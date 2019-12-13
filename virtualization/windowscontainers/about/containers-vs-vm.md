---
title: Contenedores frente a máquinas virtuales
description: En este tema se describen algunas de las similitudes y diferencias clave entre contenedores y máquinas virtuales, y Cuándo es posible que desee usar cada uno. Los contenedores y las máquinas virtuales tienen sus usos; de hecho, muchas implementaciones de contenedores usan máquinas virtuales como sistema operativo host en lugar de ejecutarse directamente en el hardware, especialmente cuando se ejecutan contenedores en la nube.
keywords: Docker, contenedores, máquinas virtuales, máquinas virtuales
author: jasongerend
ms.author: jgerend
ms.date: 10/21/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 63150dfde007ec942446387064ad59f05b0aaa43
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910825"
---
# <a name="containers-vs-virtual-machines"></a>Contenedores frente a máquinas virtuales

En este tema se describen algunas de las similitudes y diferencias clave entre contenedores y máquinas virtuales (VM), y Cuándo es posible que desee usar cada una de ellas. Los contenedores y las máquinas virtuales tienen sus usos; de hecho, muchas implementaciones de contenedores usan máquinas virtuales como el sistema operativo host en lugar de ejecutarse directamente en el hardware, especialmente cuando se ejecutan contenedores en la nube.

Para obtener información general sobre los contenedores, consulte [ventanas y contenedores](index.md).

## <a name="container-architecture"></a>Arquitectura de contenedor

Un contenedor es un silo aislado y ligero para ejecutar una aplicación en el sistema operativo host. Los contenedores se basan en el kernel del sistema operativo host (que puede considerarse como la fontanería del sistema operativo) y solo contienen aplicaciones y algunas API y servicios de sistema operativo ligeros que se ejecutan en modo de usuario, tal como se muestra en este diagrama.

![Diagrama arquitectónico en el que se muestra cómo se ejecutan los contenedores en la parte superior del kernel](media/container-diagram.svg)

## <a name="virtual-machine-architecture"></a>Arquitectura de la máquina virtual

A diferencia de los contenedores, las máquinas virtuales ejecutan un sistema operativo completo, incluido su propio kernel, tal como se muestra en este diagrama.

![Diagrama arquitectónico en el que se muestra cómo las máquinas virtuales ejecutan un sistema operativo completo junto al sistema operativo host](media/virtual-machine-diagram.svg)

## <a name="containers-vs-virtual-machines"></a>Contenedores frente a máquinas virtuales

En la tabla siguiente se muestran algunas de las similitudes y diferencias de estas tecnologías complementarias.

|                 | Máquina virtual  | Contenedor  |
| --------------  | ---------------- | ---------- |
| Aislamiento       | Proporciona un aislamiento completo del sistema operativo host y de otras máquinas virtuales. Esto resulta útil cuando un límite de seguridad fuerte es crítico, como el hospedaje de aplicaciones de compañías competidoras en el mismo servidor o clúster. | Normalmente proporciona aislamiento ligero desde el host y otros contenedores, pero no proporciona un límite de seguridad como una máquina virtual. (Puede aumentar la seguridad mediante el [modo de aislamiento de Hyper-V](../manage-containers/hyperv-container.md) para aislar cada contenedor en una máquina virtual ligera). |
| Sistema operativo | Ejecuta un sistema operativo completo incluido el kernel, lo que requiere más recursos del sistema (CPU, memoria y almacenamiento). | Ejecuta la parte del modo de usuario de un sistema operativo y se puede personalizar para que contenga solo los servicios necesarios para la aplicación, con menos recursos del sistema. |
| Compatibilidad con invitados | Ejecuta prácticamente cualquier sistema operativo dentro de la máquina virtual. | Se ejecuta en la [misma versión del sistema operativo que el host (el](../deploy-containers/version-compatibility.md) aislamiento de Hyper-V permite ejecutar versiones anteriores del mismo sistema operativo en un entorno de máquinas virtuales ligeras).
| Implementación     | Implementar máquinas virtuales individuales mediante el centro de administración de Windows o el administrador de Hyper-V; implemente varias máquinas virtuales mediante PowerShell o System Center Virtual Machine Manager. | Implementar contenedores individuales mediante Docker a través de la línea de comandos; Implemente varios contenedores con un orquestador como Azure Kubernetes Service. |
| Actualizaciones y actualizaciones del sistema operativo | Descargue e instale las actualizaciones del sistema operativo en cada máquina virtual. La instalación de una nueva versión del sistema operativo requiere actualizar o, a menudo, simplemente crear una máquina virtual totalmente nueva. Esto puede llevar mucho tiempo, especialmente si tiene una gran cantidad de máquinas virtuales... | Actualizar o actualizar los archivos del sistema operativo dentro de un contenedor es el mismo: <br><ol><li>Edite el archivo de compilación de la imagen de contenedor (conocido como Dockerfile) para que apunte a la versión más reciente de la imagen base de Windows. </li><li>Recompile la imagen de contenedor con esta nueva imagen base.</li><li>Inserte la imagen de contenedor en el registro de contenedor.</li> <li>Vuelva a implementar con un orquestador.<br>El orquestador proporciona una automatización eficaz para hacerlo a escala. Para obtener más información, consulte [Tutorial: actualización de una aplicación en Azure Kubernetes Service](https://docs.microsoft.com/azure/aks/tutorial-kubernetes-app-update).</li></ol> |
| Almacenamiento persistente | Usar un disco duro virtual (VHD) para el almacenamiento local para una sola máquina virtual o un recurso compartido de archivos SMB para el almacenamiento compartido por varios servidores | Usar discos de Azure para el almacenamiento local para un solo nodo o Azure Files (recursos compartidos de SMB) para el almacenamiento compartido por varios nodos o servidores. |
| Equilibrio de carga | El equilibrio de carga de máquinas virtuales mueve máquinas virtuales en ejecución a otros servidores de un clúster de conmutación por error. | Los propios contenedores no se mueven; en su lugar, un orquestador puede iniciar o detener automáticamente contenedores en nodos de clúster para administrar los cambios de carga y disponibilidad. |
| Tolerancia a errores | Las máquinas virtuales pueden conmutar por error a otro servidor de un clúster, con el sistema operativo de la máquina virtual reiniciando en el nuevo servidor.  | Si se produce un error en un nodo del clúster, el orquestador vuelve a crear rápidamente los contenedores que se ejecutan en él en otro nodo del clúster. |
| Funciones de red de     | Utiliza adaptadores de red virtuales. | Usa una vista aislada de un adaptador de red virtual, lo que proporciona un poco menos de virtualización: el firewall del host se comparte con los contenedores, a la vez que se usan menos recursos. Para obtener más información, consulte [redes de contenedores de Windows](../container-networking/architecture.md). |