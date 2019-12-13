---
title: Aceleración de GPU en contenedores de Windows
description: Qué nivel de aceleración de GPU existe en los contenedores de Windows
keywords: Docker, contenedores, dispositivos, hardware
author: cwilhit
ms.openlocfilehash: 8f63c74d7839385e21206188263b9e5d08e7eb60
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909915"
---
# <a name="gpu-acceleration-in-windows-containers"></a>Aceleración de GPU en contenedores de Windows

En el caso de muchas cargas de trabajo en contenedor, los recursos de proceso de CPU proporcionan un rendimiento suficiente. Sin embargo, para una determinada clase de carga de trabajo, la potencia de proceso masivamente paralela ofrecida por las GPU (unidades de procesamiento de gráficos) puede acelerar las operaciones en órdenes de magnitud, lo que reduce el costo y mejora el rendimiento de forma enorme.

Las GPU ya son una herramienta común para muchas cargas de trabajo populares, desde la representación y simulación tradicionales hasta el entrenamiento y la inferencia de aprendizaje automático. Los contenedores de Windows admiten la aceleración de GPU para DirectX y todos los marcos de trabajo que se basan en él.

> [!NOTE]
> Esta característica está disponible en Docker Desktop, la versión 2,1 y el motor de Docker-Enterprise, versión 19,03 o posterior.

## <a name="requirements"></a>Requisitos

Para que esta característica funcione, el entorno debe cumplir los siguientes requisitos:

- El host de contenedor debe ejecutar Windows Server 2019 o Windows 10, versión 1809 o posterior.
- La imagen base del contenedor debe ser [MCR.Microsoft.com/Windows:1809](https://hub.docker.com/_/microsoft-windows) o posterior. Actualmente no se admiten las imágenes de contenedor de Windows Server Core y nano Server.
- El host de contenedor debe ejecutar el motor de Docker 19,03 o una versión más reciente.
- El host de contenedor debe tener una GPU que ejecute la versión WDDM 2,5 o una más reciente.

Para comprobar la versión de WDDM de los controladores de pantalla, ejecute la herramienta de diagnóstico de DirectX (Dxdiag. exe) en el host de contenedor. En la pestaña "Mostrar" de la herramienta, mire la sección "controladores" como se indica a continuación.

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>Ejecutar un contenedor con aceleración de GPU

Para iniciar un contenedor con la aceleración de GPU, ejecute el siguiente comando:

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (y todos los marcos basados en él) son las únicas API que se pueden acelerar con una GPU hoy en día. no se admiten marcos de terceros.

## <a name="hyper-v-isolated-windows-container-support"></a>Compatibilidad con contenedores de Windows aislados de Hyper-V

La aceleración de GPU para cargas de trabajo en contenedores de Windows aislados de Hyper-V no se admite actualmente.

## <a name="hyper-v-isolated-linux-container-support"></a>Compatibilidad con contenedores de Linux aislados de Hyper-V

La aceleración de GPU para cargas de trabajo en contenedores de Linux aislados de Hyper-V no se admite actualmente.

## <a name="more-information"></a>Más información

Para ver un ejemplo completo de una aplicación de DirectX en contenedor que aprovecha la aceleración de GPU, consulte [ejemplo de contenedor de DirectX](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-container-samples/directx).
