---
title: Preguntas más frecuentes sobre los contenedores de Windows
description: Preguntas más frecuentes sobre contenedores de Windows Server
keywords: docker, contenedores
author: PatrickLang
ms.date: 10/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: aeb2b5dd0d9df95ee417b3a160d10d4991304689
ms.sourcegitcommit: 4b37076f988608b6bf1270497c24325993ef41d3
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/29/2019
ms.locfileid: "10264366"
---
# <a name="frequently-asked-questions-about-containers"></a>Preguntas más frecuentes sobre los contenedores

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>¿Cuál es la diferencia entre los contenedores de Linux y de Windows Server?

Tanto Linux como Windows Server implementan tecnologías similares dentro de sus sistemas operativos básicos y de núcleo. La diferencia reside en la plataforma y las cargas de trabajo que se ejecutan dentro de los contenedores.  

Cuando un cliente usa contenedores de Windows Server, puede integrarse con tecnologías existentes de Windows, como .NET, ASP.NET y PowerShell.

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>¿Cuáles son los requisitos previos para ejecutar contenedores en Windows?

Se introdujeron contenedores en la plataforma con Windows Server 2016. Para usar contenedores, necesitarás Windows Server 2016 o la actualización de aniversario de Windows 10 (versión 1607) o posterior. Lea los [requisitos del sistema](../deploy-containers/system-requirements.md) para obtener más información.

## <a name="what-are-wcow-and-lcow"></a>¿Qué son WCOW y LCOW?

WCOW es la abreviatura de "contenedores de Windows en Windows". LCOW es la abreviatura de "contenedores de Linux en Windows".

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>¿Cómo se concede la licencia a los contenedores? ¿Hay algún límite en la cantidad de contenedores que puedo ejecutar?

El [CLUF](../images-eula.md) de la imagen del contenedor de Windows describe un uso que depende de que un usuario tenga un sistema operativo host con licencia válida. El número de contenedores que un usuario puede ejecutar depende de la edición del sistema operativo y del [modo de aislamiento](../manage-containers/hyperv-container.md) con el que se ejecute un contenedor, así como de si estos contenedores se ejecutan con fines de desarrollo o para pruebas o en producción.

|Sistema operativo del host                                                         |Límite de contenedor aislado de proceso                   |Hyper-V: límite de contenedores aislados                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |Sin límite                                          |1                                                  |
|Windows Server Datacenter                                       |Sin límite                                          |Sin límite                                          |
|Windows 10 Pro y Enterprise                                   |Sin límites *(solo para fines de prueba o de desarrollo)*|Sin límites *(solo para fines de prueba o de desarrollo)*|
|Windows 10 IoT Core y Enterprise                             |Máxima                                         |Máxima                                          |

El uso de la imagen del contenedor de Windows Server se determina mediante la lectura del número de invitados de virtualización que se admiten en esa [edición](/windows-server/get-started-19/editions-comparison-19.md). <br/>

>[!NOTE]
>\ * El uso de la producción de contenedores en una edición de IoT de Windows depende de si ha aceptado las condiciones de uso comercial de Microsoft para las imágenes de tiempo de ejecución de Windows 10 Core o la licencia de dispositivo de Windows 10 IoT Enterprise ("contrato comercial de Windows IoT"). Las cláusulas y restricciones adicionales de los contratos de Windows IoT Commercial rigen para el uso de la imagen del contenedor en un entorno de producción. Lea el CLUF de la [imagen del contenedor](../images-eula.md) para comprender exactamente lo que está permitido y lo que no.

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>Como desarrollador, ¿tengo que volver a escribir mi aplicación para cada tipo de contenedor?

No. Las imágenes de contenedores de Windows son comunes en los contenedores de Windows Server y en el aislamiento de Hyper-V. La elección del tipo de contenedor se realiza cuando se inicia el contenedor. Desde el punto de vista del desarrollador, los contenedores de Windows Server y el aislamiento de Hyper-V son dos variantes de la misma acción. Ofrecen la misma experiencia de desarrollo, programación y administración, y están abiertos y son extensibles e incluyen el mismo nivel de integración y soporte con Docker.

Un desarrollador puede crear una imagen de contenedor con un contenedor de Windows Server e implementarla en el aislamiento de Hyper-V o viceversa sin ningún cambio que no sea especificar el indicador de tiempo de ejecución adecuado.

Los contenedores de Windows Server ofrecen mayor densidad y rendimiento para cuando la velocidad es fundamental, por ejemplo, un tiempo de giro más bajo y un rendimiento de tiempo de ejecución más rápido en comparación con las configuraciones anidadas. El aislamiento de Hyper-V, verdadero a su nombre, ofrece un mayor aislamiento, lo que garantiza que el código que se ejecuta en un contenedor no comprometa o afecte el sistema operativo del host u otros contenedores que se ejecuten en el mismo host. Esto es útil para escenarios de multiinquilino con requisitos para hospedar código que no es de confianza, incluidas las aplicaciones SaaS y el hospedaje de cálculos.

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>¿Puedo ejecutar contenedores de Windows en modo aislado de proceso en Windows 10 Enterprise o Professional?

A partir de la actualización de Windows 10 de octubre de 2018, puede ejecutar un contenedor de Windows con aislamiento de procesos, pero primero debe solicitar directamente el `--isolation=process` aislamiento de proceso usando la marca `docker run`al ejecutar los contenedores con.

Si desea ejecutar los contenedores de Windows de esta manera, tendrá que asegurarse de que su host ejecuta Windows 10 Build 17763 + y de tener una versión de Docker con Engine 18,09 o posterior.

> [!WARNING]
> Esta característica solo se ha diseñado para el desarrollo y las pruebas. Debe continuar usando Windows Server como host para las implementaciones de producción. Al usar esta característica, también debe asegurarse de que las etiquetas de versión de contenedor y host coincidan; de lo contrario, es posible que el contenedor no se inicie o que presente un comportamiento no definido.

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>¿Cómo hago para que mis imágenes de contenedor estén disponibles en máquinas con aire libre?

Las imágenes base del contenedor de Windows contienen artefactos cuya distribución está restringida por licencia. Cuando se compila en estas imágenes y se insertan en un registro privado o público, observará que la capa base nunca se inserta. En su lugar, usamos el concepto de una capa externa que apunta a la capa base real que reside en el almacenamiento en la nube de Azure.

Esto puede complicar las cosas cuando tienes un equipo con separación por aire que solo puede extraer imágenes de la dirección de tu registro de contenedor privado. En este caso, los intentos de seguir la capa externa para obtener la imagen base no funcionarán. Para invalidar el comportamiento de la capa externa, puede `--allow-nondistributable-artifacts` usar la marca en el daemon del acoplador.

> [!IMPORTANT]
> El uso de este indicador no impedirá que su obligación cumpla con las condiciones de la licencia básica de la contenedora de Windows; no debe publicar contenido de Windows para una redistribución pública o de terceros. Se permite el uso de su propio entorno.

## <a name="additional-feedback"></a>Comentarios adicionales

¿Quiere agregar algo a las preguntas más frecuentes? Abra un nuevo problema de comentarios en la sección Comentarios o configure una solicitud de extracción para esta página con GitHub.
