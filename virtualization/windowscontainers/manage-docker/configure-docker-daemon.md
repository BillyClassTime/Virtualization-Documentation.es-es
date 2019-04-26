---
title: Configuración de Docker en Windows
description: Configuración de Docker en Windows
keywords: docker, contenedores
author: PatrickLang
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: 24de3d332ae4634f7dca945c1df9182cc1310089
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9574746"
---
# <a name="docker-engine-on-windows"></a>Docker Engine en Windows

El motor de Docker y el cliente no se incluyen con Windows y que esté instalado y configurado de forma individual. Además, el motor de Docker puede aceptar varias configuraciones personalizadas. Algunos ejemplos incluyen la configuración de cómo acepta el demonio las solicitudes entrantes, las opciones de red predeterminadas y la configuración de registro y depuración. En Windows, estas configuraciones pueden especificarse en un archivo de configuración o mediante el Administrador de control de servicios de Windows. Este documento describe cómo instalar y configurar el motor de Docker y también proporciona algunos ejemplos de configuraciones frecuentes.


## <a name="install-docker"></a>Instalar Docker
Para trabajar con contenedores de Windows es necesario Docker. Docker está formado por el motor de Docker (dockerd.exe) y el cliente de Docker (docker.exe). La forma más fácil de instalar todo está en las guías de inicio rápido. Ayudan a obtener todo lo que permite configurar y ejecuta tu primer contenedor. 

* [Contenedores de Windows en Windows Server 2019](../quick-start/quick-start-windows-server.md)
* [Contenedores de Windows en Windows 10](../quick-start/quick-start-windows-10.md)

Para instalaciones con script, consulta [Usar un script para instalar Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Antes de que se puede usar Docker que se puede instalar imágenes de contenedor. Para obtener más información, consulta [Guía de inicio de la rápido para usar imágenes](../quick-start/quick-start-images.md).

## <a name="configure-docker-with-configuration-file"></a>Configurar Docker con el archivo de configuración

El método preferido para configurar el motor de Docker en Windows es usar un archivo de configuración. El archivo de configuración se encuentra en 'C:\ProgramData\Docker\config\daemon.json'. Si este archivo no existe, se puede crear.

Nota: no todas las opciones de configuración de Docker disponibles se pueden aplicar a Docker en Windows. El siguiente ejemplo muestra cuáles son. Para obtener documentación completa sobre la configuración del motor de Docker, consulte [Archivo de configuración de demonio de Docker](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

```
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

Solo se necesitan agregar los cambios de configuración deseados al archivo de configuración. Por ejemplo, en esta muestra se configura el motor de Docker para que acepte las conexiones entrantes a través del puerto TCP 2375. Las demás opciones de configuración usarán los valores predeterminados.

```
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Del mismo modo, este ejemplo configura el demonio de Docker para mantener las imágenes y los contenedores en una ruta alternativa. Si no se especifica, el valor predeterminado es c:\programdata\docker.

```
{    
    "data-root": "d:\\docker"
}
```

De la misma forma, en esta muestra se configura el demonio de Docker para aceptar únicamente conexiones seguras a través del puerto 2376.

```
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Configurar Docker en Docker Service

El motor de Docker también se puede configurar modificando el servicio de Docker con `sc config`. Con este método, se establecen marcas de motor de Docker directamente en el servicio de Docker. Ejecute el siguiente comando en un símbolo del sistema (cmd.exe, no PowerShell):


```
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

Nota: No es necesario ejecutar este comando si el archivo daemon.json ya contiene la entrada `"hosts": ["tcp://0.0.0.0:2375"]`.

## <a name="common-configuration"></a>Configuración común

Los siguientes ejemplos de archivos de configuración muestran configuraciones de Docker comunes. Esta configuración se puede combinar en un único archivo de configuración.

### <a name="default-network-creation"></a>Creación de red predeterminada 

Para configurar el motor de Docker para que no se cree la red NAT predeterminada, use lo siguiente. Para obtener más información, vea [Administrar redes de Docker](../container-networking/network-drivers-topologies.md).

```
{
    "bridge" : "none"
}
```

### <a name="set-docker-security-group"></a>Definir el grupo de seguridad de Docker

Si ha iniciado sesión en el host de Docker y ejecuta comandos de Docker de forma local, estos se ejecutan a través de una canalización con nombre. De forma predeterminada, solo los miembros del grupo de administradores pueden tener acceso al motor de Docker a través de la canalización con nombre. Para especificar un grupo de seguridad que tiene este acceso, use la marca `group`.

```
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

Para obtener más información, consulte [Archivo de configuración de Windows en Docker.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="uninstall-docker"></a>Desinstalar Docker
*Sigue los pasos de esta sección para desinstalar Docker y realizar una limpieza completa de componentes del sistema Docker del sistema Windows 10 o Windows Server 2016.*

> Nota: Todos los comandos incluidos en los siguientes pasos deben ejecutarse desde una sesión **con privilegios elevados** de PowerShell.

### <a name="step-1-prepare-your-system-for-dockers-removal"></a>PASO 1: Preparar el sistema para la eliminación de Docker 
Si no lo has hecho aún, te recomendamos asegurarte de que no hay contenedores ejecutándose en el sistema antes de eliminar Docker. Estos son algunos comandos útiles para hacerlo:
```
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```
También recomendamos eliminar todos los contenedores, imágenes de contenedor, redes y volúmenes del sistema antes de eliminar Docker:
```
docker system prune --volumes --all
```

### <a name="step-2-uninstall-docker"></a>PASO 2: Desinstalar Docker 

#### ***<a name="steps-to-uninstall-docker-on-windows-10"></a>Pasos para desinstalar Docker en Windows 10:10:***
- Ve a **"Configuración" > "Aplicaciones"** en tu equipo Windows 10
- En **"Aplicaciones y características"**, busca **"Docker para Windows"**
- Haz clic en **"Docker para Windows" > "Desinstalar"**

#### ***<a name="steps-to-uninstall-docker-on-windows-server-2016"></a>Pasos para desinstalar Docker en Windows Server 2016:16:***
Desde una sesión con privilegios elevados de PowerShell, usa los cmdlets `Uninstall-Package` y `Uninstall-Module` para eliminar el módulo Docker y su correspondiente Proveedor de administración de paquetes de tu sistema. 
> Recomendación: Puedes encontrar el Proveedor de paquetes que usaste para instalar Docker con `PS C:\> Get-PackageProvider -Name *Docker*`

*Por ejemplo*:
```
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

### <a name="step-3-cleanup-docker-data-and-system-components"></a>PASO 3: Limpiar componentes del sistema y datos de Docker
Elimina las *redes predeterminadas* de Docker para que su configuración también se elimine del sistema una vez que se quite Docker:
```
Get-HNSNetwork | Remove-HNSNetwork
```
Elimina los *datos de programa* de Docker del sistema:
```
Remove-Item "C:\ProgramData\Docker" -Recurse
```
También puedes eliminar las *características opcionales de Windows* asociadas a Docker o los contenedores en Windows. 

Como mínimo, esto incluye la característica "Contenedores", que se habilita automáticamente en Windows 10 o Windows Server 2016 cuando se instala Docker. También puede incluir la característica "Hyper-V", que se habilita automáticamente en Windows 10 cuando Docker está instalado, pero debe estar habilitado de forma explícita en Windows Server 2016.

> **NOTA IMPORTANTE SOBRE LA DESHABILITACIÓN DE HYPER-V:** [La característica Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/) es una característica de virtualización general que habilita mucho más que solo contenedores. Antes de deshabilitar la característica Hyper-V, asegúrate de que no hay otros componentes virtualizados en el sistema que la necesiten.

#### ***<a name="steps-to-remove-windows-features-on-windows-10"></a>Pasos para eliminar características de Windows en Windows 10:10:***
- Ve a **"Panel de control" > "Programas" > "Programas y características" > "Activar o desactivar características de Windows"** en tu máquina Windows 10
- Busca el nombre de la características o características que te gustaría deshabilitar, en este caso, **"Contenedores"** y (de forma opcional) **"Hyper-V"**
- **Desactiva** la casilla junto al nombre de la característica que te gustaría deshabilitar
- Haz clic en **"Aceptar"**

#### ***<a name="steps-to-remove-windows-features-on-windows-server-2016"></a>Pasos para eliminar características de Windows en Windows Server 2016:16:***
Desde una sesión con privilegios elevados de PowerShell, usa los siguientes comandos para deshabilitar las características **"Contenedores"** y (de forma opcional) **"Hyper-V"** del sistema:
```
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V 
```

### <a name="step-4-reboot-your-system"></a>PASO 4: Reiniciar el sistema
Para completar estos pasos para desinstalar y limpiar, desde una sesión con privilegios elevados de PowerShell, ejecuta:
```
Restart-Computer -Force
```
