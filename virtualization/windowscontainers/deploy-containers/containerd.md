---
title: Plataforma contenedora de Windows
description: Obtenga más información sobre los nuevos bloques de creación de contenedores disponibles en Windows.
keywords: LCOW, contenedores de Linux, Docker, contenedores, contenedor, CRI, runhcs, Runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 3107eb48dc9c75224b0c9dd9b436af6f0f451871
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998422"
---
# <a name="container-platform-tools-on-windows"></a>Herramientas de plataforma de contenedor en Windows

La plataforma de contenedor de Windows está en expansión. Docker fue la primera parte del recorrido del contenedor, ahora estamos creando otras herramientas de plataforma de contenedor.

* [contenedor/CRI](https://github.com/containerd/cri) -New en Windows Server 2019/Windows 10 1809.
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) : un host contenedor de Windows que es equivalente a Runc.
* [HCS](https://docs.microsoft.com/virtualization/api/) : el servicio de cómputo de host + correcciones útiles para que sea más fácil de usar.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

En este artículo hablaremos acerca de la plataforma de contenedor de Windows y Linux, así como de cada herramienta de plataforma de contenedor.

## <a name="windows-and-linux-container-platform"></a>Plataforma contenedora Windows y Linux

En entornos Linux, las herramientas de administración de contenedores como Docker se crean en un conjunto más granular de herramientas contenedoras: [Runc](https://github.com/opencontainers/runc) y [containered](https://containerd.io/).

![Arquitectura del acoplador en Linux](media/docker-on-linux.png)

`runc` es una herramienta de línea de comandos de Linux para crear y ejecutar contenedores según la [especificación de tiempo de ejecución del contenedor OCI](https://github.com/opencontainers/runtime-spec).

`containerd` es un daemon que administra el ciclo de vida del contenedor de descargar y desempaquetar la imagen del contenedor para la ejecución y la supervisión del contenedor.

En Windows, adoptamos un enfoque diferente.  Cuando comenzamos a trabajar con Docker para admitir contenedores de Windows, hemos creado directamente en el HCS (servicio de cómputo de host).  [Esta entrada de blog](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332) está llena de información sobre por qué hemos creado el HCS y por qué hemos tomado este enfoque para los contenedores inicialmente.

![Arquitectura del motor del acoplador inicial en Windows](media/hcs.png)

En este momento, el acoplador aún llama directamente al HCS. Sin embargo, en el futuro, las herramientas de administración de contenedores se expanden para incluir contenedores de Windows y el host contenedor de Windows podría llamar a contenedores y runhcss de la manera que llaman en contenedores y Runc en Linux.

## <a name="runhcs"></a>runhcs

`runhcs` es una horquilla de `runc`.  Like `runc`, `runhcs` es un cliente de línea de comandos para ejecutar aplicaciones empaquetadas según el formato de la iniciativa de contenedor abierto (OCI) y es una implementación conforme a la especificación de la iniciativa de contenedor abierto.

Entre las diferencias funcionales entre Runc y runhcs se incluyen:

* `runhcs` se ejecuta en Windows.  Se comunica con el [HCS](containerd.md#hcs) para crear y administrar contenedores.
* `runhcs` puede ejecutar una gran variedad de tipos de contenedor diferentes.

  * [Aislamiento de Hyper-V para](../manage-containers/hyperv-container.md) Windows y Linux
  * Contenedores de procesos de Windows (la imagen del contenedor debe coincidir con el host contenedor)

**Uso:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` es el nombre de la instancia de contenedor que está iniciando. El nombre debe ser único en el host contenedor.

El directorio del paquete ( `-b bundle`con) es opcional.  
Al igual que con Runc, los contenedores se configuran mediante paquetes. El paquete de un contenedor es el directorio con el archivo de especificación OCI del contenedor, "config. JSON".  El valor predeterminado de "paquete" es el directorio actual.

El archivo de especificación OCI, "config. JSON", debe tener dos campos para ejecutarse correctamente:

* Una ruta de acceso al espacio de la grieta del contenedor
* Una ruta de acceso al directorio de capas del contenedor

Los comandos de contenedor disponibles en runhcs incluyen:

* Herramientas para crear y ejecutar un contenedor
  * **Ejecutar** crea y ejecuta un contenedor
  * **crear** un contenedor

* Herramientas para administrar procesos que se ejecutan en un contenedor:
  * **iniciar** ejecuta el proceso definido por el usuario en un contenedor creado
  * **exec** ejecuta un nuevo proceso dentro del contenedor
  * **** pausar pausa suspende todos los procesos dentro del contenedor
  * **resume** reanuda todos los procesos que se han pausado previamente
  * **PS** PS muestra los procesos que se ejecutan en un contenedor

* Herramientas para administrar el estado de un contenedor
  * el **Estado** genera el estado de un contenedor
  * **Kill** envía la señal especificada (valor predeterminado: SIGTERM) al proceso init del contenedor
  * **eliminar** elimina todos los recursos mantenidos por el contenedor que se usa con frecuencia con el contenedor separado

El único comando que se podría considerar contenedor múltiple es una **lista**.  Muestra una lista de los contenedores en ejecución o pausados iniciados por runhcs con la raíz dada.

### <a name="hcs"></a>HCS

Tenemos dos contenedores disponibles en GitHub para la interfaz con el HCS. Puesto que HCS es una API de C, los contenedores hacen que sea más fácil llamar a HCS desde los idiomas de mayor nivel.  

* [hcsshim](https://github.com/microsoft/hcsshim) -hcsshim está escrito en Go y es la base de runhcs.
Toma las novedades de AppVeyor o escríbete tú mismo.
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet-computevirtualization es un empaquetador de C# para la HCS.

Si deseas usar el HCS (directamente o a través de un envoltorio) o quieres crear un contenedor oxidado/Haskell/InsertYourLanguage en la HCS, deja un comentario.

Para obtener más información sobre el HCS, vea la [presentación de DockerCon de John de](https://www.youtube.com/watch?v=85nCF5S8Qok)la vista.

## <a name="containerdcri"></a>contenedor/CRI

> [!IMPORTANT]
> El soporte técnico de CRI solo está disponible en Server 2019/Windows 10 1809 y versiones posteriores.  También estamos desarrollando de forma activa contenedores para Windows.
> Solo para desarrollo y pruebas.

A pesar de que las especificaciones de OCI definen un único contenedor, [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (interfaz en tiempo de ejecución del contenedor) describe los contenedores como cargas de trabajo en un entorno de recinto compartido denominado Pod.  Pods puede contener una o más cargas de trabajo de contenedor.  Pods permita que los Orchestrator de contenedores como Kubernetes y Service fabric Mesh manejen cargas de trabajo agrupadas que deberían estar en el mismo host con algunos recursos compartidos, como memoria y redes virtuales.

Container/CRI habilita la siguiente matriz de compatibilidad para pods:

| Sistema operativo del host | Sistema operativo de contenedor | Identificación | ¿Es compatible con Pod? |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | Sí: soporta los verdaderos pods con varios contenedores. |
|  | Windows Server 2019/1809 | `process`* o `hyperv` | Sí, admite los verdaderos pods con varios contenedores si cada sistema operativo de contenedor de carga coincide con el Utility OS VM. |
|  | Windows Server 2016,</br>Windows Server 1709,</br>Windows Server 1803 | `hyperv` | Partial: admite espacios aislados que pueden admitir un único contenedor aislado por utilidad para cada VM si el sistema operativo del contenedor coincide con el sistema operativo de la VM de la utilidad. |

\ * Los hosts de Windows 10 solo admiten el aislamiento de Hyper-V

Vínculos a la especificación de CRI:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) : especificación de Pod
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) : especificación de carga de trabajo

![Entornos de contenedor basados en contenedores](media/containerd-platform.png)

Aunque tanto runHCS como contenedores pueden administrarse en cualquier sistema Windows System Server 2016 o posterior, es compatible con pods (grupos de contenedores) se necesitan cambios importantes en las herramientas de contenedor de Windows.  El soporte técnico de CRI está disponible en Windows Server 2019/Windows 10 1809 y versiones posteriores.
