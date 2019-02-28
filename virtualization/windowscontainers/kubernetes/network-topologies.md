---
title: Topologías de red
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Topologías de red admitidas en Windows y Linux.
keywords: kubernetes, 1.13, windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 9f96fcc80c533b74ab46d93beecc7ca8629ce395
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120453"
---
# <a name="network-solutions"></a>Soluciones de red #

Una vez que tengas [un nodo de maestro de Kubernetes de instalación](./creating-a-linux-master.md) ya estás listo para seleccionar una solución de red. Hay varias maneras de ganar la [subred de clúster](./getting-started-kubernetes-windows.md#cluster-subnet-def) de virtual enrutable en todos los nodos. Elegir una de las siguientes opciones para Kubernetes en Windows hoy en día:

1. Usar un complemento CNI como [Flannel](#flannel-in-vxlan-mode) para configurar una red superpuesta para TI.
2. Usar un complemento de CNI como [Flannel](#flannel-in-host-gateway-mode) las rutas de programa para TI.
3. Configurar un inteligente de [cambiar la parte superior del rack (ToR)](#configuring-a-tor-switch) para enrutar a la subred.

> [!tip]  
> Hay un cuarto solución en Windows que aprovecha el vSwitch abierto (raíz) y de red Virtual abierto (OVN) de red. Documentar está fuera del ámbito de este documento, pero puede leer [estas instrucciones](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) para configurarlo.

## <a name="flannel-in-vxlan-mode"></a>Flannel en modo de vxlan

Flannel en modo de vxlan puede usarse para configurar una red de superposición virtual configurable que usa VXLAN túnel para enrutar paquetes entre los nodos.

### <a name="prepare-kubernetes-master-for-flannel"></a>Preparar el maestro de Kubernetes para Flannel
Se recomienda algunos preparación secundaria en el [maestro de Kubernetes](./creating-a-linux-master.md) en nuestro clúster. Se recomienda para permitir puente el tráfico IPv4 iptables cadenas al usar Flannel. Esto puede hacerse mediante el siguiente comando:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Descargar & configurar Flannel ###
Descarga el manifiesto Flannel más reciente:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Hay dos secciones que se debe modificar para habilitar el back-end de redes vxlan:

1. En el `net-conf.json` sección de tu `kube-flannel.yml`, vuelve a comprobar:
 * La subred de clúster (por ejemplo, "10.244.0.0/16") se establece como deseado.
 * VNI 4096 se establece en el back-end
 * Puerto 4789 se establece en el back-end
2. En el `cni-conf.json` sección de tu `kube-flannel.yml`, cambiar el nombre de la red `"vxlan0"`.

Después de aplicar los pasos anteriores, la `net-conf.json` debe tener el siguiente aspecto:
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
> Debe establecerse el VNI 4096 y puerto 4789 para Flannel en Linux para interoperar con Flannel en Windows. Compatibilidad con otros VNIs estará disponible próximamente. Para ver una explicación de estos campos, consulte [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) .

Tu `cni-conf.json` debe tener el siguiente aspecto:
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
> Para obtener más información sobre las opciones anteriores, vea a oficial CNI [flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference), [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)y [puente](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) complemento documentos para Linux.

### <a name="launch-flannel--validate"></a>Iniciar Flannel & validar ###
Iniciar Flannel usando:

```bash
kubectl apply -f kube-flannel.yml
```

A continuación, dado que los pods Flannel basado en Linux, aplicar la revisión de Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) `kube-flannel-ds` DaemonSet seleccionar como destino solo Linux (que se iniciará el Flannel "flanneld" agente de host proceso en Windows más adelante al unir):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Si todos los nodos no están en función de 64 x86, reemplaza `-amd64` anteriormente con la arquitectura del procesador.

Después de unos minutos, deberías ver todos los pods como si se ha implementado la red de pod Flannel en ejecución.

```bash
kubectl get pods --all-namespaces
```

![texto](media/kube-master.png)

El DaemonSet Flannel también debe tener la NodeSelector `beta.kubernetes.io/os=linux` aplicado.

```bash
kubectl get ds -n kube-system
```

![texto](media/kube-daemonset.png)

> [!tip]  
> Para lo demás flannel - ds-* DaemonSets, estos pueden se omiten o eliminar ya no programarse si no hay ningún nodo que coinciden con la arquitectura del procesador.

> [!tip]  
> ¿Confundirse? Este es un [ejemplo kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) de completa para Flannel v0.11.0 con estos pasos aplicado previamente de subred de clúster de forma predeterminada `10.244.0.0/16`.

Una vez que se realiza correctamente, seguir los [pasos siguientes](#next-steps).

## <a name="flannel-in-host-gateway-mode"></a>Flannel en modo host-gateway

Junto con [Flannel vxlan](#flannel-in-vxlan-mode), otra opción para las redes de Flannel es el *modo host-gateway* (gw host), que implica la programación de rutas estáticas en cada nodo para subredes del otro nodo con la dirección del host del nodo de destino como un salto siguiente.

### <a name="prepare-kubernetes-master-for-flannel"></a>Preparar el maestro de Kubernetes para Flannel

Se recomienda algunos preparación secundaria en el [maestro de Kubernetes](./creating-a-linux-master.md) en nuestro clúster. Se recomienda para permitir puente el tráfico IPv4 iptables cadenas al usar Flannel. Esto puede hacerse mediante el siguiente comando:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Descargar & configurar Flannel ###
Descarga el manifiesto Flannel más reciente:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Hay un archivo que necesitas cambiar con el fin de habilitar la host-gw redes en ambos Windows y Linux.

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
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Si todos los nodos no están en función de 64 x86, reemplaza `-amd64` anteriormente con la arquitectura de procesador deseado.

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
> Para lo demás flannel - ds-* DaemonSets, estos pueden se omiten o eliminar ya no programarse si no hay ningún nodo que coinciden con la arquitectura del procesador.

> [!tip]  
> ¿Confundirse? Este es un v0.11.0 de para Flannel completa [ejemplo kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) con estos 2 pasos aplicado previamente de subred de clúster de forma predeterminada `10.244.0.0/16`.

Una vez que se realiza correctamente, seguir los [pasos siguientes](#next-steps).

## <a name="configuring-a-tor-switch"></a>Configuración de un conmutador ToR ##
> [!NOTE]
> Puedes omitir esta sección si has elegido [Flannel como solución de red](#flannel-in-host-gateway-mode).
Configuración del conmutador ToR tiene lugar fuera de los nodos reales. Para obtener más información sobre esto, consulta [los documentos de Kubernetes oficiales](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Pasos siguientes ## 
En esta sección, hemos visto cómo seleccionar y configurar una solución de red. Ahora ya estás listo para paso 4:

> [!div class="nextstepaction"]
> [Unirse a los trabajadores de Windows](./joining-windows-workers.md)