---
title: "Implementación de contenedores de Windows en Nano Server"
description: "Implementación de contenedores de Windows en Nano Server"
keywords: docker, contenedores
author: neilpeterson
manager: timlt
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: 939a1b69f159504b998792adb95ccabc326db333
ms.openlocfilehash: 538fb27d6170f0a8dab5c189b90040e40c546e14

---

# Implementación de host de contenedor: Nano Server

**Esto es contenido preliminar y está sujeto a cambios.** 

Este documento le guiará paso a paso por una implementación básica de Nano Server con la característica de contenedor de Windows. Este es un tema avanzado y en él se presupone un conocimiento general de Windows y de los contenedores de Windows. Para obtener una introducción a los contenedores de Windows, vea [Inicio rápido de contenedores de Windows](../quick_start/quick_start.md).

## Preparar Nano Server

En la siguiente sección se describe la implementación de una configuración muy básica de Nano Server. Para obtener una explicación más detallada de las opciones de implementación y configuración de Nano Server, vea [Getting Started with Nano Server (Introducción a Nano Server)] (https://technet.microsoft.com/en-us/library/mt126167.aspx).

### Crear una máquina virtual de Nano Server

En primer lugar, descargue el VHD de evaluación de Nano Server desde [esta ubicación](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula). Cree una máquina virtual desde este VHD, iníciela y conéctese a ella con la opción de conexión de Hyper-V (o equivalente) correspondiente a la plataforma de virtualización que se esté usando.

### Crear una sesión de PowerShell remota

Dado que Nano Server carece de funciones de inicio de sesión interactivas, toda la administración se completará desde un sistema remoto con PowerShell.

Agregue el sistema Nano Server a los hosts de confianza del sistema remoto. Sustituya la dirección IP por la dirección IP de Nano Server.

```none
Set-Item WSMan:\localhost\Client\TrustedHosts 192.168.1.50 -Force
```

Cree la sesión de PowerShell remota.

```none
Enter-PSSession -ComputerName 192.168.1.50 -Credential ~\Administrator
```

Cuando finalice estos pasos, estará en una sesión de PowerShell remota con el sistema Nano Server. A menos que se indique lo contrario, el resto de este documento tendrá lugar desde la sesión remota.


## Instalar la característica de contenedor

El proveedor de administración de paquetes de Nano Server permite instalar roles y características en Nano Server. Instale el proveedor con este comando.

```none
Install-PackageProvider NanoServerPackage
```

Una vez que haya instalado el proveedor del paquete, instale la característica de contenedor.

```none
Install-NanoServerPackage -Name Microsoft-NanoServer-Containers-Package
```

Deberá reiniciar el host de Nano Server después de instalar estas características de contenedor. 

```none
Restart-Computer
```

Tras la copia de seguridad, vuelva a establecer la conexión remota de PowerShell.

## Instalar Docker

Para trabajar con contenedores de Windows es necesario el motor de Docker. Instale el motor de Docker siguiendo estos pasos.

En primer lugar, asegúrese de que el firewall de Nano Server se haya configurado para SMB. Esto se puede completar mediante la ejecución de este comando en el host de Nano Server.

```none
Set-NetFirewallRule -Name FPS-SMB-In-TCP -Enabled True
```

Cree una carpeta en el host de Nano Server para los ejecutables de Docker.

```none
New-Item -Type Directory -Path $env:ProgramFiles'\docker\'
```

Descargue el cliente y el motor de Docker y cópielos en "C:\Archivos de programa\docker\' del host de contenedor. 

> Nano Server no admite actualmente `Invoke-WebRequest`. la descarga se debe realizar en un sistema remoto y los archivos se copian en el host de Nano Server.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile .\docker-1.12.0.zip -UseBasicParsing
```

Extraiga el paquete descargado. Una vez completada la extracción, tendrá un directorio con **dockerd.exe** y **docker.exe**. Copie ambos archivos en la carpeta **C:\Program Files\docker\** en el host del contenedor de Nano Server. 

```none
Expand-Archive .\docker-1.12.0.zip
```

Agregue el directorio de Docker a la ruta de acceso del sistema en Nano Server.

> Asegúrese de volver a la sesión de Nano Server remota.

```none
# For quick use, does not require shell to be restarted.
$env:path += “;C:\program files\docker”

# For persistent use, will apply even after a reboot.
setx PATH $env:path /M
```

Instale Docker como un servicio de Windows.

```none
dockerd --register-service
```

Inicie el servicio Docker.

```none
Start-Service Docker
```

## Instalar imágenes base del contenedor

Las imágenes del sistema operativo base se usan como base para cualquier contenedor de Hyper-V o Windows Server. Las imágenes del sistema operativo base están disponibles con Windows Server Core y Nano Server como sistema operativo subyacente y se pueden instalar con `docker pull`. Para obtener información detallada sobre las imágenes del contenedor de Windows, consulte [Administración de imágenes del contenedor](../management/manage_images.md).

Para descargar e instalar la imagen base de Nano Server, ejecute lo siguiente:

```none
docker pull microsoft/nanoserver
```

> En este momento, solo la imagen base de Nano Server es compatible con un host de contenedor de Nano Server.

## Administración de Docker en Nano Server

Para obtener la mejor experiencia posible y como procedimiento recomendado, administre Docker en Nano Server desde un sistema remoto. Para ello, debe completar los pasos siguientes.

### Preparar un host de contenedor

Cree una regla de firewall en el host de contenedor para la conexión de Docker. Use el puerto `2375` para una conexión no segura o el puerto `2376` para una conexión segura.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2375
```

Configure el motor de Docker para que acepte las conexiones entrantes a través de TCP.

En primer lugar, cree un archivo `daemon.json` en `c:\ProgramData\docker\config\daemon.json` en el host de Nano Server.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Luego, ejecute el siguiente comando para agregar una configuración de conexión al archivo `daemon.json`. De este modo se configura el motor de Docker para que acepte las conexiones entrantes a través del puerto TCP 2375. Esta conexión no es segura y no se recomienda, pero se puede usar para pruebas aisladas. Para más información sobre cómo proteger esta conexión, vea [Protect the Docker Daemon on Docker.com](https://docs.docker.com/engine/security/https/) (Proteger el demonio de Docker en Docker.com).

```none
Add-Content 'c:\programdata\docker\config\daemon.json' '{ "hosts": ["tcp://0.0.0.0:2375", "npipe://"] }'
```

Reinicie el servicio Docker.

```none
Restart-Service docker
```

### Preparar el cliente remoto

En el sistema remoto donde vaya a trabajar, descargue un cliente de Docker.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Extraiga el paquete comprimido.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Ejecute los dos comandos siguientes para agregar el directorio de Docker a la ruta de acceso del sistema.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Una vez completados estos pasos, puede acceder al host de Docker remoto con el parámetro `docker -H`.

```none
docker -H tcp://<IPADDRESS>:2375 run -it microsoft/nanoserver cmd
```

Puede crear una variable de entorno `DOCKER_HOST` que eliminará el requisito del parámetro `-H`. Para ello, puede usar el siguiente comando de PowerShell.

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

Una vez que haya establecido esta variable, el comando tendrá el aspecto siguiente.

```none
docker run -it microsoft/nanoserver cmd
```

## Host de contenedor de Hyper-V

Para implementar contenedores de Hyper-V, es necesario el rol de Hyper-V en el host de contenedor. Para más información sobre los contenedores de Hyper-V, consulte [Contenedores de Hyper-V](../management/hyperv_container.md).

Si el propio host de contenedor de Windows es una máquina virtual de Hyper-V, debe habilitarse la virtualización anidada. Para más información sobre la virtualización anidada, consulte [Virtualización anidada](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).


Instale el rol de Hyper-V en el host de contenedor Nano Server.

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

Deberá reiniciar el host de Nano Server después de instalar el rol de Hyper-V.

```none
Restart-Computer
```



<!--HONumber=Sep16_HO2-->


