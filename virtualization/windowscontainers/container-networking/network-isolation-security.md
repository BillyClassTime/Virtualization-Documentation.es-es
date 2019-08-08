---
title: Red de contenedores de Windows
description: Aislamiento de red y seguridad dentro de contenedores de Windows.
keywords: docker, contenedores
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: d5081104f1614a91d6441a5e879a439f1df1bf77
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998542"
---
# <a name="network-isolation-and-security"></a>Aislamiento de redes y seguridad

## <a name="isolation-with-network-namespaces"></a>Aislamiento con espacios de nombres de red

Cada punto de conexión del contenedor está situado en su propio __espacio de nombres de red__. El vNIC del host de administración y la pila de red de host se sitúan en el espacio de nombres de la red predeterminada. Para forzar el aislamiento de red entre contenedores en el mismo host, se crea un espacio de nombres de red para cada contenedor de Windows Server y los contenedores se ejecutan en el aislamiento de Hyper-V en el que está instalado el adaptador de red para el contenedor. Los contenedores de Windows Server usan una vNIC de host para conectarse al conmutador virtual. El aislamiento de Hyper-V usa una NIC de VM sintética (no expuesta a la VM de la utilidad) para conectar con el conmutador virtual.

![texto](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>Seguridad de red

Dependiendo de qué contenedor y controlador de red se use, se aplican las ACL de puerto mediante una combinación del Firewall de Windows y [VFP](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/).

### <a name="windows-server-containers"></a>Contenedores de Windows Server

Estos usan el firewall de los hosts de Windows (habilitado con espacios de nombres de red), así como VFP

* Valor predeterminado saliente: PERMITIR TODO
* Entrada predeterminada: PERMITIR TODO (TCP, UDP, ICMP, IGMP) el tráfico de red no solicitado
  * DENEGAR TODO el otro tráfico de red que no sea de estos protocolos

  >[!NOTE]
  >Antes de Windows Server, versión 1709 y Windows 10 Fall Creators Update, la regla de entrada predeterminada fue denegada. Los usuarios que ejecutan estas versiones anteriores pueden crear reglas de ``docker run -p`` permiso de entrada con (reenvío de puerto).

### <a name="hyper-v-isolation"></a>Aislamiento de Hyper-V

Los contenedores que se ejecutan en el aislamiento de Hyper-V tienen su propio núcleo aislado y, por lo tanto, ejecutan su propia instancia de Firewall de Windows con la siguiente configuración:

* Predeterminado PERMITIR TODO en ambos Firewall de Windows (que se ejecutan en la VM de utilidad) y VFP

![texto](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes pods

En un conjunto [pod de Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/pod/), primero se crea un contenedor de infraestructura en el que se adjunta un punto final. Contenedores que pertenecen al mismo conjunto Pod, incluidos contenedores de infraestructura y trabajadores, comparten un espacio de nombres de red común (el mismo espacio de IP y puerto).

![texto](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>Personalización de los ACL de puerto predeterminado

Si desea modificar las ACL de puertos predeterminadas, lea primero la documentación del servicio de redes de host (pronto se agregará el vínculo). Tendrá que actualizar las directivas de los siguientes componentes:

>[!NOTE]
>En el caso del aislamiento Hyper-V en modo transparente y NAT, actualmente no se pueden reprogramar las ACL de puerto predeterminado. Esto se refleja mediante una "X" en la tabla.

| Controlador de red | Contenedores de Windows Server | Aislamiento de Hyper-V  |
| -------------- |-------------------------- | ------------------- |
| Transparente | Firewall de Windows | X |
| NAT | Firewall de Windows | X |
| L2Bridge | Ambos | VFP |
| L2Tunnel | Ambos | VFP |
| Superposición  | Ambos | VFP |