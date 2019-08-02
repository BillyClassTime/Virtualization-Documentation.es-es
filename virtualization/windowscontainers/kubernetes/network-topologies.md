---
title: Topologías de red
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Topologías de red compatibles en Windows y Linux.
keywords: kubernetes, 1,14, Windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6b0e13258b749ad3dfd5c8349200ca8a54908952
ms.sourcegitcommit: 42cb47ba4f3e22163869d094bd0c9cff415a43b0
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2019
ms.locfileid: "9884986"
---
# <a name="network-solutions"></a>Soluciones de red #

Una vez que haya [configurado un nodo maestro de Kubernetes](./creating-a-linux-master.md) , estará listo para elegir una solución de red. Hay varias maneras de hacer que la [subred del clúster](./getting-started-kubernetes-windows.md#cluster-subnet-def) virtual se pueda enrutar a través de los nodos. Elija una de las siguientes opciones para Kubernetes en Windows hoy:

1. Use un complemento de CNI, como [flannel](#flannel-in-vxlan-mode) , para configurar una red de superposición para usted.
2. Use un complemento de CNI, como [flannel](#flannel-in-host-gateway-mode) , para programar rutas para usted (usa el modo de red de l2bridge).
3. Configure un [conmutador inteligente de subrack (Tor)](#configuring-a-tor-switch) para enrutar la subred.

> [!tip]  
> Hay una cuarta solución de red en Windows que aprovecha el Open vSwitch (OvS) y Open Virtual Network (OVN). La documentación de este documento está fuera del ámbito, pero puede leer [estas instrucciones](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) para configurarlo.

## <a name="flannel-in-vxlan-mode"></a>Flannel en modo vxlan

Flannel en el modo vxlan se puede usar para configurar una red de superposición virtual configurable que use el túnel de VXLAN para enrutar paquetes entre nodos.

### <a name="prepare-kubernetes-master-for-flannel"></a>Preparar el maestro de Kubernetes para flannel
Se recomienda una preparación menor en el [Kubernetes maestro](./creating-a-linux-master.md) de nuestro clúster. Se recomienda habilitar el tráfico IPv4 con puente en cadenas iptables al usar flannel. Esto se puede hacer con el siguiente comando:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Descargar & configurar flannel ###
Descarga el manifiesto de flannel más reciente:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Hay dos secciones que debe modificar para habilitar el backend de redes de vxlan:

1. En la `net-conf.json` sección de su `kube-flannel.yml`, vuelva a comprobar:
 * La subred del clúster (por ejemplo, "10.244.0.0/16") se establece como se desee.
 * VNI 4096 se establece en el back-end
 * El puerto 4789 está establecido en el back-end
2. En la `cni-conf.json` sección de su `kube-flannel.yml`, cambie el nombre de red `"vxlan0"`a.

Después de aplicar los pasos anteriores, `net-conf.json` debe tener el siguiente aspecto:
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
> La VNI debe establecerse en 4096 y en el puerto 4789 para flannel en Linux para interoperar con flannel en Windows. El soporte técnico para otros VNIs estará disponible próximamente. Para obtener una explicación de estos campos, consulta [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) .

Debe `cni-conf.json` tener el siguiente aspecto:
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
> Para obtener más información sobre las opciones anteriores, consulte documentos oficiales de CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference), [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)y complementos de [puente](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) para Linux.

### <a name="launch-flannel--validate"></a>Iniciar flannel & validar ###
Inicie flannel con:

```bash
kubectl apply -f kube-flannel.yml
```

Después, dado que los pods de flannel se basan en Linux, aplique [](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) el parche de `kube-flannel-ds` NodeSelector Linux a DaemonSet solo a Linux objetivo (a partir de ahora, iniciaremos el proceso de agente de host de flannel "flanneld" en Windows más adelante):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Si alguno de los nodos no está basado en x86 `-amd64` -64, sustituya por la arquitectura del procesador.

Después de unos minutos, deberías ver que todos los pods estén ejecutándose si la red pod de flannel fue implementada.

```bash
kubectl get pods --all-namespaces
```

![texto](media/kube-master.png)

El flannel DaemonSet también debería tener el NodeSelector `beta.kubernetes.io/os=linux` aplicado.

```bash
kubectl get ds -n kube-system
```

![texto](media/kube-daemonset.png)

> [!tip]  
> Para el resto de flannel-DS-* DaemonSets, se pueden ignorar o eliminar, ya que no se programarán si no hay nodos que coincidan con la arquitectura del procesador.

> [!tip]  
> Sabe? A continuación se [muestra un ejemplo completo Kube-flannel. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) para flannel v 0.11.0 con estos pasos preaplicados para la `10.244.0.0/16`subred de clúster predeterminada.

Una vez que se haya realizado correctamente, continúe con los [pasos siguientes](#next-steps).

## <a name="flannel-in-host-gateway-mode"></a>Flannel en modo de puerta de enlace de host

Junto con [flannel vxlan](#flannel-in-vxlan-mode), otra opción para la conexión de flannel es *el modo de puerta de enlace de host* (host-GW), que conlleva la programación de rutas estáticas en cada nodo para las subredes pod de otros nodos que usan la dirección de host del nodo de destino como próximo salto.

### <a name="prepare-kubernetes-master-for-flannel"></a>Preparar el maestro de Kubernetes para flannel

Se recomienda una preparación menor en el [Kubernetes maestro](./creating-a-linux-master.md) de nuestro clúster. Se recomienda habilitar el tráfico IPv4 con puente en cadenas iptables al usar flannel. Esto se puede hacer con el siguiente comando:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Descargar & configurar flannel ###
Descarga el manifiesto de flannel más reciente:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Hay un archivo que debe cambiar para habilitar las redes host-GW en Windows/Linux.

En la `net-conf.json` sección de su Kube-flannel. yml, compruebe lo siguiente:
1. El tipo de back-end de red que se `host-gw` usa se establece `vxlan`en en lugar de.
2. La subred del clúster (por ejemplo, "10.244.0.0/16") se establece como se desee.

Después de aplicar los dos pasos, `net-conf.json` debe tener el siguiente aspecto:
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
Inicie flannel con:

```bash
kubectl apply -f kube-flannel.yml
```

A continuación, dado que los pods de flannel se basan en Linux, [](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) aplique nuestro parche `kube-flannel-ds` de NodeSelector Linux a DaemonSet solo para Linux objetivo (iniciaremos el proceso de agente de host de flannel "flanneld" en Windows más adelante, al unirse):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Si algún nodo no está basado en x86-64 `-amd64` , reemplace por encima por la arquitectura de procesador deseada.

Después de unos minutos, deberías ver que todos los pods estén ejecutándose si la red pod de flannel fue implementada.

```bash
kubectl get pods --all-namespaces
```

![texto](media/kube-master.png)

El flannel DaemonSet también debería tener el NodeSelector aplicado.

```bash
kubectl get ds -n kube-system
```

![texto](media/kube-daemonset.png)

> [!tip]  
> Para el resto de flannel-DS-* DaemonSets, se pueden ignorar o eliminar, ya que no se programarán si no hay nodos que coincidan con la arquitectura del procesador.

> [!tip]  
> Sabe? A continuación se [muestra un ejemplo completo Kube-flannel. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) para flannel v 0.11.0 con estos 2 pasos preaplicados para la `10.244.0.0/16`subred de clúster predeterminada.

Una vez que se haya realizado correctamente, continúe con los [pasos siguientes](#next-steps).

## <a name="configuring-a-tor-switch"></a>Configurar un modificador de ToR ##
> [!NOTE]
> Puede omitir esta sección si elige [flannel como su solución de red](#flannel-in-host-gateway-mode).
La configuración del parámetro de ToR se produce fuera de los nodos reales. Para obtener más información, consulta los [documentos oficiales de Kubernetes](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Pasos siguientes ## 
En esta sección, hemos explicado cómo elegir y configurar una solución de red. Ahora ya está listo para el paso 4:

> [!div class="nextstepaction"]
> [Unirse a trabajadores de Windows](./joining-windows-workers.md)