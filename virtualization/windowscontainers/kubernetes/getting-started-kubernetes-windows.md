---
title: Kubernetes en Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Unir un nodo de Windows a un clúster de Kubernetes con v1.12.
keywords: kubernetes, 1.12, windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 0e43b2ac5b19d16721c1ba0dd1f34e339223bdaf
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178908"
---
# <a name="kubernetes-on-windows"></a>Kubernetes en Windows #
Esta página sirve como una visión general para comenzar a trabajar con Kubernetes en Windows mediante la combinación de nodos de Windows a un clúster basado en Linux. Con el lanzamiento de Kubernetes 1.12 en Windows Server [versión 1803](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1803#kubernetes) beta, los usuarios pueden sacar provecho de las [características más recientes](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features) en Kubernetes en Windows:

  - **administración de red simplificada**: usar Flannel en modo host-gateway para la administración de ruta automático entre los nodos
  - **mejoras de la escalabilidad**: disfrutar tiempos de inicio más rápido y confiable contenedor gracias a [las vNICs sin dispositivos para contenedores de Windows Server](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/)
  - **aislamiento de Hyper-v (alfa)**: coordinar [contenedores de hyper-v](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) con aislamiento de modo kernel para mejorar la seguridad ([consulta los tipos de contenedores de Windows](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types))
  - **complementos de almacenamiento**: usar el [complemento de almacenamiento FlexVolume](https://github.com/Microsoft/K8s-Storage-Plugins) con soporte técnico de iSCSI y SMB para contenedores de Windows

> [!TIP] 
> Si quieres implementar un clúster en Azure, la herramienta ACS-Engine de código abierto facilita esta tarea. Hay un [tutorial](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/windows.md) paso a paso disponible.

## <a name="prerequisites"></a>Requisitos previos ##

### <a name="plan-ip-addressing-for-your-cluster"></a>Planear la asignación de direcciones IP para el clúster ###
<a name="definitions"></a>Como los clústeres de Kubernetes presentan nuevas subredes de pods y servicios, es importante asegurarse de que ninguno de ellos entran en conflicto con cualquier otra red en tu entorno existente. Estos son todos los espacios de direcciones que deben ser liberados automáticamente para implementar correctamente de Kubernetes:

| Subred / intervalo de direcciones | Descripción | Valor predeterminado |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Subred de servicio** | Una subred puramente virtual, y no enrutable que pods la usan para acceder de forma a servicios sin preocuparse por la topología de red. Se traduce al espacio de direcciones enrutables, y desde él, mediante `kube-proxy` que se ejecuta en los nodos. | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**Subred de clúster** |  Se trata de una subred global que se usa en todos los pods en el clúster. Cada nodos se asigna una menor /24 subred desde este para sus pods los usen. Debe ser lo suficientemente grande como para dar cabida a todos los pods que se usa en el clúster. Para calcular el tamaño *mínimo* de subred: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Ejemplo de un clúster de 5 nodos para 100 pods por nodo: `(5) + (5 *  100) = 505`.  | "10.244.0.0/16" |
| **IP de servicio de DNS de Kubernetes** | Dirección IP de servicio de "kube dns" que se usará para la detección de servicios de clúster y de resolución DNS. | "10.96.0.10" |
> [!NOTE]
> No hay otra red de Docker (NAT) que se crea de forma predeterminada cuando se instala Docker. No es necesaria para que funcione Kubernetes en Windows como asignamos direcciones IP de la subred de clúster en su lugar.

### <a name="disable-anti-spoofing-protection"></a>Deshabilitar la protección contra la suplantación ###
> [!Important] 
> Lea atentamente esta sección sea necesaria para cualquier usuario use correctamente las máquinas virtuales para implementar Kubernetes en Windows hoy en día.

Asegúrate de suplantación de direcciones MAC y está habilitada la virtualización para el host de contenedor de Windows (invitados) de las máquinas virtuales. Para lograr esto, debes ejecutar lo siguiente como administrador en el equipo que hospeda las máquinas virtuales (ejemplo de Hyper-V):

```powershell
Set-VMProcessor -VMName "<name>" -ExposeVirtualizationExtensions $true 
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```
> [!TIP]
> Si estás usando un producto basado en VMware para satisfacer sus necesidades de virtualización, busque en habilitar el [modo promiscuo](https://kb.vmware.com/s/article/1004099) para el requisito de suplantación de identidad de MAC.

>[!TIP]
> Si vas a implementar Kubernetes en las máquinas virtuales de Azure IaaS tú mismo, busque en máquinas virtuales que admiten [la virtualización anidada](https://azure.microsoft.com/en-us/blog/nested-virtualization-in-azure/) para este requisito.

## <a name="what-you-will-accomplish"></a>Qué lograrás ##

Al final de esta guía, habrás:

> [!div class="checklist"]
> * Crea un nodo de [maestro de Kubernetes](./creating-a-linux-master.md) .  
> * Selecciona una [solución de red](./network-topologies.md).  
> * Unido a un [nodo de trabajo de Windows](./joining-windows-workers.md) o un [nodo de trabajo de Linux](./joining-linux-workers.md) a ella.  
> * Implementa un [recurso de Kubernetes de muestra](./deploying-resources.md).  
> * Solucionado [errores y problemas comunes](./common-problems.md).

## <a name="next-steps"></a>Pasos siguientes ##
En esta sección, hablamos sobre importantes requisitos previos y suposiciones necesarias para implementar correctamente hoy en día de Kubernetes en Windows. Continuar aprender a configurar a un maestro de Kubernetes:

> [!div class="nextstepaction"]
> [Crear un maestro de Kubernetes](./creating-a-linux-master.md)