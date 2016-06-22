---
title: Administración de recursos de contenedores
description: Administre recursos de contenedores con contenedores de Windows.
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b2192e64-9d74-474e-8af0-2d8b3ad1deee
---

# Administración de recursos de contenedores

**Esto es contenido preliminar y está sujeto a cambios.** 

Los contenedores de Windows incluyen la capacidad de administrar la cantidad de recursos de CPU, entrada y salida de disco, red y memoria que pueden consumir los contenedores. Si se restringen los consumos de recursos, los recursos del host se utilizarán de forma eficiente y se evitará el consumo excesivo. En este documento se detalla la administración de recursos de contenedores con Docker.

## Administrar recursos con Docker 

Se ofrece la capacidad de administrar un subconjunto de recursos del contenedor a través de Docker. Concretamente, se permiten que los usuarios especifiquen cómo se debe compartir la CPU entre contenedores. 

### CPU

Es posible administrar los recursos compartidos de la CPU entre contenedores en tiempo de ejecución mediante la marca --cpu-shares. De forma predeterminada, todos los contenedores tienen la misma proporción de tiempo de CPU. Para cambiar el recurso compartido relativo de la CPU que usan los contenedores, ejecute la marca --cpu-shares con un valor de 1-10000. De forma predeterminada, todos los contenedores reciben un peso de 5000. Para obtener más información sobre la limitación de uso compartido de la CPU, consulte [Docker run reference]( https://docs.docker.com/engine/reference/run/#cpu-share-constraint) (Referencia de docker run). 

```none 
docker run -it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## Problemas conocidos

- Actualmente, no se admiten los controles de recursos de E/S y CPU con contenedores de Hyper-V.
- Los controles de recursos de E/S no se admiten actualmente con volúmenes de datos de contenedores.

<!--HONumber=May16_HO3-->


