---
title: Topologías de red
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Topologías de red admitidas en Windows y Linux.
keywords: kubernetes, 1.12, windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: bcbd7b530b58b663305ea5d8b84a75eaf971f997
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/15/2018
ms.locfileid: "6948064"
---
# <a name="network-solutions"></a>Soluciones de red #

Una vez que tengas [un nodo de maestro de Kubernetes de instalación](./creating-a-linux-master.md) ya estás listo para seleccionar una solución de red. Hay varias formas de realizar la [subred de clúster](./getting-started-kubernetes-windows.md#cluster-subnet-def) de virtual enrutable en todos los nodos. Elegir una de las siguientes opciones para Kubernetes en Windows hoy en día:

1. Usar un complemento CNI de terceros, como [Flannel](network-topologies.md#flannel-in-host-gateway-mode) las rutas de instalación para TI.
1. Configurar un inteligente de [cambiar la parte superior del rack (ToR)](network-topologies.md#configuring-a-tor-switch) para enrutar a la subred.

> [!tip]  
> Hay una tercera solución en Windows que aprovecha el vSwitch abierto (raíz) y de red Virtual abierto (OVN) de red. Documentar está fuera del ámbito de este documento, pero puede leer [estas instrucciones](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) para configurarlo.

## <a name="flannel-in-host-gateway-mode"></a>Flannel en modo host-gateway

Una de las opciones disponibles para las redes de Flannel es el *modo host-gateway* (gw host), que implica la configuración de rutas estáticas entre subredes de Pods en todos los nodos.
> [!NOTE]  
> Esto es diferente a modo de red de *superposición* en Flannel, que usa encapsulación VXLAN y está en desarrollo ahora mismo. Mira este espacio …

### <a name="prepare-kubernetes-master-for-flannel"></a>Preparar el maestro de Kubernetes para Flannel

Se recomienda algunos preparación secundaria en el [maestro de Kubernetes](./creating-a-linux-master.md) en nuestro clúster. Se recomienda para permitir puente el tráfico IPv4 iptables cadenas al usar Flannel. Esto puede hacerse mediante el siguiente comando:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Descargar y configurar Flannel ###
Descarga el manifiesto Flannel más reciente:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Hay dos cosas que debes hacer para habilitar host-gw redes en ambos Windows y Linux.

En el `net-conf.json` sección de tu flannel.yml kube, vuelve a comprobar que:
1. Se establece el tipo de back-end de red que se usa en `host-gw` en lugar de `vxlan`.
2. La subred de clúster (por ejemplo, "10.244.0.0/16") se establece como deseado.

Después de aplicar los 2 pasos, la `net-conf.json` debe tener el siguiente aspecto:
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Iniciar Flannel & validar ###
Iniciar Flannel usando:

```bash
kubectl apply -f kube-flannel.yml
```

A continuación, dado que los pods Flannel basado en Linux, aplicar nuestra revisión de Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) a `kube-flannel-ds` DaemonSet seleccionar como destino solo Linux (que se iniciará el Flannel "flanneld" agente de host proceso en Windows más adelante al unir):

```
kubectl patch ds/kube-flannel-ds --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

Después de unos minutos, deberías ver todos los pods como si se ha implementado la red de pod Flannel en ejecución.

```bash
kubectl get pods --all-namespaces
```

![texto](media/kube-master.png)

El DaemonSet Flannel también debe tener el NodeSelector aplicado.

```bash
kubectl get ds -n kube-system
```

![texto](media/kube-daemonset.png)
> [!tip]  
> ¿Confundirse? Este es un v0.9.1 de para Flannel completa [ejemplo kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) con estos 2 pasos aplicado previamente de subred de clúster de forma predeterminada `10.244.0.0/16`.

## <a name="configuring-a-tor-switch"></a>Configuración de un conmutador ToR ##
> [!NOTE]
> Puedes omitir esta sección si has elegido [Flannel como solución de red](#flannel-in-host-gateway-mode).
Configuración del conmutador ToR tiene lugar fuera de los nodos reales. Para obtener más detalles sobre esto, consulta [los documentos de Kubernetes oficiales](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Pasos siguientes ## 
En esta sección, hemos visto cómo seleccionar una solución de red. Ahora estás listo para paso 4:

> [!div class="nextstepaction"]
> [Unirse a los trabajadores de Windows](./joining-windows-workers.md)