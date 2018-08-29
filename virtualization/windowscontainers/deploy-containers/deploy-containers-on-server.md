---
title: Implementación de contenedores de Windows en Windows Server
description: Implementación de contenedores de Windows en Windows Server
keywords: docker, contenedores
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: b80dd0d231d0f9435b7cc1c5e2b35bbf5a59d793
ms.sourcegitcommit: a287211a0ed9cac7ebfe1718e3a46f0f26fc8843
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2018
ms.locfileid: "2748891"
---
# <a name="container-host-deployment---windows-server"></a>Implementación de host de contenedor - Windows Server

La implementación de un host de contenedor de Windows implica pasos distintos, según el sistema operativo y el tipo de sistema host (físico o virtual). En este documento se describe la implementación de un host de contenedor de Windows para Windows Server 2016 o Windows Server Core 2016, en un sistema físico o virtual.

## <a name="install-docker"></a>Instalar Docker

Para trabajar con contenedores de Windows es necesario Docker. Docker consta de motor y cliente. 

Para instalar Docker, usaremos el [módulo de PowerShell del proveedor OneGet](https://github.com/OneGet/MicrosoftDockerProvider). El proveedor habilitará la característica de contenedores en la máquina e instalará Docker, lo que requerirá un reinicio. 

Abre una sesión de PowerShell con privilegios elevados y ejecuta los comandos siguientes.

Instala el módulo OneGet de PowerShell.

```PowerShell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Usa OneGet para instalar la versión más reciente de Docker.

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Cuando la instalación se haya completado, reinicia el equipo.

```PowerShell
Restart-Computer -Force
```

## <a name="install-a-specific-version-of-docker"></a>Instalar una versión específica de Docker

Actualmente hay dos canales disponibles para EE Docker para Windows Server:

* `17.06` -Use esta versión si usa Docker Enterprise Edition (motor Docker, UCP, DTR). `17.06` es el valor predeterminado.
* `18.03` -Use esta versión si está ejecutando Docker EE motor por sí solo.

Para instalar una versión específica, use la `RequiredVersion` indicador:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

Instalación de versiones específicas de Docker EE puede requerir una actualización de los módulos de DockerMsftProvider instaladas anteriormente. Para actualizar:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Actualizar Docker

Si necesita actualizar Docker EE motor desde un canal anterior a un canal posterior, use ambos el `-Update` y `-RequiredVersion` indicadores:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>Instalar imágenes base del contenedor

Para trabajar con contenedores de Windows, debe instalarse una imagen base. Las imágenes base están disponibles con Windows Server Core o Nano Server como el sistema operativo del contenedor. Para obtener información detallada sobre las imágenes del contenedor de Docker, consulte [Build your own images (Crear sus propias imágenes) en docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

Para instalar la imagen base de Windows Server Core, ejecute lo siguiente:

```PowerShell
docker pull microsoft/windowsservercore
```

Para instalar la imagen base de Nano Server, ejecute lo siguiente:

```PowerShell
docker pull microsoft/nanoserver
```

> Lea el CLUF de la imagen de sistema operativo de contenedores de Windows que se encuentra aquí: [CLUF](../images-eula.md).

## <a name="hyper-v-container-host"></a>Host de contenedor de Hyper-V

Para ejecutar contenedores de Hyper-V, es necesario el rol de Hyper-V. Si el propio host de contenedor de Windows es una máquina virtual de Hyper-V, debe habilitarse la virtualización anidada antes de instalar el rol de Hyper-V. Para obtener más información sobre la virtualización anidada, vea [Virtualización anidada]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

### <a name="nested-virtualization"></a>Virtualización anidada

El script siguiente configurará la virtualización anidada para el host de contenedor. Este script se ejecuta en el equipo de Hyper-V primario. Asegúrese de que la máquina virtual del host de contenedor está desactivada cuando ejecute este script.

```PowerShell
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory -VMName $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="enable-the-hyper-v-role"></a>Habilitación del rol de Hyper-V

Para habilitar la característica de Hyper-V mediante PowerShell, ejecute el siguiente comando en una sesión de PowerShell con privilegios elevados.

```PowerShell
Install-WindowsFeature hyper-v
```
