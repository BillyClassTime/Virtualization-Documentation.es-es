---
title: Crear una máquina virtual con Hyper-V
description: Crear una máquina virtual con Hyper-V en Windows 10
keywords: windows 10, hyper-v
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 66723f33-b12c-49d1-82cf-71ba9d6087e9
ms.openlocfilehash: 94ac197f5bc660e52d215fa132eae78f521e1c30
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74911655"
---
# <a name="create-virtual-machine-with-hyper-v-on-windows-10"></a>Crear una máquina virtual con Hyper-V en Windows 10

Aprende a crear una máquina virtual y a instalar un sistema operativo en la nueva máquina virtual.  Necesitarás un archivo .iso para el sistema operativo que quieras ejecutar. Si es necesario, consigue una copia de evaluación de Windows 10 en el [TechNet Evaluation Center](http://www.microsoft.com/evalcenter/).

## <a name="create-a-virtual-machine-with-hyper-v-manager"></a>Crear una máquina virtual con el Administrador de Hyper-V

1. Para abrir el administrador de Hyper-V, presiona la tecla de Windows y escribe "Administrador de Hyper-V" o busca **Administrador de Hyper-V** en tus aplicaciones.

1. En el Administrador de Hyper-V, haz clic en **Acción** > **Nueva** > **Máquina virtual** para que se abra el Asistente para nueva máquina virtual.

1. Revise el contenido de "antes de empezar2 y haga clic en **Siguiente**.

1. Asigne un nombre a la máquina virtual.
  > **Nota**: Este es el nombre que usa Hyper-V para la máquina virtual, no el nombre de equipo asignado al sistema operativo invitado que se implementará en la máquina virtual.

1. Seleccione la ubicación en la que se almacenarán los archivos de la máquina virtual, por ejemplo **c:\virtualmachine**. También puede aceptar la ubicación predeterminada. Haga clic en **Siguiente** cuando haya terminado.

  ![](media/new_vm_upd.png)

1. Seleccione una generación para el equipo y haga clic en **Siguiente**.  

  Las máquinas virtuales de generación 2 se introdujeron con Windows Server 2012 R2 y ofrecen un modelo de hardware virtual simplificado y algunas funciones adicionales. Solo se puede instalar un sistema operativo de 64 bits en una máquina virtual de generación 2. Para más información sobre las máquinas virtuales de generación 2, consulte [Información general acerca de las máquinas virtuales de generación 2](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn282285(v=ws.11)>).
  
  > Si la nueva máquina virtual está configurada como generación 2 y va a ejecutar una distribución de Linux, tendrá que deshabilitar el arranque seguro. Para más información, consulte [Arranque seguro](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-8.1-and-8/dn486875(v=ws.11)>).

2. Seleccione **2048** MB como valor de **Memoria de inicio** y deje seleccionada la opción **Usar la memoria dinámica**. Haga clic en el botón **Siguiente**.

  La memoria se comparte entre un host de Hyper-V y la máquina virtual que se ejecuta en el host. El número de máquinas virtuales que se puede ejecutar en un único host depende en parte de la memoria disponible. Una máquina virtual también se puede configurar para utilizar memoria dinámica. Cuando está habilitada, la memoria dinámica reclama la memoria no utilizada de la máquina virtual en ejecución. Esto permite que varias máquinas virtuales se ejecuten en el host. Para más información sobre la memoria dinámica, consulte [Introducción a la memoria dinámica de Hyper-V](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831766(v=ws.11)).

3. En el Asistente para configurar funciones de red, seleccione un conmutador virtual para la máquina virtual y haga clic en **Siguiente**. Para más información, consulte [Crear un conmutador virtual](connect-to-network.md).

4. Asigne un nombre a la unidad de disco duro virtual, seleccione una ubicación o mantenga el valor predeterminado y, por último, especifique un tamaño. Haga clic en **Siguiente** cuando esté listo.

  Una unidad de disco duro virtual ofrece almacenamiento para una máquina virtual, de forma similar a una unidad de disco duro físico. Una unidad de disco duro virtual es necesaria para que pueda instalar un sistema operativo en la máquina virtual.
  
  ![](media/new_vhd_upd.png)

1. En el Asistente de opciones de instalación, seleccione **Instalar un sistema operativo desde un archivo de imagen de arranque** y luego seleccione un archivo .iso de sistema operativo. Haga clic en **Siguiente** cuando se haya completado.

  Al crear una máquina virtual, puede configurar algunas opciones de instalación del sistema operativo. Las tres opciones disponibles son:

  * **Instalar un sistema operativo más tarde**: esta opción no realiza ninguna modificación adicional a la máquina virtual.

  * **Instalar un sistema operativo desde un archivo de imagen de arranque**: esto es similar a insertar un CD en la unidad de CD-ROM física de un equipo. Para configurar esta opción, seleccione una imagen .iso. Esta imagen se montará en la unidad de CD-ROM virtual de la máquina virtual. El orden de arranque de la máquina virtual cambia para arrancar primero desde la unidad de CD-ROM.

  * **Instalar un sistema operativo desde un servidor de instalación en red**: esta opción no está disponible a menos que haya conectado la máquina virtual a un conmutador de red. En esta configuración, la máquina virtual intenta arrancar desde la red.

1. Revise los detalles de la máquina virtual y haga clic en **Finalizar** para completar la creación de la máquina virtual.

## <a name="create-a-virtual-machine-with-powershell"></a>Crear una máquina virtual con PowerShell

1. Abra PowerShell ISE como administrador.

2. Ejecute el script siguiente:

  ``` powershell
  # Set VM Name, Switch Name, and Installation Media Path.
  $VMName = 'TESTVM'
  $Switch = 'External VM Switch'
  $InstallMedia = 'C:\Users\Administrator\Desktop\en_windows_10_enterprise_x64_dvd_6851151.iso'

  # Create New Virtual Machine
  New-VM -Name $VMName -MemoryStartupBytes 2147483648 -Generation 2 -NewVHDPath "D:\Virtual Machines\$VMName\$VMName.vhdx" -NewVHDSizeBytes 53687091200 -Path "D:\Virtual Machines\$VMName" -SwitchName $Switch

  # Add DVD Drive to Virtual Machine
  Add-VMScsiController -VMName $VMName
  Add-VMDvdDrive -VMName $VMName -ControllerNumber 1 -ControllerLocation 0 -Path $InstallMedia

  # Mount Installation Media
  $DVDDrive = Get-VMDvdDrive -VMName $VMName

  # Configure Virtual Machine to Boot from DVD
  Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive
  ```

## <a name="complete-the-operating-system-deployment"></a>Completar la implementación del sistema operativo

Para terminar de crear la máquina virtual, debe iniciar la máquina virtual y completar la instalación del sistema operativo.

1. En el administrador de Hyper-V, haga doble clic en la máquina virtual. Esto inicia la herramienta VMConnect.

2. En VMConnect, haga clic en el botón verde de Inicio. Esto es similar a presionar el botón de encendido de un equipo físico. Es posible que se te indique "Presione cualquier tecla para arrancar desde CD o DVD". Hágalo.

  > **Nota**: Puede que necesite hacer clic en la ventana de VMConnect para garantizar que se envían las pulsaciones de tecla a la máquina virtual.

3. La máquina virtual se inicia en el programa de instalación y puede realizar la instalación como lo haría en un equipo físico.

  ![](media/OSDeploy_upd.png) 

  > **Nota**: A menos que esté ejecutando una versión con licencia por volumen de Windows, necesitarás una licencia independiente para la versión de Windows que se ejecute en una máquina virtual. El sistema operativo de la máquina virtual es independiente del sistema operativo del host.
