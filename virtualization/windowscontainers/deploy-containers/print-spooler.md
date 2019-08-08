---
title: Administrador de trabajos de impresión en contenedores de Windows
description: Explica el comportamiento de trabajo actual para el servicio Administrador de trabajos de impresión en contenedores de Windows.
keywords: acoplador, contenedores, impresora, cola de impresión
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999102"
---
# <a name="print-spooler-in-windows-containers"></a>Administrador de trabajos de impresión en contenedores de Windows

Las aplicaciones con dependencias de servicios de impresión se pueden almacenar correctamente con contenedores de Windows. Hay requisitos especiales que deben cumplirse para habilitar correctamente la funcionalidad del servicio de impresión. Esta guía explica cómo configurar correctamente su implementación.

> [!IMPORTANT]
> Aunque se puede obtener acceso a los servicios de impresión correctamente en contenedores, la funcionalidad está limitada en forma; es posible que algunas acciones relacionadas con la impresión no funcionen. Por ejemplo, las aplicaciones que tienen una dependencia en la instalación de los drivers de impresora en el host no se pueden almacenar porque no **se admite la instalación de controladores desde un contenedor**. Si encuentra una característica de impresión no admitida que quiere que se admita en los contenedores, abra una de las respuestas a continuación.

## <a name="setup"></a>Programa de instalación

* El host debe ser Windows Server 2019 o Windows 10 Pro/Enterprise de octubre de 2018 o posterior.
* La imagen de [MCR.Microsoft.com/Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) debe ser la imagen base de destino. Otras imágenes base del contenedor de Windows (como nano Server y Windows Server Core) no tienen el rol de servidor de impresión.

### <a name="hyper-v-isolation"></a>Aislamiento de Hyper-V

Le recomendamos que ejecute su contenedor con el aislamiento de Hyper-V. Cuando se ejecuta en este modo, puede tener tantos contenedores como desee usar con el acceso a los servicios de impresión. No es necesario modificar el servicio del administrador de trabajos de impresión en el host.

Puede comprobar la funcionalidad con la siguiente consulta de PowerShell:

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

Debido a la naturaleza compartida del núcleo de los contenedores aislados en el proceso, el comportamiento actual limita al usuario a ejecutar solo **una instancia** del servicio del administrador de trabajos de impresión a través del host y todos sus elementos secundarios de contenedor. Si el host tiene la cola de impresión en ejecución, debe detener el servicio en el host antes de attemping iniciar el servicio de impresora en el invitado.

> [!TIP]
> Si inicia un contenedor y consulta el servicio Administrador de trabajos de impresión tanto en el contenedor como en el host de forma simultánea, ambos notificarán su estado como "en ejecución". Pero no se puede engañar: el contenedor no podrá consultar una lista de las impresoras disponibles. El servicio de administrador de trabajos de impresión no debe ejecutarse. 

Para comprobar si el host está ejecutando el servicio de impresora, use la consulta de PowerShell a continuación:

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

Para detener el servicio de administrador de trabajos de impresión en el host, use los siguientes comandos en PowerShell a continuación:

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

Inicie el contenedor y compruebe el acceso a las impresoras.

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