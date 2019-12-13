---
title: Plataforma de contenedor de Windows
description: Obtenga más información sobre los nuevos bloques de creación de contenedores disponibles en Windows.
keywords: LCOW, contenedores de Linux, Docker, contenedores, containerd, CRI, runhcs, Runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 74e22702aa4be30055b3f4f48c7fac926d793095
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909925"
---
# <a name="container-platform-tools-on-windows"></a>Herramientas de la plataforma de contenedor en Windows

La plataforma de contenedores de Windows se está expandiendo. Docker fue la primera parte del recorrido del contenedor, ya que estamos creando otras herramientas de plataforma de contenedor.

* [containerd/CRI](https://github.com/containerd/cri) -New en Windows Server 2019/Windows 10 1809.
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) : un host de contenedor de Windows equivalente a Runc.
* [HCS](https://docs.microsoft.com/virtualization/api/) : el servicio de proceso de host + correcciones de compatibilidad (shim) útiles para que sea más fácil de usar.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

En este artículo se hablará de la plataforma de contenedores de Windows y Linux, así como de cada herramienta de plataforma de contenedor.

## <a name="windows-and-linux-container-platform"></a>Plataforma de contenedor de Windows y Linux

En entornos de Linux, las herramientas de administración de contenedores como Docker se basan en un conjunto más granular de herramientas de contenedor: [Runc](https://github.com/opencontainers/runc) y [contenedores](https://containerd.io/).

![Arquitectura de Docker en Linux](media/docker-on-linux.png)

`runc` es una herramienta de línea de comandos de Linux para crear y ejecutar contenedores de acuerdo con la [especificación del tiempo de ejecución del contenedor OCI](https://github.com/opencontainers/runtime-spec).

`containerd` es un demonio que administra el ciclo de vida del contenedor desde descargar y desempaquetar la imagen del contenedor para la ejecución y supervisión del contenedor.

En Windows, se llevó a cabo un enfoque diferente.  Cuando comenzamos a trabajar con Docker para admitir contenedores de Windows, creamos directamente en HCS (servicio de proceso de host).  [Esta entrada de blog](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332) está llena de información sobre por qué se ha creado el HCS y por qué se ha tomado este enfoque a los contenedores inicialmente.

![Arquitectura inicial del motor de Docker en Windows](media/hcs.png)

En este momento, Docker todavía llama directamente al HCS. En el futuro, sin embargo, las herramientas de administración de contenedores se expanden para incluir contenedores de Windows y el host de contenedor de Windows podía llamar a en contenedor y runhcs la forma en que llaman en contenedores y Runc en Linux.

## <a name="runhcs"></a>runhcs

`runhcs` es una bifurcación de `runc`.  Como `runc`, `runhcs` es un cliente de línea de comandos para ejecutar aplicaciones empaquetadas según el formato de la iniciativa de contenedores abiertos (OCI) y es una implementación compatible de la especificación de la iniciativa de contenedores abiertos.

Entre las diferencias funcionales entre Runc y runhcs se incluyen:

* `runhcs` se ejecuta en Windows.  Se comunica con [HCS](containerd.md#hcs) para crear y administrar contenedores.
* `runhcs` puede ejecutar diferentes tipos de contenedor.

  * [Aislamiento de Hyper-V de](../manage-containers/hyperv-container.md) Windows y Linux
  * Contenedores de procesos de Windows (la imagen de contenedor debe coincidir con el host de contenedor)

**Uso:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` es el nombre de la instancia de contenedor que se va a iniciar. El nombre debe ser único en el host de contenedor.

El directorio bundle (mediante `-b bundle`) es opcional.  
Al igual que con Runc, los contenedores se configuran mediante paquetes. El paquete de un contenedor es el directorio con el archivo de especificación OCI del contenedor, "config. JSON".  El valor predeterminado de "bundle" es el directorio actual.

El archivo de especificación OCI, "config. JSON", debe tener dos campos para ejecutarse correctamente:

* Una ruta de acceso al espacio de desecho del contenedor.
* Una ruta de acceso al directorio de capas del contenedor.

Los comandos de contenedor disponibles en runhcs incluyen:

* Herramientas para crear y ejecutar un contenedor
  * **Ejecutar** crea y ejecuta un contenedor
  * **crear** un contenedor

* Herramientas para administrar procesos que se ejecutan en un contenedor:
  * **Start** ejecuta el proceso definido por el usuario en un contenedor creado
  * **exec** ejecuta un nuevo proceso dentro del contenedor
  * **pausar pausa suspende** todos los procesos dentro del contenedor
  * **reanudar** reanuda todos los procesos que se han pausado previamente
  * **PS** PS muestra los procesos que se ejecutan dentro de un contenedor

* Herramientas para administrar el estado de un contenedor
  * el **Estado** genera el estado de un contenedor
  * **Kill** envía la señal especificada (valor predeterminado: SIGTERM) al proceso init del contenedor.
  * **Delete** elimina todos los recursos mantenidos por el contenedor que se usa a menudo con el contenedor separado.

El único comando que podría considerarse de varios contenedores es **List**.  Muestra los contenedores en ejecución o en pausa iniciados por runhcs con la raíz especificada.

### <a name="hcs"></a>HCS

Tenemos dos contenedores disponibles en GitHub para interactuar con el HCS. Dado que HCS es una API de C, los contenedores facilitan la llamada a HCS desde lenguajes de nivel superior.  

* [hcsshim](https://github.com/microsoft/hcsshim) -hcsshim está escrito en Go y es la base de runhcs.
Obtenga la versión más reciente de AppVeyor o genérelo.
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet-computevirtualization es un C# Contenedor para el HCS.

Si quiere usar HCS (ya sea directamente o a través de un contenedor) o quiere crear un contenedor de oxidación/Haskell/InsertYourLanguage alrededor de la HCS, deje un comentario.

Para ver una mirada más profunda a HCS, vea la [presentación de DockerCon de John](https://www.youtube.com/watch?v=85nCF5S8Qok).

## <a name="containerdcri"></a>contenedor/CRI

> [!IMPORTANT]
> La compatibilidad con CRI solo está disponible en el servidor 2019/Windows 10 1809 y versiones posteriores.  También estamos desarrollando de forma activa contenedores para Windows.
> Solo desarrollo y pruebas.

Mientras que las especificaciones de OCI definen un solo contenedor, [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (interfaz en tiempo de ejecución de contenedor) describe los contenedores como cargas de trabajo en un entorno de espacio aislado compartido denominado Pod.  Los pods pueden contener una o más cargas de trabajo de contenedor.  Los pods permiten que los orquestadores de contenedores como Kubernetes y Service Fabric Mesh controlen las cargas de trabajo agrupadas que deben estar en el mismo host con algunos recursos compartidos, como la memoria y las redes virtuales.

containerd/CRI habilita la siguiente matriz de compatibilidad para pods:

| SO host | SO de contenedor | Aislamiento | ¿Compatibilidad con Pod? |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | Sí: admite los conjuntos de pod de varios contenedores verdaderos. |
|  | Windows Server 2019/1809 | `process`* o `hyperv` | Sí, es compatible con los pods de varios contenedores verdaderos si cada sistema operativo de contenedor de cargas de trabajo coincide con el sistema operativo de la máquina virtual. |
|  | Windows Server 2016,</br>Windows Server 1709,</br>Windows Server 1803 | `hyperv` | Parcial: admite espacios aislados de Pod que pueden admitir un solo contenedor aislado de proceso por máquina virtual de utilidad si el sistema operativo del contenedor coincide con el sistema operativo de la máquina virtual de la utilidad. |

\*los hosts de Windows 10 solo admiten el aislamiento de Hyper-V

Vínculos a la especificación de CRI:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) : especificación de Pod
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) : especificación de carga de trabajo

![Entornos de contenedor basados en contenedores](media/containerd-platform.png)

Aunque runHCS y containered pueden administrar en cualquier sistema de Windows Server 2016 o posterior, es necesario que los pods (grupos de contenedores) admitan cambios importantes en las herramientas de contenedor de Windows.  La compatibilidad con CRI está disponible en Windows Server 2019/Windows 10 1809 y versiones posteriores.
