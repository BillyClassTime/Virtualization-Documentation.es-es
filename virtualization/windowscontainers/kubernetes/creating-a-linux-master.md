---
title: Maestro de Kubernetes desde cero
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Crear un maestro de clúster de Kubernetes.
keywords: kubernetes, 1,14, Master, Linux
ms.openlocfilehash: b1ec23b039ce6f5c42859452ecf3a8a5b35e006c
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910425"
---
# <a name="creating-a-kubernetes-master"></a>Creación de un maestro de Kubernetes #
> [!NOTE]
> Esta guía se validó en Kubernetes v 1.14. Debido a la volatilidad de Kubernetes de la versión a la versión, esta sección puede realizar suposiciones que no se conserven en todas las versiones futuras. [Aquí](https://kubernetes.io/docs/setup/independent/install-kubeadm/)puede encontrar documentación oficial sobre la inicialización de los patrones de Kubernetes con kubeadm. Simplemente habilite [la sección de programación de sistemas operativos mixtos en la](#enable-mixed-os-scheduling) parte superior.

> [!NOTE]  
> Es necesario seguir un equipo Linux actualizado recientemente; Los recursos maestros de Kubernetes como [Kube-DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), [Kube-Scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)y [Kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) no se han migrado a Windows todavía. 

> [!tip]
> Las instrucciones de Linux se adaptan hacia **Ubuntu 16,04**. Otras distribuciones de Linux certificadas para ejecutar Kubernetes también deben ofrecer comandos equivalentes que puede sustituir. También interoperarán correctamente con Windows.


## <a name="initialization-using-kubeadm"></a>Inicialización mediante kubeadm ##
A menos que se especifique explícitamente, ejecute los comandos siguientes como **raíz**.

En primer lugar, obtenga acceso a un shell raíz elevado:

```bash
sudo –s
```

Asegúrese de que la máquina está actualizada:

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>Instalar Docker ###
Para poder usar contenedores, necesita un motor de contenedor, como Docker. Para obtener la versión más reciente, puede usar [estas instrucciones](https://docs.docker.com/install/linux/docker-ce/ubuntu/) para la instalación de Docker. Puede comprobar que Docker está instalado correctamente mediante la ejecución de un contenedor de `hello-world`:

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>Instalación de kubeadm ###
Descargue `kubeadm` archivos binarios para la distribución de Linux e inicialice el clúster.

> [!Important]  
> En función de la distribución de Linux, es posible que deba reemplazar `kubernetes-xenial` siguiente por el [nombre de código](https://wiki.ubuntu.com/Releases)correcto.

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>Preparar el nodo maestro ###
Kubernetes en Linux requiere que el espacio de intercambio esté desactivado:

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>Inicializar maestro ###
Anote la subred del clúster (por ejemplo, 10.244.0.0/16) y la subred del servicio (por ejemplo, 10.96.0.0/12) e inicialice el maestro con kubeadm:

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

Esto puede tardar unos minutos. Una vez que haya finalizado, debería ver una pantalla similar a la siguiente:

![texto](media/kubeadm-init.png)

> [!tip]
> Tome nota de este comando kubeadm join. Debería el token kubeadm expira, puede usar `kubeadm token create --print-join-command` para crear un nuevo token.

> [!tip]
> Si tiene una versión de Kubernetes que desea usar, puede pasar la marca `--kubernetes-version` a kubeadm.

Todavía no hemos terminado. Para usar `kubectl` como usuario normal, ejecute lo siguiente  __**en un shell de usuario no raíz sin privilegios**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Ahora puede usar kubectl para editar o ver información sobre el clúster.

### <a name="enable-mixed-os-scheduling"></a>Habilitar la programación de sistema operativo mixto ###
De forma predeterminada, determinados recursos de Kubernetes se escriben de forma que están programados en todos los nodos. Sin embargo, en un entorno de varios sistemas operativos, no queremos que los recursos de Linux interfieran ni se programen doble en los nodos de Windows y viceversa. Por esta razón, necesitamos aplicar etiquetas [NodeSelector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) . 

En este sentido, vamos a aplicar una revisión a la [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) de Linux Kube-proxy solo para Linux como destino.

En primer lugar, vamos a crear un directorio para almacenar los archivos de manifiesto. yaml:
```bash
mkdir -p kube/yaml && cd kube/yaml
```

Confirme que la estrategia de actualización de `kube-proxy` DaemonSet está establecida en [RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/):

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

Después, aplique una revisión a la DaemonSet descargando [esta nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) y aplíquela solo a Linux de destino:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

Una vez que se realiza correctamente, debería ver "selectores de nodo" de `kube-proxy` y cualquier otro valor de DaemonSets establecido en `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![texto](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>Recopilar información del clúster ###
Para unir correctamente los nodos futuros al maestro, debe realizar un seguimiento de la siguiente información:
  1. `kubeadm join` comando de la salida ([aquí](#initialize-master))
    * Ejemplo: `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. Subred de clúster definida durante `kubeadm init` ([aquí](#initialize-master))
    * Ejemplo: `10.244.0.0/16`
  3. Subred de servicio definida durante `kubeadm init` ([aquí](#initialize-master))
    * Ejemplo: `10.96.0.0/12`
    * También se puede encontrar mediante `kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. Kube: IP de servicio DNS 
    * Ejemplo: `10.96.0.10`
    * Se puede encontrar en el campo "IP del clúster" mediante `kubectl get svc/kube-dns -n kube-system`
  5. Kubernetes `config` archivo generado después de `kubeadm init` ([aquí](#initialize-master)). Si ha seguido las instrucciones, puede encontrar en las siguientes rutas de acceso:
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>Comprobando el maestro ##
Después de unos minutos, el sistema debe estar en el estado siguiente:

  - En `kubectl get pods -n kube-system`, habrá pods para los [componentes maestros de Kubernetes](https://kubernetes.io/docs/concepts/overview/components/#master-components) en `Running` estado.
  - Al llamar a `kubectl cluster-info` se mostrará información sobre el servidor de la API maestra Kubernetes además de los complementos de DNS.
  
> [!tip]
> Dado que kubeadm no configura las redes, es posible que los pods de DNS sigan en `ContainerCreating` o `Pending` estado. Cambiarán al estado `Running` después [de elegir una solución de red](./network-topologies.md).

## <a name="next-steps"></a>Pasos siguientes ## 
En esta sección se describe cómo configurar un maestro de Kubernetes con kubeadm. Ya está listo para el paso 3:

> [!div class="nextstepaction"]
> [Elección de una solución de red](./network-topologies.md)