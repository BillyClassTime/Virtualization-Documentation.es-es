---
title: Migración de Hyper-V de WMIv1 a WMIv2
description: Obtén información sobre cómo migrar Hyper-V de WMIv1 a WMIv2
keywords: windows 10, hyper-v, WMIv1, WMIv2, WMI, Msvm_VirtualSystemGlobalSettingData, root\virtualization
author: scooley
ms.date: 04/13/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: b13a3594-d168-448b-b0a1-7d77153759a8
ms.openlocfilehash: e2d6faabe77346199a5d292fcfd92cdfd63909b8
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910905"
---
# <a name="move-from-hyper-v-wmi-v1-to-wmi-v2"></a>Migración de Hyper-V de WMI v1 a WMI v2

Instrumental de administración de Windows (WMI) es la interfaz de administración subyacente del administrador de Hyper-V y de los cmdlets de PowerShell de Hyper-V.  Aunque la mayoría usa los cmdlets de PowerShell o el administrador de Hyper-V, a veces los desarrolladores necesitaban WMI directamente.  

Han existido dos espacios de nombres WMI de Hyper-V (o versiones de la API de WMI de Hyper-V).
* El espacio de nombres WMI v1 (root\virtualization) que se introdujo en Windows Server 2008, y el último disponible en Windows Server 2012
* El espacio de nombres WMI v2 (root\virtualization\v2) que se introdujo en Windows Server 2012

Este documento contiene referencias a recursos para convertir el código que se comunica con el espacio de nombres WMI antiguo al nuevo.  Inicialmente, este artículo servirá como repositorio de información sobre API y código o scripts de ejemplo que pueden usarse para ayudar a migrar todos los programas o scripts que usan las API de WMI de Hyper-V desde el espacio de nombres v1 al espacio de nombres v2.

## <a name="msdn-samples"></a>Ejemplos de MSDN

[Ejemplo de migración de máquina virtual de Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-machine-aef356ee)  
[Ejemplo de canal de fibra virtual de Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-Fiber-35d27dcd)  
[Ejemplo de máquinas virtuales planeadas de Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-planned-virtual-8c7b7499)  
[Ejemplo de supervisión de estado de aplicaciones de Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-application-health-dc0294f2)  
[Ejemplo de administración de discos duros virtuales](http://code.msdn.microsoft.com/windowsdesktop/Virtual-hard-disk-03108ed3)  
[Ejemplo de replicación de Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-replication-sample-d2558867)  
[Ejemplo de métricas de Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-metrics-sample-2dab2cb1)  
[Ejemplo de memoria dinámica de Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-dynamic-memory-9b0b1d05)  
[Controlador de filtro de extensión de conmutador extensible de Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-Extensible-Virtual-e4b31fbb)  
[Ejemplo de red de Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-networking-sample-7c47e6f5)  
[Ejemplo de administración de grupos de recursos de Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-resource-pool-df906d95)  
[Ejemplo de instantánea de recuperación de Hyper-V](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-recovery-snapshot-ea72320c)  

## <a name="samples-from-blogs"></a>Ejemplos de blogs

[Agregar un adaptador de red a una máquina virtual mediante el espacio de nombres de WMI V2 de Hyper-V](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/adding-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Conexión de un adaptador de red de máquina virtual a un conmutador mediante el espacio de nombres de WMI V2 de Hyper-V](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/connecting-a-vm-network-adapter-to-a-switch-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Cambio de la dirección MAC de la NIC mediante el espacio de nombres de WMI V2 de Hyper-V](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/changing-the-mac-address-of-nic-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Quitar un adaptador de red a una máquina virtual mediante el espacio de nombres de WMI V2 de Hyper-V](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Conexión de un disco duro virtual a una máquina virtual mediante el espacio de nombres de WMI V2 de Hyper-V](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/attaching-a-vhd-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Eliminación de un disco duro virtual de una máquina virtual mediante el espacio de nombres de WMI V2 de Hyper-V](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-vhd-from-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Creación de una máquina virtual con el espacio de nombres de WMI V2 de Hyper-V](http://blogs.msdn.com/b/virtual_pc_guy/archive/2013/06/20/creating-a-virtual-machine-with-wmi-v2.aspx)

