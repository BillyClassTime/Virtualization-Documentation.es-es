---
title: Implementación de contenedores de Windows en Nano Server
description: Implementación de contenedores de Windows en Nano Server
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
---

# Implementación de host de contenedor: Nano Server

**Esto es contenido preliminar y está sujeto a cambios.** 

Antes de iniciar la configuración del contenedor de Windows en Nano Server, necesitará un sistema que ejecute Nano Server y una conexión remota de PowerShell con este sistema.

Para obtener más información sobre la implementación y la conexión con Nano Server, vea [Getting Started with Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx) (Introducción a Nano Server).

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

Deberá reiniciar el host de Nano Server después de instalar estas características.

## Instalar Docker

Para trabajar con contenedores de Windows es necesario Docker. Docker consta de motor y cliente. Instale el demonio de Docker siguiendo estos pasos.

Descargue el demonio de Docker y cópielo en `$env:SystemRoot\system32\` en el host de contenedor. Nano Server no admite actualmente `Invoke-Webrequest`, por lo que deberá completarlo desde un sistema remoto.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Instale Docker como un servicio de Windows.

```none
dockerd.exe --register-service
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

Por último, debe etiquetar la imagen con una versión "latest". Para ello, ejecute el comando siguiente.

```none
docker tag nanoserver:10.0.14300.1010 nanoserver:latest
```

## Host de contenedor de Hyper-V

Para implementar contenedores de Hyper-V, es necesario el rol de Hyper-V. Si el propio host de contenedor de Windows es una máquina virtual de Hyper-V, debe habilitarse la virtualización anidada antes de instalar el rol de Hyper-V. Para obtener más información sobre la virtualización anidada, vea Virtualización anidada.

### Virtualización anidada

El script siguiente configurará la virtualización anidada para el host de contenedor. Este script se ejecuta en la máquina de Hyper-V que hospeda la máquina virtual del host de contenedor. Asegúrese de que la máquina virtual del host de contenedor está desactivada cuando ejecute este script.

```none
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### Habilitación del rol de Hyper-V

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

## Administración de Docker en Nano Server

**Prepare el demonio de Docker:**

Para obtener la mejor experiencia posible, administre Docker en Nano Server desde un sistema remoto. Para ello, debe completar los pasos siguientes.

Cree una regla de firewall en el host de contenedor para la conexión de Docker. Use el puerto `2375` para una conexión no segura o el puerto `2376` para una conexión segura.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

Configure el demonio de Docker para que acepte las conexiones entrantes a través de TCP.

En primer lugar, cree un archivo `daemon.json` en `c:\ProgramData\docker\config\daemon.json`.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Después, copie este archivo JSON en el archivo. De este modo se configura el demonio de Docker para que acepte las conexiones entrantes a través del puerto TCP 2375. Esta conexión no es segura y no se recomienda, pero se puede usar para pruebas aisladas.

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

Descargue el cliente de Docker en el sistema de administración remota.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:SystemRoot\system32\docker.exe
```

Una vez completados los pasos, puede obtener acceso al demonio de Docker con el parámetro `Docker -H`.

```none
docker -H tcp://10.0.0.5:2376 run -it nanoserver cmd
```

Puede crear una variable de entorno `DOCKER_HOST` que eliminará el requisito del parámetro `-H`. Para ello, puede usar el siguiente comando de PowerShell.

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2376"
```

Una vez que haya establecido esta variable, el comando tendrá el aspecto siguiente.

```none
docker run -it nanoserver cmd
```

<!--HONumber=May16_HO5-->


