---
title: "Solución de problemas de Hyper-V en Windows 10"
description: "Solución de problemas de Hyper-V en Windows 10"
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: f0ec8eb4-ffc4-4bf1-9a19-7a8c3975b359
translationtype: Human Translation
ms.sourcegitcommit: e14ede0a2b13de08cea0a955b37a21a150fb88cf
ms.openlocfilehash: 97ae940c03df2ab18350a31c56c93da85a348110

---

# Solución de problemas de Hyper-V en Windows 10

## Actualicé a Windows 10 y ahora no puedo conectarme a mi host con una versión anterior (Windows 8.1 o Server 2012 R2)
En Windows 10, el Administrador de Hyper-V se movió a WinRM para la administración remota.  Lo que eso implica ahora es que la administración remota debe estar habilitada en el host remoto para poder utilizar el Administrador de Hyper-V para administrarlo.

Para más información, consulte [Administrar hosts remotos de Hyper-V con el Administrador de Hyper-V](remote_host_management.md).

## Cambié el tipo de punto de control, pero sigue teniendo un tipo de punto de control incorrecto
Si toma el punto de control de VMConnect y cambia el tipo de punto de control en el Administrador de Hyper-V, el punto de control que se toma será el tipo de punto de control que se especificó cuando se abrió VMConnect.

Cierre VMConnect y vuelva a abrirlo para que tenga el tipo de punto de control correcto.

## Cuando intento crear un disco duro virtual en una unidad flash, se muestra un mensaje de error
Hyper-V no admite unidades de disco con formato FAT o FAT32, ya que estos sistemas de archivos no ofrecen listas de control de acceso (ACL) y no admiten archivos mayores de 4 GB. Los discos con formato ExFAT solo ofrecen funcionalidad de ACL limitada y, por tanto, tampoco se admiten por razones de seguridad.
El mensaje de error que se muestra en PowerShell es "El sistema no pudo crear '\[path to VHD\]': no se puede completar la operación solicitada por una limitación del sistema de archivos (0x80070299).".

Use una unidad con formato NTFS en su lugar. 

## Obtengo este mensaje cuando intento la instalación: "No se puede instalar Hyper-V: el procesador no admite la traducción de direcciones de segundo nivel (SLAT).".
Hyper-V requiere SLAT para ejecutar máquinas virtuales. Si el equipo no admite SLAT, no puede ser un host de máquinas virtuales.

Si solo está intentando instalar las herramientas de administración, anule la selección de **Plataforma de Hyper-V** en **Programas y características** > **Activar o desactivar las características de Windows**.



<!--HONumber=Jun16_HO4-->


