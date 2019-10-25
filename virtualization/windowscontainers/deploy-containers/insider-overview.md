---
title: Usar contenedores con el programa Windows Insider
description: Más información sobre cómo empezar a usar contenedores de Windows con el programa Windows Insider
keywords: acoplador, contenedores, Insider, Windows
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 6080b2c5053720490d374f6fb0daa870d5ddd4e8
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/25/2019
ms.locfileid: "10257799"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Usar contenedores con el programa Windows Insider

Este ejercicio te llevará por la implementación y el uso de la función de contenedor de Windows en la última compilación para Insider de Windows Server desde el programa Windows Insider Preview. Durante este ejercicio, tendrás que instalar el rol de contenedor e implementar una edición de vista previa de las imágenes de sistema operativo base. Si necesitas familiarizarte con los contenedores, encontrarás esta información en [Acerca de los contenedores](../about/index.md).

## <a name="join-the-windows-insider-program"></a>Unirse al Programa Windows Insider

Para poder ejecutar la versión Insider de contenedores de Windows, debe tener un host que ejecute la compilación más reciente de Windows Server del programa Windows Insider o la compilación más reciente de Windows 10 desde el programa Windows Insider. Únete al [programa Windows Insider](https://insider.windows.com/GettingStarted) y revisa las condiciones de uso.

> [!IMPORTANT]
> Debe usar una compilación de Windows Server desde el programa Windows Server Insider Preview o una compilación de Windows 10 desde el programa de Windows Insider Preview para usar la imagen básica que se describe a continuación. Si no estás utilizando una de estas compilaciones, el uso de estas imágenes base dará como resultado errores al iniciar un contenedor.

## <a name="install-docker"></a>Instalar Docker

Si no tiene el acoplador ya instalado, siga los [pasos de introducción](../quick-start/set-up-environment.md) para instalar el acoplador.

## <a name="pull-an-insider-container-image"></a>Extraer una imagen del contenedor de Insider

Al formar parte del programa Windows Insider, puede usar nuestras compilaciones más recientes para las imágenes básicas.

Para recuperar la imagen base de Insider de Nano Server, ejecuta lo siguiente:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Para recuperar la imagen base de Windows Server Core Insider, ejecuta lo siguiente:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

Las imágenes base "Windows" y "IoTCore" también tienen una versión de Insider que está disponible para extracción. Puede obtener más información sobre las imágenes básicas de Insider disponibles en el documento de [imágenes base contenedor](../manage-containers/container-base-images.md) .

> [!IMPORTANT]
> Lea el [CLUF](../images-eula.md ) de la imagen de Windows Containers y las [condiciones de uso](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)del programa Windows Insider.
