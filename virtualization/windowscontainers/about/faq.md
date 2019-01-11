---
title: Preguntas más frecuentes sobre los contenedores de Windows
description: Preguntas más frecuentes sobre los contenedores de Windows
keywords: docker, contenedores
author: PatrickLang
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 36ef6df0b9736d88fec627e4cb56df023f1a7708
ms.sourcegitcommit: 4336d7617c30d26a987ad3450b048e17404c365d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 01/11/2019
ms.locfileid: "9001014"
---
# <a name="frequently-asked-questions"></a>Preguntas más frecuentes

## <a name="general"></a>General

### <a name="what-is-wcow-what-is-lcow"></a>¿Qué es WCOW? ¿Qué es LCOW?

WCOW es que una abreviatura de contenedores de Windows en Windows y LCOW es una abreviatura de contenedores de Linux en Windows.

### <a name="what-is-the-difference-between-linux-and-windows-server-containers"></a>¿Cuál es la diferencia entre los contenedores de Windows Server y Linux?

Los contenedores de Windows Server y Linux son similares, los dos implementan tecnologías similares dentro de su kernel y su sistema operativo base. La diferencia reside en la plataforma y las cargas de trabajo que se ejecutan dentro de los contenedores.  
Cuando un cliente utiliza contenedores de Windows Server, puede integrarlos con las tecnologías de Windows existentes, como. NET, ASP.NET, PowerShell y muchas más.

### <a name="as-a-developer-do-i-have-to-re-write-my-app-for-each-type-of-container"></a>Como desarrollador, ¿tengo que volver a escribir la aplicación para cada tipo de contenedor?

No, las imágenes de contenedores de Windows son comunes para todos los contenedores de Windows Server y de Hyper-V. La elección del tipo de contenedor se realiza cuando se inicia el contenedor. Desde la perspectiva del desarrollador, los contenedores de Windows Server y Hyper-V son dos versiones de lo mismo. Ofrecen la misma experiencia de desarrollo, programación y administración, son abiertos y extensibles e incluirán el mismo nivel de integración y compatibilidad a través de Docker.

Un desarrollador puede crear una imagen de contenedor con un contenedor de Windows Server e implementarla como un contenedor de Hyper-V o viceversa, sin ningún cambio aparte de especificar la marca de tiempo de ejecución adecuada.

Los contenedores de Windows Server ofrecerán mayor densidad y rendimiento (por ejemplo, un menor tiempo de spin up, un rendimiento en tiempo de ejecución más rápido en comparación con las configuraciones anidadas) cuando la velocidad es clave. Los contenedores de Hyper-V ofrecen mayor aislamiento, garantizando que el código que se ejecuta en un contenedor no se puede poner en peligro ni afectar negativamente al sistema operativo host u otros contenedores que se ejecutan en el mismo host. Esto es útil en los escenarios de varios inquilinos (con requisitos para el hospedaje de código no seguro), entre los que se incluyen las aplicaciones de SaaS y el hospedaje de procesos.

### <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>¿Cuáles son los requisitos previos para ejecutar los contenedores en Windows?

Los contenedores se introdujeron en la plataforma a partir de Windows Server 2016. Debes estar ejecutando Windows Server 2016 o Windows 10 Anniversary update (versión 1607) o posterior para usar contenedores.

### <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>¿Puedo ejecutar contenedores de Windows en el modo de aislamiento de proceso en Windows 10 Enterprise o Professional?

A partir de Windows, 10 de octubre de 2018 Update, que ya no impide un usuario ejecuta un contenedor de Windows con aislamiento de procesos. Debes solicitar directamente para el aislamiento de procesos mediante el uso de la `--isolation=process` marca al ejecutar los contenedores a través de `docker run`.

Si esto es algo que te interesan, debes Compruebe el host se está ejecutando Windows 10, compilación 17763 + y tiene una versión de docker con motor 18.09 o más reciente.

> [!WARNING]
> Esta característica solo está pensada para desarrollo y pruebas. Se debe seguir usando Windows Server que el host para entornos de producción.
>
> Al usar esta característica, también debes asegurarte de que coincidan con las etiquetas de versión de host y el contenedor, de lo contrario el contenedor puede no iniciarse o puede presentar un comportamiento no definido.

## <a name="windows-container-management"></a>Administración de contenedores de Windows

### <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>¿Cómo hacer Mis imágenes de contenedor disponibles en máquinas han?

Las imágenes base del contenedor de Windows contienen artefactos cuya distribución está restringida por la licencia. Al compilar en estas imágenes e insertarlos en un registro público o privado, verás que nunca se inserta el nivel de base. En su lugar, usamos el concepto de una capa externa que apunta a la capa de base real que se encuentran en el almacenamiento de nube de Azure.

Esto puede suponer un problema cuando tienes una máquina han que puede _solo_ las imágenes de extracción desde la dirección de seguridad _del_ registro de contenedor privada. Intenta seguir la capa externa para obtener la imagen base en este caso daría un error. Para invalidar el comportamiento de capa externa, puedes usar el `--allow-nondistributable-artifacts` marca en el demonio de Docker.

> [!IMPORTANT]
> Uso de esta marca no impedirá su obligación de cumplir con los términos de la licencia de imagen base del contenedor de Windows. no debe registrar el contenido de Windows para la redistribución de terceros público o 3. Se permite el uso dentro de su propio entorno.

## <a name="microsofts-open-ecosystem"></a>Ecosistema abierto de Microsoft

### <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>¿Microsoft participa en la iniciativa de contenedores abiertos (OCI, Open Container Initiative)?

Para garantizar que el formato de empaquetado permanece universal, Docker organizó recientemente la Open Container Initiative (OCI), con el objetivo de garantizar que el empaquetado de contenedores permanece como un formato abierto y guiado por una fundación, con Microsoft como uno de los miembros fundadores.

> [!TIP]
> ¿Tiene una recomendación para un complemento a las preguntas más frecuentes? Te animamos a un nuevo problema de comentarios debajo o abrir un PR contra estos documentos con las recomendaciones!
