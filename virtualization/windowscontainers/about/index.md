---
title: Acerca de los contenedores de Windows
description: Los contenedores son una tecnología para empaquetar y ejecutar aplicaciones, incluidas las aplicaciones de Windows, en diversos entornos locales y en la nube. En este tema se explica cómo Microsoft, Windows y Azure le ayudan a desarrollar e implementar aplicaciones en contenedores, como el uso del servicio de acoplamiento y Azure Kubernetes.
keywords: docker, contenedores
author: taylorb-microsoft
ms.date: 10/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: acce214cc8991f20c979b6dbe636590416841cb9
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254295"
---
# <a name="windows-and-containers"></a>Ventanas y contenedores

Los contenedores son una tecnología para empaquetar y ejecutar aplicaciones de Windows y Linux en diversos entornos locales y en la nube. Los contenedores proporcionan un entorno liviano y aislado que facilita el desarrollo, la implementación y la administración de las aplicaciones. Los contenedores se inician y se detienen rápidamente, lo que los hace ideales para las aplicaciones que necesitan adaptarse rápidamente a la cambia de demanda. La ligera naturaleza de los contenedores también les convierte en una herramienta útil para aumentar la densidad y la utilización de su infraestructura.

![Gráfico en el que se muestra cómo se pueden ejecutar contenedores en la nube o en el modo local, con compatibilidad con aplicaciones monolíticas o microservicios escritas en prácticamente cualquier idioma.](media/about-3-box.png)

## <a name="the-microsoft-container-ecosystem"></a>El ecosistema contenedor de Microsoft

Microsoft proporciona varias herramientas y plataformas para ayudarle a desarrollar e implementar aplicaciones en contenedores:

- <strong>Ejecute contenedores basados en Windows o en Linux en Windows 10</strong> para el desarrollo y las pruebas con el [escritorio de acoplamiento](https://store.docker.com/editions/community/docker-ce-desktop-windows), que usa la funcionalidad de contenedores integrada en Windows. También puede [Ejecutar contenedores de forma nativa en Windows Server](../quick-start/set-up-environment.md?tabs=Windows-Server).
- <strong>Desarrollar, probar, publicar e implementar contenedores basados en Windows</strong> con la [eficaz compatibilidad de contenedores en Visual Studio](https://docs.microsoft.com/visualstudio/containers/overview) y [Visual Studio](https://code.visualstudio.com/docs/azure/docker), que incluye compatibilidad con el acoplador, la redacción de acoplamiento, Kubernetes, Helm y otros útiles tecnologías.
- <strong>Publique las aplicaciones como imágenes de contenedor</strong> en el DockerHub público para que otras personas las usen o en un [registro privado del contenedor de Azure](https://azure.microsoft.com/services/container-registry/) para el desarrollo y la implementación de su organización, insertar y extraer directamente desde Visual Studio y código de Visual Studio. .
- <strong>Implementar contenedores a escala en Azure</strong> u otras nubes:

  - Extraer la aplicación (imagen del contenedor) de un registro de contenedor, como el registro del contenedor de Azure, y, a continuación, implementarla y administrarla a escala con un Orchestrator como el [servicio Azure Kubernetes (](https://docs.microsoft.com/azure/aks/intro-kubernetes) en versión preliminar para aplicaciones basadas en Windows) o el [servicio Azure Fabric](https://docs.microsoft.com/azure/service-fabric/).
  - El servicio Azure Kubernetes implementa contenedores en máquinas virtuales de Azure y los administra a escala, ya que son docenas de contenedores, cientos o incluso miles. Las máquinas virtuales de Azure ejecutan una imagen de Windows Server personalizada (si está implementando una aplicación basada en Windows) o una imagen de Ubuntu Linux personalizada (si está implementando una aplicación basada en Linux).
- <strong>Implemente contenedores locales</strong> con [la pila de Azure con el motor de AKS](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) (en versión preliminar con contenedores de Linux) o [con la pila de Azure con OpenShift](https://docs.microsoft.com/azure/virtual-machines/linux/openshift-azure-stack). También puede configurar Kubernetes en Windows Server (consulte [Kubernetes en Windows](../kubernetes/getting-started-kubernetes-windows.md)) y estamos trabajando en la compatibilidad con la ejecución de contenedores de [Windows en la plataforma de contenedor de RedHat OpenShift](https://techcommunity.microsoft.com/t5/Networking-Blog/Managing-Windows-containers-with-Red-Hat-OpenShift-Container/ba-p/339821) .

## <a name="how-containers-work"></a>Cómo funcionan los contenedores

Un contenedor es un silo liviano y aislado en el que se ejecuta una aplicación en el sistema operativo del host. Los contenedores se construyen sobre el núcleo del sistema operativo del host (que se puede considerar como la fontanería del sistema operativo enterrado), tal como se muestra en este diagrama.

![Diagrama de arquitectura en el que se muestra cómo se ejecutan los contenedores en la parte superior del núcleo](media/container-diagram.svg)

Mientras un contenedor comparte el núcleo del sistema operativo del host, el contenedor no obtiene acceso a él sin restricciones. En su lugar, el contenedor obtiene un aislamiento y, en algunos casos, la vista del sistema. Por ejemplo, un contenedor puede acceder a una versión virtualizada del sistema de archivos y al registro, pero los cambios afectan solo al contenedor y se descartan cuando se detiene. Para guardar datos, el contenedor puede montar almacenamiento persistente, como un [disco de Azure](https://azure.microsoft.com/services/storage/disks/) o un recurso compartido de archivos (incluidos [los archivos de Azure](https://azure.microsoft.com/services/storage/files/)).

Un contenedor se basa en la parte superior del núcleo, pero el núcleo no proporciona todas las API y servicios que una aplicación necesita ejecutar; los archivos del sistema (bibliotecas) que se ejecutan por encima del núcleo en modo de usuario. Dado que un contenedor se aísla del entorno de modo usuario del host, el contenedor necesita su propia copia de estos archivos de sistema de modo usuario, que se encuentran en algo conocido como una imagen base. La imagen base actúa como la capa básica en la que se ha construido el contenedor, proporcionándole servicios de sistema operativo no proporcionados por el núcleo. Pero hablaremos más sobre las imágenes de contenedor más adelante.

## <a name="containers-vs-virtual-machines"></a>Contenedores frente a máquinas virtuales

En contraste con un contenedor, una máquina virtual (VM) ejecuta un sistema operativo completo, incluido su propio kernel, como se muestra en este diagrama.

![Diagrama de arquitectura en el que se muestra cómo las máquinas virtuales ejecutan un sistema operativo completo junto al sistema operativo del host](media/virtual-machine-diagram.svg)

Los contenedores y las máquinas virtuales tienen sus usos, de hecho, muchas implementaciones de contenedores usan máquinas virtuales como el sistema operativo del host en lugar de ejecutarse directamente en el hardware, especialmente cuando se ejecutan contenedores en la nube.

Para obtener más información sobre las similitudes y diferencias de estas tecnologías complementarias, consulte [contenedores frente a máquinas virtuales](containers-vs-vm.md).

## <a name="container-images"></a>Imágenes de contenedor

Todos los contenedores se crean a partir de imágenes de contenedor. Las imágenes de contenedor son un conjunto de archivos organizados en una pila de capas que residen en su equipo local o en un registro de contenedor remoto. La imagen de contenedor consta de los archivos de sistema operativo de modo usuario necesarios para admitir la aplicación, la aplicación, cualquier Runtime o dependencia de la aplicación, así como cualquier otro archivo de configuración varios que la aplicación necesite ejecutar correctamente.

Microsoft ofrece varias imágenes (denominadas imágenes base) que puede usar como punto de partida para crear su propia imagen de contenedor:

* <strong>Windows</strong> : contiene el conjunto completo de las API y los servicios del sistema de Windows (menos roles de servidor).
* <strong>Windows Server Core</strong> : una imagen más pequeña que contiene un subconjunto de las API de Windows Server, es decir, el .NET Framework completo. También incluye la mayoría de los roles de servidor, aunque lamentablemente a pocos, no al servidor de fax.
* <strong>Nano Server</strong> : la imagen de Windows Server más pequeña, compatible con las API básicas de .net y algunos roles de servidor.
* <strong>Windows 10 IOT Core</strong> , una versión de Windows usada por los fabricantes de hardware para pequeñas Internet de los dispositivos que ejecutan procesadores de ARM o x86/x64.

Como se mencionó anteriormente, las imágenes de contenedor se componen de una serie de capas. Cada nivel contiene un conjunto de archivos que, cuando se superponen juntos, representan la imagen de su contenedor. Debido a la naturaleza por capas de los contenedores, no es necesario que siempre se dirija una imagen base para crear un contenedor de Windows. En su lugar, puede establecer como destino otra imagen que ya lleve el marco que desea. Por ejemplo, el equipo de .NET publica una [imagen central de .net](https://hub.docker.com/_/microsoft-dotnet-core) que transporta el tiempo de ejecución de .net Core. Evita que los usuarios necesiten duplicar el proceso de instalación de .NET Core; en su lugar, pueden reutilizar las capas de esta imagen de contenedor. La imagen del núcleo de .NET se crea en función de nano Server.

Para obtener más información, consulta [imágenes base del contenedor](../manage-containers/container-base-images.md).

## <a name="container-users"></a>Usuarios del contenedor

### <a name="containers-for-developers"></a>Contenedores para desarrolladores

Los contenedores ayudan a los desarrolladores a compilar y enviar aplicaciones de mayor calidad, más rápido. Con los contenedores, los desarrolladores pueden crear una imagen de contenedor que se implemente en segundos, de manera idéntica en todos los entornos. Los contenedores actúan como un mecanismo sencillo para compartir código entre equipos y para arrancar un entorno de desarrollo sin afectar al sistema de archivos de su host.

Los contenedores son portátiles y versátiles, pueden ejecutar aplicaciones escritas en cualquier idioma y son compatibles con cualquier máquina que ejecute Windows 10, versión 1607 o posterior, o Windows 2016 o posterior. Los programadores pueden crear y probar un contenedor de forma local en su equipo portátil o de escritorio y, a continuación, implementar esa misma imagen de contenedor en la nube privada de su empresa, en la nube pública o en el proveedor de servicios. La agilidad natural de los contenedores es compatible con los patrones de desarrollo de aplicaciones modernos en entornos de nube virtualizados a gran escala.

### <a name="containers-for-it-professionals"></a>Contenedores para profesionales de ti

Los contenedores ayudan a los administradores a crear una infraestructura que sea más fácil de actualizar y mantener, y que utilice más por completo los recursos de hardware. Los profesionales de TI pueden usar contenedores para proporcionar entornos estandarizados para los equipos de desarrollo, QA y producción. Mediante el uso de contenedores, los administradores de sistemas abstraen las diferencias entre las instalaciones de sistemas operativos y la infraestructura subyacente.

## <a name="container-orchestration"></a>Orquestación de contenedor

Los Orchestrator son una parte crítica de la infraestructura al configurar un entorno basado en contenedor. Aunque puede administrar unos cuantos contenedores manualmente con Docker y Windows, las aplicaciones suelen usar cinco, diez o incluso cientos de contenedores, que es donde se encuentran los Orchestrator.

Los Orchestrator de contenedor se han creado para ayudar a administrar contenedores en la escala y en la producción. Los Orchestrator proporcionan funcionalidad para:

- Implementación en escala
- Programación de carga de trabajo
- Supervisión de estado
- Conmutando por error cuando se produce un error en un nodo
- Escalar hacia arriba o hacia abajo
- Redes
- Detección de servicios
- Coordinación de las actualizaciones de aplicaciones
- Afinidad de nodo de clúster

Hay muchos Orchestrator diferentes que puede usar con contenedores de Windows; Estas son las opciones que ofrece Microsoft:
- [Servicio Azure Kubernetes (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes) : usar un servicio de Kubernetes de Azure administrado
- [Azure Service fabric](https://docs.microsoft.com/azure/service-fabric/) : usar un servicio administrado
- [Pila de Azure con el motor de AKS](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) -usar el servicio de Azure Kubernetes local
- [Kubernetes en Windows](../kubernetes/getting-started-kubernetes-windows.md) : configura Kubernetes en Windows

## <a name="try-containers-on-windows"></a>Probar contenedores en Windows

Para empezar a usar contenedores en Windows Server o Windows 10, vea lo siguiente:
> [!div class="nextstepaction"]
> [Introducción: configurar el entorno para contenedores](../quick-start/set-up-environment.md)

Para obtener ayuda para decidir qué servicios de Azure son adecuados para su escenario, vea [Azure Container Services](https://azure.microsoft.com/product-categories/containers/) y [elija qué servicios de Azure usar para hospedar su aplicación](https://docs.microsoft.com/azure/architecture/guide/technology-choices/compute-decision-tree).
