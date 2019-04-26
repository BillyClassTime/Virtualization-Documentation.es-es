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
ms.openlocfilehash: 1c0a3fd25a5572604db59e0c68d8b4a3d84b00e9
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576316"
---
# <a name="network-isolation-and-security"></a>Aislamiento de red y seguridad

## <a name="isolation-with-network-namespaces"></a>Aislamiento con espacios de nombres de red

Cada punto de conexión del contenedor está situado en su propio __espacio de nombres de red__. El vNIC del host de administración y la pila de red de host se sitúan en el espacio de nombres de la red predeterminada. Para aplicar el aislamiento de red entre contenedores del mismo host, se crea un espacio de nombres de red para cada contenedor de Windows Server y contenedores que se ejecutan en el aislamiento de Hyper-V en el que esté instalado el adaptador de red para el contenedor. Los contenedores de Windows Server usan una vNIC de host para conectarse al conmutador virtual. Aislamiento de Hyper-V usa una NIC de máquina virtual sintética (no se expone a la máquina virtual de utilidad) para conectarse al conmutador virtual.

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

  >[!NOTE]
  >Antes de Windows Server, versión 1709 y Windows 10 Fall Creators Update, la regla de entrada de forma predeterminada era denegar todo. Los usuarios que ejecuten estas versiones anteriores pueden crear las reglas de Permitir entrada con ``docker run -p`` (reenvío de puerto).

### <a name="hyper-v-isolation"></a>Aislamiento de Hyper-V

Contenedores que se ejecutan en el aislamiento de Hyper-V tienen su propio kernel aislado y, por tanto, ejecutan su propia instancia de Firewall de Windows con la siguiente configuración:

* Predeterminado PERMITIR TODO en ambos Firewall de Windows (que se ejecutan en la VM de utilidad) y VFP

![texto](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Pods de Kubernetes

En un [pod de Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/pod/), un contenedor de infraestructura primero se crea al que se une un punto de conexión. Los contenedores que pertenecen al mismo pod, incluidos los contenedores de infraestructura y trabajadores, comparten un espacio de nombres de red común (mismo IP y espacio de puerto).

![texto](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>Personalización de los ACL de puerto predeterminado

Si quieres modificar el puerto predeterminado ACL, lee nuestra documentación de servicio de redes de Host en primer lugar (vínculo se agregará pronto). Tendrás que actualizar las directivas dentro de los siguientes componentes:

>[!NOTE]
>Para el aislamiento de Hyper-V en modo transparente y NAT, actualmente no se puede reprogramar los ACL de puerto predeterminado. Esto se refleja mediante una "X" en la tabla.

| Controlador de red | Contenedores de Windows Server | Aislamiento de Hyper-V  |
| -------------- |-------------------------- | ------------------- |
| Transparente | Firewall de Windows | X |
| NAT | Firewall de Windows | X |
| L2Bridge | Ambos | VFP |
| L2Tunnel | Ambos | VFP |
| Superposición  | Ambos | VFP |