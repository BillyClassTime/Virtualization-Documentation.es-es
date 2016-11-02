---
title: "Exportar e importar máquinas virtuales"
description: "Exportar e importar máquinas virtuales"
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 7fd996f5-1ea9-4b16-9776-85fb39a3aa34
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 620a2c38514488100bae4eb71d0683e3994801b4

---

# Exportar e importar máquinas virtuales

Puede usar la funcionalidad de exportación e importación de Hyper-V para duplicar rápidamente máquinas virtuales.  Las máquinas virtuales exportadas puede usarse como copias de seguridad o como una manera de mover una máquina virtual entre hosts de Hyper-V.  

La importación le permite restaurar máquinas virtuales.  No es necesario exportar una máquina virtual para poder importarla. La importación intentará volver a crear la máquina virtual a partir de cualquier cosa disponible.  Use el Asistente para **importar máquinas virtuales** para especificar la ubicación de los archivos. Esto registra la máquina virtual con Hyper-V y hace que esté disponible para su uso.
 
Este documento le guía por los procesos de exportación e importación de una máquina virtual y algunas de las opciones que puede elegir al realizar estas tareas.

## Exportar una máquina virtual

### Uso del Administrador de Hyper-V

Al crear una exportación de una máquina virtual, todos los archivos asociados se incluyen en la exportación. Esto incluye los archivos de configuración, los archivos de la unidad de disco duro y también los archivos de puntos de control existentes. Para crear una exportación de máquina virtual:

1. En el Administrador de Hyper-V, haga clic con el botón derecho en la máquina virtual que quiera y seleccione **Exportar**.

2. Especifique la ubicación donde se almacenarán los archivos exportados y haga clic en el botón **Exportar**.

**Nota:** Este proceso se puede ejecutar en una máquina virtual que esté en un estado iniciado o detenido.

Cuando se haya completado la exportación, podrá ver todos los archivos exportados en la ubicación de exportación.

### Con PowerShell

Para exportar una máquina virtual con PowerShell, use el comando **Export-VM**. 

```powershell
Export-VM -Name <vm name> -Path <path>
```

Para obtener información sobre el uso de Windows PowerShell para exportar máquinas virtuales, vea [Export-VM](https://technet.microsoft.com/library/hh848491.aspx).

## Importar una máquina virtual. 

Al importar una máquina virtual, esta se registra con el host de Hyper-V. Es posible volver a importar la exportación de una máquina virtual en el host de la que se derivó o en un host nuevo. 

Hyper-V incluye tres tipos de importación:

- **Registro en contexto**: los archivos de exportación se colocaron en la ubicación desde donde se debería ejecutar la máquina virtual. Cuando se importa, la máquina virtual tiene el mismo identificador que tenía en el momento de la exportación. Por este motivo, si la máquina virtual ya está registrada con Hyper-V, debe eliminarse antes de que funcione la importación. Cuando finalice la importación, los archivos de exportación se convierten en los archivos de estado en ejecución y no se puede quitar.

- **Restaurar la máquina virtual**: tiene la opción de almacenar los archivos de la máquina virtual en una ubicación concreta o usar el valor predeterminado de las ubicaciones en Hyper-V. Este tipo de importación crea una copia del archivo exportado y la mueve a la ubicación seleccionada. Cuando se importa, la máquina virtual tiene el mismo identificador que tenía en el momento de la exportación. Por este motivo, si la máquina virtual ya se está ejecutando en Hyper-V, debe eliminarse antes de que se pueda completar la importación. Cuando la importación se haya completado, los archivos exportados permanecerán intactos y se podrán quitar o volver a importar.

- **Copiar la máquina virtual**: este tipo de importación es similar al tipo de restauración, en el sentido de que selecciona una ubicación para los archivos de la máquina virtual. La diferencia es que cuando se importa, la máquina virtual tiene un nuevo identificador único. Esto permite que la máquina virtual se importe en el mismo host varias veces.


### Uso del Administrador de Hyper-V

Para importar una máquina virtual en un host de Hyper-V:

1. En el Administrador de Hyper-V, haga clic en **Importar máquina virtual** en el menú de acciones.

2. Haga clic en **Siguiente** en la pantalla Antes de comenzar.

3. Seleccione la carpeta que contiene los archivos exportados y haga clic en **Siguiente**.

4. Seleccione la máquina virtual que quiere importar, aunque probablemente solo habrá una opción.

5. Elija un tipo de importación de los tres posibles y haga clic en siguiente. 

6. Seleccione **Finalizar** en la pantalla de resumen.

El Asistente para importar máquinas virtuales también le guía por los pasos necesarios para abordar las incompatibilidades al importar la máquina virtual en el nuevo host, por lo que este asistente puede ayudarle con la configuración que esté asociada al hardware físico, como la memoria, los conmutadores virtuales y los procesadores virtuales.

Para importar una máquina virtual, el asistente hace lo siguiente:  
1. Crea una copia del archivo de configuración de la máquina virtual. Esto sirve como medida de precaución en el caso de que se produjese un reinicio inesperado en el host, como el debido a una pérdida de energía.  

2. Valida el hardware. La información del archivo de configuración de la máquina virtual se compara con el hardware del nuevo host.

3. Compila una lista de errores. Esta lista identifica lo que se necesita volver a configurar y determina que páginas aparecen después en el asistente.

4. Muestra las páginas relevantes, una categoría a la vez. El asistente explica cada incompatibilidad para ayudarle a volver a configurar la máquina virtual a fin de que sea compatible con el nuevo host.

5. Quita la copia del archivo de configuración. Cuando el asistente hace esto, la máquina virtual está lista para iniciarse.


### Con PowerShell

Para importar una máquina virtual con PowerShell, use el comando **Import-VM**.  Los comandos siguientes muestran una importación de cada uno de los tres tipos de importación mediante PowerShell.

Para completar una importación en contexto de una máquina virtual, el comando tendría un aspecto similar al siguiente. Recuerde que una importación en contexto utiliza los archivos donde se almacenan en el momento de la importación y conserva el identificador de las máquinas virtuales.

```powershell
Import-VM -Path 'C:\<vm export path>\2B91FEB3-F1E0-4FFF-B8BE-29CED892A95A.vmcx' 
```

Para importar la máquina virtual especificando su propia ruta de acceso a los archivos de la máquina virtual, el comando sería similar al siguiente.

```powershell
Import-VM -Path ‘C:\<vm export path>\2B91FEB3-F1E0-4FFF-B8BE-29CED892A95A.vmcx' -Copy -VhdDestinationPath 'D:\Virtual Machines\WIN10DOC' -VirtualMachinePath 'D:\Virtual Machines\WIN10DOC'
```

Para completar una importación de copia y mover los archivos de la máquina virtual a la ubicación predeterminada de Hyper-V, el comando sería similar al siguiente.

``` PowerShell
Import-VM -Path 'C:\<vm export path>\2B91FEB3-F1E0-4FFF-B8BE-29CED892A95A.vmcx' -Copy -GenerateNewId
```

Para obtener más información, vea [Import-VM](https://technet.microsoft.com/library/hh848495.aspx).



<!--HONumber=Oct16_HO4-->


