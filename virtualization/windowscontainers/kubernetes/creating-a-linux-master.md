---
title: Maestro de Kubernetes desde cero
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Crear un Kubernetes maestro de clúster.
keywords: kubernetes, 1.12, maestro, linux
ms.openlocfilehash: 2bbcf2d382f20d140c73d9b34cf0f13a74debdfa
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178858"
---
# <a name="creating-a-kubernetes-master"></a>Crear un maestro de Kubernetes #
> [!NOTE]
> Esta guía se ha validado en v1.12 de Kubernetes. Dada la volatilidad de Kubernetes a la versión, en esta sección puede asumir que no contienen true para todas las versiones futuras. Encontrará documentación oficial para inicializar los patrones de Kubernetes con kubeadm [aquí](https://kubernetes.io/docs/setup/independent/install-kubeadm/). Simplemente habilita además de eso [sección de programación de sistemas operativos combinados](#enable-mixed-os-scheduling) .

> [!NOTE]  
> Se necesita una máquina Linux recientemente actualizada a lo largo de; Maestro de Kubernetes recursos como [kube dns](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), [Programador de kube](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)y [kube apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) no se han trasladado a Windows aún. 

> [!tip]
> Las instrucciones de Linux están adaptadas a **Ubuntu 16.04**. Otras distribuciones de Linux certificados para ejecutar Kubernetes también deben ofrecer comandos equivalentes que se pueden sustituir. También se interactuar correctamente con Windows.


## <a name="initialization-using-kubeadm"></a>Inicialización mediante kubeadm ##
A menos que se especifica explícitamente lo contrario, ejecuta los siguientes comandos como **raíz**.

Primero, Obtén en un shell de raíz con privilegios elevados:

```bash
sudo –s
```

Asegúrate de que la máquina está al día:

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>Instalar Docker ###
Para poder utilizar los contenedores, necesitas un motor de contenedor, como Docker. Para obtener la versión más reciente, puedes usar [estas instrucciones](https://docs.docker.com/install/linux/docker-ce/ubuntu/) para la instalación de Docker. Puedes comprobar que docker está instalado correctamente mediante la ejecución de un `hello-world` contenedor:

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>Instalar kubeadm ###
Descargar `kubeadm` archivos binarios para la distribución de Linux e inicializar el clúster.

> [!Important]  
> Según la distribución de Linux, tienes que reemplazar `kubernetes-xenial` a continuación con la correcta [con nombre en código](https://wiki.ubuntu.com/Releases).

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>Preparar el nodo maestro ###
Kubernetes en Linux requiere el espacio de intercambio se desactive:

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>Inicializar el maestro ###
Ten en cuenta la subred de clúster (por ejemplo, 10.244.0.0/16) y la subred de servicio (por ejemplo, 10.96.0.0/12) e inicializar al patrón de uso kubeadm:

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

Esto puede tardar unos minutos. Una vez completado, deberías ver una pantalla como este confirmar que se ha inicializado el patrón:

![texto](media/kubeadm-init.png)

> [!tip]
> Toma nota de la salida del comando kubeadm combinación de la imagen anterior *ahora* antes de se pierde.

> [!tip]
> Si tienes una versión de Kubernetes deseada que quieras usar, puedes pasar el `--kubernetes-version` marca para kubeadm.

Nos estamos todavía no ha terminado. Usar `kubectl` como un usuario normal, ejecute el siguiente __**en un shell de unelevated y que no sea de raíz para el usuario**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Ahora puedes usar kubectl para editar o ver información sobre el clúster.

### <a name="enable-mixed-os-scheduling"></a>Habilitar la programación de sistemas operativos combinados ###
De manera predeterminada, se escriben ciertos recursos de Kubernetes de tal forma que estás programados en todos los nodos. Sin embargo, en un entorno de multi-OS no conviene Linux recursos interfieran o no se programan doble en los nodos de Windows y viceversa. Por este motivo, es necesario aplicar [NodeSelector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) etiquetas. 

En este sentido, vamos a revisión linux kube proxy [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) al destino Linux.

Confirma que la estrategia de actualización de `kube-proxy` DaemonSet se establece en [RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/):

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

A continuación, revisión de la DaemonSet mediante la descarga de [este nodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) y aplicarlo para orientarlo a solo Linux:

```bash
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

Una vez que se realiza correctamente, deberías ver "Selectores de nodo" de `kube-proxy` y cualquier otro DaemonSets se establece en `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![texto](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>Recopilar información de clúster ###
Para unir correctamente futuras nodos en el patrón, debes escribir hacia abajo de la siguiente información:
  1. `kubeadm join` comando de salida ([aquí](#initialize-master))
    * Ejemplo: `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. Definida durante de subred de clúster `kubeadm init` ([aquí](#initialize-master))
    * Ejemplo: `10.244.0.0/16`
  3. Subred de servicio definida durante `kubeadm init` ([aquí](#initialize-master))
    * Ejemplo: `10.96.0.0/12`
    * También se puede encontrar mediante `kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. Dirección IP de servicio de Kube dns 
    * Ejemplo: `10.96.0.10`
    * Pueden encontrarse en el uso de campo "IP del clúster" `kubectl get svc/kube-dns -n kube-system`
  5. Kubernetes `config` archivo generado después `kubeadm init` ([aquí](#initialize-master)). Si sigue las instrucciones, esto puede encontrarse en las siguientes rutas de acceso:
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>Comprobar el maestro ##
Después de unos minutos, el sistema debe estar en el estado siguiente:

  - En `kubectl get pods -n kube-system`, habrá pods para todo el [maestro de Kubernetes componentes](https://kubernetes.io/docs/concepts/overview/components/#master-components) en `Running` estado.
  - Una llamada a `kubectl cluster-info` , mostrará información sobre el servidor de la API de maestro de Kubernetes además de complementos DNS.

## <a name="next-steps"></a>Pasos siguientes ## 
En esta sección, hemos visto cómo configurar a un maestro de Kubernetes con kubeadm. Ahora estás listo para el paso 3:

> [!div class="nextstepaction"]
> [Elegir una solución de red](./network-topologies.md)