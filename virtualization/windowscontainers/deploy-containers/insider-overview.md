---
title: Usar contenedores con el programa Windows Insider
description: Más información sobre cómo empezar a usar contenedores de Windows con el programa Windows Insider
keywords: acoplador, contenedores, Insider, Windows
author: cwilhit
ms.openlocfilehash: 137209a66c3d0b907003498fe78a04a57a140130
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254421"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Usar contenedores con el programa Windows Insider

Este ejercicio te llevará por la implementación y el uso de la función de contenedor de Windows en la última compilación para Insider de Windows Server desde el programa Windows Insider Preview. Durante este ejercicio, tendrás que instalar el rol de contenedor e implementar una edición de vista previa de las imágenes de sistema operativo base. Si necesitas familiarizarte con los contenedores, encontrarás esta información en [Acerca de los contenedores](../about/index.md).

> [!NOTE]
> Este contenido es específico de los contenedores de Windows Server en el programa de vista previa de Windows Server Insider. Si estás buscando instrucciones de Insider para usar contenedores de Windows, [consulta la guía de introducción.](../quick-start/set-up-environment.md)

## <a name="join-the-windows-insider-program"></a>Unirse al Programa Windows Insider

Para poder ejecutar la versión Insider de contenedores de Windows, debe tener un host que ejecute la compilación más reciente de Windows Server del programa Windows Insider o la compilación más reciente de Windows 10 desde el programa Windows Insider. Únete al [programa Windows Insider](https://insider.windows.com/GettingStarted) y revisa las condiciones de uso.

> [!IMPORTANT]
> Debe usar una compilación de Windows Server desde el programa Windows Server Insider Preview o una compilación de Windows 10 desde el programa de Windows Insider Preview para usar la imagen básica que se describe a continuación. Si no estás utilizando una de estas compilaciones, el uso de estas imágenes base dará como resultado errores al iniciar un contenedor.

## <a name="install-docker"></a>Instalar Docker

<!-- start tab view -->
# [<a name="windows-server-insider"></a>Windows Server Insider](#tab/Windows-Server-Insider)

Para instalar DockerEE, usaremos el módulo de PowerShell del proveedor OneGet. El proveedor habilitará la característica de contenedores en la máquina e instalará DockerEE, lo que requerirá un reinicio. Abre una sesión de PowerShell con privilegios elevados y ejecuta los comandos siguientes.

> [!NOTE]
> Instalar el acoplador EE con compilaciones de Insider de Windows Server requiere un proveedor de OneGet diferente al usado para las compilaciones que no son de Insider. Si el proveedor de DockerMsftProviderOneGet y DockerEE ya están instalados, quítalos antes de continuar.

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

Instala el módulo OneGetPowerShell para usarlo con compilaciones de WindowsInsider.

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

Usa OneGet para instalar la última versión de DockerEEPreview.

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

Cuando la instalación se haya completado, reinicia el equipo.

```powershell
Restart-Computer -Force
```

# [<a name="windows-10-insider"></a>Windows 10 Insider](#tab/Windows-10-Insider)

En Windows 10 Insider, el borde del acoplador se instala a través del mismo instalador que el escritorio estable del acoplador. Descargue el [escritorio del acoplador](https://store.docker.com/editions/community/docker-ce-desktop-windows) y ejecute el instalador. Se le pedirá que inicie sesión. Cree una cuenta si aún no la tiene. Encontrará instrucciones de instalación más detalladas en la [documentación del Docker](https://docs.docker.com/docker-for-windows/install).

Después de la instalación, abra la configuración del Docker y cambie al canal "Edge".

![](./media/docker-edge-instruction.png)

---
<!-- stop tab view -->

## <a name="pull-an-insider-container-image"></a>Extraer una imagen del contenedor de Insider

Antes de trabajar con los contenedores de Windows, debe instalarse una imagen base. Al formar parte del programa Windows Insider, puede usar nuestras compilaciones más recientes para las imágenes básicas. Puede obtener más información sobre las imágenes base disponibles en el documento de [imágenes base de contenedor](../manage-containers/container-base-images.md) .

Para recuperar la imagen base de Insider de Nano Server, ejecuta lo siguiente:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Para recuperar la imagen base de Windows Server Core Insider, ejecuta lo siguiente:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> Lea el [CLUF](../images-eula.md ) de la imagen de Windows Containers y las [condiciones de uso](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)del programa Windows Insider.
