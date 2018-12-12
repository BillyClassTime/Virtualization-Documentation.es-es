---
title: Unirse a nodos de Windows
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Unir un nodo de Windows a un clúster de Kubernetes con v1.12.
keywords: kubernetes, 1.12, windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 8051270cac6178bad9adf9a8ef9e2324932f7d01
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/08/2018
ms.locfileid: "6179096"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Unirse a nodos de servidor de Windows a un clúster #
Una vez que tengas [un nodo de maestro de Kubernetes de instalación](./creating-a-linux-master.md) y [selecciona la solución de red deseado](./network-topologies.md), estás preparado para unirse a nodos de Windows Server para formar un clúster. Esto requiere [preparación en los nodos de Windows](#preparing-a-windows-node) antes de unir.

## <a name="preparing-a-windows-node"></a>Preparar un nodo de Windows ##
> [!NOTE]  
> Todos los fragmentos de código en las secciones de Windows son para ejecutarse en PowerShell _con privilegios elevados_.

### <a name="install-docker-requires-reboot"></a>Instalar a Docker (requiere reiniciar) ###
Kubernetes usa [Docker](https://www.docker.com/) como su motor de contenedor, por lo que necesitamos instalarlo. Puedes seguir las [instrucciones oficiales de Docs](../manage-docker/configure-docker-daemon.md#install-docker), las [instrucciones de Docker](https://store.docker.com/editions/enterprise/docker-ee-server-windows) o probar estos pasos:

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

Si estás conectado a un servidor proxy, deben definirse las siguientes variables de entorno de PowerShell:
```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://proxy.example.com:80/", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://proxy.example.com:443/", [EnvironmentVariableTarget]::Machine)
```

Si tras el reinicio se muestra el error siguiente:

![texto](media/docker-svc-error.png)

A continuación, iniciar manualmente el servicio de docker:

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>Crear la imagen de "pausa" (infraestructura) ###
> [!Important]
> Es importante tener cuidado de imágenes de contenedor en conflicto. no tiene la etiqueta esperada puede provocar un `docker pull` de una imagen de contenedor incompatible, lo que provoca [problemas de implementación](./common-problems.md#when-deploying-docker-containers-keep-restarting) , como indefinido `ContainerCreating` estado.

Ahora que `docker` está instalado, debes preparar una imagen de "pausa" que Kubernetes usará para preparar los pods de infraestructura. Hay tres pasos: 
  1. [extraer la imagen](#pull-the-image)
  2. [etiquetado,](#tag-the-image) como microsoft o nanoserver:latest
  3. y [ejecutarlo](#run-the-container)


#### <a name="pull-the-image"></a>Extraiga la imagen ####     
 Extraiga la imagen de la versión específica de Windows. Por ejemplo, si estás ejecutando Windows Server 2019:

 ```powershell
docker pull microsoft/nanoserver:1803
 ```

#### <a name="tag-the-image"></a>Etiqueta de la imagen ####
Busca los Dockerfiles usará más adelante en esta guía la `:latest` etiqueta de la imagen. Etiqueta de la imagen de nanoserver extraída solo como sigue:

```powershell
docker tag microsoft/nanoserver:1803 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>Ejecutar el contenedor ####
Vuelve a comprobar que el contenedor se ejecuta realmente en el equipo:

```powershell
docker run microsoft/nanoserver:latest
```

Deberías ver algo parecido a esto:

![texto](./media/docker-run-sample.png)

> [!tip]
> Si no se puede ejecutar el contenedor por favor, consulta: [versión de host de contenedor que coincide con la imagen de contenedor](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>Preparar el directorio de Kubernetes para Windows ####
Cree un directorio de "De Kubernetes para Windows" para almacenar los archivos binarios de Kubernetes, así como los scripts de implementación y los archivos de configuración.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Copiar el certificado de Kubernetes #### 
Copia el archivo de certificado de Kubernetes (`$HOME/.kube/config`) [del maestro](./creating-a-linux-master.md#collect-cluster-information) a este nuevo `C:\k` directorio.

#### <a name="download-kubernetes-binaries"></a>Descargar archivos binarios de Kubernetes ####
Para poder ejecutar Kubernetes, primero debes descargar la `kubectl`, `kubelet`, y `kube-proxy` archivos binarios. Puedes descargarlos en los vínculos del `CHANGELOG.md` archivo de las [versiones más recientes](https://github.com/kubernetes/kubernetes/releases/).
 - Por ejemplo, estos son los [archivos binarios de nodo de v1.12](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.12.md#node-binaries).
 - Usar una herramienta como [7-Zip](http://www.7-zip.org/) para extraer el archivo y colocar los archivos binarios en `C:\k\`.

#### <a name="optional-setup-kubectl-on-windows"></a>(Opcional) El programa de instalación kubectl en Windows ####
Si desea controlar el clúster de Windows, puede hacerlo mediante la `kubectl` comando. Primero, realizar `kubectl` disponible fuera de la `C:\k\` directorio, modificar el `PATH` variable de entorno:

```powershell
$env:Path += ";C:\k"
```

Si quieres establecer este cambio de forma permanente, modifica la variable en el destino de la máquina:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

A continuación, comprobaremos que el [certificado de clúster](#copy-kubernetes-certificate) es válida. Con el fin de establecer la ubicación donde `kubectl` busca el archivo de configuración, puedes pasar el `--kubeconfig` parámetro o modificar la `KUBECONFIG` variable de entorno. Por ejemplo, si la configuración se encuentra en `C:\k\config`:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Para establecer esta configuración de forma permanente para el ámbito del usuario actual:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Por último, para comprobar si la configuración se ha detectado correctamente, puedes usar:

```powershell
kubectl config view
```

Si recibes un error de conexión,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Debe vuelve a comprobar la ubicación de kubeconfig o intentar copiarlo de nuevo.

Si no ves errores el nodo está preparado para unirse al clúster.

## <a name="joining-the-windows-node"></a>Unir el nodo de Windows ##
Dependiendo de [solución de red que has elegido](./network-topologies.md), puedes:
1. [Unirse a nodos de Windows Server a un clúster de Flannel](#joining-a-flannel-cluster)
2. [Unirse a nodos de Windows Server a un clúster con un conmutador ToR](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Unirse a un clúster de Flannel ###
Hay una colección de scripts de implementación de Flannel en [este repositorio de Microsoft](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge) que ayuda a unir este nodo al clúster.

Puedes descargar el archivo ZIP directamente [aquí](https://github.com/Microsoft/SDN/archive/master.zip). Lo único que necesitas es la `Kubernetes/flannel/l2bridge` directorio, cuyo contenido debe extraerse en `C:\k\`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mv master/SDN-master/Kubernetes/flannel/l2bridge/* C:/k/
rm -recurse -force master,master.zip
```

Además, debes asegurarte de que es correcta en la subred de clúster (p. ej., verificación "10.244.0.0/16"):
- [NET conf.json](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/net-conf.json)


Suponiendo que [preparado el nodo de Windows](#preparing-a-windows-node)y su `c:\k` directory busca al siguiente, estás listo para unir el nodo.

![texto](./media/flannel-directory.png)

#### <a name="join-node"></a>Unirse a nodos #### 
Para simplificar el proceso de unirse a un nodo de Windows, solo necesitas ejecutar un script de Windows solo para iniciar `kubelet`, `kube-proxy`, `flanneld`y unir el nodo.

> [!Note]
> [Este script](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1) se descargará archivos adicionales, como se ha actualizado `flanneld` ejecutable y el [Dockerfile de pod de infraestructura](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *y ejecutar aquellas para TI*. Puede haber varias ventanas de powershell que se está abierto o cerrado, así como unos pocos segundos de interrupción de la red mientras se crea la nueva vSwitch externo para la red de pod l2bridge la primera vez.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> 
```

> [!tip]
> Ya explicó hacia abajo de la subred de clúster, subred de servicio y kube DNS IP del maestro de Linux [anteriores](./creating-a-linux-master.md#collect-cluster-information)

Después de ejecutar este debe ser capaz de:
  * Vista unido con los nodos de Windows `kubectl get nodes`
  * Consulta 3 windows powershell abiertos, uno para `kubelet`, uno para `flanneld`y otro para `kube-proxy`
  * Ver los procesos de agente de host para `flanneld`, `kubelet`, y `kube-proxy` que se ejecuta en el nodo


## <a name="joining-a-tor-cluster"></a>Unirse a un clúster ToR ##
> [!NOTE]
> Puedes omitir esta sección si has elegido Flannel como tu red solución [anteriormente](./network-topologies.md#flannel-in-host-gateway-mode).

Para ello, debes seguir las instrucciones para [configurar los contenedores de Windows Server en Kubernetes para la topología de enrutamiento de L3 en dirección ascendente](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology). Esto incluye asegurándote de que configurar el enrutador en dirección ascendente como prefijo el pod CIDR asignadas a un nodo se asigna a su dirección IP de nodo correspondiente.

Suponiendo que el nuevo nodo aparece como "Listo para" por `kubectl get nodes`, kubelet + kube proxy se está ejecutando y has configurado el enrutador ToR en dirección ascendente, estás listo para los pasos siguientes.

## <a name="next-steps"></a>Pasos siguientes ##
En esta sección, hemos visto cómo unir los trabajadores de Windows a nuestro clúster de Kubernetes. Ahora estás listo para el paso 5:

> [!div class="nextstepaction"]
> [Unirse a los trabajadores de Linux](./joining-linux-workers.md)

Como alternativa, si no tienes los trabajadores de Linux dudes en avanza al paso 6:

> [!div class="nextstepaction"]
> [Implementación de recursos de Kubernetes](./deploying-resources.md)