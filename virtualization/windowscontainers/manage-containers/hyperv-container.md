---
title: Contenedores de Hyper-V
description: "Implementación de contenedores de Hyper-V."
keywords: docker, contenedores
author: enderb-ms
ms.date: 09/13/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: ea131dfede51ee36f7dc703511357612430ccca9
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/08/2017
---
# <a name="hyper-v-containers"></a>Contenedores de Hyper-V

**Esto es contenido preliminar y está sujeto a cambios.** 

La tecnología de contenedor de Windows incluye dos tipos distintos de contenedores, contenedores de Windows Server y de Hyper-V. Los dos tipos de contenedores se crean, se administran y funcionan de forma idéntica. También generan y usan las mismas imágenes del contenedor. La diferencia entre ellos es el nivel de aislamiento creado entre el contenedor, el sistema operativo host y todos los demás contenedores que se ejecutan en ese host.

**Contenedores de Windows Server**: Varias instancias de contenedor pueden ejecutarse simultáneamente en un host con aislamiento proporcionado a través de tecnologías de espacio de nombres, control de recursos y aislamiento de proceso.  Los contenedores de Windows Server comparten el mismo kernel con el host, así como entre sí.

**Contenedores de Hyper-V**: varias instancias de contenedor pueden ejecutarse simultáneamente en un host; pero cada contenedor se ejecuta dentro de una máquina virtual especial. Esto proporciona aislamiento de nivel de kernel entre cada contenedor de Hyper-V y el host de contenedor.

## <a name="hyper-v-container"></a>Contenedor de Hyper-V

### <a name="create-container"></a>Crear contenedor

La administración de contenedores de Hyper-V con Docker es casi idéntica a la administración de contenedores de Windows Server. Al crear un contenedor de Hyper-V con Docker, se usa el parámetro `--isolation=hyperv`.

```
docker run -it --isolation=hyperv microsoft/nanoserver cmd
```

### <a name="isolation-explanation"></a>Explicación del aislamiento

En este ejemplo se diferencian las funcionalidades de aislamiento entre contenedores de Hyper-V y Windows Server. 

En este caso, se implementa un contenedor de Windows Server que hospedará un proceso de ping de ejecución prolongada.

```
docker run -d microsoft/windowsservercore ping localhost -t
```

Mediante el comando `docker top`, el proceso de ping se devuelve tal como se muestra dentro del contenedor. El proceso de este ejemplo tiene el identificador 3964.

```
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

Por otro lado, en este ejemplo se inicia un contenedor de Hyper-V que también tiene un proceso de ping. 

```
docker run -d --isolation=hyperv microsoft/nanoserver ping -t localhost
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
