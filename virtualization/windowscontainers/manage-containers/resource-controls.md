---
title: Implementar controles de recursos
description: Detalles sobre los controles de recursos para contenedores de Windows
keywords: docker, contenedores, cpu, memoria, disco, recursos
author: taylorb-microsoft
ms.date: 11/21/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8ccd4192-4a58-42a5-8f74-2574d10de98e
ms.openlocfilehash: 3e9f7e3208222cd6c0f512c5f892453ac6e6980c
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910175"
---
# <a name="implementing-resource-controls-for-windows-containers"></a>Implementar controles de recursos para contenedores de Windows
Hay varios controles de recursos que pueden implementarse por contenedor y por recurso.  De forma predeterminada, los contenedores ejecutados están sujetos a la administración habitual de recursos de Windows, lo que, en general, se comparte de forma equitativa. Sin embargo, un desarrollador o administrador puede limitar el uso de los recursos, o influir sobre él, tras configurar estos controles.  Entre los recursos que pueden controlarse se incluyen: CPU/procesador, memoria/RAM, disco/almacenamiento y redes/rendimiento.

Los contenedores de Windows utilizan [objetos de trabajo](https://docs.microsoft.com/windows/desktop/ProcThread/job-objects) para agrupar y realizar un seguimiento de los procesos asociados a cada contenedor.  Los controles de recursos se implementan en el objeto de trabajo principal asociado al contenedor. 

En el caso de [aislamiento de Hyper-V](./hyperv-container.md), los controles de recursos se aplican a la máquina virtual y al objeto de trabajo del contenedor que se ejecuta automáticamente en dicha máquina. Esto garantiza que aunque un proceso que se ejecuta en el contenedor omitiera o pasara por alto los controles de objetos de trabajo, la máquina virtual se aseguraría de que fuera incapaz de superar los controles de recursos definidos.

## <a name="resources"></a>Recursos
Para cada recurso, esta sección ofrece una asignación entre la interfaz de línea de comandos Docker como un ejemplo de cómo debería usarse el control de recursos (un orquestador u otras herramientas pueden configurarlo) a la API de servicio de proceso de host (HCS) que ha implementado Windows (ten en cuenta que esta descripción es de nivel alto y que la implementación subyacente está sujeta a cambios).

|  | |
| ----- | ------|
| *Memoria* ||
| Interfaz de docker | [--memoria](https://docs.docker.com/engine/admin/resource_constraints/#memory) |
| Interfaz de HCS | [MemoryMaximumInMB](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Kernel compartido | [JOB_OBJECT_LIMIT_JOB_MEMORY](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_basic_limit_information) |
| Aislamiento de Hyper-V | Memoria de máquina virtual |
| _Nota sobre el aislamiento de Hyper-V en Windows Server 2016: cuando se usa un límite de memoria, verá que el contenedor asigna la cantidad de memoria del Cap inicialmente y, a continuación, comienza a devolverla al host del contenedor.  En versiones posteriores (1709 o posteriores) esto se ha optimizado._ |
| ||
| *CPU (recuento)* ||
| Interfaz de docker | [--CPU](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| Interfaz de HCS | [ProcessorCount](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Kernel compartido | Simulado con [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information)* |
| Aislamiento de Hyper-V | Número de procesadores virtuales expuestos |
| ||
| *CPU (porcentaje)* ||
| Interfaz de docker | [--CPU-porcentaje](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| Interfaz de HCS | [ProcessorMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Kernel compartido | [JOB_OBJECT_CPU_RATE_CONTROL_HARD_CAP](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Aislamiento de Hyper-V | Límites de hipervisor en procesadores virtuales |
| ||
| *CPU (recursos compartidos)* ||
| Interfaz de docker | [--CPU-recursos compartidos](https://docs.docker.com/engine/admin/resource_constraints/#cpu) |
| Interfaz de HCS | [ProcessorWeight](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Kernel compartido | [JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information) |
| Aislamiento de Hyper-V | Pesos de procesadores virtuales de hipervisor |
| ||
| *Almacenamiento (imagen)* ||
| Interfaz de docker | [--IO-maxbandwidth/--IO-maxiops](https://docs.docker.com/edge/engine/reference/commandline/run/#usage) |
| Interfaz de HCS | [StorageIOPSMaximum y StorageBandwidthMaximum](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Kernel compartido | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Aislamiento de Hyper-V | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| ||
| *Almacenamiento (volúmenes)* ||
| Interfaz de docker | [--Storage-OPC size =](https://docs.docker.com/edge/engine/reference/commandline/run/#set-storage-driver-options-per-container) |
| Interfaz de HCS | [StorageSandboxSize](https://github.com/Microsoft/hcsshim/blob/b144c605002d4086146ca1c15c79e56bfaadc2a7/interface.go#L67) |
| Kernel compartido | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |
| Aislamiento de Hyper-V | [JOBOBJECT_IO_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/jobapi2/ns-jobapi2-jobobject_io_rate_control_information) |

## <a name="additional-notes-or-details"></a>Detalles o notas adicionales

### <a name="memory"></a>Memoria

Los contenedores de Windows ejecutan algún proceso del sistema en cada contenedor; por lo general, aquellos que proporcionan funcionalidad por contenedor como la administración de usuarios, redes etc… y si bien gran parte de la memoria necesaria para estos procesos se comparte entre contenedores, el límite de memoria debe ser lo suficientemente alto como para permitirlos.  En el documento [requisitos del sistema](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/system-requirements#memory-requirments) se incluye una tabla para cada tipo de imagen base con y sin aislamiento de Hyper-V.

### <a name="cpu-shares-without-hyper-v-isolation"></a>Recursos compartidos de CPU (sin el aislamiento de Hyper-V)

Al usar los recursos compartidos de CPU, la implementación subyacente (cuando no se utiliza el aislamiento de Hyper-V) configura [JOBOBJECT_CPU_RATE_CONTROL_INFORMATION](https://docs.microsoft.com/windows/desktop/api/winnt/ns-winnt-_jobobject_cpu_rate_control_information), lo que establece específicamente la marca de control en JOB_OBJECT_CPU_RATE_CONTROL_WEIGHT_BASED y proporciona un peso adecuado.  Los intervalos de peso válido del objeto de trabajo son 1 – 9 con un valor predeterminado de 5, lo que supone una fidelidad inferior a los valores de servicios de proceso de host de 1-10000.  A modo de ejemplo, un peso de recurso compartido de 7500 produciría un peso de 7 o un peso de recurso compartido de 2500 daría como resultado un valor de 2.
