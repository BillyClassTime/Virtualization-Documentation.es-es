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
ms.openlocfilehash: 7203989483cb07423b70ff8cc644f715ba4be274
ms.sourcegitcommit: ec186664e76d413d3bf75f2056d5acb556f4205d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2018
ms.locfileid: "1876161"
---
# <a name="network-isolation-and-security"></a>Aislamiento y seguridad de red

## <a name="isolation-with-network-namespaces"></a>Aislamiento con espacio de nombres de redes
Cada punto de conexión del contenedor está situado en su propio __espacio de nombres de red__. El vNIC del host de administración y la pila de red de host se sitúan en el espacio de nombres de la red predeterminada. Para aplicar el aislamiento de red entre contenedores del mismo host, se crea un espacio de nombres de red para cada Windows Server y cada contenedor de Hyper-V en el que esté instalado el adaptador de red para el contenedor. Los contenedores de Windows Server usan una vNIC de host para conectarse al conmutador virtual. Los contenedores de Hyper-V usan una NIC de VM sintética (no expuesta a la VM de utilidad) para conectarse al conmutador virtual.


![texto](media/network-compartment-visual.png)


```powershell 
Get-NetCompartment
```

## <a name="network-security"></a>Seguridad de red
Dependiendo de qué contenedor y controlador de red se use, se aplican las ACL de puerto mediante una combinación del Firewall de Windows y [VFP](https://www.microsoft.com/en-us/research/project/azure-virtual-filtering-platform/).

### <a name="windows-server-containers"></a>Contenedores de Windows Server
Estos usan el firewall de los hosts de Windows (habilitado con espacios de nombres de red), así como VFP
  * Valor predeterminado saliente: PERMITIR TODO
  * Entrada predeterminada: PERMITIR TODO (TCP, UDP, ICMP, IGMP) el tráfico de red no solicitado
    * DENEGAR TODO el otro tráfico de red que no sea de estos protocolos

  > Nota: antes de Windows Server, versión 1709, y Windows 10 Fall Creators Update, el valor predeterminado *inbound* era DENEGAR todo. Los usuarios que ejecutan estas versiones más antiguas pueden crear reglas de PERMITIR usando ``docker run -p`` (enrutamiento de puerto)


### <a name="hyper-v-containers"></a>Contenedores de Hyper-V
Los contenedores de Hyper-V tienen su propio kernel aislado y, por tanto, ejecutan su propia instancia de Firewall de Windows con la siguiente configuración:
  * Predeterminado PERMITIR TODO en ambos Firewall de Windows (que se ejecutan en la VM de utilidad) y VFP


![texto](media/windows-firewall-containers.png)


### <a name="kubernetes-pods"></a>Pods de Kubernetes
En [Pods de Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/pod/), primero se crea un contenedor de infraestructura al que se une un punto de conexión. Los contenedores (incluidos los contenedores de infraestructura y de trabajo) que pertenecen al mismo pod comparten un espacio de nombres de red común (mismo IP y espacio de puerto).


![texto](media/pod-network-compartment.png)


### <a name="customizing-default-port-acls"></a>Personalización de los ACL de puerto predeterminado
Si deseas modificar los ACL de puerto predeterminado, haz referencia a nuestra documentación SNP (el vínculo se agregará pronto). Tendrás que actualizar las directivas dentro de los siguientes componentes:

> NOTA: Para contenedores Hyper-V en modo transparente y NAT, actualmente no puedes reprogramar los ACL de puerto predeterminado. Esto se refleja mediante una "X" en la tabla.

| Controlador de red | Contenedores de Windows Server | Contenedores de Hyper-V  |
| -------------- |-------------------------- | ------------------- |
| Transparente | Firewall de Windows | X |
| NAT | Firewall de Windows | X |
| L2Bridge | Ambos | VFP |
| L2Tunnel | Ambos | VFP |
| Superposición  | Ambos | VFP |