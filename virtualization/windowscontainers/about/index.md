---
title: Acerca de los contenedores de Windows
description: Obtenga información sobre los contenedores de Windows.
keywords: docker, contenedores
author: taylorb-microsoft
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: fc50938d87aa49c3c57bf3b1172e69a6125ed96a
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2019
ms.locfileid: "9680955"
---
# <a name="about-windows-containers"></a>Acerca de los contenedores de Windows

Imagina una cocina. Dentro de este espacio hay todo lo que necesitas para comer una comida: el horno, las pan, la lavabo, etc. Este es nuestro contenedor.

![Ilustración de una cocina completada con un papel tapiz amarillo dentro de un cuadro negro.](media/box1.png)

Ahora imagínese colocar esta cocina dentro de un edificio tan fácilmente como deslizar un libro en una estantería. Puesto que todo lo que la cocina necesita funcionar ya es así, todo lo que necesitamos para comenzar a cocina es conectar la electricidad y la fontanería.

![Una construcción de apartamentos formada por dos pilas de cuadros negros. Cuatro de estas casillas son los mismos cuadros amarillos que se usan en el ejemplo de la cocina y se colocan en lugares aleatorios en el edificio, mientras que el resto son las salas de espera de color o están vacías o están atenuadas.](media/apartment.png)

<<<<<<<, ¿por qué detener? Puede personalizar su edificio de la manera que más le guste; rellenarlo con muchos tipos de salas, rellenarlo con habitaciones idénticas o tener una mezcla de ambos.
=======
![](media/box1.png)
>>>>>>> origen o maestra

Los contenedores actúan como este salón ejecutando una aplicación como en nuestra cocina. Un contenedor coloca una aplicación y todo lo que la aplicación necesita para ejecutarse en su propio cuadro aislado. Como resultado, la aplicación aislada no tiene conocimiento alguno de las demás aplicaciones o procesos que existen fuera de su contenedor. Dado que el contenedor tiene todo lo que la aplicación necesita ejecutar, el contenedor se puede mover a cualquier parte, usando solo los recursos de su host, sin tocar ningún recurso suministrado para otros contenedores.

CABEZA de <<<<<<< en el siguiente vídeo encontrará más información sobre lo que pueden hacer los contenedores de Windows, así como la colaboración de Microsoft con el acoplador para crear un entorno sin dificultades para el desarrollo de contenedores de código abierto: = = = = = = =
![](media/apartment.png)
>>>>>>> origen o maestra

<iframe width="800" height="450" src="https://www.youtube.com/embed/Ryx3o0rD5lY" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## <a name="container-fundamentals"></a>Aspectos básicos de los contenedores

<<<<<<< HEAD vamos a conocer algunos términos que le resultarán útiles a la vez que comienza a trabajar con contenedores de Windows: = = = = = = =
## <a name="container-fundamentals"></a>Conceptos básicos de los contenedores

Los contenedores son un entorno de ejecución portátil, aislado y controlado por recursos que se ejecuta en una máquina host o virtual. Una aplicación o proceso que se ejecuta en un contenedor incluye todos los archivos de configuración y dependencias necesarios, por tanto, tiene la sensación de que no hay otros procesos en ejecución fuera del contenedor.

El host del contenedor aprovisiona un conjunto de recursos para el contenedor y este solo usará estos recursos. En lo que respecta al contenedor, no existen otros recursos fuera de lo que se ha proporcionado y, por lo tanto, el contenedor no puede tocar los recursos que pueden haberse suministrado para un contenedor vecino.

Los siguientes conceptos clave te resultarán útiles cuando empieces a crear y trabajar con contenedores de Windows.

**Host de contenedor:** equipo físico o virtual configurado con la característica de contenedor de Windows. El host de contenedor ejecutará uno o varios contenedores de Windows.

**Imagen de contenedor:** a medida que se realicen modificaciones en un registro o un sistema de archivos de contenedores, como durante la instalación de software, se capturan en un espacio aislado. En muchos casos, querrás capturar este estado de forma que se puedan crear nuevos contenedores que heredan estos cambios. Precisamente eso es una imagen: cuando se ha detenido el contenedor, puede descartar ese espacio aislado o puede convertirlo en una nueva imagen de contenedor. Por ejemplo, imaginemos que ha implementado un contenedor a partir de la imagen del sistema operativo de Windows Server Core. Después instala MySQL en este contenedor. La creación de una nueva imagen de este contenedor actuaría como una versión que se puede implementar del contenedor. Esta imagen solo contendría los cambios realizados (MySQL), aunque funcionaría como una capa encima de la imagen del sistema operativo del contenedor.

**Espacio aislado:** cuando se haya iniciado un contenedor, todas las acciones de escritura como las modificaciones del sistema de archivos, las modificaciones del Registro o las instalaciones de software se capturan en esta capa de "espacio aislado".

**Imagen del sistema operativo de contenedor:** los contenedores se implementan a partir de imágenes. La imagen del sistema operativo de contenedor es la primera capa de potencialmente muchas capas de imagen que componen un contenedor. Esta imagen ofrece el entorno del sistema operativo. Una imagen de sistema operativo del contenedor es inmutable. Es decir, no se puede modificar.

**Repositorio de contenedor:** cada vez que se crea una imagen de contenedor, esta y sus dependencias se almacenan en un repositorio local. Estas imágenes se pueden reutilizar muchas veces en el host de contenedor. Las imágenes de contenedor también pueden almacenarse en un registro público o privado, como DockerHub, de forma que se puedan usar en varios hosts de contenedor diferentes.

![](media/containerfund.png)

Para alguien familiarizado con las máquinas virtuales, puede parecer que los contenedores son increíblemente similares. Un contenedor ejecuta un sistema operativo, tiene un sistema de archivos y se puede acceder a él a través de una red, como si fuese un equipo físico o virtual. Dicho esto, la tecnología y los conceptos relacionados con los contenedores son muy diferentes de las máquinas virtuales.
>>>>>>> origen o maestra

- Host de contenedor: un sistema de equipo físico o virtual configurado con la característica contenedor de Windows. El host contenedor ejecutará uno o varios contenedores de Windows.
- Espacio aislado: la capa que captura todos los cambios que realice en el contenedor mientras se está ejecutando (como las modificaciones del sistema de archivos, las modificaciones del registro o las instalaciones de software).
- Imagen base: primera capa de las capas de imagen de un contenedor que proporciona el entorno del sistema operativo del contenedor. No se puede modificar una imagen base.
- Imagen de contenedor: plantilla de solo lectura de instrucciones para crear un contenedor. Las imágenes se pueden basar en un entorno de sistema operativo básico y sin modificar, pero también se pueden crear desde el recinto de un contenedor modificado. Estas imágenes modificadas supercapan sus cambios en la parte superior de la capa de la imagen base, y estas capas se pueden copiar y volver a aplicar a otras imágenes base para crear una nueva imagen con los mismos cambios.
- Repositorio de contenedores: el repositorio local que almacena la imagen de contenedor y sus dependencias cada vez que se crea una imagen nueva. Puede reutilizar las imágenes almacenadas tantas veces como desee en el host contenedor. También puede almacenar las imágenes de contenedor en un registro público o privado, como un concentrador acoplador, para que se puedan usar en muchos hosts de contenedor diferentes.
- Contenedor Orchestrator: un proceso que automatiza y administra un gran número de contenedores y cómo interactúan entre sí. Para obtener más información, consulte [acerca de los Windows Container orchestrators](overview-container-orchestrators.md).
- Dock: proceso automatizado que empaqueta y entrega imágenes de contenedor. Para obtener más información, vea la [información general](docker-overview.md)del acoplador, el [motor del acoplador en Windows](../manage-docker/configure-docker-daemon.md) o visite el [sitio web](https://www.docker.com)del acoplador.

![Diagrama de flujo que muestra cómo se crean los contenedores. Las imágenes de la aplicación y la base se usan para crear un espacio aislado y una nueva imagen de la aplicación, que se superponer en la parte superior de la imagen base para crear un nuevo contenedor.](media/containerfund.png)

Alguien que está familiarizado con máquinas virtuales puede pensar que los contenedores y las máquinas virtuales parecen ser similares. Un contenedor ejecuta un sistema operativo, tiene un sistema de archivos y se puede tener acceso a él a través de una red, de forma similar a un sistema informático físico o virtual. Dicho esto, la tecnología y los conceptos relacionados con los contenedores son muy diferentes de las máquinas virtuales. Para obtener más información sobre estos conceptos, lea el [blog](https://azure.microsoft.com/blog/containers-docker-windows-and-trends/) de Mark Russinovich en el que se explican las diferencias de forma más detallada.

### <a name="windows-container-types"></a>Tipos de contenedor de Windows

Otra cosa que debe saber es que hay dos tipos de contenedores diferentes, también conocidos como runtimes.

Los contenedores de Windows Server proporcionan aislamiento de aplicaciones mediante procesos y tecnología de aislamiento de espacios de nombres, motivo por el que estos contenedores también se conocen como contenedores aislados por proceso. Un contenedor de Windows Server comparte el kernel con el host de contenedor y todos los contenedores que se ejecutan en el host. Estos contenedores aislados de proceso no proporcionan un límite de seguridad hostil y no se deben usar para aislar el código que no es de confianza. Dado el espacio de kernel compartido, estos contenedores requieren la misma configuración y versión de kernel.

El aislamiento de Hyper-V se expande en el aislamiento proporcionado por los contenedores de Windows Server ejecutando cada contenedor en una máquina virtual muy optimizada. En esta configuración, el host del contenedor no comparte su núcleo con otros contenedores en el mismo host. Estos contenedores se han diseñado para el hospedaje multiinquilino hostil con las mismas garantías de seguridad de una máquina virtual. Dado que estos contenedores no comparten el núcleo con el host u otros contenedores en el host, pueden ejecutar núcleos con diferentes versiones y configuraciones (dentro de las versiones compatibles). Por ejemplo, todos los contenedores de Windows en Windows 10 usan el aislamiento de Hyper-V para usar la configuración y la versión del kernel de Windows Server.

Ejecutar un contenedor en Windows con o sin el aislamiento Hyper-V es una decisión en tiempo de ejecución. Inicialmente, puede crear el contenedor con el aislamiento de Hyper-V y, posteriormente, en tiempo de ejecución, elegir ejecutarlo como un contenedor de Windows Server.

<<<<<<< ENCABEZADO
## <a name="container-users"></a>Usuarios del contenedor
=======
![](media/docker.png)
>>>>>>> origen o maestra

### <a name="containers-for-developers"></a>Contenedores para desarrolladores

Los contenedores ayudan a los desarrolladores a crear y enviar aplicaciones de mayor calidad más rápidamente. Los programadores pueden crear una imagen de acoplador que se implementará de manera idéntica en todos los entornos en segundos. Hay un gran sistema de aplicaciones en crecimiento masivo en contenedores de Dock. DockerHub, un registro de aplicaciones de contenedor público mantenido por Dock, ha publicado más de 180.000 aplicaciones en el repositorio de la comunidad pública, y ese número sigue creciendo.

Cuando un desarrollador containerizes una aplicación, solo la aplicación y los componentes que necesita ejecutar se combinan en una imagen. Los contenedores se crean después a partir de esta imagen según sea necesario. También puede utilizar una imagen como una línea de base para crear otra imagen, con lo que la creación de imágenes será aún más rápida. Varios contenedores pueden compartir la misma imagen, lo que significa que los contenedores se inician muy rápidamente y utilizan menos recursos. Por ejemplo, un desarrollador puede usar contenedores para girar componentes de aplicaciones ligeros y portátiles, también conocidos como microservicios, para aplicaciones distribuidas y escalar rápidamente cada servicio por separado.

Los contenedores son portátiles y versátiles, pueden escribirse en cualquier idioma y son compatibles con cualquier equipo que ejecute Windows Server 2016. Los programadores pueden crear y probar un contenedor de forma local en su equipo portátil o de escritorio y, a continuación, implementar esa misma imagen de contenedor en la nube privada de su empresa, en la nube pública o en el proveedor de servicios. La agilidad natural de los contenedores es compatible con los patrones de desarrollo de aplicaciones modernos en entornos de nube virtualizados a gran escala.

### <a name="containers-for-it-professionals"></a>Contenedores para profesionales de ti

Los contenedores ayudan a los administradores a crear una infraestructura más fácil de actualizar y mantener. Los profesionales de TI pueden usar contenedores para proporcionar entornos estandarizados para los equipos de desarrollo, QA y producción. Ya no tienen que preocuparse por los procedimientos complejos de instalación y configuración. Mediante el uso de contenedores, los administradores de sistemas abstraen las diferencias entre las instalaciones del sistema operativo y la infraestructura subyacente.

## <a name="containers-101-video-presentation"></a>Presentación de vídeo de contenedores 101

La siguiente presentación de vídeo le proporcionará una información general más detallada sobre el historial y la implementación de contenedores de Windows.

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## <a name="try-windows-server-containers"></a>Usar contenedores de Windows Server

¿Estás listo para comenzar a sacar partido de la increíble potencia de los contenedores? Los artículos siguientes le ayudarán a empezar:

Para configurar un contenedor en Windows Server, vea el [tutorial rápido de Windows Server](../quick-start/quick-start-windows-server.md).

Para configurar un contenedor en Windows 10, consulta el [Inicio rápido de Windows 10](../quick-start/quick-start-windows-10.md).