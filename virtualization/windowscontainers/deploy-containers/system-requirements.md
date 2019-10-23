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
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254265"
---
# <a name="windows-container-requirements"></a>Requisitos de los contenedores de Windows

En esta guía se enumeran los requisitos para un host contenedor de Windows.

## <a name="operating-system-requirements"></a>Requisitos de sistema operativo

- La característica contenedor de Windows está disponible en Windows Server (canal semianual), Windows Server 2019, Windows Server 2016 y Windows 10 Professional y Enterprise (versión 1607 y posteriores).
- El rol de Hyper-V debe instalarse antes de poder ejecutar el aislamiento de Hyper-V
- Los hosts de contenedor de Windows Server deben tener Windows instalado en c:\. Esta restricción no se aplica si solo se implementarán los contenedores aislados de Hyper-V.

## <a name="virtualized-container-hosts"></a>Hosts de contenedor virtualizados

Si se va a ejecutar un host contenedor de Windows desde una máquina virtual de Hyper-V y también va a hospedar el aislamiento Hyper-V, se debe habilitar la virtualización anidada. La virtualización anidada consta de los siguientes requisitos:

- Al menos 4 GB de RAM disponibles para el host de Hyper-V virtualizado.
- Windows Server (canal semianual), Windows Server 2019, Windows Server 2016 o Windows 10 en el sistema host; y Windows Server (completo o Server Core) en la máquina virtual.
- Un procesador con Intel VT-x (actualmente, esta característica solo está disponible para los procesadores Intel).
- La VM host de contenedor también necesitará al menos dos procesadores virtuales.

### <a name="memory-requirements"></a>Requisitos de memoria

Se pueden configurar restricciones en la memoria disponible a través de los contenedores [controles de recursos](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls) o mediante la sobrecarga de un host de contenedor.  A continuación se muestra la cantidad mínima de memoria necesaria para iniciar un contenedor y ejecutar comandos básicos (ipconfig, dir, etc.).

>[!NOTE]
>Estos valores no tienen en cuenta el uso compartido de recursos entre contenedores o requisitos de la aplicación que se ejecuta en el contenedor.  Por ejemplo, un host con 512MB de memoria libre puede ejecutar varios contenedores de ServerCore en el aislamiento de Hyper-V porque estos contenedores comparten recursos.

#### <a name="windows-server-2016"></a>WindowsServer2016

| Imagen base  | Contenedor de Windows Server | Aislamiento de Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | archivo de paginación de 130 MB + 1 GB |
| ServerCore | 50 MB                     | archivo de paginación de 325 MB + 1 GB |

#### <a name="windows-server-semi-annual-channel"></a>Windows Server (Canal semianual)

| Imagen base  | Contenedor de Windows Server | Aislamiento de Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | archivo de paginación de 110 MB + 1 GB |
| ServerCore | 45 MB                     | archivo de paginación de 360 MB + 1 GB |

## <a name="see-also"></a>Puedes ver también

[Directiva de soporte técnico para los contenedores y el acoplador de Windows en escenarios locales](https://support.microsoft.com/help/4489234/support-policy-for-windows-containers-and-docker-on-premises)