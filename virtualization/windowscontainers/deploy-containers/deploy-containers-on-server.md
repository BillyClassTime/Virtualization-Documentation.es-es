---
title: Implementación de contenedores de Windows en Windows Server
description: Implementación de contenedores de Windows en Windows Server
keywords: docker, contenedores
author: taylorb-microsoft
ms.date: 09/09/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 6e3996af36b4a710f9a12b3a1371138b053a43d8
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909905"
---
# <a name="container-host-deployment-windows-server"></a>Implementación de host de contenedor: Windows Server

La implementación de un host de contenedor de Windows implica pasos distintos, según el sistema operativo y el tipo de sistema host (físico o virtual). En este documento se describe la implementación de un host de contenedor de Windows para Windows Server 2016 o Windows Server Core 2016, en un sistema físico o virtual.

## <a name="install-docker"></a>Instalar Docker

Para trabajar con contenedores de Windows es necesario Docker. Docker está formado por el motor de Docker y el cliente de Docker.

Para instalar Docker, usaremos el [módulo de PowerShell del proveedor OneGet](https://github.com/OneGet/MicrosoftDockerProvider). El proveedor habilitará la característica de contenedores en el equipo e instalará Docker, que requerirá un reinicio.

Abra una sesión de PowerShell con privilegios elevados y ejecute los siguientes cmdlets.

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

## <a name="install-a-specific-version-of-docker"></a>Instalación de una versión específica de Docker

Actualmente hay dos canales disponibles para Docker EE para Windows Server:

* `17.06`: Use esta versión si usa Docker Enterprise Edition (motor de Docker, UCP, DTR). `17.06` es el valor predeterminado.
* `18.03`: Use esta versión si solo está ejecutando el motor de Docker EE.

Para instalar una versión concreta, use la marca `RequiredVersion`:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

La instalación de versiones específicas de Docker EE puede requerir una actualización de los módulos de DockerMsftProvider instalados anteriormente. Para actualizar:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Actualización de Docker

Si necesita actualizar el motor de Docker EE desde un canal anterior a un canal posterior, use las marcas `-Update` y `-RequiredVersion`:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>Instalación de imágenes de contenedor base

Antes de trabajar con los contenedores de Windows, debe instalarse una imagen base. Las imágenes base están disponibles con Windows Server Core o Nano Server como el sistema operativo del contenedor. Para obtener información detallada sobre las imágenes del contenedor de Docker, consulte [Build your own images (Crear sus propias imágenes) en docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

Con el lanzamiento de Windows Server 2019, las imágenes de contenedor de origen de Microsoft se mueven a un nuevo registro llamado Microsoft Container Registry. Las imágenes de contenedor publicadas por Microsoft deben seguir siendo detectadas a través de Docker Hub. En el caso de las nuevas imágenes de contenedor publicadas con Windows Server 2019 y versiones posteriores, debe buscar para extraerlas desde MCR. En el caso de las imágenes de contenedor anteriores publicadas antes de Windows Server 2019, debe continuar con la extracción del registro de Docker.

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 y versiones más recientes

Para instalar la imagen base ' Windows Server Core ', ejecute lo siguiente:

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

Para instalar la imagen base de nano Server, ejecute lo siguiente:

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (versiones 1607-1803)

Para instalar la imagen base de Windows Server Core, ejecute lo siguiente:

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:1607
```

Para instalar la imagen base de Nano Server, ejecute lo siguiente:

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1803
```

> Lea el CLUF de la imagen de SO de contenedores de Windows, que se puede encontrar aquí: [EULA](../images-eula.md).

## <a name="hyper-v-isolation-host"></a>Host de aislamiento de Hyper-V

Debe tener el rol de Hyper-V para ejecutar el aislamiento de Hyper-V. Si el propio host de contenedor de Windows es una máquina virtual de Hyper-V, debe habilitarse la virtualización anidada antes de instalar el rol de Hyper-V. Para obtener más información sobre la virtualización anidada, vea [Virtualización anidada](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization).

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

Para habilitar la característica de Hyper-V con PowerShell, ejecute el siguiente cmdlet en una sesión de PowerShell con privilegios elevados.

```PowerShell
Install-WindowsFeature hyper-v
```
