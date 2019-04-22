---
title: Aceleración de GPU en contenedores de Windows
description: ¿Qué nivel de aceleración de GPU existe en contenedores de Windows
keywords: docker, contenedores, los dispositivos, hardware
author: cwilhit
ms.openlocfilehash: 281241e07e4bc184e73c4e74a117b44253a775be
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380059"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Aceleración de GPU en contenedores de Windows

Para muchas cargas de trabajo en contenedores, los recursos de proceso de CPU proporcionan un rendimiento suficiente. Sin embargo, para algunos tipos de carga de trabajo, la potencia de cálculo masivos en paralelo ofrecida por GPU (unidades de procesamiento de gráficos) puede acelerar las operaciones órdenes de magnitud, llevar los costos y mejora el rendimiento enormemente.

GPU ya son una herramienta común para muchas cargas de trabajo populares, de simulación de formación de aprendizaje de máquina y inferencia y representación tradicional. Los contenedores de Windows admiten la aceleración de GPU para DirectX y todos los marcos integrados en la parte superior.

> [!IMPORTANT]
> Esta característica requiere una versión de Docker que admita la `--device` opción de línea de comandos para contenedores de Windows. Se ha programado el soporte de Docker formal para la próxima versión de 19.03 motor de Docker EE. Hasta entonces, el [origen en dirección ascendente](https://master.dockerproject.org/) de Docker contiene los bits necesarios.

## <a name="requirements"></a>Requisitos

Para que funcione esta característica, el entorno debe cumplir los siguientes requisitos:

- El host de contenedor debe ejecutar Windows Server 2019 o Windows 10, versión 1809 o posterior.
- La imagen base del contenedor debe ser [mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windowsfamily-windows) o posterior. Imágenes de contenedor de Windows Server Core y Nano Server no se admiten actualmente.
- El host de contenedor debe ejecutar motor de Docker 19.03 o posterior.
- El host de contenedor debe tener una versión de los controladores de pantalla ejecución de GPU WDDM 2.5 o posterior.

Para comprobar la versión WDDM de los controladores de pantalla, ejecutar la herramienta de diagnóstico de DirectX (dxdiag.exe) en el host de contenedor. En la pestaña de "Mostrar" de la herramienta, busca en la sección "Controladores" como se indica a continuación.

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>Ejecutar un contenedor con la aceleración de GPU

Para iniciar un contenedor con la aceleración de GPU, ejecute el siguiente comando:

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (y todos los marcos integrados en la parte superior) son las únicas API que se pueden acelerar con una GPU hoy en día. no se admiten 3 marcos de terceros.

## <a name="hyper-v-isolated-windows-container-support"></a>Soporte técnico de contenedor de Hyper-V-aislada Windows

La aceleración de GPU para cargas de trabajo en contenedores de Windows aislado Hyper-V no se admite actualmente.

## <a name="hyper-v-isolated-linux-container-support"></a>Soporte de contenedor de Hyper-V-aislada Linux

La aceleración de GPU para cargas de trabajo en contenedores de Linux aislada Hyper-V no se admite actualmente.
