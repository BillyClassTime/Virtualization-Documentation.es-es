---
title: Kubernetes en Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Unirse a un nodo de Windows a un clúster de Kubernetes con v 1.14.
keywords: kubernetes, 1,14, Windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 18734f102042ec951255061dcd82229e18d29a15
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998392"
---
# <a name="kubernetes-on-windows"></a>Kubernetes en Windows

Esta página sirve como información general para empezar a usar Kubernetes en Windows uniendo nodos de Windows a un clúster basado en Linux. Con el lanzamiento de Kubernetes 1,14 en Windows Server [versión 1809](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes), los usuarios pueden aprovechar las siguientes características de Kubernetes en Windows:

- **superponer redes**: Use flannel en modo vxlan para configurar una red de superposición virtual
    - requiere Windows Server 2019 con [KB4489899](https://support.microsoft.com/help/4489899) instalado o [Windows Server vNext](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) Insider Preview compilación 18317 +
    - requiere Kubernetes v 1.14 (o una versión superior `WinOverlay` ) con la puerta de características habilitada
    - requiere flannel v 0.11.0 (o una versión posterior)
- **Administración simplificada de redes**: Use flannel en el modo de puerta de enlace de host para la administración automática de rutas entre nodos.
- **mejoras**en la escalabilidad: Disfrute de tiempos de inicio de contenedores más rápidos y confiables gracias a la vNICs no Devices [para contenedores de Windows Server](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716).
- **Aislamiento de Hyper-v (alfa)**: organice el [aislamiento Hyper-v](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) con el aislamiento de modo kernel para mayor seguridad. Para obtener más información, [tipos de contenedor de Windows](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types).
    - requiere Kubernetes v 1.10 (o superior) con `HyperVContainer` la puerta de características habilitada.
- **Complementos de almacenamiento**: Use el complemento de [almacenamiento de FlexVolume](https://github.com/Microsoft/K8s-Storage-Plugins) con compatibilidad con SMB e iSCSI para contenedores de Windows.

>[!TIP]
>Si desea implementar un clúster en Azure, la herramienta del motor de AKS de código abierto hace que esto sea más fácil. Para obtener más información, consulta nuestro [tutorial](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)paso a paso.

## <a name="prerequisites"></a>Requisitos previos

### <a name="plan-ip-addressing-for-your-cluster"></a>Planear la asignación de direcciones IP para el clúster

<a name="definitions"></a>Puesto que los clústeres de Kubernetes introducen nuevas subredes para los pods y los servicios, es importante asegurarse de que ninguna de ellas colisione con ninguna otra red existente en su entorno. Estos son todos los espacios de direcciones que deben liberarse para poder implementar Kubernetes correctamente:

| Intervalo de subred o dirección | Descripción | Valor predeterminado |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Subred de servicio** | Una subred no enrutable, puramente virtual, que usan los pods para acceder a servicios de manera uniforme sin tener que preocuparse por la topología de la red. Se traduce al espacio de direcciones enrutables, y desde él, mediante `kube-proxy` que se ejecuta en los nodos. | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**Subred del clúster** |  Esta es una subred global que usan todos los pods en el clúster. A cada uno de los nodos se les asigna una subred menor/24 a la de sus pods para usar. Debe ser lo suficientemente grande para admitir todos los pods usados en el clúster. Para calcular el tamaño *mínimo* de subred: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Ejemplo de un clúster de 5 nodos por 100 pods por nodo `(5) + (5 *  100) = 505`:.  | "10.244.0.0/16" |
| **IP del servicio DNS de Kubernetes** | Dirección IP del servicio "Kube-DNS" que se usará para la resolución DNS & la detección de servicios de clúster. | "10.96.0.10" |

> [!NOTE]
> Hay otra red de acoplamiento (NAT) que se crea de forma predeterminada al instalar el acoplador. No es necesario que funcione Kubernetes en Windows ya que, en su lugar, asignamos direcciones IP desde la subred del clúster.

## <a name="what-you-will-accomplish"></a>Qué lograrás

Al final de esta guía, habrás:

> [!div class="checklist"]
> * Creó un nodo [maestro de Kubernetes](./creating-a-linux-master.md) .  
> * Seleccionó una [solución de red](./network-topologies.md).  
> * Se unió al [nodo de trabajo de Windows](./joining-windows-workers.md) o al nodo de [trabajo Linux](./joining-linux-workers.md) .  
> * Implementó un [recurso de Kubernetes de ejemplo](./deploying-resources.md).  
> * Solucionado [errores y problemas comunes](./common-problems.md).

## <a name="next-steps"></a>Pasos siguientes

En esta sección, hemos hablado de importantes requisitos previos & supuestos necesarios para implementar Kubernetes en Windows correctamente. Aprenda a configurar un maestro de Kubernetes:

>[!div class="nextstepaction"]
>[Crear un patrón de Kubernetes](./creating-a-linux-master.md)