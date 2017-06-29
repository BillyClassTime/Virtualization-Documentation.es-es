---
title: "Solución de problemas de Hyper-V en Windows 10"
description: "Solución de problemas de Hyper-V en Windows 10"
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: f0ec8eb4-ffc4-4bf1-9a19-7a8c3975b359
ms.openlocfilehash: b4426a1a28fe3636e1a161d9ca8d09e8d1415080
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: es-ES
---
# <a name="troubleshoot-hyper-v-on-windows-10"></a>Solución de problemas de Hyper-V en Windows 10

## <a name="i-updated-to-windows-10-and-now-i-cant-connect-to-my-downlevel-windows-81-or-server-2012-r2-host"></a>Actualicé a Windows 10 y ahora no puedo conectarme a mi host con una versión anterior (Windows 8.1 o Server 2012 R2)
En Windows 10, el Administrador de Hyper-V se movió a WinRM para la administración remota.  Lo que eso implica ahora es que la administración remota debe estar habilitada en el host remoto para poder utilizar el Administrador de Hyper-V para administrarlo.

Para más información, consulte [Administrar hosts remotos de Hyper-V con el Administrador de Hyper-V](https://technet.microsoft.com/windows-server-docs/compute/hyper-v/manage/Remotely-manage-Hyper-V-hosts).

## <a name="i-changed-the-checkpoint-type-but-it-is-still-taking-the-wrong-type-of-checkpoint"></a>Cambié el tipo de punto de control, pero sigue teniendo un tipo de punto de control incorrecto
Si toma el punto de control de VMConnect y cambia el tipo de punto de control en el Administrador de Hyper-V, el punto de control que se toma será el tipo de punto de control que se especificó cuando se abrió VMConnect.

Cierre VMConnect y vuelva a abrirlo para que tenga el tipo de punto de control correcto.

## <a name="when-i-try-to-create-a-virtual-hard-disk-on-a-flash-drive-an-error-message-is-displayed"></a>Cuando intento crear un disco duro virtual en una unidad flash, se muestra un mensaje de error
Hyper-V no admite unidades de disco con formato FAT o FAT32, ya que estos sistemas de archivos no ofrecen listas de control de acceso (ACL) y no admiten archivos mayores de 4GB. Los discos con formato ExFAT solo ofrecen funcionalidad de ACL limitada y, por tanto, tampoco se admiten por razones de seguridad.
El mensaje de error que se muestra en PowerShell es "El sistema no pudo crear '\[path to VHD\]': no se puede completar la operación solicitada por una limitación del sistema de archivos (0x80070299).".

Use una unidad con formato NTFS en su lugar. 

## <a name="i-get-this-message-when-i-try-to-install-hyper-v-cannot-be-installed-the-processor-does-not-support-second-level-address-translation-slat"></a>Obtengo este mensaje cuando intento la instalación: "No se puede instalar Hyper-V: el procesador no admite la traducción de direcciones de segundo nivel (SLAT).".
Hyper-V requiere SLAT para ejecutar máquinas virtuales. Si el equipo no admite SLAT, no puede ser un host de máquinas virtuales.

Si solo está intentando instalar las herramientas de administración, anule la selección de **Plataforma de Hyper-V** en **Programas y características** > **Activar o desactivar las características de Windows**.
