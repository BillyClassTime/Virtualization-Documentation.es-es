---
title: Ejecuta Kubernetes como un servicio de Windows
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: get-started-article
ms.prod: containers
description: Cómo ejecutar Kubernetes componentes como los servicios de Windows.
keywords: kubernetes, 1.13, windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: 6c68edda6e2017640b0a490c3c30f063c81698b3
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120600"
---
# <a name="kubernetes-components-as-windows-services"></a>Componentes de Kubernetes como servicios de Windows 

Algunos usuarios querer configurar procesos como flanneld.exe, kubelet.exe, kube proxy.exe u otras para ejecutarse como servicios de Windows. Esto ofrece ventajas de tolerancia a errores adicionales, como los procesos automáticamente al reiniciar tras un bloqueo de proceso o nodo inesperado.


## <a name="prerequisites"></a>Requisitos previos
1. Has descargado [nssm.exe](https://nssm.cc/download) en el `c:\k` directory
2. Que has unido el nodo al clúster y ejecuta el script de [install.ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1) o [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) en el nodo anteriormente

## <a name="registering-windows-services"></a>Registro de servicios de Windows
Puedes ejecutar [un script de muestra](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1) qué nssm.exe usos que se registrará `kubelet`, `kube-proxy`, y `flanneld.exe` para ejecutarse como servicios de Windows en segundo plano:

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
La dirección IP asignada al nodo de Windows. Puedes usar `ipconfig` encontrar esto.

|  |  | 
|---------|---------|
|Parámetro     | `-ManagementIP`        |
|Valor predeterminado    | n.A.        |


# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
El modo de red `l2bridge` (flannel gw de host) o `overlay` (flannel vxlan) elegido como una [solución de red](./network-topologies.md).

> [!Important] 
> `overlay` modo de red (flannel vxlan) requiere los archivos binarios de Kubernetes v1.14 o posterior.

|  |  | 
|---------|---------|
|Parámetro     | `-NetworkMode`        |
|Valor predeterminado    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
El [intervalo de subred de clúster](./getting-started-kubernetes-windows.md#cluster-subnet-def).

|  |  | 
|---------|---------|
|Parámetro     | `-ClusterCIDR`        |
|Valor predeterminado    | `10.244.0.0/16`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[IP de servicio de DNS de Kubernetes](./getting-started-kubernetes-windows.md#kube-dns-def).

|  |  | 
|---------|---------|
|Parámetro     | `-KubeDnsServiceIP`        |
|Valor predeterminado    | `10.96.0.10`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
El directorio donde se redirigen kubelet y el proxy de kube registros en sus archivos de salida correspondiente.

|  |  | 
|---------|---------|
|Parámetro     | `-LogDir`        |
|Valor predeterminado    | `C:\k`        |

---


> [!TIP] 
> Debe hubo, consulte la [sección de solución de problemas](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services)

## <a name="manual-approach"></a>Enfoque manual
Debe la [por encima de la secuencia de comandos que se hace referencia](#registering-windows-services) no funcione por TI, esta sección proporciona algunos *comandos de muestra* que se puede usar para registrar estos servicios manualmente paso a paso.

> [!TIP] 
> Consulta [Kubelet y el proxy de kube ahora pueden ejecutarse como servicios de Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services) para obtener más información sobre cómo configurar `kubelet` y `kube-proxy` para ejecutarse como servicios nativos de Windows a través de `sc`.

### <a name="register-flanneldexe"></a>Registrar flanneld.exe
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>Registrar kubelet.exe
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>Registrar kube proxy.exe (l2bridge / gw de host)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>Registrar kube proxy.exe (superposición / vxlan)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```