---
title: Requisitos de los contenedores de Windows
description: Requisitos de los contenedores de Windows.
keywords: metadatos, contenedores
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: df5d8e17d0d512f7f53fcacf6c2c2a2652f3e7c0
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107869"
---
# <a name="windows-container-requirements"></a>Requisitos de los contenedores de Windows

En esta guía se enumeran los requisitos para un host contenedor de Windows.

## <a name="os-requirements"></a>Requisitos del sistema operativo

- La característica contenedor de Windows solo está disponible en Windows Server 2016 (Core y con experiencia de escritorio), Windows 10 Professional y Enterprise (edición de aniversario) y versiones posteriores.
- El rol de Hyper-V debe instalarse antes de poder ejecutar el aislamiento de Hyper-V
- Los hosts de contenedor de Windows Server deben tener Windows instalado en c:\. Esta restricción no se aplica si solo se implementarán los contenedores aislados de Hyper-V.

## <a name="virtualized-container-hosts"></a>Hosts de contenedor virtualizados

Si se va a ejecutar un host contenedor de Windows desde una máquina virtual de Hyper-V y también va a hospedar el aislamiento Hyper-V, se debe habilitar la virtualización anidada. La virtualización anidada consta de los siguientes requisitos:

- Al menos 4 GB de RAM disponibles para el host de Hyper-V virtualizado.
- Windows Server 2019, Windows Server versión 1803, Windows Server versión 1709, Windows Server 2016 o Windows 10 en el sistema host, y Windows Server (completo, núcleo) en la máquina virtual.
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

#### <a name="windows-server-version-1709"></a>WindowsServer, versión1709

| Imagen base  | Contenedor de Windows Server | Aislamiento de Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | archivo de paginación de 110 MB + 1 GB |
| ServerCore | 45 MB                     | archivo de paginación de 360 MB + 1 GB |
