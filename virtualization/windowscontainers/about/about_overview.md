---
title: Acerca de los contenedores de Windows
description: "Obtenga información sobre los contenedores de Windows."
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
translationtype: Human Translation
ms.sourcegitcommit: cfa3c14e932f8b86edf6667200ac028ea0a16b67
ms.openlocfilehash: 180ebc929e4203973ac5e0b4e108777b6c90b0fc

---

# Contenedores de Windows

**Esto es contenido preliminar y está sujeto a cambios.** 

Las aplicaciones impulsan la innovación en la era móvil y de la nube. Los contenedores y el ecosistema que se está desarrollando en torno a ellos ofrecerán a los desarrolladores de software lo necesario para crear la próxima generación de experiencias de aplicaciones.

Vea una breve introducción: [Windows-based containers: Modern app development with enterprise-grade control](https://youtu.be/Ryx3o0rD5lY) (Contenedores basados en Windows: desarrollo de aplicaciones modernas con control de nivel empresarial).

## Qué son los contenedores

Son un entorno operativo portátil, aislado y controlado por recursos.

Básicamente, un contenedor es un lugar aislado donde una aplicación puede ejecutarse sin afectar al resto del sistema y sin que el sistema afecte a la aplicación. Los contenedores son la siguiente evolución en el campo de la virtualización.

Si estuviera dentro de un contenedor, parecería que está dentro de una máquina virtual o un equipo físico recién instalados. Y, para [Docker](https://www.docker.com/), un contenedor de Windows Server se puede administrar de la misma forma que cualquier otro contenedor.

## Tipos de contenedores de Windows

Los contenedores de Windows incluyen dos tipos diferentes de contenedores o tiempos de ejecución.

**Contenedores de Windows Server**: ofrecen aislamiento de aplicaciones mediante tecnología de aislamiento de procesos y espacios de nombres. Un contenedor de Windows Server comparte el kernel con el host de contenedor y todos los contenedores que se ejecutan en el host.

**Contenedores de Hyper-V**: amplían el aislamiento que ofrecen los contenedores de Windows Server mediante la ejecución de cada contenedor en una máquina virtual altamente optimizada. En esta configuración, el kernel del host de contenedor no se comparte con los contenedores de Hyper-V.


## Conceptos básicos de los contenedores

Al empezar a trabajar con contenedores, observará muchas similitudes entre un contenedor y una máquina virtual. Un contenedor ejecuta un sistema operativo, tiene un sistema de archivos y se puede acceder a él a través de una red, como si fuese un equipo físico o virtual. Dicho esto, la tecnología y los conceptos relacionados con los contenedores son muy diferentes de las máquinas virtuales.  

[Esta entrada de blog](http://azure.microsoft.com/blog/2015/08/17/containers-docker-windows-and-trends/) de Mark Russinovich explica los contenedores muy bien.

Los siguientes conceptos clave le resultarán útiles cuando empiece a crear y trabajar con contenedores de Windows. 

**Host de contenedor:** equipo físico o virtual configurado con la característica de contenedor de Windows. El host de contenedor ejecutará uno o varios contenedores de Windows.

**Imagen de contenedor:** a medida que se realicen modificaciones en un Registro o un sistema de archivos de contenedores, como durante la instalación de software, se capturan en un espacio aislado.  En muchos casos, querrá capturar este estado de forma que se puedan crear nuevos contenedores que heredan estos cambios. Precisamente eso es una imagen: cuando se ha detenido el contenedor, puede descartar ese espacio aislado o puede convertirlo en una nueva imagen de contenedor. Por ejemplo, imaginemos que ha implementado un contenedor a partir de la imagen del sistema operativo de Windows Server Core. Después instala MySQL en este contenedor. La creación de una nueva imagen de este contenedor actuaría como una versión que se puede implementar del contenedor. Esta imagen solo contendría los cambios realizados (MySQL), aunque funcionaría como una capa encima de la imagen del sistema operativo del contenedor.

**Espacio aislado:** cuando se haya iniciado un contenedor, todas las acciones de escritura como las modificaciones del sistema de archivos, las modificaciones del Registro o las instalaciones de software se capturan en esta capa de "espacio aislado".  
 
**Imagen del sistema operativo de contenedor:** los contenedores se implementan a partir de imágenes. La imagen del sistema operativo de contenedor es la primera capa de potencialmente muchas capas de imagen que componen un contenedor. Esta imagen ofrece el entorno del sistema operativo. Una imagen del sistema operativo de contenedor es inmutable, no se puede modificar.

**Repositorio de contenedor:** cada vez que se crea una imagen de contenedor, esta y sus dependencias se almacenan en un repositorio local. Estas imágenes se pueden reutilizar muchas veces en el host de contenedor. Las imágenes de contenedor también pueden almacenarse en un registro público o privado, como DockerHub, de forma que se puedan usar en varios host de contenedor diferentes.

<center>![](media/containerfund.png)</center>

## Contenedores para desarrolladores

Del escritorio de un desarrollador a una máquina de pruebas para un conjunto de equipos de producción, se puede crear una imagen de Docker que se implementará exactamente igual en cualquier entorno en segundos. Este artículo ha creado un gran ecosistema en crecimiento de aplicaciones empaquetadas en contenedores de Docker, con DockerHub, el registro de aplicaciones en contenedores públicos que mantiene Docker. Actualmente hay publicadas más de 180.000 aplicaciones en el repositorio público de la comunidad.  

Cuando incluya una aplicación en un contenedor, solamente la aplicación y los componentes necesarios para ejecutarla se combinan en una "imagen". Los contenedores se crean después a partir de esta imagen según sea necesario. También puede utilizar una imagen como una línea de base para crear otra imagen, con lo que la creación de imágenes será aún más rápida.  Varios contenedores pueden compartir la misma imagen, lo que significa que los contenedores se inician con mucha rapidez y utilizan menos recursos. Por ejemplo, puede utilizar los contenedores para poner en marcha componentes de aplicaciones portátiles y ligeros (o "microservicios") para aplicaciones distribuidas y escalar rápidamente cada servicio por separado.

Como el contenedor tiene todo que lo necesario para ejecutar la aplicación, son muy portátiles y se pueden ejecutar en cualquier equipo que tenga Windows Server 2016. Puede crear y probar contenedores localmente y luego implementar esa misma imagen de contenedor en la nube privada, la nube pública o el proveedor de servicios de su empresa. La agilidad natural de los contenedores admite patrones de desarrollo de aplicaciones modernas en entornos de nube, virtualizados y a gran escala.

Con los contenedores, los desarrolladores pueden crear una aplicación en cualquier lenguaje. Estas aplicaciones son totalmente portátiles y pueden ejecutarse en cualquier lugar (portátil, equipo de escritorio, servidor, nube privada, nube pública o proveedor de servicios) sin cambios de código.  

Los contenedores ayudan a los desarrolladores a crear y distribuir aplicaciones de mayor calidad, más rápido.

## Contenedores para los profesionales de TI ##

Los profesionales de TI pueden utilizar los contenedores para ofrecer entornos estandarizados para su desarrollo, sus controles de calidad y sus equipos de producción. Ya no tienen que preocuparse de los complicados pasos de instalación y configuración. Mediante el uso de contenedores, los administradores de sistemas abstraen las diferencias de las instalaciones de sistemas operativos y la infraestructura subyacente.

Los contenedores ayudan a los administradores a crear una infraestructura que sea más fácil de actualizar y mantener.

## Vídeo de introducción

<iframe 
src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>


## Pruebe los contenedores de Windows Server

[Introducción a los contenedores](../quick_start/quick_start.md)




<!--HONumber=Jun16_HO4-->


