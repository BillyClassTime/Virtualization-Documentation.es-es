---
title: Unir nodos de Windows
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Unión de un nodo de Windows a un clúster de Kubernetes con v 1.14.
keywords: kubernetes, 1,14, Windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c9dbfec968d52d9fbc528892f0e3749270e3ff70
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910335"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Unir nodos de Windows Server a un clúster #
Una vez que haya [configurado un nodo principal de Kubernetes](./creating-a-linux-master.md) y [seleccionado la solución de red deseada](./network-topologies.md), estará listo para unir los nodos de Windows Server para formar un clúster. Esto requiere una [preparación en los nodos de Windows antes de](#preparing-a-windows-node) unirse.

## <a name="preparing-a-windows-node"></a>Preparar un nodo de Windows ##
> [!NOTE]  
> Todos los fragmentos de código en las secciones de Windows son para ejecutarse en PowerShell _con privilegios elevados_.

### <a name="install-docker-requires-reboot"></a>Instalación de Docker (requiere reinicio) ###
Kubernetes usa [Docker](https://www.docker.com/) como su motor de contenedor, por lo que es necesario instalarlo. Puedes seguir las [instrucciones oficiales de Docs](../manage-docker/configure-docker-daemon.md#install-docker), las [instrucciones de Docker](https://store.docker.com/editions/enterprise/docker-ee-server-windows) o probar estos pasos:

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

Si después del reinicio, verá el siguiente error:

![texto](media/docker-svc-error.png)

Después, inicie el servicio Docker manualmente:

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>Creación de la imagen de "pausa" (infraestructura) ###
> [!Important]
> Es importante tener cuidado con las imágenes de contenedor en conflicto. no tener la etiqueta esperada puede provocar un `docker pull` de una imagen de contenedor incompatible, lo que provoca [problemas de implementación](./common-problems.md#when-deploying-docker-containers-keep-restarting) como el estado indefinido de `ContainerCreating`.

Ahora que `docker` está instalado, debes preparar una imagen de "pausa" que Kubernetes usará para preparar los pods de infraestructura. Existen tres pasos: 
  1. [extracción de la imagen](#pull-the-image)
  2. [etiquetado](#tag-the-image) como Microsoft/nanoserver: latest
  3. y en [ejecución](#run-the-container)


#### <a name="pull-the-image"></a>Extracción de la imagen ####     
 Extraiga la imagen de su versión específica de Windows. Por ejemplo, si está ejecutando Windows Server 2019:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>Etiquetar la imagen ####
El Dockerfiles que utilizará más adelante en esta guía busca la etiqueta de imagen `:latest`. Etiquete la imagen de nanoserver que acaba de extraer del modo siguiente:

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>Ejecutar el contenedor ####
Compruebe que el contenedor se ejecuta realmente en el equipo:

```powershell
docker run microsoft/nanoserver:latest
```

Debería ver algo parecido a esto:

![texto](./media/docker-run-sample.png)

> [!tip]
> Si no puede ejecutar el contenedor, consulte: [coincidencia de la versión del host del contenedor con la imagen del contenedor](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>Preparar Kubernetes para el directorio de Windows ####
Cree un directorio "Kubernetes para Windows" para almacenar archivos binarios de Kubernetes, así como cualquier script de implementación y archivos de configuración.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Copiar certificado de Kubernetes #### 
Copie el archivo de certificado Kubernetes (`$HOME/.kube/config`) [de Master](./creating-a-linux-master.md#collect-cluster-information) a este nuevo directorio de `C:\k`.

> [!tip]
> Puede usar herramientas como [xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy) o [winscp](https://winscp.net/eng/download.php) para transferir el archivo de configuración entre nodos.

#### <a name="download-kubernetes-binaries"></a>Descarga de archivos binarios de Kubernetes ####
Para poder ejecutar Kubernetes, primero debe descargar los archivos binarios `kubectl`, `kubelet`y `kube-proxy`. Puede descargarlos de los vínculos del archivo de `CHANGELOG.md` de las [versiones más recientes](https://github.com/kubernetes/kubernetes/releases/).
 - Por ejemplo, estos son los [archivos binarios del nodo v 1.14](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries).
 - Use una herramienta como [Expand-Archive](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) para extraer el archivo y colocar los archivos binarios en `C:\k\`.

#### <a name="optional-setup-kubectl-on-windows"></a>Opta Configuración de kubectl en Windows ####
Si desea controlar el clúster desde Windows, puede hacerlo mediante el comando `kubectl`. En primer lugar, para que `kubectl` esté disponible fuera del directorio `C:\k\`, modifique la variable de entorno `PATH`:

```powershell
$env:Path += ";C:\k"
```

Si quieres establecer este cambio de forma permanente, modifica la variable en el destino de la máquina:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

A continuación, se comprobará que el [certificado de clúster](#copy-kubernetes-certificate) es válido. Para establecer la ubicación donde `kubectl` busca el archivo de configuración, puede pasar el parámetro `--kubeconfig` o modificar la variable de entorno `KUBECONFIG`. Por ejemplo, si la configuración se encuentra en `C:\k\config`:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Para establecer esta configuración de forma permanente para el ámbito del usuario actual:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Por último, para comprobar si la configuración se ha detectado correctamente, puede usar:

```powershell
kubectl config view
```

Si recibes un error de conexión,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Debe comprobar la ubicación de kubeconfig o intentar copiarla de nuevo.

Si no ve ningún error, el nodo ya está listo para unirse al clúster.

## <a name="joining-the-windows-node"></a>Unirse al nodo de Windows ##
Dependiendo de la [solución de red que elija](./network-topologies.md), puede:
1. [Unir nodos de Windows Server a un clúster de flannel (vxlan o host-GW)](#joining-a-flannel-cluster)
2. [Unir nodos de Windows Server a un clúster con un conmutador ToR](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Unirse a un clúster de flannel ###
Hay una colección de scripts de implementación de flannel en [este repositorio de Microsoft](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay) que le ayuda a unir este nodo al clúster.

Descargue el script [Start. PS1 de flannel](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) , cuyo contenido debe extraerse en `C:\k`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

Suponiendo que ha [preparado el nodo de Windows](#preparing-a-windows-node)y que el directorio de `c:\k` tiene el siguiente aspecto, está listo para unirse al nodo.

![texto](./media/flannel-directory.png)

#### <a name="join-node"></a>Nodo de Unión #### 
Para simplificar el proceso de unirse a un nodo de Windows, solo tiene que ejecutar un único script de Windows para iniciar `kubelet`, `kube-proxy`, `flanneld`y unir el nodo.

> [!Note]
> [Start. PS1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) References [install. PS1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1), que descargará archivos adicionales, como el `flanneld` ejecutable y [Dockerfile para la infraestructura Pod](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *e instálelos automáticamente*. En el modo de red de superposición, el [firewall](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111) se abrirá para el puerto UDP 4789 local. Puede haber varias ventanas de PowerShell abiertas o cerradas, así como algunos segundos de interrupción de la red, mientras que el nuevo vSwitch externo para la red Pod se crea por primera vez.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# <a name="managementiptabmanagementip"></a>[ManagementIP](#tab/ManagementIP)
Dirección IP asignada al nodo de Windows. Puede usar `ipconfig` para encontrar esto.

|  |  | 
|---------|---------|
|Parámetro     | `-ManagementIP`        |
|Valor predeterminado    | n.A. **required**        |

# <a name="networkmodetabnetworkmode"></a>[NetworkMode](#tab/NetworkMode)
El modo de red `l2bridge` (Flannel host-GW) o `overlay` (Flannel vxlan) elegido como una [solución de red](./network-topologies.md).

> [!Important] 
> `overlay` modo de red (Flannel vxlan) requiere archivos binarios Kubernetes v 1.14 (o superior) y [KB4489899](https://support.microsoft.com/help/4489899).

|  |  | 
|---------|---------|
|Parámetro     | `-NetworkMode`        |
|Valor predeterminado    | `l2bridge`        |


# <a name="clustercidrtabclustercidr"></a>[ClusterCIDR](#tab/ClusterCIDR)
El [intervalo de subred del clúster](./getting-started-kubernetes-windows.md#cluster-subnet-def).

|  |  | 
|---------|---------|
|Parámetro     | `-ClusterCIDR`        |
|Valor predeterminado    | `10.244.0.0/16`        |


# <a name="servicecidrtabservicecidr"></a>[ServiceCIDR](#tab/ServiceCIDR)
El [intervalo de subred del servicio](./getting-started-kubernetes-windows.md#service-subnet-def).

|  |  | 
|---------|---------|
|Parámetro     | `-ServiceCIDR`        |
|Valor predeterminado    | `10.96.0.0/12`        |


# <a name="kubednsserviceiptabkubednsserviceip"></a>[KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[Dirección IP del servicio DNS de Kubernetes](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster).

|  |  | 
|---------|---------|
|Parámetro     | `-KubeDnsServiceIP`        |
|Valor predeterminado    | `10.96.0.10`        |


# <a name="interfacenametabinterfacename"></a>[InterfaceName](#tab/InterfaceName)
El nombre de la interfaz de red del host de Windows. Puede usar `ipconfig` para encontrar esto.

|  |  | 
|---------|---------|
|Parámetro     | `-InterfaceName`        |
|Valor predeterminado    | `Ethernet`        |


# <a name="logdirtablogdir"></a>[LogDir](#tab/LogDir)
El directorio donde se redirigen los registros de kubelet y Kube-proxy a sus respectivos archivos de salida.

|  |  | 
|---------|---------|
|Parámetro     | `-LogDir`        |
|Valor predeterminado    | `C:\k`        |


---

> [!tip]
> Ya anotó la subred del clúster, la subred de servicio y la dirección IP de Kube-DNS del maestro de Linux [anterior](./creating-a-linux-master.md#collect-cluster-information)

Después de ejecutarlo, debe ser capaz de:
  * Ver nodos de Windows Unidos mediante `kubectl get nodes`
  * Vea 3 ventanas de PowerShell abiertas, una para `kubelet`, una para `flanneld`y otra para `kube-proxy`
  * Consulte procesos de agente de host para `flanneld`, `kubelet`y `kube-proxy` que se ejecutan en el nodo.

Si la operación se realiza correctamente, continúe con los [pasos siguientes](#next-steps).

## <a name="joining-a-tor-cluster"></a>Unirse a un clúster de ToR ##
> [!NOTE]
> Puede omitir esta sección si eligió flannel como su solución de red [anteriormente](./network-topologies.md#flannel-in-host-gateway-mode).

Para ello, debe seguir las instrucciones para [configurar los contenedores de Windows Server en Kubernetes para la topología de enrutamiento L3 de nivel superior](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology). Esto incluye asegurarse de que configura el enrutador ascendente de modo que el prefijo Pod CIDR asignado a un nodo se asigne a su respectivo nodo IP.

Suponiendo que el nuevo nodo aparece como "listo" por `kubectl get nodes`, kubelet + Kube-proxy se está ejecutando y ha configurado el enrutador de ToR ascendente, está listo para realizar los pasos siguientes.

## <a name="next-steps"></a>Pasos siguientes ##
En esta sección, se ha explicado cómo unir los trabajos de Windows a nuestro clúster de Kubernetes. Ya está listo para el paso 5:

> [!div class="nextstepaction"]
> [Unirse a trabajos de Linux](./joining-linux-workers.md)

Como alternativa, si no tiene ningún trabajo de Linux, no dude en avanzar al paso 6:

> [!div class="nextstepaction"]
> [Implementación de recursos de Kubernetes](./deploying-resources.md)
