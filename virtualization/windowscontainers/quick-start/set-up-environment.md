---
title: Contenedores de Windows y Linux en Windows 10
description: Inicio rápido de implementación de contenedores
keywords: acoplador, contenedores, LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 5f0922a1ee2588b6e5a06091fe34e07ceadf89cb
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129388"
---
# <a name="get-started-configure-your-environment-for-containers"></a>Introducción: configurar el entorno para contenedores

Este tutorial rápido muestra cómo:

> [!div class="checklist"]
> * Configurar el entorno para contenedores
> * Ejecutar la primera imagen de contenedor
> * Contenedores de una aplicación básica de .NET

## <a name="prerequisites"></a>Requisitos previos

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Asegúrate de cumplir los siguientes requisitos:

- Un sistema informático (físico o virtual) con Windows Server 2016 o posterior.

> [!NOTE]
> Si usa Windows Server 2019 Insider Preview, actualice a la [evaluación de Window Server 2019](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ).

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 Professional y Enterprise](#tab/Windows-10-Client)

Asegúrate de cumplir los siguientes requisitos:

- Un sistema informático físico que ejecute Windows 10 Professional o Enterprise con la actualización de aniversario (versión 1607) o posterior.
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) debería estar habilitado.

> [!NOTE]
>  A partir de la actualización 2018 de octubre de Windows 10, ya no se permite que los usuarios ejecuten un contenedor de Windows en modo de aislamiento de procesos en Windows 10 Enterprise o Professional para fines de desarrollo y pruebas. Consulte las [preguntas más frecuentes](../about/faq.md) para obtener más información. 
> 
> Los contenedores de Windows Server usan el aislamiento de Hyper-V de forma predeterminada en Windows 10 para proporcionar a los programadores la misma versión y configuración del núcleo que se usarán en producción. Obtenga más información sobre el aislamiento de Hyper-V en el área de [conceptos](../manage-containers/hyperv-container.md) de nuestros documentos.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Instalar Docker

Dock es la cadena de cadena definitiva para trabajar con contenedores de Windows. Docker ofrece una CLI para que los usuarios administren contenedores en un host dado, ensamblar contenedores, quitar contenedores, etc. Obtenga más información sobre el acoplador en el área de [conceptos](../manage-containers/configure-docker-daemon.md) de nuestros documentos.

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

En Windows Server, el acoplador se instala a través de un [módulo de PowerShell de proveedor de OneGet](https://github.com/oneget/oneget) publicado por Microsoft denominado [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider). Este proveedor:

- habilita la característica de contenedores en el equipo
- instala el motor de acoplamiento y el cliente en el equipo.

Para instalar el acoplador, abra una sesión de PowerShell con privilegios elevados e instale el acoplador (Microsoft PackageManagement Provider) de la [Galería de PowerShell](https://www.powershellgallery.com/packages/DockerMsftProvider).

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

A continuación, use el módulo de PowerShell de PackageManagement para instalar la última versión del Docker.

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Cuando PowerShell te pregunte si se debe confiar en el origen del paquete "DockerDefault", escribe `A` para continuar con la instalación. Una vez completada la instalación, debe reiniciar el equipo.

```powershell
Restart-Computer -Force
```

> [!TIP]
> Si desea actualizar el Dock más tarde:
>  - Comprueba la versión instalada con `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Busca la versión actual con `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Cuando estés listo, actualiza con `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, seguido de `Start-Service Docker`

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 Professional y Enterprise](#tab/Windows-10-Client)

En Windows 10 Professional y Enterprise, Dockr se instala a través de un instalador clásico. Descargue el [escritorio del acoplador](https://store.docker.com/editions/community/docker-ce-desktop-windows) y ejecute el instalador. Se le pedirá que inicie sesión. Cree una cuenta si aún no la tiene. Encontrará instrucciones de instalación más detalladas en la [documentación del Docker](https://docs.docker.com/docker-for-windows/install).

Después de la instalación, el escritorio del acoplador se usa de forma predeterminada para los contenedores Linux. Cambie a contenedores de Windows mediante la bandeja del Dock-menú o ejecutando el siguiente comando en un símbolo del sistema de PowerShell:

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>Pasos siguientes

Ahora que su entorno ha sido configurado correctamente, siga el vínculo para obtener información sobre cómo extraer y ejecutar un contenedor.

> [!div class="nextstepaction"]
> [Ejecutar el primer contenedor](./run-your-first-container.md)
