---
title: Aceleración de GPU en contenedores de Windows
description: Qué nivel de aceleración de la GPU existe en contenedores de Windows
keywords: acoplador, contenedores, dispositivos, hardware
author: cwilhit
ms.openlocfilehash: 6e5010efee10f9b488cbeb57b14bc86f30c1e766
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883278"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Aceleración de GPU en contenedores de Windows

Para muchas cargas de trabajo de contenedor, los recursos de cálculo de CPU proporcionan el rendimiento suficiente. Sin embargo, para una determinada clase de carga de trabajo, la potencia de cómputo altamente paralela ofrecida por las GPU (unidades de procesamiento de gráficos) puede acelerar la operación por orden de magnitud, lo que hace que el rendimiento se mejore enormemente.

Las GPU ya son una herramienta común para muchas cargas de trabajo populares, desde la representación y la simulación tradicionales hasta la inferencia de aprendizaje y formación de equipos. Los contenedores de Windows son compatibles con la aceleración GPU para DirectX y todos los marcos creados sobre él.

## <a name="requirements"></a>Requisitos

Para que esta característica funcione, su entorno debe cumplir con los siguientes requisitos:

- El host contenedor debe ejecutar Windows Server 2019 o Windows 10, versión 1809 o posterior.
- La imagen base del contenedor debe ser [MCR.Microsoft.com/Windows:1809](https://hub.docker.com/_/microsoft-windowsfamily-windows) o posterior. Actualmente no se admiten las imágenes de contenedor de Windows Server Core y nano Server.
- El host contenedor debe ejecutar el motor de acoplamiento 19,03 o una versión posterior.
- El host contenedor debe tener una GPU que ejecute los drivers de pantalla versión WDDM 2,5 o posterior.

Para comprobar la versión WDDM de los controladores de pantalla, ejecute la herramienta de diagnóstico de DirectX (Dxdiag. exe) en el host contenedor. En la ficha "Mostrar" de la herramienta, mire la sección "drivers", como se indica a continuación.

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>Ejecutar un contenedor con aceleración GPU

Para iniciar un contenedor con la aceleración GPU, ejecute el siguiente comando:

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (y todos los marcos creados a partir de él) son las únicas API que se pueden acelerar con una GPU en la actualidad. no se admiten marcos de terceros.

## <a name="hyper-v-isolated-windows-container-support"></a>Compatibilidad con Hyper-V de contenedor de Windows aislado

La aceleración de GPU para las cargas de trabajo en contenedores de Windows aislados por Hyper-V no es compatible hoy.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-V-compatibilidad con contenedores de Linux aislados

La aceleración de GPU para cargas de trabajo en contenedores de Linux aislados por Hyper-V no es compatible hoy.