---
title: Ciclos de vida de mantenimiento de imágenes base
description: Información sobre el ciclo de vida de la imagen base del contenedor de Windows.
keywords: contenedores de Windows, contenedores, ciclo de vida, información de versión, imagen base, imagen base de contenedor
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: bb5e5fabadde421de9d420edd2fc921457432930
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909995"
---
# <a name="base-image-servicing-lifecycles"></a>Ciclos de vida de mantenimiento de imágenes base

Las imágenes base del contenedor de Windows se basan en versiones de canal semianual o en versiones de canal de mantenimiento a largo plazo de Windows Server. En este artículo se indicará el tiempo que durará la compatibilidad para las diferentes versiones de imágenes base de ambos canales.

El canal semianual es una versión de actualización de características dos veces por año con escalas de tiempo de mantenimiento de 18 meses para cada versión. Esto permite a los clientes aprovechar las nuevas capacidades del sistema operativo a un ritmo más rápido, tanto en las aplicaciones (especialmente en las que se basan en contenedores y microservicios) como en el centro de centros de recursos híbridos definido por software. Para obtener más información, consulte [información general del canal semianual de Windows Server](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview).

En el caso de las imágenes Server Core, los clientes también pueden usar el canal de mantenimiento a largo plazo, que lanza una nueva versión principal de Windows Server cada dos o tres años. Las versiones de canal de mantenimiento a largo plazo reciben cinco años de soporte estándar y cinco años de soporte extendido. Este canal funciona con sistemas que requieren una opción de mantenimiento más larga y estabilidad funcional.

En la tabla siguiente se enumeran los tipos de imagen base, su canal de servicio y el tiempo que durará la compatibilidad.

|Base image                       |Canal de mantenimiento|Versión|Compilación del sistema operativo|Disponibilidad|Fecha de finalización del soporte estándar|Fecha de soporte extendido|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core, nano Server, Windows|Semestral      |1909   |18363   |12/11/2019  |11/05/2021                 |N/D                  |
|Server Core, nano Server, Windows|Semestral      |1903   |18362   |05/21/2019  |08/12/2020                 |N/D                  |
|Server Core                      |A largo plazo        |1809   |17763   |13/11/2018  |09/01/2024                 |09/01/2029           |
|Server Core, nano Server, Windows|Semestral      |1809   |17763   |13/11/2018  |05/12/2020                 |N/D                  |
|Server Core, nano Server         |Semestral      |1803   |17134   |30/04/2018  |12/11/2019                 |N/D                  |
|Server Core, nano Server         |Semestral      |1709   |16299   |17/10/2017  |09/04/2019                 |N/D                  |
|Server Core                      |A largo plazo        |1607   |14393   |15/10/2016  |11/01/2022                 |11/01/2027           |
|Nano Server                      |Semestral      |1607   |14393   |15/10/2016  |10/09/2018                 |N/D                  |

Para obtener información sobre los requisitos de servicio y otra información adicional, consulte las [preguntas más frecuentes del ciclo de vida de Windows](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products), información de la [versión de Windows Server](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info)y el [repositorio del concentrador de Docker del sistema operativo Windows base](https://hub.docker.com/_/microsoft-windows-base-os-images).
