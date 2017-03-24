---
title: Especificaciones de hipervisor
description: Especificaciones de hipervisor
keywords: Windows 10, Hyper-V
author: theodthompson
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
translationtype: Human Translation
ms.sourcegitcommit: a193002059a4f1fd0e81689deae02846da01736f
ms.openlocfilehash: 495bd9228e93a1981a646f13e2e583a741eec217
ms.lasthandoff: 03/03/2017

---

# Especificaciones de hipervisor

## Especificación funcional de nivel superior del hipervisor

La especificación funcional de nivel superior (TLFS) del hipervisor de Hyper-V describe el comportamiento externo visible del hipervisor para otros componentes del sistema operativo. Se prevé que esta especificación sea útil para los desarrolladores de sistemas operativos invitados.
  
> Esta especificación se proporciona en virtud de Microsoft Open Specification Promise.  Lea la información siguiente para obtener más detalles acerca de [Microsoft Open Specification Promise](https://msdn.microsoft.com/en-us/openspecifications).  

#### Descarga
Lanzamiento | Documento
--- | ---
Windows Server 2016 | [Hypervisor Top Level Functional Specification v5.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0.pdf)
Windows Server 2012 R2 (revisión B) | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 R2 | [Hypervisor Top Level Functional Specification v4.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## Requisitos para implementar la interfaz de hipervisor de Microsoft

Los sistemas operativos Windows requieren que un conjunto limitado de interfaces de hipervisor se ejecute en una máquina virtual invitada (también conocida como interfaz "HV #1"). Además, puede implementar varias características opcionales mediante un hipervisor que sea compatible con Microsoft. Estas opciones cambiarán el comportamiento de Windows en una máquina virtual. En "Requisitos para implementar la interfaz de hipervisor de Microsoft" se describen las características obligatorias y opcionales que se implementan mediante un hipervisor compatible con Microsoft.

#### Descarga

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)
