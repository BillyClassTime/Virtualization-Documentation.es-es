---
title: Contenedores de Windows y Linux en Windows 10
description: Configure Windows 10 o Windows Server para contenedores y luego continúe con la ejecución de la primera imagen de contenedor.
keywords: Docker, contenedores, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909565"
---
# <a name="get-started-prep-windows-for-containers"></a>Introducción: preparar Windows para contenedores

En este tutorial se describe cómo:

- Configuración de Windows 10 o Windows Server para contenedores
- Ejecutar la primera imagen de contenedor
- Inclusión de una aplicación sencilla de .NET Core en un contenedor

## <a name="prerequisites"></a>Requisitos previos

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

Para ejecutar contenedores en Windows Server, necesita un servidor físico o una máquina virtual que ejecute Windows Server (canal semianual), Windows Server 2019 o Windows Server 2016.

Para las pruebas, puede descargar una copia de la evaluación de Windows Server [2019](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ) o una [versión preliminar de Windows Server Insider](https://insider.windows.com/for-business-getting-started-server/).

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

Para ejecutar contenedores en Windows 10, necesita lo siguiente:

- Un equipo físico que ejecuta Windows 10 Professional o Enterprise con la actualización de aniversario (versión 1607) o posterior.
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) debe estar habilitado.

> [!NOTE]
>  A partir de la actualización 2018 de octubre de Windows 10, ya no se permite que los usuarios ejecuten un contenedor de Windows en modo de aislamiento de procesos en Windows 10 Enterprise o Professional para fines de desarrollo y pruebas. Consulte las [preguntas más frecuentes](../about/faq.md) para obtener más información. 
> 
> Los contenedores de Windows Server usan el aislamiento de Hyper-V de forma predeterminada en Windows 10 con el fin de proporcionar a los desarrolladores la misma versión de kernel y la misma configuración que se usarán en producción. Obtenga más información sobre el aislamiento de Hyper-V en el área de [conceptos](../manage-containers/hyperv-container.md) de nuestros documentos.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Instalar Docker

El primer paso es instalar Docker, que es necesario para trabajar con contenedores de Windows. Docker proporciona un entorno de tiempo de ejecución estándar para contenedores, con una API común y una interfaz de la línea de comandos (CLI).

Para más información sobre la configuración, consulte [motor de Docker en Windows](../manage-docker/configure-docker-daemon.md).

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

Para instalar Docker en Windows Server, puede usar un [módulo de PowerShell de proveedor de OneGet](https://github.com/oneget/oneget) publicado por Microsoft denominado [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider). Este proveedor habilita la característica de contenedores en Windows e instala el motor y el cliente de Docker. A continuación se muestra cómo hacerlo:

1. Abra una sesión de PowerShell con privilegios elevados e instale el proveedor de Microsoft PackageManagement en el [Galería de PowerShell](https://www.powershellgallery.com/packages/DockerMsftProvider).

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   Si se le pide que instale el proveedor de NuGet, escriba `Y` para instalarlo también.

2. Use el módulo de PowerShell PackageManagement para instalar la versión más reciente de Docker.

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   Cuando PowerShell te pregunte si se debe confiar en el origen del paquete "DockerDefault", escribe `A` para continuar con la instalación.
3. Una vez finalizada la instalación, reinicie el equipo.

   ```powershell
   Restart-Computer -Force
   ```

Si desea actualizar Docker más adelante:

- Compruebe la versión instalada con `Get-Package -Name Docker -ProviderName DockerMsftProvider`
- Busque la versión actual con `Find-Package -Name Docker -ProviderName DockerMsftProvider`
- Cuando esté listo, actualice con `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, seguido de `Start-Service Docker`

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

Puede instalar Docker en Windows 10 Professional y Enterprise Edition mediante los pasos siguientes. 

1. Descargue e instale [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)y cree una cuenta de Docker gratis si aún no tiene una. Para obtener más información, consulte la [documentación de Docker](https://docs.docker.com/docker-for-windows/install).

2. Durante la instalación, establezca el tipo de contenedor predeterminado en contenedores de Windows. Para cambiar una vez finalizada la instalación, puede usar el elemento de Docker en la bandeja del sistema de Windows (como se muestra a continuación) o el siguiente comando en un símbolo del sistema de PowerShell:

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![Menú de bandeja del sistema de Docker que muestra el comando "cambiar a contenedores de Windows".](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>Pasos siguientes

Ahora que el entorno se ha configurado correctamente, siga el vínculo para obtener información sobre cómo ejecutar un contenedor.

> [!div class="nextstepaction"]
> [Ejecutar el primer contenedor](./run-your-first-container.md)
