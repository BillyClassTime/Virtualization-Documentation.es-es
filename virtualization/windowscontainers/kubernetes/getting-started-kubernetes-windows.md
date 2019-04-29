---
title: Kubernetes en Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Unir un nodo de Windows a un clúster de Kubernetes con v1.13.
keywords: kubernetes, 1.13, windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 7c3a0111b3d19ae1b513a84665f870bba24ae33d
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576995"
---
# <a name="kubernetes-on-windows"></a>Kubernetes en Windows

Esta página sirve como una visión general para comenzar a trabajar con Kubernetes en Windows mediante la combinación de nodos de Windows a un clúster basado en Linux. Con el lanzamiento de 1.14 de Kubernetes en Windows Server [versión 1809](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes), los usuarios pueden sacar provecho de las siguientes características de Kubernetes en Windows:

- **redes de superposición**: usar Flannel en modo de vxlan para configurar una red virtual de superposición
    - requiere cualquier Windows Server 2019 con [KB4489899](https://support.microsoft.com/en-us/help/4489899) instalado o compilación de [Windows Server vNext Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) 18317 +
    - requiere Kubernetes v1.14 (o superior) con `WinOverlay` puerta de característica habilitada
    - requiere Flannel v0.11.0 (o anterior)
- **administración de red simplificada**: usar Flannel en modo host-gateway para la administración de ruta automático entre los nodos.
- **mejoras de escalabilidad**: disfrute tiempos de inicio más rápido y confiable contenedor gracias a [las vNICs sin dispositivos para contenedores de Windows Server](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/).
- **Aislamiento de Hyper-V (alfa)**: organizar el [aislamiento de Hyper-V](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) con aislamiento de modo kernel para mejorar la seguridad. Para obtener más información, [tipos de contenedores de Windows](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types).
    - requiere Kubernetes v1.10 (o superior) con `HyperVContainer` puerta de característica habilitada.
- **complementos de almacenamiento**: usar el [complemento de almacenamiento FlexVolume](https://github.com/Microsoft/K8s-Storage-Plugins) con soporte de iSCSI y SMB para contenedores de Windows.

>[!TIP]
>Si quieres implementar un clúster en Azure, con la herramienta AKS-Engine de código abierto facilita esta tarea. Para obtener más información, consulta nuestro [tutorial](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)de paso a paso.

## <a name="prerequisites"></a>Requisitos previos

### <a name="plan-ip-addressing-for-your-cluster"></a>Planear la asignación de direcciones IP para el clúster

<a name="definitions"></a>Como los clústeres de Kubernetes presentan nuevas subredes de pods y servicios, es importante asegurarse de que ninguno de ellos entran en conflicto con cualquier otra red en tu entorno existente. Estos son todos los espacios de direcciones que se libere para implementar Kubernetes correctamente:

| Subred / intervalo de direcciones | Descripción | Valor predeterminado |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Subred de servicio** | Una subred puramente virtual, y no enrutable que se usa Pods forma acceder a servicios sin preocuparse por la topología de red. Se traduce al espacio de direcciones enrutables, y desde él, mediante `kube-proxy` que se ejecuta en los nodos. | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**Subred de clúster** |  Se trata de una subred global que se usa en todos los pods en el clúster. Cada nodos se asigna una menor /24 subred desde este para sus pods los usen. Debe ser lo suficientemente grande como para dar cabida a todos los pods que se usa en el clúster. Para calcular el tamaño *mínimo* de subred: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Ejemplo de un clúster de 5 nodos para 100 pods por nodo: `(5) + (5 *  100) = 505`.  | "10.244.0.0/16" |
| **IP de servicio de DNS de Kubernetes** | Dirección IP de servicio de "kube dns" que se usará para la detección de servicios de clúster de & de resolución DNS. | "10.96.0.10" |

> [!NOTE]
> No hay otra red de Docker (NAT) que se crea de forma predeterminada cuando se instala Docker. No es necesaria para que funcione Kubernetes en Windows como asignamos direcciones IP de la subred de clúster en su lugar.

## <a name="what-you-will-accomplish"></a>Qué lograrás

Al final de esta guía, habrás:

> [!div class="checklist"]
> * Crea un nodo de [maestro de Kubernetes](./creating-a-linux-master.md) .  
> * Selecciona una [solución de red](./network-topologies.md).  
> * Unido a un [nodo de trabajo de Windows](./joining-windows-workers.md) o un [nodo de trabajo de Linux](./joining-linux-workers.md) a ella.  
> * Implementa un [recurso de Kubernetes de muestra](./deploying-resources.md).  
> * Solucionado [errores y problemas comunes](./common-problems.md).

## <a name="next-steps"></a>Pasos siguientes

En esta sección, hablamos sobre los requisitos previos importantes suposiciones & necesarias para implementar correctamente hoy Kubernetes en Windows. Continuar aprender a configurar a un maestro de Kubernetes:

>[!div class="nextstepaction"]
>[Crear un maestro de Kubernetes](./creating-a-linux-master.md)