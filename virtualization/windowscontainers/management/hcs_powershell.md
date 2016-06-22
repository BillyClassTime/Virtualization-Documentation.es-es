---
title: HCS PowerShell
description: Trabajo con HCS PowerShell y contenedores de Windows.
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 45144ec5-f76a-4460-abd1-9b60e47506d6
---

# Interoperabilidad de administración

**Esto es contenido preliminar y está sujeto a cambios.** 

## Mostrar todos los contenedores

Para devolver una lista de contenedores, use el comando `Get-ComputeProcess`.

```none
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## Detener un contenedor

Para detener un contenedor independientemente de si se creó con PowerShell o Docker, use el comando `Stop-ComputeProcess`.

> En el momento de la escritura, el servicio de VMMS deberá reiniciarse para que los contenedores se muestren como detenidos cuando se use el comando `Get-Container`.

```none
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```


<!--HONumber=May16_HO3-->


