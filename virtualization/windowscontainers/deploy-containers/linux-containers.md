---
title: Contenedores de Linux en Windows
description: Obtén información sobre las diferentes formas de que usar Hyper-V para ejecutar contenedores de Linux en Windows como si estuvieran nativos.
keywords: LCOW, los contenedores de linux, docker, contenedores
author: scooley
ms.date: 11/02/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 8597a93f035f5e451df8176d1563299120c95cb8
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578426"
---
# <a name="linux-containers-on-windows"></a>Contenedores de Linux en Windows

Los contenedores de Linux constituyen un gran porcentaje del ecosistema de contenedor general y son fundamentales para entornos de producción y experiencias de desarrollador.  Dado que los contenedores comparten un kernel con el host de contenedor, sin embargo, ejecuta contenedores de Linux directamente en Windows no es una opción[*](linux-containers.md#other-options-we-considered).  Aquí es donde entra virtualización en la imagen.

Ahora hay dos formas de ejecutar contenedores de Linux con Docker para Windows y Hyper-V:

1. Ejecutar contenedores de Linux en una VM Linux completa: Esto es lo que Docker normalmente hace hoy en día.
1. Ejecutar contenedores de Linux con [aislamiento de Hyper-V](../manage-containers/hyperv-container.md) (LCOW): Esto es una nueva opción de Docker para Windows.

En este artículo se describe cómo cada enfoque funciona, proporciona instrucciones sobre cuándo se elige qué soluciones y recursos compartidos de trabajo en curso.

## <a name="linux-containers-in-a-moby-vm"></a>Contenedores de Linux en una VM de Moby

Para ejecutar contenedores de Linux en una VM Linux, sigue las instrucciones de [la Guía de introducción de get de Docker](https://docs.docker.com/docker-for-windows/).

Docker ha sido capaz de ejecutar los contenedores de Linux en Windows desktop desde que se publicó por primera vez en 2016 (antes de aislamiento de Hyper-V o LCOW estuvieran disponibles) mediante un [LinuxKit](https://github.com/linuxkit/linuxkit) basada en máquina virtual en Hyper-V.

En este modelo, cliente de Docker se ejecuta en el escritorio de Windows, pero las llamadas en demonio de Docker en la VM Linux.

![VM de MOBY como el host de contenedor](media/MobyVM.png)

En este modelo, todos los contenedores de Linux compartan un host de contenedor único basado en Linux y todos los contenedores de Linux:

* Comparte un kernel entre sí y la VM de Moby, pero no con el host de Windows.
* Tengan almacenamiento coherente y propiedades de redes con contenedores de Linux que se ejecutan en Linux (ya que se están ejecutando en una VM Linux).

Esto también significa que el host de contenedor de Linux (Moby VM) debe estar ejecutando demonio de Docker y a todas las dependencias del demonio de Docker.

Para ver si estás ejecutando con VM de Moby, comprueba el Administrador de Hyper-V para la VM de Moby con la IU del Administrador de Hyper-V o mediante la ejecución de `Get-VM` en una ventana de PowerShell con privilegios elevados.

## <a name="linux-containers-with-hyper-v-isolation"></a>Contenedores de Linux con aislamiento de Hyper-V

Para probar LCOW, sigue las instrucciones de contenedores de Linux en [esta guía se empieza](../quick-start/quick-start-windows-10.md)

Los contenedores de Linux con aislamiento de Hyper-V ejecutan cada contenedor de Linux (LCOW) en una VM optimizada de Linux con suficiente sistema operativo para ejecutar contenedores.  A diferencia del enfoque de VM de Moby, cada LCOW tiene su propio kernel y su propio espacio aislado de la máquina virtual.  Se también administran por Docker en Windows directamente.

![Contenedores de Linux con aislamiento de Hyper-V (LCOW)](media/lcow-approach.png)

Echar un vistazo a cómo administración de contenedor es diferente entre el enfoque de VM de Moby y LCOW, en el LCOW administración de contenedores de modelo permanece en Windows y la administración de cada LCOW se produce a través de GRPC y containerd.  Esto significa que los contenedores de Linux distro se usan para LCOW puede tener un mucho menor de inventario.  Derecha ahora estamos usando LinuxKit para utilizan los contenedores distro optimizadas, pero otros proyectos como Kata similar altamente optimizadas las distribuciones de Linux (borrar Linux), así como a crear.

Este es un vistazo cada LCOW:

![Arquitectura LCOW](media/lcow.png)

Para ver si estás ejecutando LCOW, ve a `C:\Program Files\Linux Containers`. Si Docker está configurado para usar LCOW, habrá unos cuantos archivos aquí que contiene el distro LinuxKit mínima que se ejecuta en cada contenedor que se ejecutan en el aislamiento de Hyper-V.  Ten en cuenta que los componentes VM optimizados son menos de 100 MB, mucho más pequeño que la imagen LinuxKit en VM de Moby.

### <a name="work-in-progress"></a>Trabajo en curso

LCOW está en desarrollo activo. Realizar un seguimiento del trabajo en curso del proyecto Moby en [GitHub](https://github.com/moby/moby/issues/33850)

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

Estas aplicaciones requieren la asignación de volumen y no iniciar o se ejecutará correctamente.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>Información adicional

[Blog de docker que describe LCOW](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Vídeo de contenedores de Linux](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[Kernel LCOW LinuxKit más instrucciones de compilación](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Cuándo usar vs VM de Moby LCOW

### <a name="when-to-use-moby-vm"></a>Cuándo usar VM de Moby

Derecha ahora, te recomendamos el método de VM de Moby de ejecutar contenedores de Linux para personas que:

- Quieres un entorno de contenedor estable.  Este es el valor predeterminado de Docker para Windows.
- Ejecutar contenedores de Windows o Linux, pero rara vez ambos al mismo tiempo.
- Has complicado o personalizados redes requisitos entre los contenedores de Linux.
- No es necesario el aislamiento de kernel (aislamiento de Hyper-V) entre contenedores de Linux.

### <a name="when-to-use-lcow"></a>Cuándo usar LCOW

Derecha ahora, te recomendamos LCOW a personas que:

- ¿Quieres probar nuestra tecnología más reciente.
- Ejecutar contenedores de Windows y Linux al mismo tiempo.
- Es necesario el aislamiento de kernel (aislamiento de Hyper-V) entre contenedores de Linux.

## <a name="other-options-we-considered"></a>Otras opciones consideramos

Cuando estábamos buscando en formas de ejecutar contenedores de Linux en Windows, consideramos WSL. En última instancia, elegimos un enfoque basada en virtualización para que los contenedores de Linux en Windows son coherentes con los contenedores de Linux en Linux. Uso de Hyper-V también hace que LCOW sea más segura. Podemos volver a evaluar en el futuro, pero por ahora, LCOW seguirán usando Hyper-V.

Si tienes ideas, envía comentarios a través de GitHub o UserVoice.  Agradecemos especialmente comentarios sobre la experiencia específica que te gustaría ver.
