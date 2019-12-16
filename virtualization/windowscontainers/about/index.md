---
title: Acerca de los contenedores de Windows
description: Los contenedores son una tecnología para empaquetar y ejecutar aplicaciones, incluidas las aplicaciones de Windows, en diversos entornos locales y en la nube. En este tema se describe cómo Microsoft, Windows y Azure le ayudan a desarrollar e implementar aplicaciones en contenedores, incluido el uso de Docker y Azure Kubernetes Service.
keywords: docker, contenedores
author: taylorb-microsoft
ms.date: 10/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 4fad299db2c897a6be860ef0cc71e80969c75357
ms.sourcegitcommit: 8dedb887b038fbff872327f51c7416454b301b86
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/06/2019
ms.locfileid: "74909415"
---
# <a name="windows-and-containers"></a>Windows y contenedores

Los contenedores son una tecnología para empaquetar y ejecutar aplicaciones de Windows y Linux en diversos entornos locales y en la nube. Los contenedores proporcionan un entorno ligero y aislado que facilita el desarrollo, implementación y administración de las aplicaciones. Los contenedores se inician y detienen rápidamente, por lo que son ideales para las aplicaciones que necesitan adaptarse rápidamente a la demanda cambiante. La naturaleza ligera de los contenedores también los convierte en una herramienta útil para aumentar la densidad y el uso de la infraestructura.

![Gráfico en el que se muestra cómo se pueden ejecutar los contenedores en la nube o de forma local para admitir aplicaciones monolíticas o microservicios escritos en casi cualquier lenguaje.](media/about-3-box.png)

## <a name="the-microsoft-container-ecosystem"></a>El ecosistema de contenedores de Microsoft

Microsoft ofrece una serie de herramientas y plataformas que le ayudarán a desarrollar e implementar aplicaciones en contenedores:

- <strong>Ejecute contenedores basados en Windows o Linux en Windows 10</strong> para procesos de desarrollo y pruebas con [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows), que usa la funcionalidad de contenedores integrada en Windows. También puede [ejecutar contenedores de forma nativa en Windows Server](../quick-start/set-up-environment.md?tabs=Windows-Server).
- <strong>Desarrolle, pruebe, publique e implemente contenedores basados en Windows</strong> gracias a la [eficaz compatibilidad con contenedores en Visual Studio](https://docs.microsoft.com/visualstudio/containers/overview) y [Visual Studio Code](https://code.visualstudio.com/docs/azure/docker), que incluyen compatibilidad con Docker, Docker Compose, Kubernetes, Helm y otras tecnologías útiles.
- <strong>Publique sus aplicaciones como imágenes de contenedor</strong> en DockerHub público para que las usen otros usuarios, o en una instancia privada de [Azure Container Registry](https://azure.microsoft.com/services/container-registry/) para los procesos de desarrollo e implementación de su organización. De este modo, podrá enviar ("push") e incorporar ("pull") cambios directamente desde Visual Studio y Visual Studio Code.
- <strong>Implemente contenedores a escala en Azure</strong> u otras nubes:

  - Extraiga la aplicación (imagen de contenedor) de un registro de contenedor (como Azure Container Registry) y después impleméntela y adminístrela a escala mediante un orquestador como [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes) (en versión preliminar para aplicaciones basadas en Windows) o [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/).
  - Azure Kubernetes Service implementa contenedores en máquinas virtuales de Azure y los administra a escala, independientemente de si son docenas de contenedores, cientos o incluso miles. Las máquinas virtuales de Azure ejecutan una imagen de Windows Server personalizada (si va a implementar una aplicación basada en Windows) o una imagen de Ubuntu Linux personalizada (si va a implementar una aplicación basada en Linux).
- <strong>Implemente contenedores locales</strong> mediante [Azure Stack con el motor de AKS](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) (en versión preliminar con contenedores de Linux) o [Azure Stack con OpenShift](https://docs.microsoft.com/azure/virtual-machines/linux/openshift-azure-stack). También puede configurar Kubernetes en Windows Server (consulte [Kubernetes en Windows](../kubernetes/getting-started-kubernetes-windows.md)) y estamos trabajando en admitir la ejecución de [contenedores de Windows en RedHat OpenShift Container Platform](https://techcommunity.microsoft.com/t5/Networking-Blog/Managing-Windows-containers-with-Red-Hat-OpenShift-Container/ba-p/339821).

## <a name="how-containers-work"></a>Cómo funcionan los contenedores

Un contenedor es un silo aislado y ligero para ejecutar una aplicación en el sistema operativo host. Los contenedores se basan en el kernel del sistema operativo host (que puede considerarse la fontanería del sistema operativo), tal como se muestra en este diagrama.

![Diagrama arquitectónico en el que se muestra cómo se ejecutan los contenedores en un nivel superior del kernel](media/container-diagram.svg)

Aunque un contenedor comparte el kernel del sistema operativo host, no obtiene acceso sin restricciones a dicho kernel. En su lugar, el contenedor obtiene una vista aislada (y, en ocasiones, virtualizada) del sistema. Por ejemplo, un contenedor puede tener acceso a una versión virtualizada del sistema de archivos y el registro, pero los cambios solo afectan al contenedor y se descartan cuando se detiene. Para guardar los datos, el contenedor puede montar una instancia de almacenamiento persistente, como [Azure Disk](https://azure.microsoft.com/services/storage/disks/) o un recurso compartido de archivos (incluido [Azure Files](https://azure.microsoft.com/services/storage/files/)).

Un contenedor se basa en el kernel, pero el kernel no proporciona todas las API y servicios que una aplicación necesita para ejecutarse; la mayoría de estos últimos se proporcionan a través de archivos del sistema (bibliotecas) que se ejecutan sobre el kernel en modo de usuario. Dado que un contenedor está aislado del entorno de modo de usuario del host, el contenedor necesita su propia copia de estos archivos del sistema de modo de usuario, que se empaquetan en algo conocido como imagen base. La imagen base cumple el papel de la capa fundamental en la que se basa el contenedor y le proporciona los servicios de sistema operativo que no ofrece el kernel. Sin embargo, mencionaremos más detalles acerca de las imágenes de contenedor más adelante.

## <a name="containers-vs-virtual-machines"></a>Contenedores frente a máquinas virtuales

A diferencia de un contenedor, una máquina virtual (VM) ejecuta un sistema operativo completo, incluido su propio kernel, tal como se muestra en este diagrama.

![Diagrama arquitectónico en el que se muestra cómo las máquinas virtuales ejecutan un sistema operativo completo junto al sistema operativo host](media/virtual-machine-diagram.svg)

Los contenedores y las máquinas virtuales tienen sus usos particulares; de hecho, muchas implementaciones de contenedores usan máquinas virtuales como sistema operativo host en lugar de ejecutarse directamente en el hardware, sobre todo cuando se ejecutan contenedores en la nube.

Para obtener más información sobre las similitudes y las diferencias de estas tecnologías complementarias, consulte [Contenedores frente a máquinas virtuales](containers-vs-vm.md).

## <a name="container-images"></a>Imágenes del contenedor

Todos los contenedores se crean a partir de imágenes de contenedor. Las imágenes de contenedor son un conjunto de archivos organizados en una pila de capas que se hospedan en el equipo local o en un registro de contenedor remoto. La imagen de contenedor está compuesta por los archivos de sistema operativo del modo de usuario necesarios para admitir la aplicación, los entornos de ejecución o las dependencias de la aplicación, y cualquier otro archivo de configuración que la aplicación necesite para ejecutarse correctamente.

Microsoft ofrece varias imágenes (denominadas imágenes base) que puede usar como punto de partida para crear su propia imagen de contenedor:

* <strong>Windows</strong>: contiene el conjunto completo de API y servicios del sistema de Windows (menos los roles de servidor).
* <strong>Windows Server Core</strong>: una imagen más pequeña que contiene un subconjunto de las API de Windows Server; es decir, la versión completa de .NET Framework. También incluye la mayoría de los roles de servidor, aunque lamentablemente, no el servidor de fax.
* <strong>Nano Server</strong>: la imagen de Windows Server más pequeña, compatible con las API de .NET Core y algunas funciones de servidor.
* <strong>Windows 10 IoT Core</strong>: una versión de Windows usada por los fabricantes de hardware para pequeños dispositivos con Internet de las cosas que ejecutan procesadores de ARM o x86/x64.

Como se mencionó anteriormente, las imágenes de contenedor se componen de una serie de capas. Cada capa contiene un conjunto de archivos que, cuando se superponen, representan la imagen de contenedor. Debido a la naturaleza por capas de los contenedores, no es necesario que siempre tenga como destino una imagen base para compilar un contenedor de Windows. En su lugar, puede elegir como destino otra imagen que ya contenga el marco de trabajo que quiere. Por ejemplo, el equipo de .NET publica una [imagen de .NET Core](https://hub.docker.com/_/microsoft-dotnet-core) que incluye el entorno de ejecución de .NET Core. Esto evita que los usuarios tengan que realizar nuevamente el proceso de instalación de .NET Core. En su lugar, pueden volver a usar las capas de esta imagen de contenedor. La imagen de .NET Core está basada en Nano Server.

Para obtener más información, consulte [Imágenes base del contenedor](../manage-containers/container-base-images.md).

## <a name="container-users"></a>Usuarios de los contenedores

### <a name="containers-for-developers"></a>Contenedores para desarrolladores

Los contenedores ayudan a los desarrolladores a crear y distribuir aplicaciones de mayor calidad, más rápido. Con los contenedores, los desarrolladores pueden crear una imagen de contenedor que se implementa en segundos de forma idéntica en todos los entornos. Los contenedores actúan como un mecanismo sencillo para compartir código entre equipos y arrancar un entorno de desarrollo sin afectar al sistema de archivos del host.

Los contenedores son portátiles y versátiles, pueden ejecutar aplicaciones escritas en cualquier lenguaje y son compatibles con cualquier equipo que ejecute Windows 10, versión 1607 o posterior, o Windows Server 2016 o posterior. Los desarrolladores pueden crear y probar contenedores localmente en su equipo portátil o de escritorio y luego implementar esa misma imagen de contenedor en la nube privada, la nube pública o el proveedor de servicios de su empresa. La agilidad natural de los contenedores admite patrones de desarrollo de aplicaciones modernas en entornos de nube virtualizados y a gran escala.

### <a name="containers-for-it-professionals"></a>Contenedores para los profesionales de TI

Los contenedores ayudan a los administradores a crear una infraestructura que sea más fácil de actualizar y mantener, y que el uso de los recursos de hardware sea más completo. Los profesionales de TI pueden utilizar los contenedores para ofrecer entornos estandarizados para su desarrollo, sus controles de calidad y sus equipos de producción. Mediante el uso de contenedores, los administradores de sistemas abstraen las diferencias en las instalaciones de sistemas operativos y la infraestructura subyacente.

## <a name="container-orchestration"></a>Orquestación de contenedores

Los orquestadores son una parte fundamental de la infraestructura al configurar un entorno basado en contenedores. Aunque puede administrar algunos contenedores manualmente mediante Docker y Windows, las aplicaciones suelen usar cinco, diez o incluso cientos de contenedores, donde se encuentran los orquestadores.

Los orquestadores de contenedor se crearon para ayudar a administrar contenedores a escala y en producción. Los orquestadores proporcionan las funcionalidades para:

- Implementación a escala
- Programación de cargas de trabajo
- Supervisión de estado
- Conmutación por error cuando se produce un error en un nodo
- Escalado o reducción vertical
- Funciones de red
- Detección de servicios
- Coordinación de las actualizaciones de aplicaciones
- Afinidad de nodos de clúster

Hay muchos orquestadores diferentes que puede utilizar con los contenedores de Windows. Estas son las opciones que proporciona Microsoft:
- [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes): use un servicio administrado de Azure Kubernetes.
- [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/): use un servicio administrado.
- [Azure Stack con el motor de AKS](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview): use Azure Kubernetes Service de forma local.
- [Kubernetes en Windows](../kubernetes/getting-started-kubernetes-windows.md): configure Kubernetes en Windows.

## <a name="try-containers-on-windows"></a>Prueba de contenedores en Windows

Para empezar a trabajar con contenedores en Windows Server o Windows 10, consulte lo siguiente:
> [!div class="nextstepaction"]
> [Introducción: configuración del entorno para los contenedores](../quick-start/set-up-environment.md)

Para obtener ayuda a la hora de decidir qué servicios de Azure son adecuados para su escenario, consulte [Azure Container Services](https://azure.microsoft.com/product-categories/containers/) y [Selección de los servicios de Azure que se usarán para hospedar la aplicación](https://docs.microsoft.com/azure/architecture/guide/technology-choices/compute-decision-tree).
