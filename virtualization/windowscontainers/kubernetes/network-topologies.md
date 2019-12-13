---
title: Topologías de red
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Topologías de red admitidas en Windows y Linux.
keywords: kubernetes, 1,14, Windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6b0e13258b749ad3dfd5c8349200ca8a54908952
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910315"
---
# <a name="network-solutions"></a>Network Solutions #

Una vez que haya [configurado un nodo principal de Kubernetes](./creating-a-linux-master.md) , estará listo para elegir una solución de red. Hay varias maneras de hacer que la [subred del clúster](./getting-started-kubernetes-windows.md#cluster-subnet-def) virtual se pueda enrutar entre nodos. Elija una de las siguientes opciones para Kubernetes en Windows hoy:

1. Use un complemento de CNI, como [flannel](#flannel-in-vxlan-mode) , para configurar una red de superposición.
2. Use un complemento de CNI, como [flannel](#flannel-in-host-gateway-mode) , para programar rutas automáticamente (usa el modo de red de l2bridge).
3. Configure un [conmutador de la parte superior del rack (Tor)](#configuring-a-tor-switch) para enrutar la subred.

> [!tip]  
> Hay una cuarta solución de red en Windows que aprovecha el vSwitch abierto (OvS) y Open Virtual Network (OVN). Documentar esto está fuera del ámbito de este documento, pero puede leer [estas instrucciones](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) para configurarlo.

## <a name="flannel-in-vxlan-mode"></a>Flannel en modo vxlan

Flannel en modo vxlan se puede usar para configurar una red de superposición virtual configurable que use el túnel VXLAN para enrutar paquetes entre nodos.

### <a name="prepare-kubernetes-master-for-flannel"></a>Preparar el maestro de Kubernetes para flannel
Se recomienda alguna preparación mínima en el [maestro de Kubernetes](./creating-a-linux-master.md) en el clúster. Se recomienda habilitar el tráfico IPv4 con puente en las cadenas de iptables cuando se usa flannel. Esto puede hacerse mediante el siguiente comando:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Descargar & configurar flannel ###
Descargue el manifiesto de flannel más reciente:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Hay dos secciones que debe modificar para habilitar el back-end de redes vxlan:

1. En la sección `net-conf.json` de la `kube-flannel.yml`, haga doble comprobación:
 * La subred del clúster (por ejemplo, "10.244.0.0/16") se establece como se desea.
 * VNI 4096 se establece en el back-end
 * El puerto 4789 se establece en el back-end
2. En la sección `cni-conf.json` del `kube-flannel.yml`, cambie el nombre de red a `"vxlan0"`.

Después de aplicar los pasos anteriores, el `net-conf.json` debe tener el siguiente aspecto:
```json
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan",
        "VNI" : 4096,
        "Port": 4789
      }
    }
```

> [!NOTE]  
> El valor de VNI debe establecerse en 4096 y el puerto 4789 para flannel en Linux para interoperar con flannel en Windows. Próximamente se admitirá la compatibilidad con otras VNIs. Consulte [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) para obtener una explicación de estos campos.

El `cni-conf.json` debe tener el siguiente aspecto:
```json
cni-conf.json: |
    {
      "name": "vxlan0",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
```
> [!tip]  
> Para más información sobre las opciones anteriores, consulte los documentos oficiales de CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference), [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)y [Bridge](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) plugin para Linux.

### <a name="launch-flannel--validate"></a>Iniciar flannel & validar ###
Inicie flannel mediante:

```bash
kubectl apply -f kube-flannel.yml
```

Después, dado que los pods de flannel están basados en Linux, aplique la revisión [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) de linux a `kube-flannel-ds` DaemonSet solo para Linux de destino (se iniciará el proceso de agente de host de flannel "flanneld" en Windows más adelante al unirse):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Si algún nodo no se basa en x86-64, reemplace `-amd64` anterior por la arquitectura del procesador.

Después de unos minutos, debería ver todos los pods como en ejecución si se ha implementado la red pod de flannel.

```bash
kubectl get pods --all-namespaces
```

![texto](media/kube-master.png)

Flannel DaemonSet también debe tener el `beta.kubernetes.io/os=linux` NodeSelector aplicado.

```bash
kubectl get ds -n kube-system
```

![texto](media/kube-daemonset.png)

> [!tip]  
> En el resto de flannel-DS-* DaemonSets, estos se pueden omitir o eliminar, ya que no se programarán si no hay ningún nodo que coincida con la arquitectura del procesador.

> [!tip]  
> ¿Está confundido? A continuación se muestra un ejemplo completo de [Kube-flannel. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) para flannel v 0.11.0 con estos pasos aplicados previamente para `10.244.0.0/16`de la subred de clúster predeterminada.

Una vez que se realice correctamente, continúe con los [pasos siguientes](#next-steps).

## <a name="flannel-in-host-gateway-mode"></a>Flannel en modo de puerta de enlace host

Junto a [flannel vxlan](#flannel-in-vxlan-mode), otra opción para redes de flannel es el *modo de puerta de enlace de host* (host-GW), que conlleva la programación de rutas estáticas en cada nodo a las subredes pod de otro nodo mediante la dirección de host del nodo de destino como próximo salto.

### <a name="prepare-kubernetes-master-for-flannel"></a>Preparar el maestro de Kubernetes para flannel

Se recomienda alguna preparación mínima en el [maestro de Kubernetes](./creating-a-linux-master.md) en el clúster. Se recomienda habilitar el tráfico IPv4 con puente en las cadenas de iptables cuando se usa flannel. Esto puede hacerse mediante el siguiente comando:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Descargar & configurar flannel ###
Descargue el manifiesto de flannel más reciente:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Hay un archivo que debe cambiar para habilitar las redes de host-GW en Windows/Linux.

En la sección `net-conf.json` de Kube-flannel. yml, compruebe lo siguiente:
1. El tipo de back-end de red que se está usando se establece en `host-gw` en lugar de `vxlan`.
2. La subred del clúster (por ejemplo, "10.244.0.0/16") se establece como se desea.

Después de aplicar los 2 pasos, el `net-conf.json` debe tener el siguiente aspecto:
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Iniciar flannel & validar ###
Inicie flannel mediante:

```bash
kubectl apply -f kube-flannel.yml
```

Después, dado que los pods de flannel están basados en Linux, aplique la revisión [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) de linux a `kube-flannel-ds` DaemonSet solo para Linux de destino (se iniciará el proceso de agente de host de flannel "flanneld" en Windows más adelante al unirse):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Si algún nodo no se basa en x86-64, reemplace `-amd64` anterior por la arquitectura de procesador deseada.

Después de unos minutos, debería ver todos los pods como en ejecución si se ha implementado la red pod de flannel.

```bash
kubectl get pods --all-namespaces
```

![texto](media/kube-master.png)

Flannel DaemonSet también debe tener el NodeSelector aplicado.

```bash
kubectl get ds -n kube-system
```

![texto](media/kube-daemonset.png)

> [!tip]  
> En el resto de flannel-DS-* DaemonSets, estos se pueden omitir o eliminar, ya que no se programarán si no hay ningún nodo que coincida con la arquitectura del procesador.

> [!tip]  
> ¿Está confundido? Este es un ejemplo completo de [Kube-flannel. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) para flannel v 0.11.0 con estos 2 pasos previamente aplicados para la `10.244.0.0/16`de la subred de clúster predeterminada.

Una vez que se realice correctamente, continúe con los [pasos siguientes](#next-steps).

## <a name="configuring-a-tor-switch"></a>Configuración de un conmutador ToR ##
> [!NOTE]
> Puede omitir esta sección si elige [flannel como solución de red](#flannel-in-host-gateway-mode).
La configuración del conmutador ToR se produce fuera de los nodos reales. Para obtener más información sobre esto, consulte [documentos oficiales de Kubernetes](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Pasos siguientes ## 
En esta sección, se ha explicado cómo elegir y configurar una solución de red. Ya está listo para el paso 4:

> [!div class="nextstepaction"]
> [Unirse a trabajos de Windows](./joining-windows-workers.md)