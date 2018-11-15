---
title: Unirse a nodos Linux
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Unir un nodo de Linux a un clúster de Kubernetes con v1.12.
keywords: kubernetes, 1.12, windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 97c12d70db9679dbb85877f0985c6053f95fa500
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/15/2018
ms.locfileid: "6947984"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Unirse a nodos Linux a un clúster

Una vez que tengas [un nodo de maestro de Kubernetes de instalación](creating-a-linux-master.md) y [selecciona la solución de red deseado](network-topologies.md), estás listo para unirse a nodos Linux para el clúster. Esto requiere [preparación en el nodo de Linux](joining-linux-workers.md#preparing-a-linux-node) antes de unir.
> [!tip]
> Las instrucciones de Linux están adaptadas a **Ubuntu 16.04**. Otras distribuciones de Linux certificados para ejecutar Kubernetes también deben ofrecer comandos equivalentes que se pueden sustituir. También se interactuar correctamente con Windows.

## <a name="preparing-a-linux-node"></a>Preparar un nodo de Linux

> [!NOTE]
> A menos que se especifica explícitamente lo contrario, ejecuta los comandos en un **shell de usuario de raíz con privilegios elevados**.

Primero, Obtén en un shell de raíz:

```bash
sudo –s
```

Asegúrate de que la máquina está al día:

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>Instalar Docker

Para poder utilizar los contenedores, necesitas un motor de contenedor, como Docker. Para obtener la versión más reciente, puedes usar [estas instrucciones](https://docs.docker.com/install/linux/docker-ce/ubuntu/) para la instalación de Docker. Puedes comprobar que docker está instalado correctamente, ejecute `hello-world` imagen:

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>Instalar kubeadm

Descargar `kubeadm` archivos binarios para la distribución de Linux e inicializar el clúster.

> [!Important]  
> Según la distribución de Linux, tienes que reemplazar `kubernetes-xenial` a continuación con la correcta [con nombre en código](https://wiki.ubuntu.com/Releases).

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>Deshabilitar el intercambio

Kubernetes en Linux requiere el espacio de intercambio se desactive:

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(Sólo flannel) Permitir el tráfico de IPv4 con puente a iptables

Si has elegido Flannel como la solución de red se recomienda habilitar con puente el tráfico de IPv4 a iptables cadenas. Debe tener [ya hecho esto para el patrón de](network-topologies.md#flannel-in-host-gateway-mode) y necesita ahora para incluirla en el nodo de Linux que se va a unir. Puede hacerse mediante el siguiente comando:

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>Copiar el certificado de Kubernetes

**Como normal, usuario (que no sea de raíz)**, sigue estos 3 pasos.

1. Crear Kubernetes para el directorio de Linux:

```bash
mkdir -p $HOME/.kube
```

1. Copia el archivo de certificado de Kubernetes (`$HOME/.kube/config`) [del maestro](./creating-a-linux-master.md#collect-cluster-information) y guardar como `$HOME/.kube/config` en el trabajo.

1. Establece la propiedad del archivo del archivo config copiado como sigue:

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>Nodo de unión

Por último, para unir el clúster, ejecute el `kubeadm join` comando [mencionamos hacia abajo anteriormente](./creating-a-linux-master.md#initialize-master) **como raíz**:

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

Si se realiza correctamente, verás un resultado similar a este:

![texto](./media/node-join.png)

## <a name="next-steps"></a>Pasos siguientes

En esta sección, hemos visto cómo unir los trabajadores de Linux a nuestro clúster de Kubernetes. Ahora estás listo para el paso 6:
> [!div class="nextstepaction"]
> [Implementación de recursos de Kubernetes](./deploying-resources.md)