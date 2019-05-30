---
title: Introducción a Hyper-V en Windows 10
description: Introducción a Hyper-V, la virtualización y tecnologías relacionadas.
keywords: windows 10, hyper-v
author: scooley
ms.date: 06/25/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
ms.openlocfilehash: aab6a285e9c1ed9918b39cb1de88e3e2243fb3a9
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2019
ms.locfileid: "9680965"
---
# <a name="introduction-to-hyper-v-on-windows-10"></a>Introducción a Hyper-V en Windows 10

> Hyper-V reemplaza a Microsoft Virtual PC.

Tanto los desarrolladores de software como los profesionales de TI o los aficionados a la tecnología necesitan a veces ejecutar varios sistemas operativos. Hyper-V te permite ejecutar varios sistemas operativos como máquinas virtuales en Windows.

![Máquina virtual ejecutando Windows](media/HyperVNesting.png)

Específicamente, Hyper-V proporciona virtualización de hardware.  Eso significa que cada máquina virtual se ejecuta en hardware virtual.  Hyper-V permite crear unidades de disco duro virtuales, conmutadores virtuales y otros dispositivos virtuales, y todos ellos pueden agregarse a máquinas virtuales.

## <a name="reasons-to-use-virtualization"></a>Motivos para utilizar la virtualización

La virtualización permite:

* Ejecutar software que requiere una versión anterior de cualquier sistema operativo Windows o que no sea Windows.

* Experimentar con otros sistemas operativos. Hyper-V facilita la creación y eliminación de distintos sistemas operativos.

* Pruebe el software en varios sistemas operativos mediante varias máquinas virtuales. Con Hyper-V, se pueden ejecutar en un único equipo de escritorio o portátil. Estas máquinas virtuales se pueden exportar y después importar en cualquier otro sistema de Hyper-V, incluido Azure.

## <a name="system-requirements"></a>Requisitos del sistema

Hyper-V está disponible en las versiones de 64 de Windows 10 Pro, Enterprise y Education. No está disponible en la edición de tu hogar.

> Actualice de Windows 10 Home Edition a Windows 10 Pro y Abra **configuración** > **actualización y** > **activación**de seguridad. Aquí puedes visitar store y comprar una actualización.

La mayoría de los equipos ejecutan Hyper-V, pero cada máquina virtual ejecuta un sistema operativo completamente independiente.  Por lo general, puedes ejecutar una o más máquinas virtuales en un equipo con 4GB de RAM, pero necesitarás más recursos para otras máquinas virtuales o para instalar y ejecutar software que requiere muchos recursos, como juegos, programas de edición de vídeo o software de diseño de ingeniería.

Para obtener más información acerca de los requisitos del sistema de Hyper-V y sobre cómo comprobar que Hyper-V se ejecuta en un equipo, consulta la [Referencia de requisitos de Hyper-V](../reference/hyper-v-requirements.md).

## <a name="operating-systems-you-can-run-in-a-virtual-machine"></a>Sistemas operativos que se pueden ejecutar en una máquina virtual

Hyper-V en Windows admite muchos sistemas operativos diferentes en las máquinas virtuales, incluyendo diversas versiones de Linux, FreeBSD y Windows.

Como recordatorio, debe tener una licencia válida de los sistemas operativos que use en las VM.

Para obtener información sobre los sistemas operativos que se admiten como invitados en Hyper-V en Windows, consulta [Sistemas operativos invitados de Windows admitidos](supported-guest-os.md) y [Supported Linux Guest Operating Systems](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows) (Sistemas operativos invitados de Linux admitidos).

## <a name="differences-between-hyper-v-on-windows-and-hyper-v-on-windows-server"></a>Diferencias entre Hyper-V en Windows y Hyper-V en Windows Server

Existen algunas funciones que trabajan de forma diferente en Hyper-V en Windows con respecto a Hyper-V en Windows Server.

Funciones de Hyper-V disponibles solo en Windows Server:

* Migración en vivo de máquinas virtuales de un host a otro
* Réplica de Hyper-V
* Canal de fibra virtual
* Redes de SR-IOV
* .VHDX compartido

Funciones de Hyper-V disponibles solo en Windows 10:

* Creación rápida y galería de VM
* Red predeterminada (conmutador NAT)

El modelo de administración de memoria es diferente para Hyper-V en Windows. En un servidor, la memoria de Hyper-V se administra presuponiendo que solo las máquinas virtuales se ejecutan en el servidor. En Hyper-V en Windows, la memoria se administra con las expectativas de que la mayoría de las máquinas clientes ejecuten software en el host, además de ejecutar máquinas virtuales.

## <a name="limitations"></a>Limitaciones

Los programas que dependan de un hardware concreto no funcionarán bien en una máquina virtual. Por ejemplo, es posible que los juegos o aplicaciones que requieran un procesamiento con GPU no funcionen bien. Además, las aplicaciones que dependen de los temporizadores por debajo de 10 ms, como las aplicaciones de mezcla de música en directo o tiempos de alta precisión, pueden tener problemas para ejecutarse en una máquina virtual.

Además, si ha habilitado Hyper-V, dichas aplicaciones de alta precisión sensibles a la latencia también podrían tener problemas para ejecutarse en el host.  Esto se debe a que, con la virtualización habilitada, el sistema operativo host también se ejecuta sobre el nivel de virtualización de Hyper-V, al igual que los sistemas operativos invitados. Pero, a diferencia de los invitados, el sistema operativo host es especial, en el sentido de que tiene acceso directo a todo el hardware, lo que significa que las aplicaciones con requisitos de hardware especiales pueden continuar ejecutándose sin problemas en el sistema operativo host.

## <a name="next-step"></a>Paso siguiente

[Instalar Hyper-V en Windows 10](../quick-start/enable-hyper-v.md)
