---
title: "Implementación de contenedores de Windows en Windows Server"
description: "Implementación de contenedores de Windows en Windows Server"
keywords: docker, contenedores
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
translationtype: Human Translation
ms.sourcegitcommit: f721639b1b10ad97cc469df413d457dbf8d13bbe
ms.openlocfilehash: 4d7e8fb1fcbb7e9680b7d5bd143ef6d59e45035e

---

# Implementación de host de contenedor - Windows Server

La implementación de un host de contenedor de Windows implica pasos distintos, según el sistema operativo y el tipo de sistema host (físico o virtual). En este documento se describe la implementación de un host de contenedor de Windows para Windows Server 2016 o Windows Server Core 2016, en un sistema físico o virtual.

## Instalar la característica de contenedor

La característica de contenedor debe habilitarse antes de trabajar con contenedores de Windows. Para ello, ejecute el comando siguiente en una sesión de PowerShell con privilegios elevados.

```none
Install-WindowsFeature containers
```

Cuando la instalación de la característica haya finalizado, reinicie el equipo.

```none
Restart-Computer -Force
```

## Instalar Docker

Para trabajar con contenedores de Windows es necesario Docker. Docker consta de motor y cliente. En este ejercicio se instalarán ambos.

Descargue el motor de Docker y el cliente como un archivo zip.

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Expanda el archivo zip en Archivos de programa. El contenido del archivo ya está en el directorio de Docker.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Ejecute los dos comandos siguientes para agregar el directorio de Docker a la ruta de acceso del sistema.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Para instalar Docker como un servicio de Windows, ejecute lo siguiente.

```none
dockerd --register-service
```

Una vez instalado, puede iniciar el servicio.

```none
Start-Service Docker
```

## Instalar imágenes base del contenedor

Para trabajar con contenedores de Windows, debe instalarse una imagen base. Las imágenes base están disponibles con Windows Server Core o Nano Server como el sistema operativo subyacente. Para obtener información detallada sobre las imágenes del contenedor de Docker, consulte [Build your own images (Crear sus propias imágenes) en docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

Para instalar la imagen base de Windows Server Core, ejecute lo siguiente:

```none
docker pull microsoft/windowsservercore
```

Para instalar la imagen base de Nano Server, ejecute lo siguiente:

```none
docker pull microsoft/nanoserver
```

> Lea el CLUF de la imagen de sistema operativo de contenedores de Windows que se encuentra aquí: [CLUF](../Images_EULA.md).

## Host de contenedor de Hyper-V

Para ejecutar contenedores de Hyper-V, es necesario el rol de Hyper-V. Si el propio host de contenedor de Windows es una máquina virtual de Hyper-V, debe habilitarse la virtualización anidada antes de instalar el rol de Hyper-V. Para obtener más información sobre la virtualización anidada, vea [Virtualización anidada]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

### Virtualización anidada

El script siguiente configurará la virtualización anidada para el host de contenedor. Este script se ejecuta en el equipo de Hyper-V primario. Asegúrese de que la máquina virtual del host de contenedor está desactivada cuando ejecute este script.

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

Para habilitar la característica de Hyper-V mediante PowerShell, ejecute el siguiente comando en una sesión de PowerShell con privilegios elevados.

```none
Install-WindowsFeature hyper-v
```



<!--HONumber=Sep16_HO4-->


