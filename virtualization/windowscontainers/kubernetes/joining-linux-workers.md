---
title: Unión de nodos de Linux
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Unión de un nodo de Linux a un clúster de Kubernetes con v 1.14.
keywords: kubernetes, 1,14, Windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 88207939c82bfe8ffa0b088cfd61cf4ab22cb10a
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909955"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Unión de nodos de Linux a un clúster

Una vez que haya [configurado un nodo principal de Kubernetes](creating-a-linux-master.md) y [seleccionado la solución de red deseada](network-topologies.md), estará listo para unir los nodos de Linux al clúster. Esto requiere una [preparación en el nodo de Linux antes de](joining-linux-workers.md#preparing-a-linux-node) unirse.
> [!tip]
> Las instrucciones de Linux se adaptan hacia **Ubuntu 16,04**. Otras distribuciones de Linux certificadas para ejecutar Kubernetes también deben ofrecer comandos equivalentes que puede sustituir. También interoperarán correctamente con Windows.

## <a name="preparing-a-linux-node"></a>Preparación de un nodo de Linux

> [!NOTE]
> A menos que se especifique explícitamente lo contrario, ejecute cualquier comando en un **Shell de usuario raíz con privilegios elevados**.

En primer lugar, obtenga acceso a un shell raíz:

```bash
sudo –s
```

Asegúrese de que la máquina está actualizada:

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>Instalar Docker

Para poder usar contenedores, necesita un motor de contenedor, como Docker. Para obtener la versión más reciente, puede usar [estas instrucciones](https://docs.docker.com/install/linux/docker-ce/ubuntu/) para la instalación de Docker. Para comprobar que Docker está instalado correctamente, ejecute `hello-world` imagen:

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>Instalación de kubeadm

Descargue `kubeadm` archivos binarios para la distribución de Linux e inicialice el clúster.

> [!Important]  
> En función de la distribución de Linux, es posible que deba reemplazar `kubernetes-xenial` siguiente por el [nombre de código](https://wiki.ubuntu.com/Releases)correcto.

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>Deshabilitar intercambio

Kubernetes en Linux requiere que el espacio de intercambio esté desactivado:

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(Solo flannel) Habilitación del tráfico IPv4 de puente a iptables

Si elige flannel como solución de red, se recomienda habilitar el tráfico IPv4 con puente en las cadenas iptables. Ya debe hacer [esto para el maestro](network-topologies.md#flannel-in-host-gateway-mode) y ahora debe repetirlo para el nodo de Linux que se va a combinar. Puede realizarse mediante el comando siguiente:

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>Copiar certificado de Kubernetes

**Como usuario normal (no raíz)** , lleve a cabo los tres pasos siguientes.

1. Cree el directorio Kubernetes para Linux:

```bash
mkdir -p $HOME/.kube
```

2. Copie el archivo de certificado Kubernetes (`$HOME/.kube/config`) [de Master](./creating-a-linux-master.md#collect-cluster-information) y guárdelo como `$HOME/.kube/config` en el trabajo.

> [!tip]
> Puede usar herramientas basadas en SCP, como [winscp](https://winscp.net/eng/download.php) , para transferir el archivo de configuración entre nodos.

3. Establezca la propiedad del archivo de configuración copiado de la siguiente manera:

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>Unir nodo

Por último, para unirse al clúster, ejecute el comando `kubeadm join` que [hemos anotado anteriormente](./creating-a-linux-master.md#initialize-master) **como raíz**:

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

Si se realiza correctamente, debería ver una salida similar a la siguiente:

![texto](./media/node-join.png)

## <a name="next-steps"></a>Pasos siguientes

En esta sección, trataremos cómo unir los trabajos de Linux a nuestro clúster de Kubernetes. Ya está listo para el paso 6:
> [!div class="nextstepaction"]
> [Implementación de recursos de Kubernetes](./deploying-resources.md)