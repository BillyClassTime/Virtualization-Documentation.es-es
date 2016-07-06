---
title: Trabajar con Hyper-V y Windows PowerShell
description: Trabajar con Hyper-V y Windows PowerShell
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6d1ae036-0841-4ba5-b7e0-733aad31e9a7
translationtype: Human Translation
ms.sourcegitcommit: e14ede0a2b13de08cea0a955b37a21a150fb88cf
ms.openlocfilehash: a8e567b6447aa73f14825b7054d977d2b003a726

---

# Trabajar con Hyper-V y Windows PowerShell

Ahora que ya ha visto los conceptos básicos de implementación de Hyper-V, la creación de máquinas virtuales y la administración de dichas máquinas virtuales, es hora de explorar cómo automatizar muchas de estas actividades con PowerShell.

### Devolver una lista de comandos de Hyper-V

1.  Haga clic en el botón de inicio de Windows y escriba **PowerShell**.
2.  Ejecute el siguiente comando para mostrar una lista de comandos de PowerShell que permite búsqueda, que están disponibles con el módulo de PowerShell de Hyper-V.

 ```powershell
get-command -module hyper-v | out-gridview
```
  Obtendrá algo parecido a esto:

  ![](media\command_grid.png)

3. Para más información sobre un comando de PowerShell determinado, use `get-help`. Por ejemplo, al ejecutar el comando siguiente se devuelve información sobre el comando `get-vm` de Hyper-V.

  ```powershell
get-help get-vm
```
 El resultado muestra cómo estructurar el comando, cuáles son los parámetros obligatorios y opcionales y los alias que puede utilizar.

 ![](media\get_help.png)


### Devolver una lista de máquinas virtuales

Use el comando `get-vm` para devolver una lista de máquinas virtuales.

1. En PowerShell, ejecute el siguiente comando:
 
 ```powershell
get-vm
```
 Se muestra algo parecido a esto:

 ![](media\get_vm.png)

2. Para devolver una lista que solo incluya las máquinas virtuales encendidas, agregue un filtro al comando `get-vm`. Puede agregar un filtro si usa el comando where-object. Para más información sobre el filtrado, consulte la documentación de [Uso de Where-Object](https://technet.microsoft.com/en-us/library/ee177028.aspx).   

 ```powershell
 get-vm | where {$_.State -eq ‘Running’}
 ```
3.  Para enumerar todas las máquinas virtuales que se encuentran en un estado apagado, ejecute el siguiente comando. Este comando es una copia del comando del paso 2 con el filtro cambiado de "Running" a "Off".

 ```powershell
 get-vm | where {$_.State -eq ‘Off’}
 ```

### Iniciar y apagar las máquinas virtuales

1. Para iniciar una máquina virtual determinada, ejecute el comando siguiente con el nombre de la máquina virtual:

 ```powershell
 Start-vm -Name <virtual machine name>
 ```

2. Para iniciar todas las máquinas virtuales actualmente apagadas, obtenga una lista de esas máquinas y canalice la lista al comando "start-vm":

  ```powershell
 get-vm | where {$_.State -eq ‘Off’} | start-vm
 ```
3. Para cerrar todas las máquinas virtuales en ejecución, ejecute lo siguiente:
 
  ```powershell
 get-vm | where {$_.State -eq ‘Running’} | stop-vm
 ```

### Crear un punto de control de máquina virtual

Para crear un punto de control con PowerShell, seleccione la máquina virtual con el comando `get-vm` y canalícela al comando `checkpoint-vm`. Finalmente, asigne un nombre al punto de control con `-snapshotname`. El comando completo tendrá el siguiente aspecto:

 ```powershell
 get-vm -Name <VM Name> | checkpoint-vm -snapshotname <name for snapshot>
 ```
### Crear una máquina virtual nueva

En el ejemplo siguiente se muestra cómo crear una nueva máquina virtual en el Entorno de scripting integrado de PowerShell (ISE). Esto es un ejemplo sencillo que podría ampliarse para incluir características adicionales de PowerShell e implementaciones de máquinas virtuales más avanzadas.

1. Para abrir PowerShell ISE, haga clic en Inicio y escriba **PowerShell ISE**.
2. Ejecute el código siguiente para crear una máquina virtual. Consulte la documentación sobre [New-VM](https://technet.microsoft.com/en-us/library/hh848537.aspx) para obtener información detallada sobre el comando New-VM.

  ```powershell
 $VMName = "VMNAME"

 $VM = @{
     Name = $VMName 
     MemoryStartupBytes = 2147483648
     Generation = 2
     NewVHDPath = "C:\Virtual Machines\$VMName\$VMName.vhdx"
     NewVHDSizeBytes = 53687091200
     BootDevice = "VHD"
     Path = "C:\Virtual Machines\$VMName "
     SwitchName = (get-vmswitch).Name[0]
 }

 New-VM @VM
  ```

## Resumen y referencias

En este documento se han mostrado algunos pasos sencillos para explorar el módulo de PowerShell de Hyper-V, así como algunos escenarios de ejemplo. Para más información sobre el módulo de PowerShell de Hyper-V, consulte la [referencia de cmdlets de Hyper-V en Windows PowerShell](https://technet.microsoft.com/%5Clibrary/Hh848559.aspx).  
 


<!--HONumber=Jun16_HO4-->


