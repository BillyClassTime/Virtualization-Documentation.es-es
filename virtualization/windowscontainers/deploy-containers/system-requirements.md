---
title: Requisitos de los contenedores de Windows
description: Requisitos de los contenedores de Windows.
keywords: metadatos, contenedores
author: taylorb-microsoft
ms.author: taylorb
ms.date: 10/22/2019
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 74f501e5efab3a93e60c9d4797464cea283cdc0b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910505"
---
# <a name="windows-container-requirements"></a>Requisitos de los contenedores de Windows

En esta guía se enumeran los requisitos de un host de contenedor de Windows.

## <a name="operating-system-requirements"></a>Requisitos de sistema operativo

- La característica de contenedor de Windows está disponible en Windows Server (canal semianual), Windows Server 2019, Windows Server 2016 y Windows 10 Professional y Enterprise Edition (versión 1607 y posteriores).
- El rol de Hyper-V debe instalarse antes de ejecutar el aislamiento de Hyper-V
- Los hosts de contenedor de Windows Server deben tener Windows instalado en c:\. Esta restricción no se aplica si solo se implementarán los contenedores aislados de Hyper-V.

## <a name="virtualized-container-hosts"></a>Hosts de contenedor virtualizados

Si un host de contenedor de Windows se va a ejecutar desde una máquina virtual de Hyper-V y también va a hospedar el aislamiento de Hyper-V, será necesario habilitar la virtualización anidada. La virtualización anidada consta de los siguientes requisitos:

- Al menos 4 GB de RAM disponibles para el host de Hyper-V virtualizado.
- Windows Server (canal semianual), Windows Server 2019, Windows Server 2016 o Windows 10 en el sistema host; y Windows Server (completo o Server Core) en la máquina virtual.
- Un procesador con Intel VT-x (actualmente, esta característica solo está disponible para los procesadores Intel).
- La máquina virtual del host de contenedor también necesitará al menos dos procesadores virtuales.

### <a name="memory-requirements"></a>Requisitos de memoria

Se pueden configurar restricciones en la memoria disponible a través de los contenedores [controles de recursos](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls) o mediante la sobrecarga de un host de contenedor.  A continuación se enumera la cantidad mínima de memoria necesaria para iniciar un contenedor y ejecutar comandos básicos (ipconfig, dir, etc.).

>[!NOTE]
>Estos valores no tienen en cuenta el uso compartido de recursos entre contenedores o requisitos de la aplicación que se ejecuta en el contenedor.  Por ejemplo, un host con 512 MB de memoria libre puede ejecutar varios contenedores de Server Core en el aislamiento de Hyper-V porque estos contenedores comparten recursos.

#### <a name="windows-server-2016"></a>Windows Server 2016

| Base image  | Contenedor de Windows Server | Aislamiento de Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | archivo de paginación de 130 MB + 1 GB |
| Server Core | 50 MB                     | archivo de paginación de 325 MB + 1 GB |

#### <a name="windows-server-semi-annual-channel"></a>Windows Server (Canal semianual)

| Base image  | Contenedor de Windows Server | Aislamiento de Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | archivo de paginación de 110 MB + 1 GB |
| Server Core | 45 MB                     | archivo de paginación de 360 MB + 1 GB |

## <a name="see-also"></a>Consulta también

[Directiva de compatibilidad para contenedores de Windows y Docker en escenarios locales](https://support.microsoft.com/help/4489234/support-policy-for-windows-containers-and-docker-on-premises)