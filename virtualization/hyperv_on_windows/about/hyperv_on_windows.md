---
title: "Introducción a Hyper-V en Windows 10"
description: "Introducción a Hyper-V en Windows 10."
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
translationtype: Human Translation
ms.sourcegitcommit: c3e7cc07ac7e7d4e1c5f1827deb5951daa1e3749
ms.openlocfilehash: ad84961d0a79853e2aadcf9ed0e37e340103835a

---

# Introducción a Hyper-V en Windows 10

Tanto los desarrolladores de software como los profesionales de TI o los aficionados a la tecnología a veces necesitan ejecutar varios sistemas operativos.  En lugar de dedicar hardware físico a cada una de las máquinas, Hyper-V permite ejecutar varios sistemas operativos en un equipo de Windows como máquinas virtuales (VM).

> Microsoft Virtual PC alcanzará el final del ciclo de vida en abril de 2017. En Windows 10 Enterprise y Windows 10 Professional, será su sustituto compatible.  

## Motivos para utilizar la virtualización
La virtualización permite que cualquier usuario ejecute fácilmente varios sistemas operativos, configuraciones de software y configuraciones de hardware en la misma máquina física.  Hyper-V proporciona virtualización y las herramientas necesarias para administrar máquinas virtuales.

Hyper-V puede usarse de muchas maneras. Por ejemplo:

* Ejecutar software que requiere una versión anterior de Windows o de cualquier sistema operativo que no sea Windows. 

* Experimentar con otros sistemas operativos. Hyper-V facilita la creación y eliminación de distintos sistemas operativos.

* Pruebe el software en varios sistemas operativos mediante varias máquinas virtuales. Con Hyper-V, se pueden ejecutar en un único equipo de escritorio o portátil. Estas máquinas virtuales se pueden exportar y después importar en cualquier otro sistema de Hyper-V, incluido Azure.

* Solucionar problemas de máquinas virtuales desde cualquier implementación de Hyper-V. Puede exportar una máquina virtual del entorno de producción, abrirla en el escritorio que ejecuta Hyper-V, solucionar el problemas de la máquina virtual y luego volver a exportarla al entorno de producción. 

* Mediante redes virtuales, puede crear un entorno con varios equipos para pruebas, desarrollo y demostración para garantizar que la red de producción no se vea afectada.

## Requisitos del sistema
Hyper-V solo está disponible en las ediciones Professional, Enterprise y Education de Windows 8, y en las versiones superiores.

Requiere un sistema de 64 bits que tenga traducción de direcciones de segundo nivel (SLAT). SLAT es una característica presente en la generación actual de los procesadores de 64 bits de Intel y AMD.  También necesitará una versión de 64 bits de Windows.  
Dicho esto, Hyper-V es compatible con sistemas operativos de 32 y 64 bits en las máquinas virtuales.

Puede ejecutar tres o cuatro máquinas virtuales básicas en un host que tenga 4 GB de RAM, aunque se necesitarán más recursos si se desea ejecutar más. En el otro extremo del espectro, también se pueden crear máquinas virtuales grandes con 32 procesadores y 512 GB de RAM, en función del hardware físico.

Para más información acerca de los requisitos del sistema de Hyper-V y cómo comprobar que Hyper-V se ejecuta en un equipo, consulte [Instalar Hyper-V en Windows 10](..\quick_start\walkthrough_install.md).


## Sistemas operativos que puede ejecutar en una máquina virtual
El término "invitado" hace referencia a una máquina virtual y "host" se refiere al equipo que ejecuta la máquina virtual. Hyper-V en Windows admite muchos sistemas operativos invitados diferentes, entre los que se incluyen varias versiones de Linux, FreeBSD y Windows. 

Como recordatorio, debe tener una licencia válida de los sistemas operativos que use en las máquinas virtuales. 

Para obtener información sobre los sistemas operativos que se admiten como invitados en Hyper-V en Windows, consulte [Sistemas operativos invitados de Windows admitidos](supported_guest_os.md) y [Máquinas virtuales de Linux y FreeBSD en Hyper-V](https://technet.microsoft.com/library/dn531030.aspx). 


## Diferencias entre Hyper-V en Windows y Hyper-V en Windows Server
Existen algunas características que funcionan de forma diferente en Hyper-V en Windows con respecto a Hyper-V en Windows Server. 

El modelo de administración de memoria es diferente para Hyper-V en Windows. En un servidor, la memoria de Hyper-V se administra presuponiendo que solo las máquinas virtuales se ejecutan en el servidor. En Hyper-V en Windows, la memoria se administra con las expectativas de que la mayoría de las máquinas clientes ejecuten software en el host, además de ejecutar máquinas virtuales. Por ejemplo, un programador podría ejecutar Visual Studio y varias máquinas virtuales en el mismo equipo.

### Características de Hyper-V disponibles solo en Windows Server
Existen algunas características que se incluyen en Hyper-V en Windows Server, pero no están incluidas en Hyper-V en Windows. Entre ellos se incluyen los siguientes:

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
[Instalar Hyper-V en Windows 10](..\quick_start\walkthrough_install.md) 

Descubra las [novedades](whats_new.md) de Hyper-V en Windows 10.




<!--HONumber=Jul16_HO2-->


