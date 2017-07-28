---
title: "Configuración de Docker en Windows"
description: "Configuración de Docker en Windows"
keywords: docker, contenedores
author: PatrickLang
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: f266404f12e47c8605436af44e636c54ec6ef8e5
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/21/2017
---
# Docker Engine en Windows

El cliente y el motor de Docker no se incluyen con Windows y deberán instalarse y configurarse por separado. Además, el motor de Docker puede aceptar varias configuraciones personalizadas. Algunos ejemplos incluyen la configuración de cómo acepta el demonio las solicitudes entrantes, las opciones de red predeterminadas y la configuración de registro y depuración. En Windows, estas configuraciones pueden especificarse en un archivo de configuración o mediante el Administrador de control de servicios de Windows. En este documento se detallará cómo instalar y configurar el motor de Docker. Además, se proporcionarán algunos ejemplos de configuraciones frecuentes.


## Instalar Docker
Para trabajar con contenedores de Windows es necesario Docker. Docker está formado por el motor de Docker (dockerd.exe) y el cliente de Docker (docker.exe). La forma más fácil de instalar todo está en las guías de inicio rápido. Le ayudará a configurar todo y a poner en ejecución su primer contenedor. 

* [Contenedores de Windows en Windows Server 2016](../quick-start/quick-start-windows-server.md)
* [Contenedores de Windows en Windows 10](../quick-start/quick-start-windows-10.md)


### Instalación manual
Si deseas utilizar una versión en desarrollo del motor y el cliente de Docker, puedes seguir los siguientes pasos. Esto instalará el motor de Docker y el cliente. Si eres un desarrollador probando nuevas características o usando una versión de compilación de Windows Insider, piensa en usar una versión en desarrollo de Docker. De lo contrario, sigue los pasos en la sección "Instalar Docker" que está más arriba para obtener las versiones más recientes.

> Si has instalado Docker para Windows, asegúrate de quitarlo antes de seguir estos pasos de instalación manual. 

Descargar el motor de Docker

Siempre puedes encontrar la versión más reciente en https://master.dockerproject.org. Este ejemplo usa la última versión de la rama principal. 

```powershell
$version = (Invoke-WebRequest -UseBasicParsing https://raw.githubusercontent.com/docker/docker/master/VERSION).Content.Trim()
Invoke-WebRequest "https://master.dockerproject.org/windows/x86_64/docker-$($version).zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Expande el archivo zip en Archivos de programa.

```powershell
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Agrega el directorio de Docker a la ruta de acceso del sistema. Una vez hecho esto, reinicie la sesión de PowerShell para que reconozca la ruta de acceso modificada.

```powershell
# Add path to this PowerShell session immediately
$env:path += ";$env:ProgramFiles\Docker"

# For persistent use after a reboot
$existingMachinePath = [Environment]::GetEnvironmentVariable("Path",[System.EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("Path", $existingMachinePath + ";$env:ProgramFiles\Docker", [EnvironmentVariableTarget]::Machine)
```

Para instalar Docker como un servicio de Windows, ejecute lo siguiente.

```none
dockerd --register-service
```

Una vez instalado, puede iniciar el servicio.

```powershell
Start-Service Docker
```

Tendrán que instalarse imágenes de contenedor antes de poder usar Docker. Para obtener más información, consulta [Guía de inicio de la rápido para usar imágenes](../quick-start/quick-start-images.md).

## Configurar Docker con el archivo de configuración

El método preferido para configurar el motor de Docker en Windows es usar un archivo de configuración. El archivo de configuración se encuentra en 'C:\ProgramData\Docker\config\daemon.json'. Si este archivo no existe, se puede crear.

Nota: no todas las opciones de configuración de Docker disponibles se pueden aplicar a Docker en Windows. El siguiente ejemplo muestra cuáles son. Para obtener documentación completa sobre la configuración del motor de Docker, consulte [Archivo de configuración de demonio de Docker](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

```none
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
    "graph": "",
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

```none
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Del mismo modo, este ejemplo configura el demonio de Docker para mantener las imágenes y los contenedores en una ruta alternativa. Si no se especifica, el valor predeterminado es c:\programdata\docker.

```none
{    
    "graph": "d:\\docker"
}
```

De la misma forma, en esta muestra se configura el demonio de Docker para aceptar únicamente conexiones seguras a través del puerto 2376.

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## Configurar Docker en Docker Service

El motor de Docker también se puede configurar modificando el servicio de Docker con `sc config`. Con este método, se establecen marcas de motor de Docker directamente en el servicio de Docker. Ejecute el siguiente comando en un símbolo del sistema (cmd.exe, no PowerShell):


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

Nota: No es necesario ejecutar este comando si el archivo daemon.json ya contiene la entrada `"hosts": ["tcp://0.0.0.0:2375"]`.

## Configuración común

Los siguientes ejemplos de archivos de configuración muestran configuraciones de Docker comunes. Esta configuración se puede combinar en un único archivo de configuración.

### Creación de red predeterminada 

Para configurar el motor de Docker para que no se cree la red NAT predeterminada, use lo siguiente. Para obtener más información, vea [Administrar redes de Docker](../manage-containers/container-networking.md).

```none
{
    "bridge" : "none"
}
```

### Definir el grupo de seguridad de Docker

Si ha iniciado sesión en el host de Docker y ejecuta comandos de Docker de forma local, estos se ejecutan a través de una canalización con nombre. De forma predeterminada, solo los miembros del grupo de administradores pueden tener acceso al motor de Docker a través de la canalización con nombre. Para especificar un grupo de seguridad que tiene este acceso, use la marca `group`.

```none
{
    "group" : "docker"
}
```

## Configuración de proxy

Para establecer la información del proxy para `docker search` y `docker pull`, cree una variable de entorno de Windows con el nombre `HTTP_PROXY` o `HTTPS_PROXY` y un valor de información del proxy. Esto se puede completar con PowerShell mediante un comando similar al siguiente:

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

Una vez que se ha establecido la variable, reinicie el servicio Docker.

```powershell
Restart-Service docker
```

Para obtener más información, consulte [Archivo de configuración de Windows en Docker.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

