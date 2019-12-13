---
title: Kubernetes en Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Unión de un nodo de Windows a un clúster de Kubernetes con v 1.14.
keywords: kubernetes, 1,14, Windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c380f5dc10430a94959718a5ce92f311603db733
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910365"
---
# <a name="kubernetes-on-windows"></a>Kubernetes en Windows

Esta página sirve como introducción para empezar a trabajar con Kubernetes en Windows mediante la Unión de nodos de Windows a un clúster basado en Linux. Con la versión de Kubernetes 1,14 en Windows Server [versión 1809](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes), los usuarios pueden aprovechar las siguientes características de Kubernetes en Windows:

- **redes de superposición**: Use flannel en modo vxlan para configurar una red de superposición virtual
    - requiere Windows Server 2019 con [KB4489899](https://support.microsoft.com/help/4489899) instalado o [Windows Server vNext Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) compilación 18317 +
    - requiere Kubernetes v 1.14 (o superior) con `WinOverlay` puerta de características habilitada
    - requiere flannel v 0.11.0 (o superior)
- **Administración de red simplificada**: Use flannel en modo de puerta de enlace host para la administración automática de rutas entre nodos.
- **mejoras de escalabilidad**: Disfrute de tiempos de inicio de contenedor más rápidos y confiables gracias a [VNIC sin dispositivos para contenedores de Windows Server](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716).
- **Aislamiento de Hyper-v (alfa)** : organice el [aislamiento de Hyper-v](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) con el aislamiento de modo kernel para mejorar la seguridad. Para obtener más información, los [tipos de contenedor de Windows](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types).
    - requiere Kubernetes v 1.10 (o superior) con `HyperVContainer` puerta de características habilitada.
- **Complementos de almacenamiento**: Use el [complemento de almacenamiento de FlexVolume](https://github.com/Microsoft/K8s-Storage-Plugins) con compatibilidad con SMB e iSCSI para contenedores de Windows.

>[!TIP]
>Si desea implementar un clúster en Azure, la herramienta de código abierto AKS-Engine facilita esta tarea. Para obtener más información, consulte nuestro [tutorial](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)paso a paso.

## <a name="prerequisites"></a>Requisitos previos

### <a name="plan-ip-addressing-for-your-cluster"></a>Planeamiento de direccionamiento IP del clúster

<a name="definitions"></a>Dado que los clústeres de Kubernetes introducen nuevas subredes para pods y servicios, es importante asegurarse de que ninguno de ellos entra en conflicto con otras redes existentes en el entorno. Estos son todos los espacios de direcciones que deben liberarse para poder implementar Kubernetes correctamente:

| Intervalo de direcciones/subred | Descripción | Valor predeterminado |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Subred de servicio** | Una subred virtual no enrutable que usan los pods para acceder a los servicios de forma uniforme sin preocuparse por la topología de red. Se traduce al espacio de direcciones enrutables, y desde él, mediante `kube-proxy` que se ejecuta en los nodos. | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**Subred del clúster** |  Se trata de una subred global que usan todos los pods del clúster. A cada nodo se le asigna una subred menor/24 de este para que sus pods lo usen. Debe ser lo suficientemente grande como para alojar todos los pods usados en el clúster. Para calcular el tamaño *mínimo* de la subred: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Ejemplo para un clúster de 5 nodos para los pods 100 por nodo: `(5) + (5 *  100) = 505`.  | "10.244.0.0/16" |
| **IP del servicio DNS de Kubernetes** | Dirección IP del servicio "Kube-DNS" que se usará para la resolución de DNS & la detección del servicio de Cluster Server. | "10.96.0.10" |

> [!NOTE]
> Hay otra red de Docker (NAT) que se crea de forma predeterminada al instalar Docker. No es necesario que funcione Kubernetes en Windows, ya que asignamos direcciones IP de la subred del clúster en su lugar.

## <a name="what-you-will-accomplish"></a>Qué lograrás

Al final de esta guía, habrás:

> [!div class="checklist"]
> * Creó un nodo [maestro Kubernetes](./creating-a-linux-master.md) .  
> * Seleccionó una [solución de red](./network-topologies.md).  
> * Unido a un [nodo de trabajo de Windows](./joining-windows-workers.md) o a un nodo de trabajo de [Linux](./joining-linux-workers.md) .  
> * Implementó un [recurso Kubernetes de ejemplo](./deploying-resources.md).  
> * Solucionado [errores y problemas comunes](./common-problems.md).

## <a name="next-steps"></a>Pasos siguientes

En esta sección, hemos hablado de los requisitos previos importantes & supuestos necesarios para implementar Kubernetes en Windows correctamente. Continúe con la información sobre cómo configurar un maestro de Kubernetes:

>[!div class="nextstepaction"]
>[Creación de un maestro de Kubernetes](./creating-a-linux-master.md)