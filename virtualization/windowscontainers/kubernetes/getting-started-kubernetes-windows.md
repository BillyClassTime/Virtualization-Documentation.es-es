---
title: Kubernetes en Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: Unir un nodo de Windows a un clúster de Kubernetes con v1.9 beta.
keywords: kubernetes, 1.9, windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c6127fe8ab9de6a56816fb8187d4dec525425510
ms.sourcegitcommit: 7c3af076eb8bad98e1c3de0af63dacd842efcfa3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2018
---
# <a name="kubernetes-on-windows"></a>Kubernetes en Windows #

Con la versión más reciente de Kubernetes 1.9 y WindowsServer [versión 1709](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1709#networking), los usuarios pueden sacar partido de las características más recientes de las redes de Windows:

  - **compartimentos de pods compartidos**: los pods de infraestructuras y trabajadores ahora comparten un compartimento de red (análogo a un espacio de nombre de Linux)
  - **optimización de punto de conexión**: gracias al uso compartido de compartimentos, los servicios de contenedor necesitan realizar un seguimiento (al menos) de la mitad de los puntos de conexión anteriores
  - **optimización de ruta de datos**: las mejoras en la plataforma de filtrado virtual y el servicio de redes de host permiten el equilibrio de carga basado en kernel


Esta página sirve como guía para empezar a unir un nuevo nodo de Windows a un clúster existente basado en Linux. Para comenzar completamente desde cero, consulta [esta página](./creating-a-linux-master.md) &mdash; uno de los numerosos recursos disponibles para la implementación de un clúster de Kubernetes &mdash; para configurar un maestro desde cero y de la misma forma que lo hicimos.

> [!TIP] 
> Si quieres implementar un clúster en Azure, la herramienta ACS-Engine de código abierto facilita esta tarea. Hay un [tutorial](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/windows.md) paso a paso disponible.

<a name="definitions"></a> Estas son definiciones de algunos términos a los que se hacen referencia en esta guía:

  - La **red externa** es la red en la que se comunican los nodos.
  - <a name="cluster-subnet-def"></a>La **subred de clúster** es una red virtual enrutable; a los nodos se les asignan subredes más pequeñas de esta red para que sus pods los usen.
  - La **subred de servicio** es una subred puramente virtual y no enrutable en 11.0/16 que los pods la usan para acceder de forma uniforme a los servicios sin preocuparse por la topología de red. Se traduce al espacio de direcciones enrutables, y desde él, mediante `kube-proxy` que se ejecuta en los nodos.

## <a name="what-you-will-accomplish"></a>Qué lograrás ##

Al final de esta guía, habrás:

> [!div class="checklist"]  
> * Configurado un nodo de [maestro de Linux](#preparing-the-linux-master).  
> * Unido un [nodo de trabajo de Windows](#preparing-a-windows-node) a él.  
> * Preparado nuestra [topología de red](#network-topology).  
> * Implementado un [servicio de Windows de ejemplo](#running-a-sample-service).  
> * Solucionado [errores y problemas comunes](./common-problems.md).  

## <a name="preparing-the-linux-master"></a>Preparar el maestro de Linux ##

Independientemente de si has seguido [las instrucciones](./creating-a-linux-master.md) o ya tienes un clúster, lo único que necesitas del maestro de Linux es la configuración de certificado de Kubernetes. Esto podría ser en `/etc/kubernetes/admin.conf`, `~/.kube/config`, o en otro lugar en función de la configuración.

## <a name="preparing-a-windows-node"></a>Preparar un nodo de Windows ##

> [!NOTE]  
> Todos los fragmentos de código en las secciones de Windows son para ejecutarse en PowerShell _con privilegios elevados_.

Kubernetes usa [Docker](https://www.docker.com/) como su orquestador de contenedores, por lo que necesitamos para instalarlo. Puedes seguir las [instrucciones oficiales de Docs](../manage-docker/configure-docker-daemon.md#install-docker), las [instrucciones de Docker](https://store.docker.com/editions/enterprise/docker-ee-server-windows) o probar estos pasos:

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

Hay una colección de scripts en [este repositorio de Microsoft](https://github.com/Microsoft/SDN) que te ayuda a unir este nodo al clúster. Puedes descargar el archivo ZIP directamente [aquí](https://github.com/Microsoft/SDN/archive/master.zip). Lo único que necesitas es la carpeta `Kubernetes/windows`, cuyo contenido debe moverse a `C:\k\`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mkdir C:/k/
mv master/SDN-master/Kubernetes/windows/* C:/k/
rm -recurse -force master,master.zip
```

Copia el archivo de certificado [que se ha identificado anteriormente](#preparing-the-linux-master) a este nuevo directorio `C:\k`.

## <a name="network-topology"></a>Topología de red ##

Hay varias maneras de hacer enrutable la [subred de clúster](#cluster-subnet-def) virtual. Se puede hacer lo siguiente:

  - Configurar el [modo host-gateway](./configuring-host-gateway-mode.md), configurar rutas estáticas de siguiente salto entre nodos para habilitar la comunicación de pod a pod.
  - Configurar un conmutador Top of Rack (ToR) para enrutar a la subred.
  - Usar un complemento de superposición de terceros como, por ejemplo, [Flannel](https://coreos.com/flannel/docs/latest/kubernetes.html) (el soporte técnico de Windows para Flannel está en la versión beta).

### <a name="creating-the-pause-image"></a>Crear la imagen de "pausa" ###

Ahora que `docker` está instalado, debes preparar una imagen de "pausa" que Kubernetes usará para preparar los pods de infraestructura.

```powershell
docker pull microsoft/windowsservercore:1709
docker tag microsoft/windowsservercore:1709 microsoft/windowsservercore:latest
cd C:/k/
docker build -t kubeletwin/pause .
```

> [!NOTE]
> Se etiqueta como `:latest` porque el servicio de muestra que se implementará más adelante depende de ella, aunque podría no _ser_ la última imagen de WindowsServerCore disponible. Es importante tener cuidado y no usar imágenes de contenedor en conflicto; no contar con la etiqueta esperada puede provocar un `docker pull` de una imagen de contenedor incompatible, lo que puede provocar [problemas de implementación](./common-problems.md#when-deploying-docker-containers-keep-restarting). 


### <a name="downloading-binaries"></a>Descargar archivos binarios ###
Entretanto mientras se produce `pull`, descarga los siguientes archivos binarios de cliente de Kubernetes:

  - `kubectl.exe`
  - `kubelet.exe`
  - `kube-proxy.exe`

Puedes descargarlos en los vínculos del archivo `CHANGELOG.md` de la última versión 1.9. En el momento de la redacción de este documento, la versión es [1.9.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1), y los archivos binarios de Windows están [aquí](https://storage.googleapis.com/kubernetes-release/release/v1.9.1/kubernetes-node-windows-amd64.tar.gz). Usa una herramienta como [7-Zip](http://www.7-zip.org/) para extraer el archivo y colocar los archivos binarios en `C:\k\`.

Para que el comando `kubectl` esté disponible fuera del directorio `C:\k\`, modifica la variable de entorno de `PATH`:

```powershell
$env:Path += ";C:\k"
```

Si quieres establecer este cambio de forma permanente, modifica la variable en el destino de la máquina:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

### <a name="joining-the-cluster"></a>Unir el clúster ###
Comprueba que la configuración del clúster es válida mediante:

```powershell
kubectl version
```

Si recibes un error de conexión,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

comprueba si la configuración se ha detectado correctamente:

```powershell
kubectl config view
```

Para cambiar la ubicación donde `kubectl` busca el archivo de configuración, puedes pasar el parámetro `--kubeconfig` o modificar la variable de entorno `KUBECONFIG`. Por ejemplo, si la configuración se encuentra en `C:\k\config`:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Para establecer esta configuración de forma permanente para el ámbito del usuario actual:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

El nodo ya está preparado para unirse al clúster. En dos ventanas diferentes y *con privilegios elevados* de PowerShell, ejecuta estos scripts (en este orden). El parámetro `-ClusterCidr` del primer script es la [subred de clúster](#cluster-subnet-def); configurada aquí, es `192.168.0.0/16`.

```powershell
./start-kubelet.ps1 -ClusterCidr 192.168.0.0/16
./start-kubeproxy.ps1
```

El nodo de Windows estará visible desde el maestro de Linux en `kubectl get nodes` en el plazo de un minuto.


### <a name="validating-your-network-topology"></a>Validar la topología de red ###

Hay algunas pruebas básicas que validan una configuración de red correcta:

  - **Conectividad de nodo a nodo**: los pings entre los nodos de trabajo de Windows y de maestro deben tener éxito en ambas direcciones.

  - **Subred de Pod a conectividad de nodo**: los pings entre la interfaz de pod virtual y los nodos. Busca la dirección de puerta de enlace en `route -n` y `ipconfig` en Windows y Linux, respectivamente, busca la interfaz `cbr0`.

Si alguna de estas pruebas básicas no funcionan, prueba la [página de solución de problemas](./common-problems.md#common-networking-errors) para solucionar problemas comunes.


## <a name="running-a-sample-service"></a>Ejecutar un servicio de muestra ##

Implementarás un [servicio web basado en PowerShell](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) muy sencillo para garantizar que has unido el clúster correctamente y que nuestra red está configurada correctamente.

En el maestro de Linux, descarga y ejecuta el servicio:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/WebServer.yaml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Esto creará una implementación y un servicio. A continuación, puedes visualizar los pods indefinidamente para realizar un seguimiento de su estado. Cuando hayas finalizado, simplemente presiona `Ctrl+C` para salir del comando `watch`.

Si todo ha funcionado correctamente, podrás:

  - ver 4 contenedores en un comando `docker ps` en el nodo de Windows.
  - ver 2 pods en un comando `kubectl get pods` desde el maestro de Linux
  - `curl` en las IP de *pod* del puerto 80 del maestro de Linux que obtiene una respuesta del servidor web; esto demuestra el nodo adecuado a la comunicación de pod de la red.
  - hacer ping *entre pods* (incluidos entre hosts, si tienes más de un nodo de Windows) a través de `docker exec`; esto demuestra la comunicación adecuada de pod a pod
  - `curl` la *IP de servicio* virtual (que se muestra en `kubectl get services`) del maestro de Linux y de pods individuales.
  - `curl` el *nombre de servicio* con el [sufijo DNS predeterminado](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services) de Kubernetes, lo que demuestra la funcionalidad DNS.

> [!Warning]  
> Los nodos de Windows no podrán obtener acceso a la dirección IP de servicio. Se trata de una [limitación de plataforma conocida](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) que se mejorará en la próxima actualización a Windows Server.


### <a name="port-mapping"></a>Asignación de puertos ### 
También es posible tener acceso a servicios hospedados en pods a través de sus respectivos nodos mediante la asignación de un puerto en el nodo. Hay [otra muestra YAML disponible](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml) con una asignación de puerto 4444 en el nodo para el puerto 80 en el pod para demostrar esta característica. Para implementarlo, sigue los mismos pasos que antes:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

Ahora debería ser posible la acción `curl` en el IP del *nodo* del puerto 4444 y recibir una respuesta del servidor web. Ten en cuenta que esto limita el escalado a un solo pod por nodo ya que debe aplicar una asignación uno a uno.
