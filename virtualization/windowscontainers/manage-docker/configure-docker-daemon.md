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
ms.openlocfilehash: 354469199f3c7e886760e8a391edccde067986af
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610295"
---
# <a name="docker-engine-on-windows"></a>Docker Engine en Windows

El motor de Docker y el cliente no se incluyen con Windows y que esté instalado y configurado de forma individual. Además, el motor de Docker puede aceptar varias configuraciones personalizadas. Algunos ejemplos incluyen la configuración de cómo acepta el demonio las solicitudes entrantes, las opciones de red predeterminadas y la configuración de registro y depuración. En Windows, estas configuraciones pueden especificarse en un archivo de configuración o mediante el Administrador de control de servicios de Windows. Este documento describe cómo instalar y configurar el motor de Docker y también proporciona algunos ejemplos de configuraciones frecuentes.

## <a name="install-docker"></a>Instalar Docker

Es necesario Docker para trabajar con contenedores de Windows. Docker está formado por el motor de Docker (dockerd.exe) y el cliente de Docker (docker.exe). La forma más fácil de instalar todo está en el inicio rápido guías, lo que te ayudarán a obtener todo lo que permite configurar y ejecutan tu primer contenedor.

- [Contenedores de Windows en Windows Server 2019](../quick-start/quick-start-windows-server.md)
- [Contenedores de Windows en Windows 10](../quick-start/quick-start-windows-10.md)

Para instalaciones con script, vea [usar un script para instalar Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Antes de poder usar Docker, tendrás que instalar las imágenes del contenedor. Para obtener más información, consulta [la Guía de inicio rápido para el uso de imágenes](../quick-start/quick-start-images.md).

## <a name="configure-docker-with-a-configuration-file"></a>Configurar Docker con un archivo de configuración

El método preferido para configurar el motor de Docker en Windows es usar un archivo de configuración. El archivo de configuración se encuentra en 'C:\ProgramData\Docker\config\daemon.json'. Puedes crear este archivo si aún no existe.

>[!NOTE]
>No todas las opciones de configuración Docker disponibles se aplica a Docker en Windows. El siguiente ejemplo muestra las opciones de configuración que se aplican. Para obtener más información acerca de la configuración del motor de Docker, consulte el [archivo de configuración de demonio de Docker](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

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

Solo debes agregar los cambios de configuración deseados al archivo de configuración. Por ejemplo, en el siguiente ejemplo configura el motor de Docker para que acepte las conexiones entrantes en el puerto TCP 2375. Las demás opciones de configuración usarán los valores predeterminados.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

De igual modo, el siguiente ejemplo configura el demonio de Docker para mantener las imágenes y los contenedores en una ruta alternativa. Si no se especifica, el valor predeterminado es `c:\programdata\docker`.

```json
{    
    "data-root": "d:\\docker"
}
```

En el siguiente ejemplo configura el demonio de Docker para aceptar únicamente conexiones seguras a través del puerto 2376.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Configurar Docker en Docker service

El motor de Docker también se puede configurar modificando el servicio de Docker con `sc config`. Con este método, se establecen marcas de motor de Docker directamente en el servicio de Docker. Ejecute el siguiente comando en un símbolo del sistema (cmd.exe, no PowerShell):

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>No es necesario ejecutar este comando si el archivo daemon.json ya contiene la `"hosts": ["tcp://0.0.0.0:2375"]` entrada.

## <a name="common-configuration"></a>Configuración común

Los siguientes ejemplos de archivos de configuración muestran configuraciones de Docker comunes. Esta configuración se puede combinar en un único archivo de configuración.

### <a name="default-network-creation"></a>Creación de una red predeterminada

Para configurar el motor de Docker para que no crea una red NAT predeterminada, usa la siguiente configuración.

```json
{
    "bridge" : "none"
}
```

Para obtener más información, vea [Administrar redes de Docker](../container-networking/network-drivers-topologies.md).

### <a name="set-docker-security-group"></a>Definir el grupo de seguridad de Docker

Cuando hayas iniciado sesión en el host de Docker y se ejecutan localmente comandos de Docker, estos comandos se ejecutan a través de una canalización con nombre. De forma predeterminada, solo los miembros del grupo de administradores pueden tener acceso al motor de Docker a través de la canalización con nombre. Para especificar un grupo de seguridad que tiene este acceso, use la marca `group`.

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

Para obtener más información, consulta el [Archivo de configuración de Windows en Docker.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="how-to-uninstall-docker"></a>Cómo desinstalar Docker

En esta sección te indicará cómo desinstalar Docker y realizar una limpieza completa de componentes del sistema Docker del sistema de Windows 10 o Windows Server 2016.

>[!NOTE]
>Debes ejecutar todos los comandos en estas instrucciones desde una sesión de PowerShell con privilegios elevados.

### <a name="prepare-your-system-for-dockers-removal"></a>Preparación del sistema para la eliminación de Docker

Antes de desinstalar Docker, asegúrate de que no hay contenedores ejecutándose en el sistema.

Ejecute los siguientes cmdlets para buscar de contenedores en ejecución:

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

También es recomendable eliminar todos los contenedores, imágenes de contenedor, redes y volúmenes del sistema antes de eliminar Docker. Para ello, ejecuta el siguiente cmdlet:

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Desinstalar Docker

A continuación, tendrás que realmente desinstalar Docker.

Para desinstalar Docker en Windows 10

- Ve a **configuración** > **aplicaciones** en la máquina de Windows 10
- En **las aplicaciones & características**, encontrar **Docker para Windows**
- Ve a **Docker para Windows** > **desinstalar**

Para desinstalar Docker en Windows Server 2016:

Desde una sesión de PowerShell con privilegios elevados, usa los cmdlets de **Paquete de la desinstalación** y el **Módulo de desinstalación** para quitar el módulo Docker y su correspondiente proveedor de administración de paquetes de su sistema, como se muestra en el ejemplo siguiente:

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Puedes encontrar el proveedor de paquetes que usaste para instalar a Docker con `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Limpiar componentes de sistema y los datos de Docker

Después de desinstalar Docker, tendrás que eliminar las redes de Docker de forma predeterminada para que su configuración no permanecen en el sistema después de que se quite Docker. Para ello, ejecuta el siguiente cmdlet:

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Ejecuta el siguiente cmdlet para quitar los datos del programa de Docker del sistema:

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

También puedes eliminar las características opcionales de Windows asociadas a Docker o los contenedores en Windows.

Esto incluye la característica "Contenedores", que se habilita automáticamente en Windows 10 o Windows Server 2016 cuando se instala Docker. También puede incluir la característica "Hyper-V", que se habilita automáticamente en Windows 10 cuando Docker está instalado, pero debe estar habilitado de forma explícita en Windows Server 2016.

>[!IMPORTANT]
>[Característica el Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/) es una característica de virtualización general que habilita mucho más que solo contenedores. Antes de deshabilitar la característica Hyper-V, asegúrate de que no existen otros componentes virtualizados en el sistema que requieren Hyper-V.

Para quitar las características de Windows en Windows 10:

- Ve a **Panel de Control** > **programas** > **programas y características** > **activar características de Windows activado o desactivado**.
- Busca el nombre de la característica o características que se va a deshabilitar: en este caso, **Hyper-V** **contenedores** y (opcionalmente).
- Desactiva la casilla situada junto al nombre de la característica que desee deshabilitar.
- Selecciona **"Aceptar"**

Para quitar las características de Windows en Windows Server 2016:

Desde una sesión de PowerShell con privilegios elevados, ejecute los siguientes cmdlets para deshabilitar las características de **Hyper-V** desde el sistema de **los contenedores** y (opcionalmente):

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>Reiniciar el sistema

Para finalizar la desinstalación y la limpieza, ejecuta el siguiente cmdlet desde una sesión de PowerShell con privilegios elevados para reiniciar el sistema:

```powershell
Restart-Computer -Force
```
