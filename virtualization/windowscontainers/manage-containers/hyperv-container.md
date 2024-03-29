---
title: Modos de aislamiento
description: Explicación de cómo difiere el aislamiento de Hyper-V de los contenedores aislados del proceso.
keywords: docker, contenedores
author: crwilhit
ms.date: 09/26/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: fa95ffe1c699a2c837076fcc1b662f6b792b7dfb
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909755"
---
# <a name="isolation-modes"></a>Modos de aislamiento

Los contenedores de Windows ofrecen dos modos distintos de aislamiento en tiempo de ejecución: `process` y aislamiento de `Hyper-V`. Los contenedores que se ejecutan en ambos modos de aislamiento se crean, administran y funcionan de forma idéntica. También generan y usan las mismas imágenes del contenedor. La diferencia entre los modos de aislamiento es el grado de aislamiento que se crea entre el contenedor, el sistema operativo host y todos los demás contenedores que se ejecutan en ese host.

## <a name="process-isolation"></a>Aislamiento de procesos

Este es el modo de aislamiento "tradicional" para los contenedores y es lo que se describe en [Introducción a los contenedores de Windows](../about/index.md). Con el aislamiento de procesos, se ejecutan varias instancias de contenedor simultáneamente en un host dado con aislamiento proporcionado mediante tecnologías de espacio de nombres, control de recursos y aislamiento de procesos. Cuando se ejecuta en este modo, los contenedores comparten el mismo kernel con el host, así como entre sí.  Esto equivale aproximadamente al modo en que se ejecutan los contenedores de Linux.

![](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Aislamiento de Hyper-V
Este modo de aislamiento ofrece seguridad mejorada y mayor compatibilidad entre las versiones del host y del contenedor. Con el aislamiento de Hyper-V, varias instancias de contenedor se ejecutan simultáneamente en un host; Sin embargo, cada contenedor se ejecuta dentro de una máquina virtual altamente optimizada y, de hecho, obtiene su propio kernel. La presencia de la máquina virtual proporciona aislamiento de nivel de hardware entre cada contenedor y el host de contenedor.

![](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>Ejemplos de aislamiento

### <a name="create-container"></a>Crear contenedor

La administración de contenedores aislados con Hyper-V con Docker es casi idéntica a la administración de contenedores aislados de proceso. Para crear un contenedor con el aislamiento de Hyper-V minucioso Docker, use el parámetro `--isolation` para establecer `--isolation=hyperv`.

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Para crear un contenedor con Docker exhaustivo de aislamiento de proceso, use el parámetro `--isolation` para establecer `--isolation=process`.

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Los contenedores de Windows que se ejecutan en Windows Server de forma predeterminada se ejecutan con aislamiento de procesos. Los contenedores de Windows que se ejecutan en Windows 10 Pro y Enterprise de forma predeterminada se ejecutan con aislamiento de Hyper-V. A partir de la actualización 2018 de octubre de Windows 10, los usuarios que ejecutan un host de Windows 10 Pro o Enterprise pueden ejecutar un contenedor de Windows con aislamiento de procesos. Los usuarios deben solicitar directamente el aislamiento del proceso mediante el uso de la marca `--isolation=process`.

> [!WARNING]
> La ejecución con el aislamiento de procesos en Windows 10 Pro y Enterprise está pensada para desarrollo y pruebas. El host debe ejecutar Windows 10 Build 17763 + y debe tener una versión de Docker con el motor 18,09 o posterior.
> 
> Debe seguir usando Windows Server como host para las implementaciones de producción. Al usar esta característica en Windows 10 Pro y Enterprise, también debe asegurarse de que las etiquetas del host y de la versión del contenedor coinciden; de lo contrario, el contenedor puede no iniciarse o mostrar un comportamiento indefinido.

### <a name="isolation-explanation"></a>Explicación del aislamiento

En este ejemplo se muestran las diferencias en las funcionalidades de aislamiento entre el proceso y el aislamiento de Hyper-V.

En este caso, se implementa un contenedor aislado de proceso y se hospedará un proceso de ping de ejecución prolongada.

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

Mediante el comando `docker top`, el proceso de ping se devuelve tal como se muestra dentro del contenedor. El proceso de este ejemplo tiene el identificador 3964.

``` cmd
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

En el host de contenedor, puede usarse el comando `get-process` para devolver los procesos de ping que se están ejecutando en el host. En este ejemplo hay uno, y el identificador del proceso coincide con el del contenedor. Se trata del mismo proceso visible en el contenedor y en el host.

```
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

Por el contrario, en este ejemplo se inicia un contenedor de Hyper-V-solated también con un proceso de ping.

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

Del mismo modo, puede usarse `docker top` para devolver los procesos que se están ejecutando en el contenedor.

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

Sin embargo, al buscar el proceso en el host de contenedor, no se encuentra un proceso ping y se produce un error.

```
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

Por último, en el host, puede verse el proceso `vmwp`, que es la máquina virtual que encapsula el contenedor en ejecución y que protege los procesos en ejecución del sistema operativo host.

```
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```
