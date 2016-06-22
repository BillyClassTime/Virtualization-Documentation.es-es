---
title: Implementación de contenedores de Windows en Windows Server
description: Implementación de contenedores de Windows en Windows Server
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
---

# Implementación de host de contenedor - Windows Server

**Esto es contenido preliminar y está sujeto a cambios.** 

La implementación de un host de contenedor de Windows implica pasos distintos, según el sistema operativo y el tipo de sistema host (físico o virtual). En este documento se describe la implementación de un host de contenedor de Windows para Windows Server 2016 o Windows Server Core 2016, en un sistema físico o virtual.

## Instalar la característica de contenedor

La característica de contenedor debe habilitarse antes de trabajar con contenedores de Windows. Para ello, ejecute el comando siguiente en una sesión de PowerShell con privilegios elevados. 

```none
Install-WindowsFeature containers
```

Cuando la instalación de la característica haya finalizado, reinicie el equipo.

## Instalar Docker

Para trabajar con contenedores de Windows es necesario Docker. Docker consta de motor y cliente. En este ejercicio se instalarán ambos.

Cree una carpeta para los ejecutables de Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Descargue el demonio de Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Descargue el cliente de Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

Agregue el directorio de Docker a la ruta de acceso del sistema.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Reinicie la sesión de PowerShell para que reconozca la ruta de acceso modificada.

Para instalar Docker como un servicio de Windows, ejecute lo siguiente.

```none
dockerd --register-service
```

Una vez instalado, puede iniciar el servicio.

```none
Start-Service Docker
```

## Instalar imágenes base del contenedor

Para poder implementar un contenedor, es necesario descargar una imagen base del sistema operativo del contenedor. En el ejemplo siguiente, se descargará la imagen base del sistema operativo de Windows Server Core. Puede completar este mismo procedimiento para instalar la imagen base de Nano Server. Puede completar este mismo procedimiento para instalar la imagen base de Nano Server. Para obtener información detallada sobre las imágenes del contenedor de Windows, consulte [Administración de imágenes del contenedor](../management/manage_images.md).
    
En primer lugar, instale el proveedor de paquetes de imágenes del contenedor.

```none
Install-PackageProvider ContainerImage -Force
```

Después, instale la imagen de Windows Server Core. Este proceso puede tardar algún tiempo, por lo que puede dedicarse a otros asuntos y retomarlo cuando se haya completado la descarga.

```none 
Install-ContainerImage -Name WindowsServerCore    
```

Cuando se haya instalado la imagen base, debe reiniciar el servicio Docker.

```none
Restart-Service docker
```

Por último, debe etiquetar la imagen con una versión "latest". Para ello, ejecute el comando siguiente.

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

## Host de contenedor de Hyper-V

Para implementar contenedores de Hyper-V, es necesario el rol de Hyper-V. Si el propio host de contenedor de Windows es una máquina virtual de Hyper-V, debe habilitarse la virtualización anidada antes de instalar el rol de Hyper-V. Para obtener más información sobre la virtualización anidada, vea [Virtualización anidada]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

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

Para habilitar la característica de Hyper-V mediante PowerShell, ejecute el siguiente comando en una sesión de PowerShell con privilegios elevados.

```none
Install-WindowsFeature hyper-v
```



<!--HONumber=May16_HO4-->


