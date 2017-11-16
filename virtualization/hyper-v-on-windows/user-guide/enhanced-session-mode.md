---
title: "Compartir dispositivos con máquinas virtuales de Windows"
description: "En este tema se describe cómo compartir dispositivos con máquinas virtuales de Hyper-V (USB, audio, micrófono y unidades montadas)."
keywords: windows 10, hyper-v
ms.author: scooley
ms.date: 10/20/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d1aeb9cb-b18f-43cb-a568-46b33346a188
ms.openlocfilehash: 52d51fca03f454a311a123f20e5aeda9376fdc3d
ms.sourcegitcommit: 3ec9917c456875f68180323eb9e0470d5115325a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 10/27/2017
---
# <a name="share-devices-with-your-virtual-machine"></a>Compartir dispositivos con la máquina virtual

> Solo está disponible para las máquinas virtuales de Windows.

El Modo de sesión mejorada permite que Hyper-V se conecte a las máquinas virtuales a través de RDP (protocolo de escritorio remoto).  Esto no solo mejora la experiencia general de visualización de la máquina virtual, sino que la conexión mediante RDP también permite que la máquina virtual comparta dispositivos con el equipo.  Dado que está opción está activada de manera predeterminada en Windows 10, probablemente ya estés usando RDP para conectarte a tus máquinas virtuales de Windows.  En este artículo se destacan algunas de las ventajas y las opciones ocultas del cuadro de diálogo de configuración de conexión.

Modo de sesión mejorada o RDP:

* Permite ajustar el tamaño de las máquinas virtuales y que las VM reconozcan valores altos de PPP.
* Mejora la integración de las máquinas virtuales.
  * Portapapeles compartido
  * Uso compartido de archivos mediante las acciones arrastrar y colocar, y copiar y pegar
* Permite el uso compartido de dispositivos.
  * Micrófono y altavoces
  * Dispositivos USB
  * Discos de datos (incluido C:)
  * Impresoras

En este artículo se muestra cómo ver el tipo de sesión, pasar al modo de sesión mejorada y configurar las opciones de la sesión.

## <a name="share-drives-and-devices"></a>Compartir unidades y dispositivos

Las funciones de uso compartido de dispositivos del modo de sesión mejorada están ocultas en esta ventana de conexión poco notoria que aparece cuando te conectas a una máquina virtual:

![](media/esm-default-view.png)

De manera predeterminada, las máquinas virtuales que usan el modo de sesión mejorada compartirán el portapapeles y las impresoras.  También se han configurado para transmitir el audio de la máquina virtual a los altavoces del equipo de manera predeterminada.

Para compartir dispositivos con la máquina virtual o para cambiar estos ajustes predeterminados:

1. Mostrar más opciones

  ![](media/esm-show-options.png)

1. Ver recursos locales

  ![](media/esm-local-resources.png)

### <a name="share-storage-and-usb-devices"></a>Compartir almacenamiento y dispositivos USB

De manera predeterminada, las máquinas virtuales que usan el modo de sesión mejorada comparten las impresoras, el portapapeles, la tarjeta inteligente de acceso y otros dispositivos de seguridad a través de la máquina virtual para que puedas usar herramientas de inicio de sesión más seguras desde la máquina virtual.

Para compartir otros dispositivos, como los dispositivos USB o la unidad C:, selecciona el menú "Más…":  
![](media/esm-more-devices.png)

Allí puedes seleccionar los dispositivos que te gustaría compartir con la máquina virtual.  La unidad del sistema (C: en Windows) resulta especialmente útil para el uso compartido de archivos.  
![](media/esm-drives-usb.png)

### <a name="share-audio-devices-speakers-and-microphones"></a>Compartir dispositivos de audio (altavoces y micrófonos)

De manera predeterminada, las máquinas virtuales que usan el modo de sesión mejorada transmiten audio para poder escucharlo desde la máquina virtual.  La máquina virtual usará el dispositivo de audio actualmente seleccionado en el equipo host.

Para cambiar estos valores o agregar un micrófono de transmisión (para que se pueda grabar audio en una máquina virtual):

Selecciona el menú "Configuración…" para configurar las opciones de audio remoto.  
![](media/esm-audio.png)

A continuación, configura las opciones de audio y micrófono.  
![](media/esm-audio-settings.png)

Dado que la máquina virtual probablemente se ejecute de forma local, las opciones "Reproducir en este equipo" y "Reproducir en el equipo remoto" generarán los mismos resultados.

## <a name="re-launching-the-connection-settings"></a>Volver a iniciar la configuración de conexión

Si no aparece el cuadro de diálogo de resolución y uso compartido de dispositivos, intenta iniciar VMConnect de forma independiente desde el menú de Windows o desde la línea de comandos como Administrador.  

``` Powershell
vmconnect.exe
```

## <a name="check-session-type"></a>Comprobar el tipo de sesión

Puedes comprobar el tipo de conexión que tienes mediante el icono del modo de sesión mejorada situado en la parte superior de la herramienta Conexión a máquina virtual (VMConnect).  Este botón también te permite alternar entre el modo de sesión básica y sesión mejorada.

![](media/esm-button-location.png)

| icono | estado de conexión |
|:-----|:---------|
|![](media/esm-basic.png)| La VM se está ejecutando en modo de sesión mejorada.  Al hacer clic en este icono, se volverá a establecer la conexión a la máquina virtual en modo básico. |
|![](media/esm-connect.png)| La VM se está ejecutando en modo de sesión básica, pero el modo de sesión mejorada está disponible.  Al hacer clic en este icono, se volverá a establecer la conexión a la máquina virtual en modo de sesión mejorada.  |
|![](media/esm-stop.png)| La VM se está ejecutando en modo básico.  El modo de sesión mejorada no está disponible para esta máquina virtual. |