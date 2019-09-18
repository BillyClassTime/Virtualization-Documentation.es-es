---
title: Contenedores de Linux en Windows
description: Obtenga más información sobre las diferentes formas de usar Hyper-V para ejecutar los contenedores de Linux en Windows como si estuvieran nativos.
keywords: LCOW, contenedores de Linux, Docker, contenedores
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 14445f3e9d292dbdab28986e772d0c045fca1586
ms.sourcegitcommit: 9100d2218c160bbe9fbf24f3524c8ff5e3dd826c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/18/2019
ms.locfileid: "10135328"
---
# <a name="linux-containers-on-windows"></a>Contenedores de Linux en Windows

Los contenedores de Linux conforman un enorme porcentaje del ecosistema de contenedor general y son fundamentales tanto para las experiencias de los programadores como para los entornos de producción.  Como los contenedores comparten un núcleo con el host contenedor, sin embargo, no es opción[*](linux-containers.md#other-options-we-considered)ejecutar contenedores Linux directamente en Windows.  Aquí es donde se encuentra la virtualización en la imagen.

Ahora, hay dos formas de ejecutar contenedores Linux con Docker para Windows y Hyper-V:

- Ejecuta los contenedores de Linux en una VM completa de Linux: lo que normalmente hace el acoplador hoy.
- Ejecute los contenedores de Linux con el [aislamiento de Hyper-V](../manage-containers/hyperv-container.md) (LCOW): esta es una nueva opción en el Docker para Windows.

En este artículo se describe cómo funciona cada enfoque, se proporcionan instrucciones sobre cuándo elegir qué solución y compartir el trabajo en curso.

## <a name="linux-containers-in-a-moby-vm"></a>Contenedores de Linux en una VM de Moby

Para ejecutar los contenedores de Linux en una VM de Linux, siga las instrucciones de la [Guía de introducción del Docker](https://docs.docker.com/docker-for-windows/).

El acoplador ha podido ejecutar contenedores Linux en el escritorio de Windows desde que se lanzó por primera vez en 2016 (antes de que estuviera disponible el aislamiento de Hyper-V o los contenedores Linux en Windows) usando una máquina virtual basada en [LinuxKit](https://github.com/linuxkit/linuxkit) que se ejecuta en Hyper-v.

En este modelo, el cliente del acoplador se ejecuta en el escritorio de Windows, pero llama al demonio del acoplador en la máquina virtual de Linux.

![VM de Moby como host de contenedor](media/MobyVM.png)

En este modelo, todos los contenedores de Linux comparten un único host de contenedor basado en Linux y todos los contenedores de Linux:

* Comparta un núcleo entre sí y la VM de Moby, pero no con el host de Windows.
* Tienen propiedades de red y almacenamiento de información coherentes con contenedores Linux que se ejecutan en Linux (ya que se ejecutan en una VM Linux).

También significa que el host contenedor Linux (VM de Moby) necesita ejecutar el demonio de acoplamiento y todas las dependencias de demonio de acoplamiento.

Para ver si está ejecutando con una VM de Moby, consulte administrador de Hyper-V para VM de Moby con la interfaz de usuario del administrador de `Get-VM` Hyper-v o ejecutando en una ventana de PowerShell con privilegios elevados.

## <a name="linux-containers-with-hyper-v-isolation"></a>Contenedores de Linux con aislamiento de Hyper-V

Para probar los contenedores de Linux en Windows (LCOW), siga las instrucciones del contenedor de Linux en [contenedores de Linux en Windows 10](../quick-start/quick-start-windows-10-linux.md).

Los contenedores de Linux con el aislamiento de Hyper-V ejecutan cada contenedor Linux en una VM optimizada de Linux con el SO suficiente para ejecutar contenedores. En contraste con el enfoque de la VM de Moby, cada contenedor de Linux tiene su propio núcleo y su propio espacio aislado de VM. También están administrados por el acoplador en Windows directamente.

![Contenedores de Linux con aislamiento de Hyper-V (LCOW)](media/lcow-approach.png)

Con más profundidad sobre el modo en que la administración de contenedores se diferencia entre el enfoque de la VM de Moby y el LCOW, en la administración del contenedor de LCOW de Windows, y la administración de LCOW se realiza mediante GRPC y contenedores.  Esto significa que los contenedores de Linux distro usan para LCOW pueden tener un inventario mucho más pequeño.  En este momento, usamos LinuxKit para el uso optimizado de los contenedores de distro, pero otros proyectos como Kata están generando Linux Distros muy optimizados (desmarca Linux) también.

Aquí encontrará más detalles sobre cada LCOW:

![Arquitectura de LCOW](media/lcow.png)

Para ver si está ejecutando LCOW, vaya a `C:\Program Files\Linux Containers`. Si el acoplador está configurado para usar LCOW, habrá unos cuantos archivos que contengan la distro mínima de LinuxKit que se ejecuta en cada contenedor que se ejecuta en el aislamiento de Hyper-V.  Observe que los componentes de VM optimizados son menos de 100 MB, mucho más pequeños que la imagen de LinuxKit en una VM de Moby.

### <a name="work-in-progress"></a>Trabajo en curso

LCOW está en desarrollo activo. Realizar un seguimiento del progreso continuo en el proyecto de Moby en [GitHub](https://github.com/moby/moby/issues/33850)

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

Estas aplicaciones requieren una asignación de volumen y no se iniciarán ni se ejecutarán correctamente.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>Información adicional

[Blog de Docks que describe LCOW](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Video contenedor de Linux](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-kernel Plus instrucciones de compilación](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Cuándo usar VM de Moby vs LCOW

### <a name="when-to-use-moby-vm"></a>Cuándo usar una VM de Moby

En este momento, recomendamos el método de la VM de Moby que ejecuta los contenedores de Linux para personas que:

- Desea un entorno de contenedor estable.  Este es el acoplador para Windows de forma predeterminada.
- Ejecutar contenedores Windows o Linux, pero rara vez ambos al mismo tiempo.
- Tener requisitos de conexión de red complicados o personalizados entre contenedores de Linux.
- No necesita aislamiento de kernel (aislamiento de Hyper-V) entre contenedores de Linux.

### <a name="when-to-use-lcow"></a>Cuándo usar LCOW

En este momento, recomendamos LCOW a las personas que:

- Desea probar nuestra tecnología más reciente.
- Ejecutar contenedores de Windows y Linux al mismo tiempo.
- Necesita aislamiento de kernel (aislamiento de Hyper-V) entre contenedores de Linux.

## <a name="other-options-we-considered"></a>Otras opciones que consideramos

Cuando estábamos buscando formas de ejecutar los contenedores de Linux en Windows, se consideró WSL. En última instancia, elegimos un enfoque basado en la virtualización para que los contenedores de Linux en Windows sean coherentes con los contenedores de Linux en Linux. El uso de Hyper-V también hace que LCOW sea más seguro. Podemos volver a evaluar en el futuro, pero por ahora, LCOW continuará usando Hyper-V.

Si tienes pensamientos, envía tus comentarios a través de GitHub o UserVoice.  Nos Agradecemos especialmente los comentarios sobre la experiencia específica que deseas ver.
