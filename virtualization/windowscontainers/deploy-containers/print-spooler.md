---
title: Administrador de trabajos de impresión en contenedores de Windows
description: Explica el comportamiento de trabajo actual para el servicio Administrador de trabajos de impresión en contenedores de Windows
keywords: Docker, contenedores, impresora, administrador de trabajos de impresión
author: cwilhit
ms.openlocfilehash: 48130bc6a826a45dfa49d0a3b4600d227f34704e
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910535"
---
# <a name="print-spooler-in-windows-containers"></a>Administrador de trabajos de impresión en contenedores de Windows

Las aplicaciones con una dependencia en los servicios de impresión se pueden incluir correctamente en contenedores de Windows. Existen requisitos especiales que se deben cumplir para poder habilitar correctamente la funcionalidad del servicio de impresora. En esta guía se explica cómo configurar correctamente la implementación.

> [!IMPORTANT]
> Aunque el acceso a los servicios de impresión se realiza correctamente en los contenedores funciona, la funcionalidad tiene un formato limitado. es posible que algunas acciones relacionadas con la impresión no funcionen. Por ejemplo, las aplicaciones que dependen de la instalación de controladores de impresora en el host no se pueden incluir en contenedores porque no **se admite la instalación de controladores desde dentro de un contenedor**. Abra una información a continuación si encuentra una característica de impresión no compatible que desea que se admita en los contenedores.

## <a name="setup"></a>Instalación

* El host debe ser Windows Server 2019 o Windows 10 Pro/Enterprise de octubre de 2018 o posterior.
* La imagen de [MCR.Microsoft.com/Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) debe ser la imagen base de destino. Otras imágenes base del contenedor de Windows (como nano Server y Windows Server Core) no llevan el rol del servidor de impresión.

### <a name="hyper-v-isolation"></a>Aislamiento de Hyper-V

Se recomienda ejecutar el contenedor con el aislamiento de Hyper-V. Cuando se ejecuta en este modo, puede tener tantos contenedores como desee ejecutar con acceso a los servicios de impresión. No es necesario modificar el servicio Administrador de trabajos de impresión en el host.

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

Debido a la naturaleza del kernel compartido de los contenedores aislados del proceso, el comportamiento actual limita al usuario a ejecutar solo **una instancia** del servicio de cola de impresión en el host y todos sus elementos secundarios del contenedor. Si el host tiene la cola de impresión en ejecución, debe detener el servicio en el host antes de que intentar inicie el servicio de impresora en el invitado.

> [!TIP]
> Si inicia un contenedor y consulta el servicio de cola de impresión en el contenedor y el host simultáneamente, ambos notificarán su estado como "en ejecución". Pero no se puede engañar: el contenedor no podrá consultar una lista de las impresoras disponibles. No se debe ejecutar el servicio Administrador de trabajos de impresión del host. 

Para comprobar si el host está ejecutando el servicio de impresión, use la consulta de PowerShell siguiente:

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

Para detener el servicio Administrador de trabajos de impresión en el host, use los siguientes comandos en PowerShell a continuación:

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