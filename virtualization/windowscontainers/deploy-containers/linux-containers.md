---
title: Contenedores de Linux en Windows
description: Obtenga información sobre las distintas formas en que puede usar Hyper-V para ejecutar contenedores de Linux en Windows como si fueran nativos.
keywords: LCOW, contenedores de Linux, Docker, contenedores
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 14445f3e9d292dbdab28986e772d0c045fca1586
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910575"
---
# <a name="linux-containers-on-windows"></a>Contenedores de Linux en Windows

Los contenedores de Linux componen un gran porcentaje del ecosistema de contenedores general y son fundamentales para las experiencias de desarrollo y los entornos de producción.  Dado que los contenedores comparten un kernel con el host de contenedor, sin embargo, la ejecución de contenedores de Linux directamente en Windows no es una opción[*](linux-containers.md#other-options-we-considered).  Aquí es donde la virtualización entra en la imagen.

En la actualidad hay dos maneras de ejecutar contenedores de Linux con Docker para Windows e Hyper-V:

- Ejecución de contenedores de Linux en una máquina virtual de Linux completa: esto es lo que normalmente tiene Docker.
- Ejecutar contenedores de Linux con el [aislamiento de Hyper-V](../manage-containers/hyperv-container.md) (LCOW): se trata de una nueva opción de Docker para Windows.

En este artículo se describe cómo funciona cada enfoque, se proporcionan instrucciones sobre cuándo elegir qué solución y qué recursos compartidos funcionan en curso.

## <a name="linux-containers-in-a-moby-vm"></a>Contenedores de Linux en una máquina virtual de Moby

Para ejecutar contenedores de Linux en una máquina virtual Linux, siga las instrucciones de la guía de introducción a [Docker](https://docs.docker.com/docker-for-windows/).

Docker ha podido ejecutar contenedores de Linux en el escritorio de Windows, ya que se lanzó por primera vez en 2016 (antes de que el aislamiento de Hyper-V o los contenedores de Linux en Windows estuvieran disponibles) mediante una máquina virtual basada en [LinuxKit](https://github.com/linuxkit/linuxkit) que se ejecuta en Hyper-v.

En este modelo, el cliente de Docker se ejecuta en el escritorio de Windows, pero llama al demonio de Docker en la máquina virtual Linux.

![Máquina virtual de Moby como host de contenedor](media/MobyVM.png)

En este modelo, todos los contenedores de Linux comparten un único host de contenedor basado en Linux y todos los contenedores de Linux:

* Comparta un kernel entre sí y la máquina virtual de Moby, pero no con el host de Windows.
* Tienen propiedades de red y almacenamiento coherentes con contenedores de Linux que se ejecutan en Linux (ya que se ejecutan en una máquina virtual Linux).

También significa que el host de contenedor de Linux (máquina virtual de Moby) debe ejecutar el demonio de Docker y todas las dependencias del demonio de Docker.

Para ver si está ejecutando con una máquina virtual de Moby, consulte administrador de Hyper-V para VM de Moby con la interfaz de usuario del administrador de Hyper-V o mediante la ejecución de `Get-VM` en una ventana de PowerShell con privilegios elevados.

## <a name="linux-containers-with-hyper-v-isolation"></a>Contenedores de Linux con aislamiento de Hyper-V

Para probar los contenedores de Linux en Windows (LCOW), siga las instrucciones del contenedor de Linux en [contenedores de Linux en Windows 10](../quick-start/quick-start-windows-10-linux.md).

Los contenedores de Linux con aislamiento de Hyper-V ejecutan cada contenedor de Linux en una máquina virtual Linux optimizada con suficiente sistema operativo para ejecutar contenedores. A diferencia del enfoque de máquina virtual de Moby, cada contenedor de Linux tiene su propio kernel y su propio espacio aislado de VM. También se administran mediante Docker en Windows directamente.

![Contenedores de Linux con aislamiento de Hyper-V (LCOW)](media/lcow-approach.png)

Si observamos más de cerca cómo se diferencia la administración de contenedores entre el enfoque de la máquina virtual de Moby y LCOW, en la administración del contenedor de modelos de LCOW permanece en Windows y cada administración de LCOW se produce a través de GRPC y contenedores.  Esto significa que el uso de los contenedores de Linux distribución para LCOW puede tener un inventario mucho más pequeño.  En este momento, usamos LinuxKit para el uso optimizado de los contenedores de distribución, pero otros proyectos como Kata están compilando Linux distribuciones de alta optimización similar.

Esta es una mirada más detallada a cada LCOW:

![Arquitectura de LCOW](media/lcow.png)

Para ver si está ejecutando LCOW, vaya a `C:\Program Files\Linux Containers`. Si Docker está configurado para usar LCOW, habrá unos cuantos archivos aquí que contengan la distribución de LinuxKit mínima que se ejecuta en cada contenedor que se ejecuta en el aislamiento de Hyper-V.  Observe que los componentes de la máquina virtual optimizada son inferiores a 100 MB, mucho más pequeños que la imagen LinuxKit en la máquina virtual de Moby.

### <a name="work-in-progress"></a>Trabajo en curso

LCOW está en desarrollo activo. Seguimiento del progreso continuo en el proyecto de Moby en [GitHub](https://github.com/moby/moby/issues/33850)

#### <a name="bind-mounts"></a>Enlazar montajes

Al enlazar volúmenes de montaje con `docker run -v ...`, se almacenan los archivos en el sistema de archivos NTFS de Windows, por lo que es necesaria una traducción de las operaciones de POSIX. Algunas operaciones del sistema de archivos están implementadas parcialmente o no se han implementado, lo que puede provocar problemas de compatibilidad con algunas aplicaciones.

Las siguientes operaciones no funcionan actualmente para volúmenes montados mediante enlace:

* MkNod
* XAttrWalk
* XAttrCreate
* Lock
* Getlock
* Auth
* Flush
* INotify

También hay algunas que no se han implementado completamente:

* GetAttr: el recuento de Nlink siempre se notifica como 2
* Open: solo se implementan las marcas ReadWrite, WriteOnly y ReadOnly

Todas estas aplicaciones requieren la asignación de volúmenes y no se iniciarán ni ejecutarán correctamente.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>Información adicional

[Blog de Docker que describe LCOW](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Vídeo de contenedor de Linux](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-kernel Plus instrucciones de compilación](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Cuándo usar una máquina virtual de Moby en lugar de LCOW

### <a name="when-to-use-moby-vm"></a>Cuándo usar una máquina virtual de Moby

En este momento, se recomienda usar el método de máquina virtual de Moby para ejecutar contenedores de Linux para personas que:

- Desea un entorno de contenedor estable.  Esta es la Docker para Windows predeterminada.
- Ejecutar contenedores de Windows o Linux, pero rara vez al mismo tiempo.
- Tenga requisitos de red complicados o personalizados entre los contenedores de Linux.
- No es necesario el aislamiento de kernel (aislamiento de Hyper-V) entre los contenedores de Linux.

### <a name="when-to-use-lcow"></a>Cuándo usar LCOW

En este momento, se recomienda LCOW a las personas que:

- Quiere probar la tecnología más reciente.
- Ejecutar contenedores de Windows y Linux al mismo tiempo.
- Necesita aislamiento de kernel (aislamiento de Hyper-V) entre contenedores de Linux.

## <a name="other-options-we-considered"></a>Otras opciones que consideramos

Cuando examinamos maneras de ejecutar contenedores de Linux en Windows, se consideró WSL. En última instancia, elegimos un enfoque basado en la virtualización para que los contenedores de Linux en Windows sean coherentes con los contenedores de Linux en Linux. El uso de Hyper-V también hace que LCOW sea más seguro. Es posible que se vuelva a evaluar en el futuro, pero por ahora, LCOW continuará usando Hyper-V.

Si tiene alguna idea, envíe sus comentarios a través de GitHub o UserVoice.  Es especialmente apreciamos comentarios sobre la experiencia específica que le gustaría ver.
