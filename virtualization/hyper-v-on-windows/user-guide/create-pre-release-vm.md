---
title: "Pruebe las características de la versión preliminar para Hyper-V"
description: "Pruebe las características de la versión preliminar para Hyper-V"
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 426c87cc-fa50-4b8d-934e-0b653d7dea7d
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: ea91ea0ffca5479cb0593ef9961625f7b7ab1f42

---

# Pruebe las características de la versión preliminar para Hyper-V

> Esto es contenido preliminar y está sujeto a cambios.  
  Las máquinas virtuales de versión preliminar están diseñadas únicamente para entornos de desarrollo o de prueba, ya que Microsoft no las admite.

Obtenga acceso anticipado a las características de la versión preliminar para Hyper-V en Windows Server 2016 Technical Preview para probarlas en el entorno de desarrollo o de prueba. Puede ser el primero en ver las últimas características de Hyper-V y ayudar a darle forma al producto con sus comentarios anticipados.

Las máquinas virtuales que se crean como versión preliminar no tienen compatibilidad de una compilación a otra ni soporte técnico en el futuro.  No las use en un entorno de producción.

Estas son algunas de las razones por las que solo están destinadas a entornos que no sean de producción:

* No existe compatibilidad con versiones posteriores para las máquinas virtuales de versión preliminar. No se puede actualizar estas máquinas virtuales a una nueva versión de configuración.
* Las máquinas virtuales de versión preliminar no tienen una definición coherente entre compilaciones. Si actualiza el sistema operativo host, las máquinas virtuales de versión preliminar existentes podrían ser incompatibles con el host. Estas máquinas virtuales podrían no iniciarse, o en un principio podría parecer que funcionan pero más adelante presentar graves problemas de compatibilidad.
* Si importa una máquina virtual de versión preliminar en un host con una compilación diferente, los resultados son imprevisibles. Puede mover una máquina virtual de versión preliminar a otro host. Pero se espera que este escenario funcione solamente si ambos hosts ejecutan la misma compilación.

## Creación de una máquina virtual de versión preliminar

Puede crear una máquina virtual de versión preliminar en hosts de Hyper-V que ejecutan Windows Server 2016 Technical Preview.

1. En el Escritorio de Windows, haga clic en el botón Inicio y escriba cualquier parte del nombre **Windows PowerShell**.
2. Haga clic con el botón derecho en **Windows PowerShell** y seleccione **Ejecutar como administrador**.
3. Use el cmdlet [New-VM](https://technet.microsoft.com/library/hh848537.aspx) con la marca -Prerelease para crear la máquina virtual de versión preliminar. Por ejemplo, ejecute el comando siguiente, donde VM Name es el nombre de la máquina virtual que quiere crear.

``` PowerShell
New-VM -Name <VM Name> -Prerelease
```
Otros ejemplos con los que puede usar la marca -Prerelease:
 - Para crear una máquina virtual que use un disco duro virtual existente o un disco duro nuevo, vea los ejemplos de PowerShell de [Create a virtual machine in Hyper-V on Windows Server 2016 Technical Preview](https://technet.microsoft.com/library/mt126140.aspx#BKMK_PowerShell) (Crear una máquina virtual de Hyper-V en Windows Server 2016 Technical Preview).
 - Para crear un disco duro virtual nuevo que se inicie con una imagen de sistema operativo, vea el ejemplo de PowerShell de [Implementar una máquina virtual de Windows en Hyper-V en Windows 10](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_create_vm).

 Los ejemplos descritos en estos artículos funcionan con los hosts de Hyper-V que ejecutan Windows 10 o Windows Server 2016 Technical Preview. Pero, por el momento, solo puede usar la marca -Prerelease para crear una máquina virtual de versión preliminar en hosts de Hyper-V que ejecutan Windows Server 2016 Technical Preview.

## Consulte también
-  [Virtualization Blog](https://blogs.technet.microsoft.com/virtualization/) (Blog de virtualización): obtenga información sobre las características de la versión preliminar que están disponibles y cómo probarlas.
- [Supported virtual machine configuration versions](https://technet.microsoft.com/library/mt695898.aspx#BKMK_SupportedConfigVersions) (Versiones de configuración de máquina virtual admitidas): obtenga información sobre cómo comprobar la versión de configuración de la máquina virtual y qué versiones admite Microsoft.



<!--HONumber=Oct16_HO4-->


