---
title: Administrador de trabajos de impresión en contenedores de Windows
description: Se explica el comportamiento de trabajo actual para el servicio de cola de impresión en contenedores de Windows
keywords: docker, contenedores, impresoras, Administrador de trabajos
author: cwilhit
ms.openlocfilehash: 48130bc6a826a45dfa49d0a3b4600d227f34704e
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576666"
---
# <a name="print-spooler-in-windows-containers"></a>Administrador de trabajos de impresión en contenedores de Windows

Las aplicaciones con una dependencia en los servicios de impresión pueden ser en contenedores correctamente con contenedores de Windows. Existen requisitos especiales que se deben cumplir para poder habilitar correctamente la funcionalidad de servicio de impresora. Esta guía explica cómo configurar la implementación de forma adecuada.

> [!IMPORTANT]
> Mientras que obtiene acceso a la impresión de servicios correctamente en contenedores funciona, funcionalidad está limitada, de forma. Algunas acciones relacionadas con la impresión no funcionen. Por ejemplo, las aplicaciones que tienen una dependencia en instalar controladores de impresora en el host no pueden ser en contenedores porque **no se admite la instalación de controladores desde dentro de un contenedor**. Abre tus comentarios a continuación si te encuentras con una función no admitida de impresión que quieres que se admiten en contenedores.

## <a name="setup"></a>Programa de instalación

* El host debe ser Windows Server 2019 o Windows 10 Pro/Enterprise octubre de 2018 update o posterior.
* La imagen [mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) debe ser la imagen base de destino. Otras imágenes base de contenedor de Windows (por ejemplo, Nano Server y Windows Server Core) no llevan el rol de servidor de impresión.

### <a name="hyper-v-isolation"></a>Aislamiento de Hyper-V

Se recomienda ejecutar el contenedor con aislamiento de Hyper-V. Cuando se ejecuta en este modo, puedes tener tantas contenedores que quieras ejecutando con acceso a los servicios de impresión. No es necesario modificar el servicio de cola en el host.

Puedes comprobar la funcionalidad con la siguiente consulta de PowerShell:

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation hyperv mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```

### <a name="process-isolation"></a>Aislamiento de procesos

Debido a la naturaleza de kernel compartido de contenedores de aislamiento de proceso, comportamiento actual limita al usuario que ejecuta solo **una instancia** del servicio de administrador de trabajos de impresora entre el host y todos sus elementos secundarios de contenedor. Si el host tiene la cola de impresión que se ejecuta, debe detener el servicio en el host antes de attemping para iniciar el servicio de impresora en el invitado.

> [!TIP]
> Si puedes iniciar un contenedor y consultan el servicio de cola en el contenedor y el host al mismo tiempo, ambos informará de su estado como "running". Pero no ser convencidos mediante engaño: el contenedor no podrán consultar para obtener una lista de impresoras disponibles. No debe ejecutar el servicio de cola del host. 

Para comprobar si el host está ejecutando el servicio de la impresora, usa la consulta en PowerShell siguiente:

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

Para detener el servicio de cola en el host, usa los siguientes comandos en PowerShell siguiente:

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

Inicia el contenedor y comprobar el acceso a las impresoras.

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation process mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.


PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```