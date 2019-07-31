---
title: Unir nodos de Windows
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Unirse a un nodo de Windows a un clúster de Kubernetes con v 1.14.
keywords: kubernetes, 1,14, Windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c9dbfec968d52d9fbc528892f0e3749270e3ff70
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882988"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Unir nodos de Windows Server a un clúster #
Una vez que haya [configurado un nodo maestro de Kubernetes](./creating-a-linux-master.md) y haya [seleccionado la solución de red deseada](./network-topologies.md), estará listo para unirse a los nodos de Windows Server para formar un clúster. Esto requiere [la preparación de los nodos de Windows antes de](#preparing-a-windows-node) unirlos.

## <a name="preparing-a-windows-node"></a>Preparar un nodo de Windows ##
> [!NOTE]  
> Todos los fragmentos de código en las secciones de Windows son para ejecutarse en PowerShell _con privilegios elevados_.

### <a name="install-docker-requires-reboot"></a>Instalar el acoplador (requiere reiniciar) ###
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

Si después de reiniciar el usuario ve el siguiente error:

![texto](media/docker-svc-error.png)

A continuación, inicie el servicio de Dock manualmente:

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>Crear la imagen de "pausa" (infraestructura) ###
> [!Important]
> Es importante tener cuidado con las imágenes de contenedores en conflicto; Si no tiene la etiqueta esperada, `docker pull` puede provocar una de una imagen de contenedor incompatible, lo que `ContainerCreating` provoca problemas de [implementación](./common-problems.md#when-deploying-docker-containers-keep-restarting) , como el estado indefinido.

Ahora que `docker` está instalado, debes preparar una imagen de "pausa" que Kubernetes usará para preparar los pods de infraestructura. Hay tres pasos: 
  1. [extracción de la imagen](#pull-the-image)
  2. [etiquetado](#tag-the-image) como Microsoft/nanoserver: último
  3. y [lo ejecuta](#run-the-container)


#### <a name="pull-the-image"></a>Tire de la imagen ####     
 Tire de la imagen para su versión específica de Windows. Por ejemplo, si está ejecutando Windows Server 2019:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>Etiquetar la imagen ####
La Dockerfiles que usará más adelante en esta guía, busque la `:latest` etiqueta de imagen. Etiquete la imagen de nanoservidors que acaba de extraer de la siguiente manera:

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>Ejecutar el contenedor ####
Vuelva a comprobar que el contenedor se ejecuta realmente en su equipo:

```powershell
docker run microsoft/nanoserver:latest
```

Debería ver algo parecido a esto:

![texto](./media/docker-run-sample.png)

> [!tip]
> Si no puede ejecutar el contenedor, vea: [versión del host del contenedor que coincide con la imagen del contenedor](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>Preparar Kubernetes para el directorio de Windows ####
Cree un directorio "Kubernetes para Windows" para almacenar los archivos binarios de Kubernetes, así como los scripts de implementación y los archivos de configuración.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Copiar certificado de Kubernetes #### 
Copie el archivo de certificado Kubernetes`$HOME/.kube/config`() [desde el patrón](./creating-a-linux-master.md#collect-cluster-information) a `C:\k` este nuevo directorio.

> [!tip]
> Puede usar herramientas como [xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy) o [winscp](https://winscp.net/eng/download.php) para transferir el archivo de configuración entre los nodos.

#### <a name="download-kubernetes-binaries"></a>Descargar archivos binarios de Kubernetes ####
Para poder ejecutar Kubernetes, primero debe descargar el, `kubectl` `kubelet`el y `kube-proxy` los binarios. Puede descargarlos de los vínculos en el `CHANGELOG.md` archivo de las [versiones más recientes](https://github.com/kubernetes/kubernetes/releases/).
 - Por ejemplo, estos son los [binarios del nodo v 1.14](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries).
 - Use una herramienta como [expandir-archivar](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) para extraer el archivo y colocar los binarios `C:\k\`en.

#### <a name="optional-setup-kubectl-on-windows"></a>Faculta Configuración de kubectl en Windows ####
Si desea controlar el clúster desde Windows, puede hacerlo con el `kubectl` comando. En primer lugar, `kubectl` para que esté disponible `C:\k\` fuera del directorio, `PATH` modifique la variable de entorno:

```powershell
$env:Path += ";C:\k"
```

Si quieres establecer este cambio de forma permanente, modifica la variable en el destino de la máquina:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

A continuación, comprobaremos que el [certificado de clúster](#copy-kubernetes-certificate) es válido. Para establecer la ubicación donde `kubectl` se busca el archivo de configuración, puede pasar el `--kubeconfig` parámetro o modificar la variable de `KUBECONFIG` entorno. Por ejemplo, si la configuración se encuentra en `C:\k\config`:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Para establecer esta configuración de forma permanente para el ámbito del usuario actual:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Por último, para comprobar si la configuración se ha descubierto correctamente, puede usar:

```powershell
kubectl config view
```

Si recibes un error de conexión,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Debe comprobar la ubicación del kubeconfig o intentar copiarlo de nuevo.

Si no ve ningún error, el nodo ya está listo para unirse al clúster.

## <a name="joining-the-windows-node"></a>Unirse al nodo de Windows ##
Dependiendo de la [solución de red que elija](./network-topologies.md), puede hacer lo siguiente:
1. [Unir nodos de Windows Server a un clúster de flannel (vxlan o host-GW)](#joining-a-flannel-cluster)
2. [Unir nodos de Windows Server a un clúster con un modificador de ToR](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Unirse a un clúster de flannel ###
Hay una colección de scripts de implementación de flannel en [este Microsoft Repository](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay) que le ayuda a unir este nodo al clúster.

Descargue el [flannel Start. PS1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) , cuyo contenido debe ser extraído a `C:\k`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

Suponiendo que haya [preparado el nodo de Windows](#preparing-a-windows-node)y `c:\k` su directorio tenga el aspecto que se muestra a continuación, ya puede unirse al nodo.

![texto](./media/flannel-directory.png)

#### <a name="join-node"></a>Nodo de Unión #### 
Para simplificar el proceso de unirse a un nodo de Windows, solo tiene que ejecutar un único script de `kubelet`Windows `kube-proxy`para `flanneld`iniciar,, y unirse al nodo.

> [!Note]
> las referencias de [Start. PS1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) [install. PS1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1), que descargarán archivos adicionales `flanneld` , como el ejecutable y el [Dockerfile para la caja pod de la infraestructura](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *y*los instalarán. Para el modo superpuesto de redes, se abrirá el [firewall](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111) para el puerto UDP local 4789. Es posible que se abran o cierren varias ventanas de PowerShell, así como algunos segundos de interrupción en la red, mientras que el nuevo vSwitch externo para la red Pod se crea por primera vez.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
La dirección IP asignada al nodo de Windows. Puede usar `ipconfig` para encontrar esto.

|  |  | 
|---------|---------|
|Parámetro     | `-ManagementIP`        |
|Valor predeterminado    | n.A. **obligatorio**        |

# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
El modo `l2bridge` de red (Flannel host-GW) `overlay` o (Flannel vxlan) ha sido elegido como una [solución de red](./network-topologies.md).

> [!Important] 
> `overlay` el modo de red (Flannel vxlan) requiere Kubernetes v 1.14 binarios (o versiones posteriores) y [KB4489899](https://support.microsoft.com/help/4489899).

|  |  | 
|---------|---------|
|Parámetro     | `-NetworkMode`        |
|Valor predeterminado    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
El [intervalo de subred del clúster](./getting-started-kubernetes-windows.md#cluster-subnet-def).

|  |  | 
|---------|---------|
|Parámetro     | `-ClusterCIDR`        |
|Valor predeterminado    | `10.244.0.0/16`        |


# [<a name="servicecidr"></a>ServiceCIDR](#tab/ServiceCIDR)
El [intervalo de subred del servicio](./getting-started-kubernetes-windows.md#service-subnet-def).

|  |  | 
|---------|---------|
|Parámetro     | `-ServiceCIDR`        |
|Valor predeterminado    | `10.96.0.0/12`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
La [IP del servicio DNS de Kubernetes](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster).

|  |  | 
|---------|---------|
|Parámetro     | `-KubeDnsServiceIP`        |
|Valor predeterminado    | `10.96.0.10`        |


# [<a name="interfacename"></a>Interfaz](#tab/InterfaceName)
El nombre de la interfaz de red del host de Windows. Puede usar `ipconfig` para encontrar esto.

|  |  | 
|---------|---------|
|Parámetro     | `-InterfaceName`        |
|Valor predeterminado    | `Ethernet`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
El directorio donde se redirigen los registros de kubelet y Kube-proxy en sus archivos de salida respectivos.

|  |  | 
|---------|---------|
|Parámetro     | `-LogDir`        |
|Valor predeterminado    | `C:\k`        |


---

> [!tip]
> Ya ha observado la subred del clúster, la subred de servicio y la IP Kube-DNS del maestro de Linux [antes](./creating-a-linux-master.md#collect-cluster-information)

Después de ejecutarlo, deberías poder:
  * Ver nodos de Windows combinados mediante `kubectl get nodes`
  * Vea 3 ventanas de PowerShell abiertas, una `kubelet`para, una `flanneld`para y otra para `kube-proxy`
  * Ver los procesos de agente de `flanneld`host `kubelet`para, `kube-proxy` y ejecutar en el nodo

Si es correcto, continúe con los [pasos siguientes](#next-steps).

## <a name="joining-a-tor-cluster"></a>Unirse a un clúster de ToR ##
> [!NOTE]
> Puede omitir esta sección si elige flannel como su solución de red [anteriormente](./network-topologies.md#flannel-in-host-gateway-mode).

Para ello, debe seguir las instrucciones para [configurar contenedores de Windows Server en Kubernetes para la topología de enrutamiento L3 de nivel superior](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology). Esto incluye asegurarse de configurar el router de nivel superior de modo que el prefijo CIDR del Pod asignado a un nodo se asigne a su respectivo IP de nodo.

Suponiendo que el nuevo nodo aparece como "listo" por `kubectl get nodes`, kubelet + Kube-proxy se está ejecutando y ha configurado su enrutador de Tor de nivel superior y está listo para seguir los pasos siguientes.

## <a name="next-steps"></a>Pasos siguientes ##
En esta sección, hemos explicado cómo unir a los trabajadores de Windows a nuestro clúster de Kubernetes. Ahora ya está listo para el paso 5:

> [!div class="nextstepaction"]
> [Unirse a trabajadores de Linux](./joining-linux-workers.md)

Como alternativa, si no tiene ningún trabajador de Linux, no dude en ir directamente al paso 6:

> [!div class="nextstepaction"]
> [Implementación de recursos de Kubernetes](./deploying-resources.md)
