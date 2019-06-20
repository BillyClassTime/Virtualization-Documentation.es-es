---
title: Ciclos de vida de mantenimiento de imágenes básicos
description: Información sobre el ciclo de vida de la imagen base del contenedor de Windows.
keywords: contenedores de Windows, contenedores, ciclo de vida, información de la versión, imagen base, imagen base del contenedor
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: c26f4b225287fbc25566e36376eb8cd604d45a68
ms.sourcegitcommit: 9cd1aa792a417e71192c7aa39e409ae6ca0bc710
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/20/2019
ms.locfileid: "9788558"
---
# <a name="base-image-servicing-lifecycles"></a>Ciclos de vida de mantenimiento de imágenes básicos

Las imágenes base del contenedor de Windows se basan en las versiones del canal semianual o en las versiones de canal de mantenimiento a largo plazo de Windows Server. Este artículo le indicará cuánto durará la compatibilidad en diferentes versiones de las imágenes base de ambos canales.

El canal semestral es un lanzamiento de actualización de características dos veces por año con escalas de tiempo de mantenimiento de dieciocho meses para cada versión. Esto permite que los clientes aprovechen las nuevas capacidades del sistema operativo a un ritmo más rápido, tanto en las aplicaciones (especialmente en las que se basan en contenedores y microservicios) como en el centro de recursos híbrido definido por software. Para obtener más información, consulte [información general del canal semianual de Windows Server](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview).

Para las imágenes de Server Core, los clientes también pueden usar el canal de mantenimiento de larga duración, que libera una nueva versión principal de Windows Server cada dos o tres años. Las versiones de canal de mantenimiento a largo plazo reciben cinco años de soporte técnico general y cinco años de soporte ampliado. Este canal funciona con sistemas que requieren una opción de mantenimiento más larga y una estabilidad funcional.

En la siguiente tabla se enumeran los distintos tipos de imagen base, su canal de mantenimiento y cuánto tiempo durará la compatibilidad.

|Imagen base                       |Canal de mantenimiento|Versión|Compilación del sistema operativo|Disponibilidad|Fecha de finalización del soporte estándar|Fecha de soporte extendido|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core, nano Server, Windows|Semestral      |1903   |18362   |05/21/2019  |12/08/2020                 |N/D                  |
|ServerCore                      |A largo plazo        |1809   |17763   |13/11/2018  |9/1/2024                 |9/1/2029           |
|Server Core, nano Server, Windows|Semestral      |1809   |17763   |13/11/2018  |05/12/2020                 |N/D                  |
|Server Core, nano Server         |Semestral      |1803   |17134   |30/04/2018  |12/11/2019                 |N/D                  |
|Server Core, nano Server         |Semestral      |1709   |16299   |17/10/2017  |09/04/2019                 |N/A                  |
|ServerCore                      |A largo plazo        |1607   |14393   |15/10/2016  |11/01/2022                 |11/1/2027           |
|Nano Server                      |Semestral      |1607   |14393   |15/10/2016  |10/09/2018                 |N/D                  |

Para obtener información sobre los requisitos de servicio y otra información adicional, consulte las [preguntas más frecuentes](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products)sobre la versión de Windows Lifecycle, la información de la [versión de Windows](https://docs.microsoft.com/en-us/windows-server/get-started/windows-server-release-info)y el repositorio del concentrador de acoplamiento de [imágenes de sistema operativo Windows base](https://hub.docker.com/_/microsoft-windows-base-os-images).
