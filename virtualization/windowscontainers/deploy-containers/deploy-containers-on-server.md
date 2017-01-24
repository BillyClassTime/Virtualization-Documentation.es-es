---
title: "Implementación de contenedores de Windows en Windows Server"
description: "Implementación de contenedores de Windows en Windows Server"
keywords: docker, contenedores
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
translationtype: Human Translation
ms.sourcegitcommit: 54eff4bb74ac9f4dc870d6046654bf918eac9bb5
ms.openlocfilehash: 6e113b5ddc74b5a4e6ee23b06ef635a7ba0d4693

---

# Implementación de host de contenedor - Windows Server

La implementación de un host de contenedor de Windows implica pasos distintos, según el sistema operativo y el tipo de sistema host (físico o virtual). En este documento se describe la implementación de un host de contenedor de Windows para Windows Server 2016 o Windows Server Core 2016, en un sistema físico o virtual.

## Instalar Docker

Para trabajar con contenedores de Windows es necesario Docker. Docker consta de motor y cliente. 

Para instalar Docker, usaremos el [módulo de PowerShell del proveedor OneGet](https://github.com/oneget/oneget). El proveedor habilitará la característica de contenedores en la máquina e instalará Docker, lo que requerirá un reinicio. 

Abra una sesión de PowerShell con privilegios elevados y ejecute los comandos siguientes.

Primero instalaremos el módulo de PowerShell de OneGet.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Después, usaremos OneGet para instalar la versión más reciente de Docker.

```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Cuando finalice la instalación, reinicie el equipo.

```none
Restart-Computer -Force
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

> Lea el CLUF de la imagen de sistema operativo de contenedores de Windows que se encuentra aquí: [CLUF](../images-eula.md).

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



<!--HONumber=Jan17_HO4-->


