---
title: Habilitar Hyper-V en Windows 10
description: Instalar Hyper-V en Windows 10
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 02/15/2019
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 752dc760-a33c-41bb-902c-3bb2ecd9ac86
ms.openlocfilehash: e1b6b55b2e17ac4f0883078748d75f6d4b9fcafa
ms.sourcegitcommit: 3c06d6b5cebeacc4c0185dca5c20680ad67655bd
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681906"
---
# <a name="install-hyper-v-on-windows-10"></a>Instalar Hyper-V en Windows 10

Habilitar Hyper-V para crear máquinas virtuales en Windows 10.  
Hyper-V se puede habilitar de varias maneras, entre las que se incluyen el panel de control de Windows 10, PowerShell o el uso de la herramienta de administración y mantenimiento de imágenes de implementación (DISM). Este documento explica paso a paso cada una de las opciones.

> **Nota:**  Hyper-V está integrado en Windows como función opcional: no existen descargas de Hyper-V.

## <a name="check-requirements"></a>Comprobación de los requisitos

* Windows 10 Enterprise, Pro o Education
* Procesador de 64 bits con traducción de direcciones de segundo nivel (SLAT).
* Compatibilidad de CPU para la extensión de modo de monitor de VM (VT-c en CPU Intel).
* Mínimo de 4 GB de memoria.

El rol de Hyper-V **no** se puede instalar en Windows 10 Home.

Actualice de Windows 10 Home Edition a Windows 10 Pro abriendo la actualización de la **configuración** > **y** > la**activación**de seguridad.

Para más información y solución de problemas, consulta [Requisitos de sistema de Hyper-V en Windows 10](../reference/hyper-v-requirements.md).

## <a name="enable-hyper-v-using-powershell"></a>Habilitar Hyper-V usando PowerShell

1. Abre una consola de PowerShell como administrador.

2. Ejecuta el siguiente comando:

  ```powershell
  Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
  ```

  Si no fue posible encontrar el comando, asegúrate de que estás ejecutando PowerShell como administrador.

Cuando la instalación haya finalizado, reinicia.

## <a name="enable-hyper-v-with-cmd-and-dism"></a>Habilitar Hyper-V con CMD y DISM

La herramienta Administración y mantenimiento de imágenes de implementación (DISM) ayuda a configurar Windows y las imágenes de Windows.  Entre sus muchas aplicaciones, DISM puede habilitar características de Windows mientras se ejecuta el sistema operativo.

Para habilitar el rol de Hyper-V mediante DISM:

1. Abra una sesión de PowerShell o CMD como administrador.

1. Escribe el comando siguiente:

  ```powershell
  DISM /Online /Enable-Feature /All /FeatureName:Microsoft-Hyper-V
  ```

  ![Ventana de la consola que muestra a Hyper-V habilitándose.](media/dism_upd.png)

Para más información sobre DISM, consulta [Referencia técnica de DISM](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-8.1-and-8/hh824821(v=win.10)>).

## <a name="enable-the-hyper-v-role-through-settings"></a>Habilitar el rol de Hyper-V en Configuración

1. Haz clic con el botón derecho en el botón Windows y selecciona ‘Aplicaciones y funciones’.

2. En la sección configuración relacionada **, seleccione programas y características** en la parte derecha. 

3. Selecciona **Activar o desactivar las características de Windows**.

4. Selecciona **Hyper-V** y haz clic en **Aceptar**.

![Cuadro de diálogo de programas y funciones de Windows](media/enable_role_upd.png)

Cuando la instalación se complete, se te pedirá confirmación para reiniciar el equipo.

## <a name="make-virtual-machines"></a>Crear máquinas virtuales

[Crea tu primera máquina virtual](quick-create-virtual-machine.md)
