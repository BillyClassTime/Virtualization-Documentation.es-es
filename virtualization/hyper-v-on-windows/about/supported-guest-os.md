---
title: Invitados de Windows admitidos
description: Invitados de Windows admitidos.
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
ms.openlocfilehash: 25c72b910af15fc0b498a5b2abce72d32e6d1efd
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999042"
---
# <a name="supported-windows-guests"></a>Invitados de Windows admitidos

En este artículo se enumeran las combinaciones de sistemas operativos que se admiten en Hyper-V en Windows.  También sirve como una introducción a los servicios de integración y otros factores de compatibilidad.

Microsoft ha probado estas combinaciones de hosts e invitados.  Los problemas con estas combinaciones pueden recibir atención del Servicio de soporte técnico de productos.

Microsoft proporciona soporte técnico de la siguiente manera:

* Los problemas encontrados en sistemas operativos y en servicios de integración de Microsoft los atiende el soporte técnico de Microsoft.

* Los problemas encontrados en otros sistemas operativos cuya ejecución en Hyper-V se encuentra certificada por el proveedor del sistema operativo, se admiten en el soporte del proveedor.

* Microsoft envía los problemas detectados en otros sistemas operativos a la comunidad de soporte técnico de varios proveedores, [TSANet](http://www.tsanet.org/).

Para poder recibir soporte técnico, todos los sistemas operativos (cliente y host) deben estar actualizados.  Comprueba si hay actualizaciones críticas en Windows Update.

## <a name="supported-guest-operating-systems"></a>Sistemas operativos invitados compatibles

| Sistema operativo invitado |  Número máximo de procesadores virtuales | Notas |
|:-----|:-----|:-----|
| Windows 10 | 32 |El Modo de sesión mejorada no funciona en Windows10HomeEdition |
| Windows8.1 | 32 | |
| Windows 8 | 32 ||
| Windows7 con Service Pack1 (SP1) | cuatro | Ediciones Ultimate, Enterprise y Professional  (32 y 64bits). |
| Windows 7 | cuatro | Ediciones Ultimate, Enterprise y Professional  (32 y 64bits). |
| Windows Vista con Service Pack2 (SP2) | 1 | Ediciones Business, Enterprise y Ultimate, incluidas las ediciones N y KN. |
| - | | |
| [Canal semianual de WindowsServer](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview) | 64 | |
| Windows Server 2019 | 64 | |
| WindowsServer2016 | 64 | |
| WindowsServer2012R2 | 64 | |
| Windows Server 2012 | 64 | |
| Windows Server2008R2 con Service Pack1 (SP1) | 64 | Ediciones Datacenter, Enterprise, Standard y Web. |
| Windows Server2008 con Service Pack2 (SP 2) | cuatro | Ediciones Datacenter, Enterprise, Standard y Web (32 y 64bits). |
| Windows Home Server2011 | cuatro | |
| Windows Small Business Server2011 | Essentials edition: 2; Standard edition: 4 | |

> Windows 10 puede ejecutarse como un sistema operativo invitado en hosts de Hyper-V de Windows 8.1 y Windows Server 2012 R2.

## <a name="supported-linux-and-free-bsd"></a>Linux y Free BSD admitidos

| Sistema operativo invitado |  |
|:-----|:------|
| [CentOS y Red Hat Enterprise Linux](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-CentOS-and-Red-Hat-Enterprise-Linux-virtual-machines-on-Hyper-V) | |
| [Máquinas virtuales de Debian en Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Debian-virtual-machines-on-Hyper-V) | |
| [SUSE](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-SUSE-virtual-machines-on-Hyper-V) | |
| [Oracle Linux](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Oracle-Linux-virtual-machines-on-Hyper-V)  | |
| [Ubuntu](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Ubuntu-virtual-machines-on-Hyper-V) | |
| [FreeBSD](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-FreeBSD-virtual-machines-on-Hyper-V) | |

Para más información, incluida información de soporte técnico sobre las versiones anteriores de Hyper-V, consulte [Máquinas virtuales de Linux y FreeBSD en Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows).
