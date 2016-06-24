---
title: &1067729278 Requisitos de sistema de Hyper-V en Windows 10
description: Requisitos de sistema de Hyper-V en Windows 10
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &319284841 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6e5e6b01-7a9d-4123-8cc7-f986e10cd372
---

# Requisitos de sistema de Hyper-V en Windows 10

Hyper-V en Windows 10 solo funciona en un conjunto específico de configuraciones de hardware y sistema operativo. En este documento se indican los requisitos de Hyper-V y cómo se puede comprobar si el sistema es compatible.

## Requisitos del sistema operativo

El rol de Hyper-V puede habilitarse en estas versiones de Windows 10:

- Windows 10 Enterprise
- Windows 10 Professional
- Windows 10 Education

El rol de Hyper-V **no** se puede instalar en:

- Windows 10 Home
- Windows 10 Mobile
- Windows 10 Mobile Enterprise

>10 de Windows Home edition puede actualizarse a Windows 10 Professional. Para hacerlo, abra **Configuración** > **Actualización y seguridad** > **Activación**. Aquí puede visitar la tienda y comprar una actualización.

## Requisitos de hardware

Aunque este documento no ofrece una lista completa del hardware compatible con Hyper-V, son necesarios los siguientes elementos:

- Procesador de 64 bits con traducción de direcciones de segundo nivel (SLAT).
- Compatibilidad de CPU con la extensión del modo monitor de la máquina virtual (VT-c en CPU de Intel).
- Mínimo de 4 GB de memoria. Como las máquinas virtuales comparten memoria con el host de Hyper-V, debe proporcionar memoria suficiente para administrar la carga de trabajo virtual prevista.

Los siguientes elementos se deben habilitar en la BIOS del sistema:
- Tecnología de virtualización (puede tener un nombre diferente según el fabricante de la placa base).
- Prevención de ejecución de datos aplicada por hardware.

## Comprobar la compatibilidad de hardware

Para comprobar la compatibilidad, abra PowerShell o un símbolo del sistema (cmd.exe) y escriba **systeminfo.exe**. Si todos los requisitos que se enumeran de Hyper-V tienen un valor de **Sí**, el sistema puede ejecutar el rol de Hyper-V. Si algún elemento devuelve **No**, compruebe los requisitos que se muestran en el documento y realice ajustes cuando sea posible.

![](media/SystemInfo_upd.png)

Si ejecuta **systeminfo.exe** en un host de Hyper-V existente, la sección de requisitos de Hyper-V se indica:

```
Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V are not be displayed.
```

## Siguiente paso: instalar Hyper-V

[Instalar Hyper-V](walkthrough_install.md)






<!--HONumber=May16_HO1-->


