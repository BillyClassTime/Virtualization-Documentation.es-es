---
title: Crear una máquina virtual con Hyper-V
description: Crear una nueva máquina virtual con Hyper-V en Windows 10 Creators Update
keywords: windows 10, hyper-v, máquina virtual
author: scooley
ms.date: 04/07/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: f1e75efa-8745-4389-b8dc-91ca931fe5ae
ms.openlocfilehash: 970e92def02e5386d38a2e72d5ef921aa8321fdf
ms.sourcegitcommit: 08cc38955faad26f075b912a64b8ffb6b36f190c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578688"
---
# <a name="create-a-virtual-machine-with-hyper-v"></a>Crear una máquina virtual con Hyper-V

Crea una máquina virtual e instala su sistema operativo.

Hemos desarrollado nuevas herramientas para la creación de máquinas virtuales para que las instrucciones han cambiado significativamente en las tres últimas versiones.

Elige el sistema operativo para el conjunto correcto de instrucciones:

* [Windows 10 Fall Creators Update (v1709) y versiones posteriores](quick-create-virtual-machine.md#windows-10-fall-creators-update-windows-10-version-1709)
* [Windows 10 Creators Update (v1703)](quick-create-virtual-machine.md#windows-10-creators-update-windows-10-version-1703)
* [Actualización de aniversario de Windows 10 (v1607) y versiones anteriores](quick-create-virtual-machine.md#before-windows-10-creators-update-windows-10-version-1607-and-earlier)

Empecemos.

## <a name="windows-10-fall-creators-update-windows-10-version-1709"></a>Windows 10 Fall Creators Update (versión 1709 de Windows 10)

En la actualización Fall Creators Update, Creación rápida se ha ampliado para incluir una galería de máquinas virtuales que se puede iniciar independientemente del administrador de Hyper-V.

Para crear una nueva máquina virtual en la actualización Fall Creators Update:

1. Abre Creación rápida en Hyper-V desde el menú Inicio.

    ![Galería de Creación rápida en el menú Inicio de Windows](media/quick-create-start-menu.png)

1. Selecciona un sistema operativo o elige el tuyo usando un origen de instalación local.

    ![Vista de galería](media/vmgallery.png)

    1. Si quieres usar tu propia imagen para crear la máquina virtual, selecciona **Origen de instalación Local**.
    1. Selecciona **Cambiar el origen de instalación**.
      ![Botón para usar un origen de instalación local](media/change-source.png)
    1. Elige la .iso o el .vhdx que quieres convertir en una nueva máquina virtual.
    1. Si la imagen es una imagen de Linux, desactiva la opción Arranque seguro.
      ![Botón para usar un origen de instalación local](media/toggle-secure-boot.png)

1. Selecciona "Crear máquina virtual"

Eso es todo.  Creación rápida se encargará del resto.

## <a name="windows-10-creators-update-windows-10-version-1703"></a>Windows 10 Creators Update (Windows 10 versión 1703)

![Captura de pantalla de la interfaz de usuario de creación rápida](media/quickcreatesteps_inked.jpg)

1. Abre el administrador de Hyper-V desde el menú de inicio.

1. En el administrador de Hyper-V, busca **Creación rápida** en el menú **Acciones** del lado derecho.

1. Personaliza la máquina virtual.

    * (opcional) Asigna un nombre a la máquina virtual.
    * Selecciona el medio de instalación para la máquina virtual. Puedes instalar desde un archivo .iso o .vhdx.
    Si vas a instalar Windows en la máquina virtual, puedes habilitar el arranque seguro de Windows. Si no, déjalo sin seleccionar.
    * Configurar la red.
    Si tienes un conmutador virtual existente, puedes seleccionarlo en la lista desplegable de redes. Si no tienes ningún conmutador existente, verás un botón para configurar una red automática, lo que configurará automáticamente una red virtual.

1. Haz clic en **Conectar** para iniciar la máquina virtual. No te preocupes por editar la configuración. Puedes volver y cambiarla en cualquier momento.

    Es posible que se te indique "Presione cualquier tecla para arrancar desde CD o DVD". Hágalo.  En lo que respecta a la máquina virtual, estás instalando desde un CD.

Enhorabuena, ya tienes una nueva máquina virtual.  Ahora estás listo para instalar el sistema operativo.

La máquina virtual debe ser algo parecido a esto:

![Pantalla de inicio de la máquina virtual](media/OSDeploy_upd.png)

> **Nota**: a menos que estés ejecutando una versión con licencia por volumen de Windows, necesitarás una licencia independiente para la ejecución de Windows en una máquina virtual. El sistema operativo de la máquina virtual es independiente del sistema operativo del host.

## <a name="before-windows-10-creators-update-windows-10-version-1607-and-earlier"></a>Antes de Windows 10 Creators Update (versión 1607 y versiones anterior de Windows 10)

Si no ejecutas Windows 10 Creators Update o una versión posterior, sigue estas instrucciones usando en su lugar el Asistente para crear una nueva máquina virtual:

1. [Crear una red virtual](connect-to-network.md)
1. [Crear una nueva máquina virtual](create-virtual-machine.md)
