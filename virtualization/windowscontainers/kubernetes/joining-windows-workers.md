---
title: Unirse a nodos de Windows
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Unir un nodo de Windows a un clúster de Kubernetes con v1.14.
keywords: kubernetes, 1.14, windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c1781a6ce48ebaa8433f5649a34ac79b852beae6
ms.sourcegitcommit: aaf115a9de929319cc893c29ba39654a96cf07e1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "9622970"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Unirse a nodos de servidor de Windows a un clúster #
Una vez que tengas [un nodo de maestro de Kubernetes de instalación](./creating-a-linux-master.md) y [selecciona la solución de red deseado](./network-topologies.md), estás preparado para unirse a nodos de Windows Server para formar un clúster. Esto requiere [preparación en los nodos de Windows](#preparing-a-windows-node) antes de unir.

## <a name="preparing-a-windows-node"></a>Preparar un nodo de Windows ##
> [!NOTE]  
> Todos los fragmentos de código en las secciones de Windows son para ejecutarse en PowerShell _con privilegios elevados_.

### <a name="install-docker-requires-reboot"></a>Instalar a Docker (requiere reiniciar) ###
Kubernetes usa [Docker](https://www.docker.com/) como motor de contenedor, por lo que necesitamos instalarlo. Puedes seguir las [instrucciones oficiales de Docs](../manage-docker/configure-docker-daemon.md#install-docker), las [instrucciones de Docker](https://store.docker.com/editions/enterprise/docker-ee-server-windows) o probar estos pasos:

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

A continuación, iniciar manualmente el servicio docker:

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
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>Etiqueta de la imagen ####
Busca los Dockerfiles usará más adelante en esta guía la `:latest` etiqueta de la imagen. Etiqueta de la imagen de nanoserver extraída simplemente como sigue:

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>Ejecutar el contenedor ####
Vuelve a comprobar que el contenedor se ejecuta realmente en el equipo:

```powershell
docker run microsoft/nanoserver:latest
```

Deberías ver algo parecido a esto:

![texto](./media/docker-run-sample.png)

> [!tip]
> Si no se puede ejecutar el contenedor por favor, consulta: [versión de host de contenedor que coincide con la imagen de contenedor](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>Preparar el directorio de Kubernetes para Windows ####
Cree un directorio de "De Kubernetes para Windows" para almacenar los archivos binarios de Kubernetes, así como los scripts de implementación y los archivos de configuración.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Copiar el certificado de Kubernetes #### 
Copia el archivo de certificado de Kubernetes (`$HOME/.kube/config`) [del maestro](./creating-a-linux-master.md#collect-cluster-information) a este nuevo `C:\k` directorio.

> [!tip]
> Puedes usar herramientas como [xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy) o [WinSCP](https://winscp.net/eng/download.php) para transferir el archivo de configuración entre los nodos.

#### <a name="download-kubernetes-binaries"></a>Descargar archivos binarios de Kubernetes ####
Para poder ejecutar Kubernetes, primero debes descargar la `kubectl`, `kubelet`, y `kube-proxy` archivos binarios. Puedes descargarlos en los vínculos del `CHANGELOG.md` archivo de las [versiones más recientes](https://github.com/kubernetes/kubernetes/releases/).
 - Por ejemplo, estos son los [archivos binarios de nodo v1.14](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries).
 - Usar una herramienta como [Archivo de expandir](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) para extraer el archivo y colocar los archivos binarios en `C:\k\`.

#### <a name="optional-setup-kubectl-on-windows"></a>(Opcional) El programa de instalación kubectl en Windows ####
Si desea controlar el clúster de Windows, puede hacerlo mediante la `kubectl` comando. Primero, realizar `kubectl` esté disponible fuera de la `C:\k\` directorio, modificar la `PATH` variable de entorno:

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
1. [Unirse a nodos de Windows Server a un clúster de Flannel (vxlan o gw de host)](#joining-a-flannel-cluster)
2. [Unirse a nodos de Windows Server a un clúster con un conmutador ToR](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Unirse a un clúster de Flannel ###
Hay una colección de scripts de implementación de Flannel en [este repositorio de Microsoft](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay) que ayuda a unir este nodo al clúster.

Descargar el script [Flannel start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) , cuyo contenido debe extraerse en `C:\k`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

Suponiendo que [preparado el nodo de Windows](#preparing-a-windows-node)y su `c:\k` directory al siguiente aspecto, estás listo para unir el nodo.

![texto](./media/flannel-directory.png)

#### <a name="join-node"></a>Unirse a nodos #### 
Para simplificar el proceso de unirse a un nodo de Windows, solo necesitas ejecutar un script de Windows solo para iniciar `kubelet`, `kube-proxy`, `flanneld`y unir el nodo.

> [!Note]
> [Start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) hace referencia a [install.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1), que se descargará archivos adicionales, como el `flanneld` ejecutable y el [Dockerfile de pod de infraestructura](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *e instalarlas para usted*. Para el modo de red de superposición, se abrirá el [firewall](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111) para el puerto UDP 4789 local. Puede haber varias ventanas de powershell que se está abierto o cerrado, así como unos pocos segundos de interrupción de la red mientras se crea la primera vez que el nuevo vSwitch externo para la red de pod.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
La dirección IP asignada al nodo de Windows. Puedes usar `ipconfig` encontrar esto.

|  |  | 
|---------|---------|
|Parámetro     | `-ManagementIP`        |
|Valor predeterminado    | n.A. **obligatorio**        |

# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
El modo de red `l2bridge` (flannel gw de host) o `overlay` (flannel vxlan) elegido como una [solución de red](./network-topologies.md).

> [!Important] 
> `overlay` modo de red (flannel vxlan) requiere los archivos binarios de Kubernetes v1.14 (o superior) y [KB4489899](https://support.microsoft.com/help/4489899).

|  |  | 
|---------|---------|
|Parámetro     | `-NetworkMode`        |
|Valor predeterminado    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
El [intervalo de subred de clúster](./getting-started-kubernetes-windows.md#cluster-subnet-def).

|  |  | 
|---------|---------|
|Parámetro     | `-ClusterCIDR`        |
|Valor predeterminado    | `10.244.0.0/16`        |


# [<a name="servicecidr"></a>ServiceCIDR](#tab/ServiceCIDR)
El [intervalo de subred de servicio](./getting-started-kubernetes-windows.md#service-subnet-def).

|  |  | 
|---------|---------|
|Parámetro     | `-ServiceCIDR`        |
|Valor predeterminado    | `10.96.0.0/12`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[IP de servicio de DNS de Kubernetes](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster).

|  |  | 
|---------|---------|
|Parámetro     | `-KubeDnsServiceIP`        |
|Valor predeterminado    | `10.96.0.10`        |


# [<a name="interfacename"></a>Interfaz](#tab/InterfaceName)
El nombre de la interfaz de red del host de Windows. Puedes usar `ipconfig` encontrar esto.

|  |  | 
|---------|---------|
|Parámetro     | `-InterfaceName`        |
|Valor predeterminado    | `Ethernet`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
El directorio donde se redirigen kubelet y el proxy de kube registros en sus archivos de salida correspondiente.

|  |  | 
|---------|---------|
|Parámetro     | `-LogDir`        |
|Valor predeterminado    | `C:\k`        |


---

> [!tip]
> Ya explicó hacia abajo de la subred de clúster, subred de servicio y kube DNS IP del maestro de Linux [anteriores](./creating-a-linux-master.md#collect-cluster-information)

Después de ejecutar este debe ser capaz de:
  * Vista unido con los nodos de Windows `kubectl get nodes`
  * Consulta 3 windows powershell abiertos, uno para `kubelet`, uno para `flanneld`y otro para `kube-proxy`
  * Ver los procesos de agente de host para `flanneld`, `kubelet`, y `kube-proxy` que se ejecuta en el nodo

Si se realiza correctamente, seguir los [pasos siguientes](#next-steps).

## <a name="joining-a-tor-cluster"></a>Unirse a un clúster ToR ##
> [!NOTE]
> Puedes omitir esta sección si has elegido Flannel como tu red solución [anteriormente](./network-topologies.md#flannel-in-host-gateway-mode).

Para ello, debes seguir las instrucciones para [configurar los contenedores de Windows Server en Kubernetes para la topología de enrutamiento de L3 en dirección ascendente](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology). Esto incluye asegurándote de que configurar el enrutador en dirección ascendente como prefijo en el pod CIDR asignadas a un nodo se asigna a su dirección IP de nodo correspondiente.

Suponiendo que el nuevo nodo aparece como "Listo para" por `kubectl get nodes`, kubelet + kube proxy se está ejecutando y has configurado el enrutador ToR en dirección ascendente, estás listo para los pasos siguientes.

## <a name="next-steps"></a>Pasos siguientes ##
En esta sección, hemos visto cómo unir los trabajadores de Windows a nuestro clúster de Kubernetes. Ahora ya estás listo para paso 5:

> [!div class="nextstepaction"]
> [Unirse a los trabajadores de Linux](./joining-linux-workers.md)

Como alternativa, si no tienes ninguna trabajadores de Linux no dude en avanza al paso 6:

> [!div class="nextstepaction"]
> [Implementación de recursos de Kubernetes](./deploying-resources.md)
