---
title: Contenedores de Windows y Linux en Windows 10
description: Configure Windows 10 o Windows Server para contenedores y, a continuación, vaya a la ejecución de su primera imagen de contenedor.
keywords: acoplador, contenedores, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288123"
---
# <a name="get-started-prep-windows-for-containers"></a>Introducción: preparar Windows para contenedores

En este tutorial se describe cómo:

- Configurar Windows 10 o Windows Server para contenedores
- Ejecutar la primera imagen de contenedor
- Contenedores de una aplicación básica de .NET

## <a name="prerequisites"></a>Requisitos previos

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Para ejecutar contenedores en Windows Server, necesita un servidor físico o una máquina virtual que ejecute Windows Server (canal semi-anual), Windows Server 2019 o Windows Server 2016.

Para las pruebas, puede descargar una copia de la [evaluación de Window Server 2019](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ) o una [versión preliminar de Windows Server Insider](https://insider.windows.com/for-business-getting-started-server/).

# [<a name="windows-10"></a>Windows 10](#tab/Windows-10-Client)

Para ejecutar contenedores en Windows 10, necesita lo siguiente:

- Un sistema informático físico que ejecute Windows 10 Professional o Enterprise con la actualización de aniversario (versión 1607) o posterior.
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) debería estar habilitado.

> [!NOTE]
>  A partir de la actualización 2018 de octubre de Windows 10, ya no se permite que los usuarios ejecuten un contenedor de Windows en modo de aislamiento de procesos en Windows 10 Enterprise o Professional para fines de desarrollo y pruebas. Consulte las [preguntas más frecuentes](../about/faq.md) para obtener más información. 
> 
> Los contenedores de Windows Server usan el aislamiento de Hyper-V de forma predeterminada en Windows 10 para proporcionar a los programadores la misma versión y configuración del núcleo que se usarán en producción. Obtenga más información sobre el aislamiento de Hyper-V en el área de [conceptos](../manage-containers/hyperv-container.md) de nuestros documentos.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Instalar Docker

El primer paso es instalar el acoplador, que es necesario para trabajar con contenedores de Windows. Dock ofrece un entorno de tiempo de ejecución estándar para contenedores, con una API común y una interfaz de línea de comandos (CLI).

Para obtener más detalles de configuración, vea [motor de acoplamiento en Windows](../manage-docker/configure-docker-daemon.md).

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Para instalar el acoplador en Windows Server, puede usar un [módulo de PowerShell de proveedor de OneGet](https://github.com/oneget/oneget) publicado por Microsoft denominado [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider). Este proveedor habilita la característica de contenedores en Windows e instala el motor de acoplamiento y el cliente. Se hace así:

1. Abra una sesión de PowerShell con privilegios elevados e instale el acoplador-proveedor de Microsoft PackageManagement desde la [Galería de PowerShell](https://www.powershellgallery.com/packages/DockerMsftProvider).

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   Si se le pide que instale el proveedor de NuGet, escríbalo `Y` también.

2. Use el módulo de PowerShell de PackageManagement para instalar la última versión del Docker.

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   Cuando PowerShell te pregunte si se debe confiar en el origen del paquete "DockerDefault", escribe `A` para continuar con la instalación.
3. Una vez completada la instalación, reinicie el equipo.

   ```powershell
   Restart-Computer -Force
   ```

Si desea actualizar el Dock más tarde:

- Comprueba la versión instalada con `Get-Package -Name Docker -ProviderName DockerMsftProvider`
- Busca la versión actual con `Find-Package -Name Docker -ProviderName DockerMsftProvider`
- Cuando estés listo, actualiza con `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, seguido de `Start-Service Docker`

# [<a name="windows-10"></a>Windows 10](#tab/Windows-10-Client)

Puede instalar el acoplador en Windows 10 Professional y en las ediciones Enterprise siguiendo los pasos siguientes. 

1. Descargue e instale [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)y cree una cuenta de Dock libres si aún no tiene una. Para obtener más información, consulte la [documentación del acoplador](https://docs.docker.com/docker-for-windows/install).

2. Durante la instalación, establezca el tipo de contenedor predeterminado en contenedores de Windows. Para cambiar una vez finalizada la instalación, puede usar el elemento del acoplador de la bandeja del sistema de Windows (como se muestra a continuación) o el siguiente comando en un símbolo del sistema de PowerShell:

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![Menú de la bandeja del sistema del acoplador que muestra el comando "cambiar a contenedores de Windows".](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>Pasos siguientes

Ahora que su entorno ha sido configurado correctamente, siga el vínculo para obtener información sobre cómo ejecutar un contenedor.

> [!div class="nextstepaction"]
> [Ejecutar el primer contenedor](./run-your-first-container.md)
