---
title: "Introducción a Hyper-V en Windows 10"
description: "Introducción a Hyper-V, la virtualización y tecnologías relacionadas."
keywords: windows 10, hyper-v
author: scooley
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
ms.openlocfilehash: 307cd592a9deda41fd2a892d49eadbc5ae436d84
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/21/2017
---
# Introducción a Hyper-V en Windows 10

> Hyper-V reemplaza a Microsoft Virtual PC. 

Tanto los desarrolladores de software como los profesionales de TI o los aficionados a la tecnología a veces necesitan ejecutar varios sistemas operativos. En lugar de dedicar hardware físico a cada una de las máquinas, Hyper-V permite ejecutar un sistema operativo o un equipo informático como una máquina virtual en Windows.  

![](media/HyperVNesting.png)

Específicamente, Hyper-V proporciona virtualización de hardware.  Eso significa que cada máquina virtual se ejecuta en hardware virtual.  Hyper-V permite crear unidades de disco duro virtuales, conmutadores virtuales y otros dispositivos virtuales, y todos ellos pueden agregarse a máquinas virtuales.

## Motivos para utilizar la virtualización

La virtualización permite:  
* Ejecutar software que requiere una versión anterior de cualquier sistema operativo Windows o que no sea Windows. 

* Experimentar con otros sistemas operativos. Hyper-V facilita la creación y eliminación de distintos sistemas operativos.

* Pruebe el software en varios sistemas operativos mediante varias máquinas virtuales. Con Hyper-V, se pueden ejecutar en un único equipo de escritorio o portátil. Estas máquinas virtuales se pueden exportar y después importar en cualquier otro sistema de Hyper-V, incluido Azure.

* Solucionar problemas de máquinas virtuales desde cualquier implementación de Hyper-V. Puede exportar una máquina virtual del entorno de producción, abrirla en el escritorio que ejecuta Hyper-V, solucionar el problemas de la máquina virtual y luego volver a exportarla al entorno de producción. 

* Mediante redes virtuales, puede crear un entorno con varios equipos para pruebas, desarrollo y demostración para garantizar que la red de producción no se vea afectada.

## Requisitos del sistema
Hyper-V solo está disponible en las versiones de 64 bits de las ediciones Professional, Enterprise y Education de Windows 8, y en las versiones superiores.  No está disponible en la edición Windows Home.  

>  Puedes actualizar Windows 10 Home Edition a Windows 10 Professional abriendo **Configuración** > **Actualización y seguridad** > **Activación**. Aquí puedes visitar la tienda y comprar una actualización.

La mayoría de los equipos ejecutarán Hyper-V; sin embargo, las máquinas virtuales requieren muchos recursos ya que ejecutan un sistema operativo completo.  Por lo general, puedes ejecutar una o más máquinas virtuales en un equipo con 4GB de RAM, pero necesitarás más recursos para otras máquinas virtuales o para instalar y ejecutar software que requiere muchos recursos, como juegos, programas de edición de vídeo o software de diseño de ingeniería. 

El equipo necesitará traducción de direcciones de segundo nivel (SLAT), que está presente en la generación actual de procesadores de 64 bits de Intel y AMD.  También necesitarás una versión de 64 bits de Windows.

Para obtener más información acerca de los requisitos del sistema de Hyper-V y sobre cómo comprobar que Hyper-V se ejecuta en un equipo, consulta la [Referencia de requisitos de Hyper-V](..\reference\hyper-v-requirements.md).

## Sistemas operativos que se pueden ejecutar en una máquina virtual
El término "invitado" hace referencia a una máquina virtual y "host" se refiere al equipo que ejecuta la máquina virtual. Hyper-V en Windows admite muchos sistemas operativos invitados diferentes, entre los que se incluyen varias versiones de Linux, FreeBSD y Windows. 

Como recordatorio, debe tener una licencia válida de los sistemas operativos que use en las máquinas virtuales. 

Para obtener información sobre los sistemas operativos que se admiten como invitados en Hyper-V en Windows, consulta [Sistemas operativos invitados de Windows admitidos](supported-guest-os.md) y [Supported Linux Guest Operating Systems](https://technet.microsoft.com/library/dn531030.aspx) (Sistemas operativos invitados de Linux admitidos). 


## Diferencias entre Hyper-V en Windows y Hyper-V en Windows Server
Existen algunas características que funcionan de forma diferente en Hyper-V en Windows con respecto a Hyper-V en Windows Server. 

El modelo de administración de memoria es diferente para Hyper-V en Windows. En un servidor, la memoria de Hyper-V se administra presuponiendo que solo las máquinas virtuales se ejecutan en el servidor. En Hyper-V en Windows, la memoria se administra con las expectativas de que la mayoría de las máquinas clientes ejecuten software en el host, además de ejecutar máquinas virtuales. Por ejemplo, un desarrollador podría ejecutar Visual Studio y varias máquinas virtuales en el mismo equipo.

Existen algunas características incluidas en Hyper-V en Windows Server que no se incluyen en Hyper-V en Windows. Entre ellos se incluyen los siguientes:

* Virtualización de GPU con RemoteFX 
* Migración en vivo de máquinas virtuales de un host a otro
* Réplica de Hyper-V
* Canal de fibra virtual
* Redes de SR-IOV
* .VHDX compartido

## Limitaciones
El uso de la virtualización tiene limitaciones. Las características o aplicaciones que dependen de un hardware concreto no funcionarán bien en una máquina virtual. Por ejemplo, es posible que los juegos o aplicaciones que requieran un procesamiento con GPU no funcionen bien. Además, las aplicaciones que dependen de los temporizadores por debajo de 10 ms, como las aplicaciones de mezcla de música en directo o tiempos de alta precisión, pueden tener problemas para ejecutarse en una máquina virtual.

Además, si ha habilitado Hyper-V, dichas aplicaciones de alta precisión sensibles a la latencia también podrían tener problemas para ejecutarse en el host.  Esto se debe a que, con la virtualización habilitada, el sistema operativo host también se ejecuta sobre el nivel de virtualización de Hyper-V, al igual que los sistemas operativos invitados. Pero, a diferencia de los invitados, el sistema operativo host es especial, en el sentido de que tiene acceso directo a todo el hardware, lo que significa que las aplicaciones con requisitos de hardware especiales pueden continuar ejecutándose sin problemas en el sistema operativo host.

## Paso siguiente
[Instalar Hyper-V en Windows 10](..\quick-start\enable-hyper-v.md) 
