---
title: Preguntas más frecuentes sobre los contenedores de Windows
description: Preguntas más frecuentes sobre los contenedores de Windows Server
keywords: docker, contenedores
author: PatrickLang
ms.date: 10/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 405b2abc43a4ae2c546de351679deb755e4a9317
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910805"
---
# <a name="frequently-asked-questions-about-containers"></a>Preguntas más frecuentes sobre los contenedores

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>¿Cuál es la diferencia entre los contenedores de Windows Server y Linux?

Tanto Linux como Windows Server implementan tecnologías similares en sus sistemas operativos kernel y Core. La diferencia reside en la plataforma y las cargas de trabajo que se ejecutan dentro de los contenedores.  

Cuando un cliente utiliza contenedores de Windows Server, se pueden integrar con las tecnologías de Windows existentes, como .NET, ASP.NET y PowerShell.

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>¿Cuáles son los requisitos previos para ejecutar contenedores en Windows?

Los contenedores se introdujeron en la plataforma con Windows Server 2016. Para usar contenedores, necesitará Windows Server 2016 o la actualización de aniversario de Windows 10 (versión 1607) o posterior. Lea los [requisitos del sistema](../deploy-containers/system-requirements.md) para obtener más información.

## <a name="what-are-wcow-and-lcow"></a>¿Qué son WCOW y LCOW?

WCOW es la abreviatura de "Windows containers on Windows". LCOW es la abreviatura de "contenedores de Linux en Windows".

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>¿Cómo se concede la licencia a los contenedores? ¿Hay un límite en el número de contenedores que puedo ejecutar?

El [CLUF](../images-eula.md) de la imagen de contenedor de Windows describe un uso que depende de que un usuario tenga un sistema operativo de host con licencia válida. El número de contenedores que un usuario puede ejecutar depende de la edición del sistema operativo host y del [modo de aislamiento](../manage-containers/hyperv-container.md) con el que se ejecuta un contenedor, así como de si estos contenedores se ejecutan para fines de desarrollo y pruebas o en producción.

|SO host                                                         |Límite de contenedor aislado de proceso                   |Hyper-V: límite de contenedores aislados                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |Sin límite                                          |2                                                  |
|Windows Server Datacenter                                       |Sin límite                                          |Sin límite                                          |
|Windows 10 Pro y Enterprise                                   |Sin límite *(solo con fines de prueba o desarrollo)*|Sin límite *(solo con fines de prueba o desarrollo)*|
|Windows 10 IoT Core y Enterprise                             |Limitada                                         |Limitada                                          |

El uso de imágenes de contenedor de Windows Server se determina mediante la lectura del número de invitados de virtualización admitidos para esa [edición](/windows-server/get-started-19/editions-comparison-19.md). <br/>

>[!NOTE]
>\*uso de producción de contenedores en una edición de IoT de Windows depende de si ha aceptado los términos de uso comerciales de Microsoft para imágenes de tiempo de ejecución de Windows 10 Core o la licencia de dispositivo de Windows 10 IoT Enterprise ("contrato comercial de IoT Windows"). Se aplican términos y restricciones adicionales en los acuerdos comerciales de IoT Windows al uso de la imagen de contenedor en un entorno de producción. Lea el CLUF de la [imagen de contenedor](../images-eula.md) para comprender exactamente lo que se permite y qué no.

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>Como desarrollador, ¿tengo que volver a escribir la aplicación para cada tipo de contenedor?

No. Las imágenes de contenedor de Windows son comunes en los contenedores de Windows Server y en el aislamiento de Hyper-V. La elección del tipo de contenedor se realiza cuando se inicia el contenedor. Desde el punto de vista del desarrollador, los contenedores de Windows Server y el aislamiento de Hyper-V son dos versiones de lo mismo. Ofrecen la misma experiencia de desarrollo, programación y administración, y son abiertas y extensibles e incluyen el mismo nivel de integración y compatibilidad con Docker.

Un desarrollador puede crear una imagen de contenedor con un contenedor de Windows Server e implementarla en el aislamiento de Hyper-V, o viceversa, sin ningún cambio que no sea especificando la marca de tiempo de ejecución adecuada.

Los contenedores de Windows Server ofrecen mayor densidad y rendimiento para cuando la velocidad es clave, como un menor tiempo de arranque y un rendimiento de tiempo de ejecución más rápido en comparación con las configuraciones anidadas. El aislamiento de Hyper-V, que es true para su nombre, ofrece un aislamiento mayor, lo que garantiza que el código que se ejecuta en un contenedor no puede poner en peligro ni afectar al sistema operativo host u otros contenedores que se ejecutan en el mismo host. Esto resulta útil para escenarios multiinquilino con requisitos para hospedar código que no es de confianza, incluidas las aplicaciones SaaS y el hospedaje de procesos.

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10"></a>¿Puedo ejecutar contenedores de Windows en modo aislado de proceso en Windows 10?

A partir de la actualización del 2018 de octubre de Windows 10, puede ejecutar un contenedor de Windows con aislamiento de procesos, pero primero debe solicitar directamente el aislamiento de procesos mediante el `--isolation=process` marca al ejecutar los contenedores con `docker run`. El aislamiento de procesos es compatible con Windows 10 Pro, Windows 10 Enterprise, Windows 10 IoT Core y Windows 10 IoT Enterprise.

Si desea ejecutar los contenedores de Windows de esta manera, debe asegurarse de que el host ejecuta Windows 10 Build 17763 + y de que tiene una versión de Docker con el motor 18,09 o posterior.

> [!WARNING]
> Aparte de los hosts de IoT Core y IoT Enterprise (después de aceptar términos y restricciones adicionales), esta característica solo está pensada para desarrollo y pruebas. Debe seguir usando Windows Server como host para las implementaciones de producción. Con esta característica, también debe asegurarse de que las etiquetas de la versión del contenedor y del host coincidan; de lo contrario, el contenedor puede no iniciarse o mostrar un comportamiento indefinido.

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>Cómo hacer que mis imágenes de contenedor estén disponibles en máquinas con separación aérea?

Las imágenes base del contenedor de Windows contienen artefactos cuya distribución está restringida por licencia. Al compilar en estas imágenes e insertarlas en un registro privado o público, observará que la capa base nunca se inserta. En su lugar, usamos el concepto de una capa externa que apunta a la capa base real que reside en el almacenamiento en la nube de Azure.

Esto puede complicar las cosas cuando tiene un equipo con separación de aire que solo puede extraer imágenes de la dirección de su registro de contenedor privado. En este caso, los intentos de seguir la capa externa para obtener la imagen base no funcionarán. Para invalidar el comportamiento de la capa externa, puede usar la marca `--allow-nondistributable-artifacts` en el demonio de Docker.

> [!IMPORTANT]
> El uso de esta marca no impedirá la obligación de cumplir los términos de la licencia de imagen base del contenedor de Windows. no debe publicar el contenido de Windows para la redistribución pública o de otro fabricante. Se permite el uso dentro de su propio entorno.

## <a name="additional-feedback"></a>Comentarios adicionales

¿Desea agregar algo a las preguntas más frecuentes? Abra un nuevo problema de comentarios en la sección de comentarios o configure una solicitud de incorporación de cambios para esta página con GitHub.
