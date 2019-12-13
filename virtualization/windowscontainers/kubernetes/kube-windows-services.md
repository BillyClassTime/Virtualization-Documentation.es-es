---
title: Ejecución de Kubernetes como un servicio de Windows
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: get-started-article
ms.prod: containers
description: Cómo ejecutar los componentes de Kubernetes como servicios de Windows.
keywords: kubernetes, 1,14, Windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: cd5026a244b57b5c70d4abfe076839130315a4f5
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909805"
---
# <a name="kubernetes-components-as-windows-services"></a>Componentes de Kubernetes como servicios de Windows 

Es posible que algunos usuarios deseen configurar procesos como flanneld. exe, kubelet. exe, Kube-proxy. exe u otros para que se ejecuten como servicios de Windows. Esto aporta ventajas adicionales de tolerancia a errores, como los procesos que se reinician automáticamente tras un proceso inesperado o un bloqueo de nodo.


## <a name="prerequisites"></a>Requisitos previos
1. Ha descargado [NSSM. exe](https://nssm.cc/download) en el directorio `c:\k`
2. Ha unido el nodo al clúster y ejecutado el script [install. PS1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1) o [Start. PS1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) en el nodo previamente.

## <a name="registering-windows-services"></a>Registro de servicios de Windows
Puede ejecutar [un script de ejemplo](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1) que use NSSM. exe que registrará `kubelet`, `kube-proxy`y `flanneld.exe` para ejecutarse como servicios de Windows en segundo plano:

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# <a name="managementiptabmanagementip"></a>[ManagementIP](#tab/ManagementIP)
Dirección IP asignada al nodo de Windows. Puede usar `ipconfig` para encontrar esto.

|  |  | 
|---------|---------|
|Parámetro     | `-ManagementIP`        |
|Valor predeterminado    | n.A.        |


# <a name="networkmodetabnetworkmode"></a>[NetworkMode](#tab/NetworkMode)
El modo de red `l2bridge` (Flannel host-GW) o `overlay` (Flannel vxlan) elegido como una [solución de red](./network-topologies.md).

> [!Important] 
> `overlay` modo de red (Flannel vxlan) requiere archivos binarios Kubernetes v 1.14 o superiores.

|  |  | 
|---------|---------|
|Parámetro     | `-NetworkMode`        |
|Valor predeterminado    | `l2bridge`        |


# <a name="clustercidrtabclustercidr"></a>[ClusterCIDR](#tab/ClusterCIDR)
El [intervalo de subred del clúster](./getting-started-kubernetes-windows.md#cluster-subnet-def).

|  |  | 
|---------|---------|
|Parámetro     | `-ClusterCIDR`        |
|Valor predeterminado    | `10.244.0.0/16`        |


# <a name="kubednsserviceiptabkubednsserviceip"></a>[KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[Dirección IP del servicio DNS de Kubernetes](./getting-started-kubernetes-windows.md#kube-dns-def).

|  |  | 
|---------|---------|
|Parámetro     | `-KubeDnsServiceIP`        |
|Valor predeterminado    | `10.96.0.10`        |


# <a name="logdirtablogdir"></a>[LogDir](#tab/LogDir)
El directorio donde se redirigen los registros de kubelet y Kube-proxy a sus respectivos archivos de salida.

|  |  | 
|---------|---------|
|Parámetro     | `-LogDir`        |
|Valor predeterminado    | `C:\k`        |

---


> [!TIP] 
> Si algo va mal, consulte la [sección de solución de problemas](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services) .

## <a name="manual-approach"></a>Enfoque manual
Si el [script al que se hace referencia anteriormente](#registering-windows-services) no funciona para usted, en esta sección se proporcionan algunos *comandos de ejemplo* que se pueden usar para registrar estos servicios manualmente paso a paso.

> [!TIP] 
> Consulte [Kubelet y Kube-proxy ahora puede ejecutarse como servicios de Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services) para obtener más información sobre cómo configurar `kubelet` y `kube-proxy` para que se ejecuten como servicios nativos de Windows a través de `sc`.

### <a name="register-flanneldexe"></a>Registrar flanneld. exe
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>Registrar kubelet. exe
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>Registrar Kube-proxy. exe (l2bridge/host-GW)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>Registrar Kube-proxy. exe (Overlay/vxlan)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```