---
title: Crear una galería de máquinas virtuales personalizada
description: Crea tus propias entradas en la galería de máquinas virtuales de Windows 10 Creators Update y versiones posteriores.
keywords: windows 10, hyper-v, creación rápida, máquina virtual, galería
ms.author: scooley
ms.date: 05/04/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d9238389-7028-4015-8140-27253b156f37
ms.openlocfilehash: c7a6462b331f469148eb4cf5a0a2740c9929fa29
ms.sourcegitcommit: 2b5d806fc978e60fb71ce33ef491d4cfd6fc4456
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2018
ms.locfileid: "2596073"
---
# <a name="create-a-custom-virtual-machine-gallery"></a>Crear una galería de máquinas virtuales personalizada

> Windows 10 Fall Creators Update y versiones posteriores.

En la actualización Fall Creators Update, Creación rápida se ha ampliado para incluir una galería de máquinas virtuales.

![Creación rápida de galerías de VM con imágenes personalizadas](media/vmgallery.png)

Aunque existe un conjunto de imágenes proporcionadas por Microsoft y los partners de Microsoft, la galería también puede incluir tus propias imágenes.

Este artículo trata sobre:

* creación de máquinas virtuales compatibles con la galería.
* creación de un nuevo origen de galerías.
* agregar el origen de la galería personalizada a la galería.

## <a name="gallery-architecture"></a>Arquitectura de galerías

La galería de máquinas virtuales es una vista gráfica de un conjunto de orígenes de máquinas virtuales que se definen en el registro de Windows.  Cada origen de máquina virtual es una ruta de acceso (ruta de acceso local o URI) a un archivo JSON con máquinas virtuales como elementos de lista.

La lista de máquinas virtuales que ves en la galería es todo el contenido del primer origen, seguido por el contenido del segundo origen, y así sucesivamente hasta que se enumeran todas las máquinas virtuales disponibles.  La lista se crea dinámicamente cada vez que inicies la galería.

![arquitectura de galerías](media/vmgallery-architecture.png)

Clave del Registro: `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization`

Nombre del valor: `GalleryLocations`

Tipo: `REG_MULTI_SZ`

## <a name="create-gallery-compatible-virtual-machines"></a>Crear máquinas virtuales compatibles con la galería

Las máquinas virtuales de la galería pueden ser una imagen de disco (.iso) o un disco duro virtual (.vhdx).

Las máquinas virtuales realizadas desde una unidad de disco duro virtual tiene algunos requisitos de configuración:

1. Creado para admitir el firmware UEFI. Si se crean mediante Hyper-V, se trata de VM de generación 2.
1. La unidad de disco duro virtual debe tener al menos 20GB; atención, ya que es el tamaño máximo.  Hyper-V no tomará el espacio que la VM no esté usando activamente.

### <a name="testing-a-new-vm-image"></a>Prueba de una nueva imagen de VM

La galería de máquina virtual crea máquinas virtuales mediante el mismo mecanismo que cuando la instalación se realiza desde un origen de instalación local.

Para validar que una imagen de máquina virtual será capaz de arrancar y ejecutarse:

1. Abre la Galería de VM (Creación rápida de Hyper-V) y selecciona **Origen de instalación Local **.
  ![Botón para usar un origen de instalación local](media/use-local-source.png)
1. Selecciona **Cambiar el origen de instalación**.
  ![Botón para usar un origen de instalación local](media/change-source.png)
1. Elige la .iso o el .vhdx que se usará en la galería.
1. Si la imagen es una imagen de Linux, desactiva la opción Arranque seguro.
  ![Botón para usar un origen de instalación local](media/toggle-secure-boot.png)
1. Crea la máquina virtual.  Si la máquina virtual arranca correctamente, está lista para la galería.

## <a name="build-a-new-gallery-source"></a>Crea un nuevo origen de galería.

El paso siguiente es crear un nuevo origen de galería.  Este es el archivo JSON que enumera las máquinas virtuales y agrega toda la información adicional que ves en la galería.

Información de texto:

![Ubicaciones etiquetadas como texto de galería](media/gallery-text.png)

* **name**: necesario, es el nombre que aparece en la columna izquierda y también en la parte superior de la vista de la máquina virtual.
* **publicador**: requerido
* **descripción**: necesaria, lista de cadenas que describen la VM.
* **version**: requerido
* lastUpdated: el valor predeterminado es el lunes, 1 de enero de 0001.

  El formato debe ser: aaaa-mm-ddThh:mm:ssZ

  El siguiente comando de PowerShell proporciona la fecha de hoy en el formato adecuado y la coloca en el Portapapeles:

  ``` PowerShell
  Get-Date -UFormat "%Y-%m-%dT%TZ" | clip.exe
  ```

* local: el valor predeterminado es en blanco.

Imágenes:

![Ubicaciones etiquetadas como imágenes de galería](media/gallery-pictures.png)

* **logo**: obligatorio
* símbolo
* miniatura

Y, por supuesto, la máquina virtual (.iso o .vhdx).

Para generar los valores hash, puede usar el siguiente comando de powershell:

  ``` PowerShell
  Get-FileHash -Path .\TMLogo.jpg -Algorithm SHA256
  ```

La plantilla JSON siguiente tiene elementos de comienzo y el esquema de la galería.  Si la editas en VSCode, proporcionará automáticamente IntelliSense.

[!code-json[main](../../../hyperv-tools/vmgallery/vm-gallery-template.json)]

## <a name="connect-your-gallery-to-the-vm-gallery-ui"></a>Conectar la galería a la interfaz de usuario de la galería de VM

La forma más sencilla de agregar el origen de la galería personalizada a la galería de VM es hacerlo en regedit.

1. Abre **regedit.exe**
1. Ve a `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\`
1. Busca el elemento `GalleryLocations`.

    Si ya existe, ve al menú **Editar** y **modifica**.

    Si aún no existe, ve al menú **Editar**, navega por **Nuevo** hasta **Valor de cadena múltiple**

1. Agrega tu galería a la clave del Registro `GalleryLocations`.

    ![Clave del Registro de la galería con el nuevo elemento](media/new-gallery-uri.png)

## <a name="troubleshooting"></a>Solución de problemas

### <a name="check-for-errors-loading-gallery"></a>Comprobar si hay errores al cargar la galería

La galería de máquinas virtuales proporciona informes de errores en el Visor de eventos de Windows.  Para comprobar si hay errores:

1. Abre el Visor de eventos
1. Ve a **Registros de Windows** -> **Aplicación**
1. Busca eventos desde VMCreate de origen.

## <a name="resources"></a>Recursos

Hay una serie de scripts y aplicaciones auxiliares de galerías en GitHub [vínculo](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/live/hyperv-tools/vmgallery).

Consulta un ejemplo de entrada de galería [aquí](https://go.microsoft.com/fwlink/?linkid=851584).  Este es el archivo JSON que define la galería en el equipo.
