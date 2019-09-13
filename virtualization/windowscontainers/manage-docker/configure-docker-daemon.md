---
title: Configuración de Docker en Windows
description: Configuración de Docker en Windows
keywords: docker, contenedores
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: c84a6652b5918238ee8ef6e1fa7a9b2aa596aefd
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129265"
---
# <a name="docker-engine-on-windows"></a>Docker Engine en Windows

El motor de acoplamiento y el cliente no se incluyen con Windows y deben instalarse y configurarse individualmente. Además, el motor de Docker puede aceptar varias configuraciones personalizadas. Algunos ejemplos incluyen la configuración de cómo acepta el demonio las solicitudes entrantes, las opciones de red predeterminadas y la configuración de registro y depuración. En Windows, estas configuraciones pueden especificarse en un archivo de configuración o mediante el Administrador de control de servicios de Windows. En este documento se explica cómo instalar y configurar el motor de acoplamiento, y también se proporcionan algunos ejemplos de las configuraciones más usadas.

## <a name="install-docker"></a>Instalar Docker

Para trabajar con contenedores de Windows, necesita acoplador. Docker está formado por el motor de Docker (dockerd.exe) y el cliente de Docker (docker.exe). La forma más sencilla de obtener todo lo que se ha instalado está en la guía de inicio rápido, que le ayudará a configurarlo todo y ejecutarlo.

- [Instalar Docker](../quick-start/set-up-environment.md)

Para instalaciones con scripts, consulte [usar un script para instalar docko EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Antes de que pueda usar el acoplador, tendrá que instalar las imágenes de contenedor. Para obtener más información, vea [documentos para nuestras imágenes base de contenedor](../manage-containers/container-base-images.md).

## <a name="configure-docker-with-a-configuration-file"></a>Configurar el acoplador con un archivo de configuración

El método preferido para configurar el motor de Docker en Windows es usar un archivo de configuración. El archivo de configuración se encuentra en 'C:\ProgramData\Docker\config\daemon.json'. Puede crear este archivo si aún no existe.

>[!NOTE]
>No todas las opciones de configuración del acoplador disponibles se aplican al acoplador en Windows. En el ejemplo siguiente se muestran las opciones de configuración que se aplican. Para obtener más información sobre la configuración del motor del acoplador, consulte [archivo de configuración de demonio de Docker](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

```json
{
    "authorization-plugins": [],
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "storage-driver": "",
    "storage-opts": [],
    "labels": [],
    "log-driver": "", 
    "mtu": 0,
    "pidfile": "",
    "data-root": "",
    "cluster-store": "",
    "cluster-advertise": "",
    "debug": true,
    "hosts": [],
    "log-level": "",
    "tlsverify": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "group": "",
    "default-ulimits": {},
    "bridge": "",
    "fixed-cidr": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "insecure-registries": [],
    "disable-legacy-registry": false
}
```

Solo necesita agregar los cambios de configuración que desee en el archivo de configuración. Por ejemplo, el siguiente ejemplo configura el motor de acoplamiento para aceptar las conexiones entrantes en el puerto 2375. Las demás opciones de configuración usarán los valores predeterminados.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Del mismo modo, en la siguiente muestra se configura el daemon del acoplador para mantener las imágenes y los contenedores en una ruta de acceso alternativa. Si no se especifica, el valor `c:\programdata\docker`predeterminado es.

```json
{    
    "data-root": "d:\\docker"
}
```

En el siguiente ejemplo se configura el demonio de acoplamiento para que solo acepte conexiones seguras por el puerto 2376.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Configurar el acoplador en el servicio del acoplador

El motor de acoplamiento también se puede configurar modificando el servicio del acoplador `sc config`. Con este método, se establecen marcas de motor de Docker directamente en el servicio de Docker. Ejecute el siguiente comando en un símbolo del sistema (cmd.exe, no PowerShell):

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>No es necesario que ejecute este comando si el archivo daemon. JSON ya contiene la `"hosts": ["tcp://0.0.0.0:2375"]` entrada.

## <a name="common-configuration"></a>Configuración común

Los siguientes ejemplos de archivos de configuración muestran configuraciones de Docker comunes. Esta configuración se puede combinar en un único archivo de configuración.

### <a name="default-network-creation"></a>Creación de red predeterminada

Para configurar el motor de acoplamiento de modo que no cree una red NAT predeterminada, use la configuración siguiente.

```json
{
    "bridge" : "none"
}
```

Para obtener más información, vea [Administrar redes de Docker](../container-networking/network-drivers-topologies.md).

### <a name="set-docker-security-group"></a>Establecer grupo de seguridad del acoplador

Cuando haya iniciado sesión en el host del acoplador y esté ejecutando de forma local comandos del Docker, estos comandos se ejecutarán en una canalización con nombre. De forma predeterminada, solo los miembros del grupo de administradores pueden tener acceso al motor de Docker a través de la canalización con nombre. Para especificar un grupo de seguridad que tiene este acceso, use la marca `group`.

```json
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>Configuración de proxy

Para establecer la información del proxy para `docker search` y `docker pull`, cree una variable de entorno de Windows con el nombre `HTTP_PROXY` o `HTTPS_PROXY` y un valor de información del proxy. Esto se puede completar con PowerShell mediante un comando similar al siguiente:

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

Una vez que se ha establecido la variable, reinicie el servicio Docker.

```powershell
Restart-Service docker
```

Para obtener más información, vea [archivo de configuración de Windows en Docker.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="how-to-uninstall-docker"></a>Cómo desinstalar el acoplador

En esta sección se explica cómo desinstalar el acoplador y cómo realizar una limpieza completa de los componentes del sistema de acoplamiento desde el sistema Windows 10 o Windows Server 2016.

>[!NOTE]
>Debe ejecutar todos los comandos de estas instrucciones desde una sesión de PowerShell con privilegios elevados.

### <a name="prepare-your-system-for-dockers-removal"></a>Preparar el sistema para la eliminación de un acoplador

Antes de desinstalar el acoplador, asegúrese de que no haya contenedores en el sistema.

Ejecute los siguientes cmdlets para comprobar los contenedores en ejecución:

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

También es una buena práctica quitar todos los contenedores, imágenes de contenedor, redes y volúmenes de su sistema antes de quitar el acoplador. Para hacerlo, ejecute el siguiente cmdlet:

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Desinstalar Docker

A continuación, tendrá que desinstalar el acoplador.

Para desinstalar el acoplador en Windows 10

- Ve a **configuración** > **aplicaciones** en tu equipo con Windows 10
- En **aplicaciones & características**, busque **acoplador para Windows**
- Ir a la**desinstalación** **del acoplador para Windows** > 

Para desinstalar el acoplador en Windows Server 2016:

Desde una sesión de PowerShell con privilegios elevados, use los cmdlets **Uninstall-Package** and **Uninstall-Module** para quitar el módulo del acoplador y su proveedor de administración de paquetes correspondiente del sistema, tal y como se muestra en el siguiente ejemplo:

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Puede encontrar el proveedor de paquetes que usó para instalar el Dock `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Limpiar los componentes del sistema y los datos del Dock

Después de desinstalar el acoplador, tendrá que quitar las redes predeterminadas de los acopladores, de modo que la configuración no permanezca en el sistema después de que el acoplador haya desaparecido. Para hacerlo, ejecute el siguiente cmdlet:

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Ejecute el siguiente cmdlet para quitar los datos del programa del acoplador de su sistema:

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

También puedes eliminar las características opcionales de Windows asociadas a Docker o los contenedores en Windows.

Esto incluye la característica "contenedores", que se habilita automáticamente en cualquier Windows 10 o Windows Server 2016 cuando se instala el acoplador. También puede incluir la característica "Hyper-V", que se habilita automáticamente en Windows 10 cuando Docker está instalado, pero debe estar habilitado de forma explícita en Windows Server 2016.

>[!IMPORTANT]
>[La característica Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/) es una característica general de la virtualización que permite mucho más que solo contenedores. Antes de deshabilitar la característica Hyper-V, asegúrese de que no hay otros componentes virtualizados en su sistema que necesiten Hyper-V.

Para quitar las características de Windows en Windows 10:

- Vaya al **Panel** > de control**programas** > **y características** > activan**o desactivan las características de Windows**.
- Busque el nombre de la característica o características que desea deshabilitar, en este caso, **contenedores** y (opcionalmente) **Hyper-V**.
- Desactive la casilla situada junto al nombre de la característica que desea deshabilitar.
- Seleccione **"Aceptar"** .

Para quitar características de Windows en Windows Server 2016:

Desde una sesión de PowerShell con privilegios elevados, ejecute los siguientes cmdlets para deshabilitar los **contenedores** y las características de **Hyper-V** (opcionalmente) de su sistema:

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>Reiniciar el sistema

Para finalizar la desinstalación y la limpieza, ejecute el siguiente cmdlet desde una sesión de PowerShell con privilegios elevados para reiniciar el sistema:

```powershell
Restart-Computer -Force
```
