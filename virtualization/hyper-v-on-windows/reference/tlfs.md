---
title: Especificaciones de hipervisor
description: Especificaciones de hipervisor
keywords: Windows 10, Hyper-V
author: allenma
ms.date: 06/26/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
ms.openlocfilehash: 26eaca5a583f8b2e11ca1e05e2fa9a4366fd8837
ms.sourcegitcommit: 594cc1728347646609ae1952ecc6c97fc659d0a9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/07/2018
ms.locfileid: "2226211"
---
# <a name="hypervisor-specifications"></a>Especificaciones de hipervisor

## <a name="hypervisor-top-level-functional-specification"></a>Especificación funcional de nivel superior del hipervisor

La especificación funcional de nivel superior (TLFS) del hipervisor de Hyper-V describe el comportamiento externo visible del hipervisor para otros componentes del sistema operativo. Se prevé que esta especificación sea útil para los desarrolladores de sistemas operativos invitados.
  
> Esta especificación se proporciona en virtud de Microsoft Open Specification Promise.  Lea la información siguiente para obtener más detalles acerca de [Microsoft Open Specification Promise](https://msdn.microsoft.com/en-us/openspecifications).  

#### <a name="download"></a>Descarga
Lanzamiento | Documento
--- | ---
Windows Server 2016 (revisión C) | [Hypervisor Top Level Functional Specification v5.0c.pdf](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/live/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0C.pdf)
Windows Server 2012 R2 (revisión B) | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## <a name="requirements-for-implementing-the-microsoft-hypervisor-interface"></a>Requisitos para implementar la interfaz de hipervisor de Microsoft

La TLFS describe completamente todos los aspectos de la arquitectura de hipervisor específico de Microsoft, que se declara en las máquinas virtuales invitadas como interfaz "HV#1".  Sin embargo, no todas las interfaces descritas en la TLFS son necesarias para implementarse mediante hipervisor de terceros con la finalidad de declarar conformidad con la especificación de hipervisor HV#1 de Microsoft. El documento "Requisitos de implementación de la interfaz de hipervisor de Microsoft" describe el conjunto mínimo de interfaces de hipervisor que debe implementarse mediante cualquier hipervisor que asegure compatibilidad con la interfaz HV#1 de Microsoft.

#### <a name="download"></a>Descarga

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)
