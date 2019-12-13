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
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910165"
---
# <a name="docker-engine-on-windows"></a>Docker Engine en Windows

El motor de Docker y el cliente no se incluyen en Windows y deben instalarse y configurarse individualmente. Además, el motor de Docker puede aceptar varias configuraciones personalizadas. Algunos ejemplos incluyen la configuración de cómo acepta el demonio las solicitudes entrantes, las opciones de red predeterminadas y la configuración de registro y depuración. En Windows, estas configuraciones pueden especificarse en un archivo de configuración o mediante el Administrador de control de servicios de Windows. En este documento se detalla cómo instalar y configurar el motor de Docker y también se proporcionan algunos ejemplos de configuraciones de uso frecuente.

## <a name="install-docker"></a>Instalar Docker

Necesita Docker para trabajar con contenedores de Windows. Docker está formado por el motor de Docker (dockerd.exe) y el cliente de Docker (docker.exe). La forma más fácil de conseguir todo instalado es la guía de inicio rápido, que le ayudará a preparar todo el equipo y ejecutar el primer contenedor.

- [Instalación de Docker](../quick-start/set-up-environment.md)

Para las instalaciones con scripts, consulte [uso de un script para instalar Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Antes de poder usar Docker, deberá instalar las imágenes de contenedor. Para obtener más información, consulte [docs para nuestras imágenes base del contenedor](../manage-containers/container-base-images.md).

## <a name="configure-docker-with-a-configuration-file"></a>Configuración de Docker con un archivo de configuración

El método preferido para configurar el motor de Docker en Windows es usar un archivo de configuración. El archivo de configuración se encuentra en 'C:\ProgramData\Docker\config\daemon.json'. Puede crear este archivo si aún no existe.

>[!NOTE]
>No todas las opciones de configuración de Docker disponibles se aplican a Docker en Windows. En el ejemplo siguiente se muestran las opciones de configuración que se aplican. Para más información sobre la configuración del motor de Docker, vea [archivo de configuración de demonio de Docker](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

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

Solo tiene que agregar los cambios de configuración deseados al archivo de configuración. Por ejemplo, en el ejemplo siguiente se configura el motor de Docker para que acepte las conexiones entrantes en el puerto 2375. Las demás opciones de configuración usarán los valores predeterminados.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Del mismo modo, en el ejemplo siguiente se configura el demonio de Docker para mantener las imágenes y los contenedores en una ruta de acceso alternativa. Si no se especifica, el valor predeterminado es `c:\programdata\docker`.

```json
{    
    "data-root": "d:\\docker"
}
```

En el ejemplo siguiente se configura el demonio de Docker para aceptar solo conexiones seguras a través del puerto 2376.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Configuración de Docker en el servicio Docker

El motor de Docker también se puede configurar modificando el servicio de Docker con `sc config`. Con este método, se establecen marcas de motor de Docker directamente en el servicio de Docker. Ejecute el siguiente comando en un símbolo del sistema (cmd.exe, no PowerShell):

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>No es necesario ejecutar este comando si el archivo daemon. JSON ya contiene la entrada `"hosts": ["tcp://0.0.0.0:2375"]`.

## <a name="common-configuration"></a>Configuración común

Los siguientes ejemplos de archivos de configuración muestran configuraciones de Docker comunes. Esta configuración se puede combinar en un único archivo de configuración.

### <a name="default-network-creation"></a>Creación de red predeterminada

Para configurar el motor de Docker para que no cree una red NAT predeterminada, use la configuración siguiente.

```json
{
    "bridge" : "none"
}
```

Para obtener más información, vea [Administrar redes de Docker](../container-networking/network-drivers-topologies.md).

### <a name="set-docker-security-group"></a>Establecimiento de un grupo de seguridad de Docker

Cuando ha iniciado sesión en el host de Docker y está ejecutando localmente comandos de Docker, estos comandos se ejecutan a través de una canalización con nombre. De forma predeterminada, solo los miembros del grupo de administradores pueden tener acceso al motor de Docker a través de la canalización con nombre. Para especificar un grupo de seguridad que tiene este acceso, use la marca `group`.

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

Para obtener más información, consulte [archivo de configuración de Windows en Docker.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="how-to-uninstall-docker"></a>Desinstalación de Docker

En esta sección se explica cómo desinstalar Docker y realizar una limpieza completa de los componentes del sistema de Docker desde el sistema Windows 10 o Windows Server 2016.

>[!NOTE]
>Debe ejecutar todos los comandos de estas instrucciones desde una sesión de PowerShell con privilegios elevados.

### <a name="prepare-your-system-for-dockers-removal"></a>Preparación del sistema para la eliminación de Docker

Antes de desinstalar Docker, asegúrese de que no haya contenedores en ejecución en el sistema.

Ejecute los siguientes cmdlets para comprobar los contenedores en ejecución:

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

También se recomienda quitar todos los contenedores, imágenes de contenedor, redes y volúmenes del sistema antes de quitar Docker. Puede hacerlo mediante la ejecución del siguiente cmdlet:

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Desinstalar Docker

A continuación, deberá desinstalar Docker.

Para desinstalar Docker en Windows 10

- Vaya a **configuración** > **aplicaciones** en el equipo con Windows 10.
- En **aplicaciones & características**, busque **Docker para Windows**
- Vaya a **Docker para Windows** > **desinstalar** .

Para desinstalar Docker en Windows Server 2016:

En una sesión de PowerShell con privilegios elevados, use los cmdlets **Uninstall-Package** y **Uninstall-Module** para quitar el módulo de Docker y su proveedor de administración de paquetes correspondiente del sistema, tal como se muestra en el ejemplo siguiente:

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Puede encontrar el proveedor de paquetes que usó para instalar Docker con `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Limpieza de los componentes del sistema y los datos de Docker

Después de desinstalar Docker, deberá quitar las redes predeterminadas de Docker para que su configuración no permanezca en el sistema después de que haya salido de Docker. Puede hacerlo mediante la ejecución del siguiente cmdlet:

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Ejecute el siguiente cmdlet para quitar los datos del programa de Docker del sistema:

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

También puede que desee quitar las características opcionales de Windows asociadas a Docker/containers en Windows.

Esto incluye la característica "contenedores", que se habilita automáticamente en cualquier Windows 10 o Windows Server 2016 cuando se instala Docker. También puede incluir la característica "Hyper-V", que se habilita automáticamente en Windows 10 cuando Docker está instalado, pero debe estar habilitado de forma explícita en Windows Server 2016.

>[!IMPORTANT]
>[La característica de Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/) es una característica de virtualización general que permite mucho más que simplemente contenedores. Antes de deshabilitar la característica de Hyper-V, asegúrese de que no hay ningún otro componente virtualizado en el sistema que requiera Hyper-V.

Para quitar las características de Windows en Windows 10:

- Vaya al **Panel de Control** > **programas** > **programas y características** > **activar o desactivar las características de Windows**.
- Busque el nombre de la característica o características que desea deshabilitar; en este caso, **contenedores** y (opcionalmente) **Hyper-V**.
- Desactive la casilla situada junto al nombre de la característica que desea deshabilitar.
- Seleccione **"Aceptar"** .

Para quitar las características de Windows en Windows Server 2016:

Desde una sesión de PowerShell con privilegios elevados, ejecute los siguientes cmdlets para deshabilitar los **contenedores** y, opcionalmente, las características de **Hyper-V** del sistema:

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>Reiniciar el sistema

Para finalizar la desinstalación y la limpieza, ejecute el siguiente cmdlet desde una sesión de PowerShell con privilegios elevados para reiniciar el sistema:

```powershell
Restart-Computer -Force
```
