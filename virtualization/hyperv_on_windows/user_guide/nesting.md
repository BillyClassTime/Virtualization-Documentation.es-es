---
title: Virtualización anidada
description: Virtualización anidada
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
---

# Virtualización anidada

La virtualización anidada proporciona la capacidad de ejecutar hosts de Hyper-V en un entorno virtualizado. En otras palabras: un host de Hyper-V puede virtualizarse con la virtualización anidada. Algunos casos de uso para la virtualización anidada podrían ser la ejecución de un laboratorio de Hyper-V en un entorno virtualizado, los servicios de virtualización que se proporcionan a otros usuarios sin la necesidad de hardware individual y la tecnología de contenedor de Windows que confía en la virtualización anidada al ejecutar contenedores de Hyper-V en un host contenedor virtualizado. En este documento se detallan los requisitos previos de hardware y software, los pasos de configuración y la información para solucionar errores.

> La virtualización anidada está en fase preliminar y no se debería usar en entornos de producción.

## Requisitos previos

- Versión de Windows Insider (Windows Server 2016, Nano Server o Windows 10) que ejecute la versión 10565 o una versión posterior.
- Los dos hipervisores (principal y secundario) deben ejecutar la misma versión de Windows (10565 o una posterior).
- Mínimo de 4 GB de RAM disponible.
- Un procesador Intel con tecnología VT-x de Intel.

## Configurar la virtualización anidada

Primero cree una máquina virtual que ejecute la misma versión que el host; **no apague la máquina virtual**. Para obtener más información, consulte [Crear una máquina virtual](../quick_start/walkthrough_create_vm.md).

Una vez se haya creado la máquina virtual, ejecute el siguiente comando en el hipervisor principal, que habilitará la virtualización anidada en la máquina virtual.

```none
Set-VMProcessor -VMName <virtual machine> -ExposeVirtualizationExtensions $true -Count 2
```

Al ejecutar un host de Hyper-V anidado, debe deshabilitarse la memoria dinámica en la máquina virtual. Esto puede configurarse en las propiedades de la máquina virtual o usando el siguiente comando de PowerShell.

```none
Set-VMMemory <virtual machine> -DynamicMemoryEnabled $false
```

Para que las máquinas virtuales anidadas reciban direcciones IP, debe habilitarse la suplantación de dirección MAC. Esto se completa con el siguiente comando de PowerShell.

```none
Get-VMNetworkAdapter -VMName <virtual machine> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

Cuando haya completado estos pasos, se podrá iniciar una máquina virtual y se podrá instalar Hyper-V. Para obtener más información sobre la instalación de Hyper-V, consulte [Instalar Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install).

## Script de configuración

También puede usar el siguiente script para habilitar y configurar la virtualización anidada. Con los siguientes comandos, se descargará y ejecutará el script.
  
```none
# download script
Invoke-WebRequest https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Enable-NestedVm.ps1 -OutFile .\Enable-NestedVm.ps1 

# run script
.\Enable-NestedVm.ps1 -VmName "DemoVM"
```

## Problemas conocidos

- Los hosts con Device Guard habilitado no pueden exponer las extensiones de virtualización a los invitados.
- Los hosts con seguridad basada en la virtualización (VBS) habilitada no pueden exponer las extensiones de virtualización a los invitados. Primero debe deshabilitar VBS para usar la virtualización anidada.
- Es posible que se pierda la conexión a la máquina virtual si usa una contraseña en blanco. Cambie la contraseña; el problema debería resolverse.
- Cuando se habilite la virtualización anidada en una máquina virtual, las siguientes características ya no serán compatibles con esa máquina virtual.  
  * Cambio de tamaño de memoria en tiempo de ejecución.
  * Aplicación de un punto de control a una máquina virtual en ejecución.
  * Una máquina virtual que hospeda otras máquinas virtuales no se puede migrar mientras está activa.
  * No podrá guardar ni restaurar el contenido.

## Preguntas más frecuentes y solución de problemas

Mi máquina virtual no se inicia, ¿qué debo hacer?

1. Asegúrese de que la memoria dinámica esté desactivada.
2. Ejecute [este script de PowerShell](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1) en el equipo host desde un símbolo de sistema con privilegios elevados.
  
## Comentarios

Informe de otros problemas a través de la aplicación de comentarios de Windows, los [foros de virtualización](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv) o a través de [GitHub](https://github.com/Microsoft/Virtualization-Documentation).



<!--HONumber=Jun16_HO2-->


