---
title: "Implementación de contenedores de Windows en Nano Server"
description: "Implementación de contenedores de Windows en Nano Server"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/17/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: 1ba6af300d0a3eba3fc6d27598044f983a4c9168
ms.openlocfilehash: f2790186aa641378b1981a1f946665ca46fdbd73

---

# Implementación de host de contenedor: Nano Server

**Esto es contenido preliminar y está sujeto a cambios.** 

Antes de iniciar la configuración del contenedor de Windows en Nano Server, necesitará un sistema que ejecute Nano Server y una conexión remota de PowerShell con este sistema. Para obtener más información sobre la implementación y la conexión con Nano Server, vea [Getting Started with Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx) (Introducción a Nano Server).

Encontrará una copia de evaluación de Nano Server [aquí](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula).

## Instalar la característica de contenedor

Instale el proveedor de administración del paquete de Nano Server.

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

## Instalar Docker

Para trabajar con contenedores de Windows es necesario Docker. Docker consta de motor y cliente. Instale el cliente y el demonio de Docker mediante estos pasos.

Cree una carpeta en el host de Nano Server para los ejecutables de Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Descargue el cliente y el demonio de Docker y cópielos en 'C:\Program Files\docker\' del host del contenedor. 

**Nota**: Nano Server no admite `Invoke-WebRequest` en estos momentos, por lo que deberá completarlo desde un sistema remoto.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Descargue el cliente de Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile .\docker.exe
```

Cuando se hayan descargado y copiado el cliente y el demonio de Docker en el host de contenedor de Nano Server, ejecute este comando en el host para instalar Docker como un servicio de Windows.

```none
& 'C:\Program Files\docker\dockerd.exe' --register-service
```

Inicie el servicio Docker.

```none
Start-Service Docker
```

## Instalar imágenes base del contenedor

Las imágenes del sistema operativo base se usan como base para cualquier contenedor de Hyper-V o Windows Server. Las imágenes del sistema operativo base están disponibles con Windows Server Core y Nano Server como sistema operativo subyacente y se pueden instalar mediante el proveedor de imágenes de contenedores. Para obtener información detallada sobre las imágenes del contenedor de Windows, consulte [Administración de imágenes del contenedor](../management/manage_images.md).

Se puede usar el siguiente comando para instalar el proveedor de imágenes de contenedores.

```none
Install-PackageProvider ContainerImage -Force
```

Para descargar e instalar la imagen base de Nano Server, ejecute lo siguiente:

```none
Install-ContainerImage -Name NanoServer
```

**Nota**: En este momento, solo la imagen base de Nano Server es compatible con un host de contenedor de Nano Server.

Reinicie el servicio Docker.

```none
Restart-Service Docker
```

Etiquete la imagen base de Nano Server como la más reciente.

```none
& 'C:\Program Files\docker\docker.exe' tag nanoserver:10.0.14300.1016 nanoserver:latest
```

## Administración de Docker en Nano Server

Para obtener la mejor experiencia posible y como procedimiento recomendado, administre Docker en Nano Server desde un sistema remoto. Para ello, debe completar los pasos siguientes.

**Prepare el demonio de Docker:**

Cree una regla de firewall en el host de contenedor para la conexión de Docker. Use el puerto `2375` para una conexión no segura o el puerto `2376` para una conexión segura.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

Configure el demonio de Docker para que acepte las conexiones entrantes a través de TCP.

En primer lugar, cree un archivo `daemon.json` en `c:\ProgramData\docker\config\daemon.json`.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Después, copie este archivo JSON en el archivo de configuración. De este modo se configura el demonio de Docker para que acepte las conexiones entrantes a través del puerto TCP 2375. Esta conexión no es segura y no se recomienda, pero se puede usar para pruebas aisladas.

```none
{
    "hosts": ["tcp://0.0.0.0:2375", "npipe://"]
}
```

En el ejemplo siguiente se configura una conexión remota segura. Los certificados TLS deben crearse y copiarse en las ubicaciones correctas. Para obtener más información, vea [Docker Daemon on Windows](./docker_windows.md) (El demonio de Docker en Windows).

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

Reinicie el servicio Docker.

```none
Restart-Service docker
```

**Prepare el cliente de Docker:**

Cree un directorio para almacenar el cliente de Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Descargue el cliente de Docker en el sistema de administración remota.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile "C:\Program Files\docker\docker.exe"
```

Agregue el directorio de Docker a la ruta de acceso del sistema.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Reinicie la sesión de PowerShell o de comando para que se reconozca la ruta de acceso modificada.

Una vez completados estos pasos, puede acceder al host de Docker remoto con el parámetro `docker -H`.

```none
docker -H tcp://10.0.0.5:2375 run -it nanoserver cmd
```

Puede crear una variable de entorno `DOCKER_HOST` que eliminará el requisito del parámetro `-H`. Para ello, puede usar el siguiente comando de PowerShell.

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2375"
```

Una vez que haya establecido esta variable, el comando tendrá el aspecto siguiente.

```none
docker run -it nanoserver cmd
```

## Host de contenedor de Hyper-V

Para implementar contenedores de Hyper-V, es necesario el rol de Hyper-V. Para más información sobre los contenedores de Hyper-V, consulte [Contenedores de Hyper-V](../management/hyperv_container.md).

Si el propio host de contenedor de Windows es una máquina virtual de Hyper-V, debe habilitarse la virtualización anidada. Para más información sobre la virtualización anidada, consulte [Virtualización anidada](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).


Instale el rol de Hyper-V:

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

Deberá reiniciar el host de Nano Server después de instalar el rol de Hyper-V.

```none
Restart-Computer
```






<!--HONumber=Jun16_HO4-->


