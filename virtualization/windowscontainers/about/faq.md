---
title: Preguntas más frecuentes sobre los contenedores de Windows
description: Preguntas más frecuentes sobre contenedores de Windows Server
keywords: docker, contenedores
author: PatrickLang
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 90894278885fde54feab222bb2bf44ca3eba331b
ms.sourcegitcommit: daf1d2b5879c382404fc4d59f1c35c88650e20f7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/23/2019
ms.locfileid: "9674740"
---
# <a name="frequently-asked-questions-about-containers"></a>Preguntas más frecuentes sobre los contenedores

## <a name="what-are-wcow-and-lcow"></a>¿Qué son WCOW y LCOW?

WCOW es la abreviatura de "contenedores de Windows en Windows". LCOW es la abreviatura de "contenedores de Linux en Windows".

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>¿Cuál es la diferencia entre los contenedores de Linux y de Windows Server?

Tanto Linux como Windows Server implementan tecnologías similares dentro de sus sistemas operativos básicos y de núcleo. La diferencia reside en la plataforma y las cargas de trabajo que se ejecutan dentro de los contenedores.  

Cuando un cliente usa contenedores de Windows Server, puede integrarse con tecnologías existentes de Windows, como .NET, ASP.NET y PowerShell.

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>Como desarrollador, ¿tengo que volver a escribir mi aplicación para cada tipo de contenedor?

No. Las imágenes de contenedores de Windows son comunes en los contenedores de Windows Server y en el aislamiento de Hyper-V. La elección del tipo de contenedor se realiza cuando se inicia el contenedor. Desde el punto de vista del desarrollador, los contenedores de Windows Server y el aislamiento de Hyper-V son dos variantes de la misma acción. Ofrecen la misma experiencia de desarrollo, programación y administración, y están abiertos y son extensibles e incluyen el mismo nivel de integración y soporte con Docker.

Un desarrollador puede crear una imagen de contenedor con un contenedor de Windows Server e implementarla en el aislamiento de Hyper-V o viceversa sin ningún cambio que no sea especificar el indicador de tiempo de ejecución adecuado.

Los contenedores de Windows Server ofrecen mayor densidad y rendimiento para cuando la velocidad es fundamental, por ejemplo, un tiempo de giro más bajo y un rendimiento de tiempo de ejecución más rápido en comparación con las configuraciones anidadas. El aislamiento de Hyper-V, verdadero a su nombre, ofrece un mayor aislamiento, lo que garantiza que el código que se ejecuta en un contenedor no comprometa o afecte el sistema operativo del host u otros contenedores que se ejecuten en el mismo host. Esto es útil para escenarios de multiinquilino con requisitos para hospedar código que no es de confianza, incluidas las aplicaciones SaaS y el hospedaje de cálculos.

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>¿Cuáles son los requisitos previos para ejecutar contenedores en Windows?

Se introdujeron contenedores en la plataforma con Windows Server 2016. Para usar contenedores, necesitarás Windows Server 2016 o la actualización de aniversario de Windows 10 (versión 1607) o posterior.

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>¿Puedo ejecutar contenedores de Windows en modo aislado de proceso en Windows 10 Enterprise o Professional?

A partir de la actualización de Windows 10 de octubre de 2018, puede ejecutar un contenedor de Windows con aislamiento de procesos, pero primero debe solicitar directamente el `--isolation=process` aislamiento de proceso usando la marca `docker run`al ejecutar los contenedores con.

Si desea ejecutar los contenedores de Windows de esta manera, tendrá que asegurarse de que su host ejecuta Windows 10 Build 17763 + y de tener una versión de Docker con Engine 18,09 o posterior.

> [!WARNING]
> Esta característica solo está destinada a desarrollos y pruebas. Debe continuar usando Windows Server como host para las implementaciones de producción. Al usar esta característica, también debe asegurarse de que las etiquetas de versión de contenedor y host coincidan; de lo contrario, es posible que el contenedor no se inicie o que presente un comportamiento no definido.

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>¿Cómo hago para que mis imágenes de contenedor estén disponibles en máquinas con aire libre?

Las imágenes base del contenedor de Windows contienen artefactos cuya distribución está restringida por licencia. Cuando se compila en estas imágenes y se insertan en un registro privado o público, observará que la capa base nunca se inserta. En su lugar, usamos el concepto de una capa externa que apunta a la capa base real que reside en el almacenamiento en la nube de Azure.

Esto puede complicar las cosas cuando tienes un equipo con separación por aire que solo puede extraer imágenes de la dirección de tu registro de contenedor privado. En este caso, los intentos de seguir la capa externa para obtener la imagen base no funcionarán. Para invalidar el comportamiento de la capa externa, puede `--allow-nondistributable-artifacts` usar la marca en el daemon del acoplador.

> [!IMPORTANT]
> El uso de este indicador no impedirá que su obligación cumpla con las condiciones de la licencia básica de la contenedora de Windows; no debe publicar contenido de Windows para una redistribución pública o de terceros. Se permite el uso de su propio entorno.

## <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>¿Microsoft participa en la iniciativa de contenedores abiertos (OCI, Open Container Initiative)?

Para garantizar que el formato del embalaje sea universal, el acoplador organizó recientemente la iniciativa Open Container Initiative (OCI), que tiene el objetivo de garantizar que el embalaje del contenedor siga siendo abierto y en formato de cimentación, con Microsoft como uno de sus miembros fundadores.

## <a name="additional-feedback"></a>Comentarios adicionales

¿Quiere agregar algo a las preguntas más frecuentes? Abra un nuevo problema de comentarios en la sección Comentarios o configure una solicitud de extracción para esta página con GitHub.
