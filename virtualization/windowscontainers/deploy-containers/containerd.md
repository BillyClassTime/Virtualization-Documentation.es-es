---
title: Plataforma de contenedor de Windows
description: Más información sobre el nuevo contenedor bloques de creación disponibles en Windows.
keywords: LCOW, los contenedores de linux, docker, contenedores, containerd, cri, runhcs, runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 74e22702aa4be30055b3f4f48c7fac926d793095
ms.sourcegitcommit: 03e9203e9769997d8be3f66dc7935a3e5c0a83e1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621613"
---
# <a name="container-platform-tools-on-windows"></a>Herramientas de la plataforma de contenedor en Windows

¡La plataforma de contenedor de Windows está en expansión! Docker es la primera parte del viaje de contenedor, ahora estamos creando otras herramientas de la plataforma de contenedor.

* [containerd/cri](https://github.com/containerd/cri) - nuevo en Windows Server 2019 y Windows 10 1809.
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) - un equivalente de host de contenedor de Windows para runc.
* [hcs](https://docs.microsoft.com/virtualization/api/) - el servicio de cálculo de Host + correcciones útiles para que sea más fácil de usar.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

En este artículo se habla sobre la plataforma de contenedor de Windows y Linux, así como cada herramienta de plataforma de contenedor.

## <a name="windows-and-linux-container-platform"></a>Plataforma de contenedor de Windows y Linux

En entornos de Linux, herramientas de administración de contenedor como Docker se basan en un conjunto más detallado de herramientas de contenedor: [runc](https://github.com/opencontainers/runc) y [containerd](https://containerd.io/).

![Arquitectura de docker en Linux](media/docker-on-linux.png)

`runc` es una herramienta de línea de comandos de Linux para crear y ejecutar contenedores de acuerdo con la [especificación de OCI contenedor en tiempo de ejecución](https://github.com/opencontainers/runtime-spec).

`containerd` es un demonio que administra el ciclo de vida del contenedor descarguen y desempaquetar la imagen de contenedor en ejecución de contenedor y supervisión.

En Windows, tomamos un enfoque diferente.  Cuando empezamos a trabajar con Docker para admitir los contenedores de Windows, hemos creado directamente en el HCS (servicio de cálculo de Host).  [Esta entrada de blog](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332) está lleno de información acerca de por qué se ha creado el HCS y por qué tomamos este enfoque para contenedores inicialmente.

![Arquitectura del motor de Docker inicial en Windows](media/hcs.png)

En este punto, Docker aún llama directamente en el HCS. En el futuro, sin embargo, las herramientas de administración de contenedor expansión para incluir los contenedores de Windows y las ventanas de host de contenedor puede llamar a containerd y runhcs la manera llaman en containerd y runc en Linux.

## <a name="runhcs"></a>runhcs

`runhcs` es una bifurcación de `runc`.  Al igual que `runc`, `runhcs` es un cliente de línea de comandos para ejecutar aplicaciones empaquetadas según el formato abierto contenedor Initiative (OCI) y es una implementación compatible de la especificación abierta iniciativa de contenedor.

Diferencias funcionales entre runc y runhcs incluyen:

* `runhcs` se ejecuta en Windows.  Se comunica con el [HCS](containerd.md#hcs) para crear y administrar contenedores.
* `runhcs` puede ejecutar una variedad de tipos diferentes de contenedores.

  * Windows y Linux [aislamiento de Hyper-V](../manage-containers/hyperv-container.md)
  * Windows procesar contenedores (imagen de contenedor debe coincidir con el host de contenedor)

**Uso:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` es el nombre de la instancia de contenedor que se está iniciando. El nombre debe ser único en el host de contenedor.

El directorio de recopilación (con `-b bundle`) es opcional.  
Al igual que con runc, los contenedores se configuran mediante paquetes. Recopilación de un contenedor es el directorio con el archivo de especificación de OCI del contenedor, "config.json".  El valor predeterminado de "agrupación" es el directorio actual.

El archivo de especificación de OCI, "config.json", debe tener dos campos para ejecutarse correctamente:

* Una ruta de acceso al espacio de memoria virtual del contenedor
* Una ruta de acceso al directorio de nivel del contenedor

Comandos de contenedor disponibles en runhcs incluyen:

* Herramientas para crear y ejecutar un contenedor
  * **Ejecutar** crea y ejecuta un contenedor
  * **crear** , crear un contenedor

* Herramientas para administrar los procesos que se ejecutan en un contenedor:
  * **Inicio** se ejecuta el proceso definido por el usuario en un contenedor creado
  * un nuevo proceso dentro del contenedor se ejecuta el **método exec**
  * **Pausar** pause suspende todos los procesos dentro del contenedor
  * **Reanudar** reanuda todos los procesos que se han pausado anteriormente
  * **PS** ps muestra los procesos que se ejecutan dentro de un contenedor

* Herramientas para administrar el estado de un contenedor
  * **estado** envía el estado de un contenedor
  * **Kill** envía la señal especificada (valor predeterminado: SIGTERM) al proceso de inicialización del contenedor
  * **Delete** elimina todos los recursos mantenidos por el contenedor suelen usado con contenedores desasociados

El único comando que podría considerarse varios contenedor es la **lista**.  Enumera los contenedores en pausa o en ejecución iniciados por runhcs con la raíz determinada.

### <a name="hcs"></a>HCS

Tenemos dos contenedores disponibles en GitHub para interactuar con el HCS. Dado que el HCS es una API C, contenedores facilitan llamar a la HCS de idiomas de nivel superiores.  

* [hcsshim](https://github.com/microsoft/hcsshim) - HCSShim se escriben en Ir y es la base de runhcs.
Obtener la última versión de AppVeyor o crearla tú mismo.
* [computevirtualization de dotnet](https://github.com/microsoft/dotnet-computevirtualization) -dotnet-computevirtualization es un contenedor de C# para el HCS.

Si quieres usar el HCS (directamente o a través de un contenedor), o que quieras realizar un contenedor óxido/Haskell/InsertYourLanguage el HCS, deje un comentario.

Para obtener un vistazo más profundo a la HCS, mira [presentación de DockerCon de John Stark](https://www.youtube.com/watch?v=85nCF5S8Qok).

## <a name="containerdcri"></a>containerd/cri

> [!IMPORTANT]
> Soporte técnico CRI solo está disponible en Windows Server 2019 10 1809 y posteriores.  Estamos desarrollando activamente aún containerd para Windows.
> Desarrollo o prueba solamente.

Mientras que las especificaciones OCI define un contenedor único, [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (interfaz de tiempo de ejecución de contenedor) describe los contenedores como workload(s) en un espacio aislado compartido llama un pod de entorno.  Pods pueden contener uno o más cargas de trabajo de contenedor.  Pods permiten orquestadores de contenedor como Kubernetes y Service Fabric malla controlan las cargas de trabajo agrupados que deben estar en el mismo host con algunos recursos compartidos como la memoria y vNETs.

containerd/cri permite la siguiente matriz de compatibilidad para pods:

| Sistema operativo del host | Contenedor del sistema operativo | Aislamiento de | ¿Soporte técnico de pod? |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | Sí, admite true pods de varios contenedores. |
|  | Windows Server 2019/1809 | `process`* o `hyperv` | Sí, admite true pods de varios contenedores si cada contenedor de carga de trabajo del sistema operativo coincide con la utilidad del sistema operativo de máquina virtual. |
|  | Windows Server 2016,</br>Windows Server 1709,</br>1803 de Windows Server | `hyperv` | Parcial, admite pod espacios aislados que pueden admitir un contenedor de aislamiento de proceso único por VM de utilidad si el sistema operativo de contenedor coincide con la utilidad del sistema operativo de máquina virtual. |

Hosts \*Windows 10 solo admiten el aislamiento de Hyper-V

Vínculos a la especificación de CRI:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) - especificación de Pod
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) - especificación de carga de trabajo

![Entornos de contenedor en función de Containerd](media/containerd-platform.png)

Mientras runHCS y containerd pueden administrar en un sistema de Windows Server 2016 o posterior, compatibilidad con Pods (grupos de contenedores) requiere cambios importantes a las herramientas de contenedor de Windows.  Soporte técnico CRI está disponible en Windows Server 2019 y Windows 10 1809 y posteriores.
