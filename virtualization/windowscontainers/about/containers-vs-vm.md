---
title: Contenedores frente a máquinas virtuales
description: En este tema se describen algunas de las similitudes y diferencias clave entre los contenedores y las máquinas virtuales, y Cuándo es posible que desee usar cada uno de ellos. Los contenedores y las máquinas virtuales tienen sus usos, de hecho, muchas implementaciones de contenedores usan máquinas virtuales como el sistema operativo del host en lugar de ejecutarse directamente en el hardware, especialmente cuando se ejecutan contenedores en la nube.
keywords: acoplador, contenedores, VM, máquinas virtuales
author: jasongerend
ms.author: jgerend
ms.date: 10/21/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 63150dfde007ec942446387064ad59f05b0aaa43
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254424"
---
# <a name="containers-vs-virtual-machines"></a>Contenedores frente a máquinas virtuales

En este tema se describen algunas de las similitudes y diferencias clave entre los contenedores y las máquinas virtuales (MV), y Cuándo es posible que desee usar cada una de ellas. Los contenedores y las VM tienen sus usos, de hecho, muchas implementaciones de contenedores usan VM como el sistema operativo del host en lugar de ejecutarse directamente en el hardware, especialmente cuando se ejecutan contenedores en la nube.

Para obtener información general sobre contenedores, vea [ventanas y contenedores](index.md).

## <a name="container-architecture"></a>Arquitectura de contenedor

Un contenedor es un silo liviano y aislado en el que se ejecuta una aplicación en el sistema operativo del host. Los contenedores se construyen sobre el kernel del sistema operativo del host (que se puede considerar como la fontanería del sistema operativo enterrado) y solo contienen aplicaciones y API de sistemas operativos livianos y servicios que se ejecutan en modo de usuario, como se muestra en este diagrama.

![Diagrama de arquitectura en el que se muestra cómo se ejecutan los contenedores en la parte superior del núcleo](media/container-diagram.svg)

## <a name="virtual-machine-architecture"></a>Arquitectura de máquina virtual

A diferencia de los contenedores, las VM ejecutan un sistema operativo completo, incluido su propio kernel, tal como se muestra en este diagrama.

![Diagrama de arquitectura en el que se muestra cómo las máquinas virtuales ejecutan un sistema operativo completo junto al sistema operativo del host](media/virtual-machine-diagram.svg)

## <a name="containers-vs-virtual-machines"></a>Contenedores frente a máquinas virtuales

En la tabla siguiente se muestran algunas de las similitudes y diferencias de estas tecnologías complementarias.

|                 | Máquina virtual  | Contenedor  |
| --------------  | ---------------- | ---------- |
| Identificación       | Proporciona aislamiento total desde el sistema operativo del host y otras máquinas virtuales. Esto es útil cuando un límite de seguridad fuerte es crítico, como hospedar aplicaciones de empresas de la competencia en el mismo servidor o clúster. | Por lo general, ofrece aislamiento liviano del host y otros contenedores, pero no proporciona como fuerte un límite de seguridad como una VM. (Puede aumentar la seguridad con el [modo de aislamiento de Hyper-V](../manage-containers/hyperv-container.md) para aislar cada contenedor en una VM ligera). |
| Sistema operativo | Ejecuta un sistema operativo completo incluido el núcleo, lo que requiere más recursos del sistema (CPU, memoria y almacenamiento). | Ejecuta la parte de modo de usuario de un sistema operativo y se puede personalizar para que contenga solo los servicios necesarios para la aplicación, con menos recursos del sistema. |
| Compatibilidad de invitados | Ejecuta casi cualquier sistema operativo dentro de la máquina virtual | Se ejecuta en la [misma versión de sistema operativo que el host (el](../deploy-containers/version-compatibility.md) aislamiento de Hyper-V le permite ejecutar versiones anteriores del mismo sistema operativo en un entorno de VM ligero).
| Implementación     | Implementar máquinas virtuales individuales con el centro de administración de Windows o el administrador de Hyper-V; implemente varias máquinas virtuales con PowerShell o System Center Virtual Machine Manager. | Implementar contenedores individuales con el acoplador a través de la línea de comandos; Implemente varios contenedores con un Orchestrator como el servicio de Azure Kubernetes. |
| Actualizaciones y actualizaciones del sistema operativo | Descargar e instalar actualizaciones del sistema operativo en cada VM. Instalar una nueva versión de sistema operativo requiere actualizar o, a menudo, simplemente crear una máquina virtual completamente nueva. Esto puede requerir mucho tiempo, especialmente si tienes muchas VMs... | Actualizar o actualizar los archivos del sistema operativo dentro de un contenedor es el mismo: <br><ol><li>Edite el archivo de compilación de la imagen de contenedor (conocido como Dockerfile) para que apunte a la última versión de la imagen base de Windows. </li><li>Vuelva a crear la imagen de contenedor con esta nueva imagen base.</li><li>Inserte la imagen del contenedor en el registro del contenedor.</li> <li>Vuelva a implementar con un organizador.<br>El Orchestrator proporciona una eficaz automatización para hacerlo a escala. Para obtener más información, consulte [Tutorial: actualizar una aplicación en el servicio de Kubernetes de Azure](https://docs.microsoft.com/azure/aks/tutorial-kubernetes-app-update).</li></ol> |
| Almacenamiento persistente | Usar un disco duro virtual (VHD) para almacenamiento local para una única VM o un recurso compartido de archivos SMB para el almacenamiento compartido por varios servidores | Use los discos de Azure para el almacenamiento local para un solo nodo o archivos de Azure (recursos compartidos SMB) para almacenamiento compartido por varios nodos o servidores. |
| Equilibrio de carga | El equilibrio de carga de la máquina virtual pasa las máquinas virtuales a otros servidores en un clúster de conmutación por error. | Los propios contenedores no se mueven; en su lugar, un organizador puede iniciar o detener automáticamente los contenedores en los nodos del clúster para administrar los cambios en la carga y la disponibilidad. |
| Tolerancia a errores | Las máquinas virtuales pueden migrar tras error a otro servidor de un clúster, con el sistema operativo de la máquina virtual reiniciando en el nuevo servidor.  | Si se produce un error en un nodo del clúster, el Orchestrator vuelve a crear rápidamente los contenedores que se ejecutan en él en otro nodo del clúster. |
| Redes     | Usa adaptadores de red virtuales. | Usa una vista aislada de un adaptador de red virtual, lo que proporciona un poco menos de virtualización: el firewall del host se comparte con contenedores, mientras se usan menos recursos. Para obtener más información, consulta [redes de contenedor de Windows](../container-networking/architecture.md). |