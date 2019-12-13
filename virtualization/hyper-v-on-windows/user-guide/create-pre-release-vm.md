---
title: Pruebe las características de la versión preliminar para Hyper-V
description: Pruebe las características de la versión preliminar para Hyper-V
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 426c87cc-fa50-4b8d-934e-0b653d7dea7d
ms.openlocfilehash: 8f1c1b96fe88f46a24b8ebb46d4f387c9717f6ba
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74911165"
---
# <a name="try-pre-release-features-for-hyper-v"></a>Pruebe las características de la versión preliminar para Hyper-V

> Este contenido es preliminar y está sujeto a cambios.  
  Las máquinas virtuales de versión preliminar están diseñadas únicamente para entornos de desarrollo o de prueba, ya que Microsoft no las admite.

Obtenga acceso anticipado a las características de la versión preliminar para Hyper-V en Windows Server 2016 Technical Preview para probarlas en el entorno de desarrollo o de prueba. Puede ser el primero en ver las últimas características de Hyper-V y ayudar a darle forma al producto con sus comentarios anticipados.

Las máquinas virtuales que se crean como versión preliminar no tienen compatibilidad de una compilación a otra ni soporte técnico en el futuro.  No las use en un entorno de producción.

Estas son algunas de las razones por las que solo están destinadas a entornos que no sean de producción:

* No existe compatibilidad con versiones posteriores para las máquinas virtuales de versión preliminar. No se puede actualizar estas máquinas virtuales a una nueva versión de configuración.
* Las máquinas virtuales de versión preliminar no tienen una definición coherente entre compilaciones. Si actualiza el sistema operativo host, las máquinas virtuales de versión preliminar existentes podrían ser incompatibles con el host. Estas máquinas virtuales podrían no iniciarse, o en un principio podría parecer que funcionan pero más adelante presentar graves problemas de compatibilidad.
* Si importa una máquina virtual de versión preliminar en un host con una compilación diferente, los resultados son imprevisibles. Puede mover una máquina virtual de versión preliminar a otro host. Pero se espera que este escenario funcione solamente si ambos hosts ejecutan la misma compilación.

## <a name="create-a-pre-release-virtual-machine"></a>Creación de una máquina virtual de versión preliminar

Puede crear una máquina virtual de versión preliminar en hosts de Hyper-V que ejecutan Windows Server 2016 Technical Preview.

1. En el Escritorio de Windows, haga clic en el botón Inicio y escriba cualquier parte del nombre **Windows PowerShell**.
2. Haga clic con el botón derecho en **Windows PowerShell** y seleccione **Ejecutar como administrador**.
3. Use el cmdlet [New-VM](https://docs.microsoft.com/powershell/module/hyper-v/new-vm?view=win10-ps) con la marca -Prerelease para crear la máquina virtual de versión preliminar. Por ejemplo, ejecute el comando siguiente, donde VM Name es el nombre de la máquina virtual que quiere crear.

``` PowerShell
New-VM -Name <VM Name> -Prerelease
```
Otros ejemplos con los que puede usar la marca -Prerelease:
 - Para crear una máquina virtual que use un disco duro virtual existente o un disco duro nuevo, vea los ejemplos de PowerShell de [Create a virtual machine in Hyper-V on Windows Server 2016 Technical Preview](https://docs.microsoft.com/windows-server/virtualization/hyper-v/get-started/Create-a-virtual-machine-in-Hyper-V#BKMK_PowerShell) (Crear una máquina virtual de Hyper-V en Windows Server 2016 Technical Preview).
 - Para crear un disco duro virtual nuevo que se inicie con una imagen de sistema operativo, vea el ejemplo de PowerShell de [Implementar una máquina virtual de Windows en Hyper-V en Windows 10](https://docs.microsoft.com/virtualization/hyper-v-on-windows/quick-start/create-virtual-machine).

 Los ejemplos descritos en estos artículos funcionan con los hosts de Hyper-V que ejecutan Windows 10 o Windows Server 2016 Technical Preview. Pero, por el momento, solo puede usar la marca -Prerelease para crear una máquina virtual de versión preliminar en hosts de Hyper-V que ejecutan Windows Server 2016 Technical Preview.

## <a name="see-also"></a>Consulta también
-  [Virtualization Blog](https://techcommunity.microsoft.com/t5/Virtualization/bg-p/Virtualization) (Blog de virtualización): obtenga información sobre las características de la versión preliminar que están disponibles y cómo probarlas.
- [Supported virtual machine configuration versions](https://docs.microsoft.com/windows-server/virtualization/hyper-v/deploy/Upgrade-virtual-machine-version-in-Hyper-V-on-Windows-or-Windows-Server#BKMK_SupportedConfigVersions) (Versiones de configuración de máquina virtual admitidas): obtenga información sobre cómo comprobar la versión de configuración de la máquina virtual y qué versiones admite Microsoft.
