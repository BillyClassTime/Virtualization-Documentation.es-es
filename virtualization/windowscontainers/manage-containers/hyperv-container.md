---
title: Aislamiento de Hyper-V
description: Explicación de las diferencias entre los contenedores de proceso aislado aislamiento de Hyper-V cuales.
keywords: docker, contenedores
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 2ff2d1204e1f973d49af5e1d4441e4eacd946101
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576906"
---
# <a name="hyper-v-isolation"></a>Aislamiento de Hyper-V

La tecnología de contenedor de Windows incluye dos niveles distintos de aislamiento de contenedores, el proceso y el aislamiento de Hyper-V. Ambos tipos se crean, se administran y funcionan de forma idéntica. También generan y usan las mismas imágenes del contenedor. La diferencia entre ellos es el nivel de aislamiento creado entre el contenedor, el sistema operativo host y todos los demás contenedores que se ejecutan en ese host.

**Aislamiento de procesos** : contenedor varias instancias pueden ejecutarse simultáneamente en un host con aislamiento proporcionado a través del espacio de nombres, control de recursos y aislamiento de procesos tecnologías.  Los contenedores comparten el mismo kernel con el host, así como entre sí.  Esto es aproximadamente el mismo como el comportamiento de los contenedores puede ejecutarse en Linux.

**Aislamiento de Hyper-V** : varias instancias de contenedor pueden ejecutarse simultáneamente en un host, sin embargo, cada contenedor se ejecuta dentro de una máquina virtual especial. Esto proporciona aislamiento de nivel de kernel entre cada contenedor, así como el host de contenedor.

## <a name="hyper-v-isolation-examples"></a>Ejemplos de aislamiento de Hyper-V

### <a name="create-container"></a>Crear contenedor

Administración de contenedores de Hyper-V aislado con Docker es casi idéntica a la administración de contenedores de Windows Server. Para crear un contenedor con aislamiento de Hyper-V Docker exhaustiva, usa el `--isolation` parámetro para establecer `--isolation=hyperv`.

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>Explicación del aislamiento

En este ejemplo se muestra las diferencias de las funcionalidades de aislamiento entre el aislamiento de Hyper-V y Windows Server.

Aquí, un proceso de contenedor aislado se está implementando y que hospedará un proceso de ping de ejecución larga.

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:1809 ping localhost -t
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

Por otro lado, en este ejemplo se inicia un contenedor aislado de Hyper-V, así como un proceso de ping.

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 ping -t localhost
```

Del mismo modo, puede usarse `docker top` para devolver los procesos que se están ejecutando en el contenedor.

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

Pero cuando se busca el proceso de host de contenedor, no se encuentra ningún proceso de ping y se produce un error.

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
