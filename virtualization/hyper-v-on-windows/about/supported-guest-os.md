---
title: Invitados de Windows admitidos
description: Invitados de Windows admitidos.
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
ms.openlocfilehash: 7ef4d5aa084d199abfb39d1c44e4dd1305e07904
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/21/2017
---
# Invitados de Windows admitidos 

En este artículo se enumeran las combinaciones de sistemas operativos que se admiten en Hyper-V en Windows.  También sirve como una introducción a los servicios de integración y otros factores de compatibilidad.

## ¿Qué significa compatibilidad? 
La compatibilidad significa que Microsoft ha probado estas combinaciones de hosts e invitados.  Los problemas con estas combinaciones pueden recibir atención del Servicio de soporte técnico del producto.
 
Microsoft ofrece compatibilidad para los sistemas operativos invitados de la siguiente manera:
* Los problemas encontrados en sistemas operativos y en servicios de integración de Microsoft se admiten en el soporte técnico de Microsoft.
* Los problemas encontrados en otros sistemas operativos cuya ejecución en Hyper-V se encuentra certificada por el proveedor del sistema operativo, se admiten en el soporte del proveedor.
* Microsoft envía los problemas detectados en otros sistemas operativos a la comunidad de soporte técnico de varios proveedores, [TSANet](http://www.tsanet.org/).

Para que se admitan, tanto el host de Hyper-V como el invitado deben estar actualizados con todas las actualizaciones críticas disponibles a través de Windows Update.

## Sistemas operativos invitados compatibles

Para recibir soporte técnico, tanto el sistema operativo invitado de Windows como el sistema operativo host deben estar actualizados y disponer de todas las actualizaciones críticas disponibles a través de Windows Update.

| Sistema operativo invitado |  Número máximo de procesadores virtuales | Notas | 
|:-----|:-----|:-----|
| Windows 10 | 32 | |
| Windows 8.1 | 32 | |
| Windows 8 | 32 |  |
| Windows7 con Service Pack1 (SP1) | 4 | Ediciones Ultimate, Enterprise y Professional  (32 y 64bits). |
| Windows 7 | 4 | Ediciones Ultimate, Enterprise y Professional  (32 y 64bits). |
| Windows Vista con Service Pack2 (SP2) | 2 | Ediciones Business, Enterprise y Ultimate, incluidas las ediciones N y KN. | 
| - | | |
| Windows Server2012R2 | 64 | |
| Windows Server 2012 | 64 | |
| Windows Server2008R2 con Service Pack1 (SP1) | 64 | Ediciones Datacenter, Enterprise, Standard y Web. |
| Windows Server2008 con Service Pack2 (SP 2) | 4 | Ediciones Datacenter, Enterprise, Standard y Web (32 y 64bits). |
| Windows Home Server2011 | 4 | |
| Windows Small Business Server2011 | Essentials edition: 2; Standard edition: 4 | |
  
 > Windows 10 puede ejecutarse como un sistema operativo invitado en hosts de Hyper-V de Windows 8.1 y Windows Server 2012 R2.

## Linux y Free BSD admitidos

| Sistema operativo invitado |  |
|:-----|:------|
| [CentOS y Red Hat Enterprise Linux ](https://technet.microsoft.com/library/dn531026.aspx) | |
| [Máquinas virtuales de Debian en Hyper-V](https://technet.microsoft.com/library/dn614985.aspx) | |
| [SUSE](https://technet.microsoft.com/en-us/library/dn531027.aspx) | |
| [Oracle Linux](https://technet.microsoft.com/en-us/library/dn609828.aspx)  | |
| [Ubuntu](https://technet.microsoft.com/en-us/library/dn531029.aspx) | |
| [FreeBSD](https://technet.microsoft.com/library/dn848318.aspx) | |

Para más información, incluida información de soporte técnico sobre las versiones anteriores de Hyper-V, consulte [Máquinas virtuales de Linux y FreeBSD en Hyper-V](https://technet.microsoft.com/library/dn531030.aspx).
