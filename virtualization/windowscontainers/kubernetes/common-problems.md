---
title: Solución de problemas de Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Soluciones para problemas comunes al implementar Kubernetes y unirse a nodos de Windows.
keywords: kubernetes, 1,14, Linux, compilación
ms.openlocfilehash: bdf1fd78bbbebcad3562872d9e71c961be6c64eb
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883008"
---
# <a name="troubleshooting-kubernetes"></a>Solución de problemas de Kubernetes #
Esta página te guía a través de varios problemas comunes con las implementaciones, redes y configuración de Kubernetes.

> [!tip]
> Recomienda un elemento de Preguntas frecuentes enviando una PR a [nuestro repositorio de documentación](https://github.com/MicrosoftDocs/Virtualization-Documentation/).

Esta página se subdivide en las siguientes categorías:
1. [Preguntas generales](#general-questions)
2. [Errores comunes de red](#common-networking-errors)
3. [Errores comunes de Windows](#common-windows-errors)
4. [Errores comunes de los maestros de Kubernetes](#common-kubernetes-master-errors)

## <a name="general-questions"></a>Preguntas generales ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>¿Cómo sé que Start. PS1 en Windows se completó correctamente? ###
Debe ver kubelet, Kube-proxy y (si ha elegido flannel como solución de red) flanneld procesos de hosts que se ejecutan en el nodo, con la ejecución de registros que se muestran en distintas ventanas de PoSh. Además, el nodo Windows debe aparecer como "listo" en el clúster Kubernetes.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>¿Puedo configurar para ejecutar todo esto en segundo plano en lugar de PoSh Windows? ###
A partir de Kubernetes versión 1,11, kubelet & Kube-proxy se puede ejecutar como [servicios](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)nativos de Windows. También puedes usar administradores de servicios alternativos, como [NSSM. exe](https://nssm.cc/) , para ejecutar siempre estos procesos (flanneld, kubelet & Kube-proxy) en segundo plano. Vea [servicios de Windows en Kubernetes](./kube-windows-services.md) por pasos de ejemplo.

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>Tengo problemas al ejecutar los procesos de Kubernetes como servicios de Windows ###
Para la solución de problemas iniciales, puede usar los siguientes marcadores en [NSSM. exe](https://nssm.cc/) para redirigir stdout y stderr a un archivo de salida:
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
Para obtener más información, consulte documentos oficiales sobre el [uso de NSSM](https://nssm.cc/usage) .

## <a name="common-networking-errors"></a>Errores comunes de red ##

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>Veo errores como "error de hnsCall en Win32: el disco equivocado está en la unidad". ###
Este error puede producirse cuando se realizan modificaciones personalizadas en objetos SNP o se instala una nueva actualización de Windows que introduce cambios en SNP sin recortar los objetos SNP anteriores. Indica que un objeto SNP creado previamente antes de una actualización es incompatible con la versión SNP instalada actualmente.

En Windows Server 2019 (y versiones anteriores), los usuarios pueden eliminar objetos SNP eliminando el archivo SNP. Data 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

Los usuarios deben poder eliminar directamente los puntos de conexión o redes SNP incompatibles:
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Los usuarios de Windows Server, versión 1903, pueden ir a la siguiente ubicación del registro y eliminar cualquier NIC empezando con el nombre de `vxlan0` red `cbr0`(por ejemplo, o):
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```


### <a name="my-windows-pods-cannot-ping-external-resources"></a>Mis pods de Windows no pueden hacer ping a recursos externos ###
Los pods de Windows no tienen reglas de salida programadas para el protocolo ICMP hoy. Sin embargo, se admite TCP/UDP. Al intentar demostrar la conectividad de recursos fuera del clúster, sustituya `ping <IP>` los comandos correspondientes `curl <IP>` .

Si sigues teniendo problemas, lo más probable es que la configuración de red de [CNI. conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) sea más importante. Siempre puede editar este archivo estático, la configuración se aplicará a los recursos de Kubernetes recién creados.

¿Por qué?
Uno de los requisitos de redes de Kubernetes (consulte el [modelo Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) es para que la comunicación de clústeres se produzca sin NAT internamente. Para cumplir este requisito, tenemos una de las [excepciones](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) para todas las comunicaciones en las que no desea que se produzca NAT saliente. Sin embargo, esto también significa que tiene que excluir la dirección IP externa que intenta consultar desde la misma. Solo entonces el tráfico originario de sus pods de Windows se SNAT'ed correctamente para recibir una respuesta del mundo exterior. En este sentido, tu cuenta de `cni.conf` excepciones en debería tener el siguiente aspecto:
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>Mi nodo Windows no puede obtener acceso a un servicio NodePort ###
El acceso local de NodePort desde el nodo mismo fallará. Se trata de una limitación conocida. NodePort Access funcionará desde otros nodos o clientes externos.

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>Después de un tiempo, se eliminan los extremos de vNICs y SNP de los contenedores ###
Este problema se puede producir cuando el `hostname-override` parámetro no se pasa a [Kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/). Para resolverlo, los usuarios deben pasar el nombre de host a Kube-proxy de la siguiente manera:
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>En el modo flannel (vxlan), mis pods tienen problemas de conectividad después de volver a unir el nodo ###
Cada vez que se vuelva a unir a un clúster un nodo previamente eliminado, flannelD intentará asignar una nueva subred Pod al nodo. Los usuarios deben quitar los antiguos archivos de configuración de subred del conjunto Pod en las siguientes rutas:
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Después de iniciar Start. ps1, Flanneld se bloquea en "esperando que se cree la red" ###
Hay numerosos informes de este problema que se están investigando; lo más probable es que se produzca un problema de temporización cuando se establece la IP de administración de la red flannel. Una solución es reiniciar simplemente Start. PS1 o volver a iniciarlo manualmente de la siguiente manera:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

En la actualidad, hay un [PR](https://github.com/coreos/flannel/pull/1042) que soluciona este problema.


### <a name="on-flannel-host-gw-my-windows-pods-do-not-have-network-connectivity"></a>En flannel (host-GW), mis pods de Windows no tienen conectividad de red ###
Si desea usar l2bridge para redes (también conocido como [flannel host-Gateway](./network-topologies.md#flannel-in-host-gateway-mode)), debe asegurarse de habilitar la suplantación de direcciones MAC para las máquinas virtuales host contenedoras de Windows. Para ello, debe ejecutar lo siguiente como administrador en el equipo que hospeda las máquinas virtuales (ejemplo proporcionado para Hyper-V):

```powershell
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> [!TIP]
> Si está usando un producto basado en VMware para satisfacer sus necesidades de virtualización, consulte habilitar el [modo promiscuo](https://kb.vmware.com/s/article/1004099) para el requisito de suplantación de equipos Mac.

>[!TIP]
> Si implementa Kubernetes en las máquinas virtuales de Azure o IaaS desde otros proveedores de nube, también puede usar la [red superpuesta](./network-topologies.md#flannel-in-vxlan-mode) en su lugar.

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Mis pods de Windows no se pueden iniciar porque falta/Run/flannel/subnet.env ###
Esto indica que flannel no se inició correctamente. Puede intentar reiniciar flanneld. exe o puede copiar los archivos de forma manual desde `/run/flannel/subnet.env` el maestro de Kubernetes al `C:\run\flannel\subnet.env` nodo de trabajo de Windows y modificar la `FLANNEL_SUBNET` fila a la subred asignada. Por ejemplo, si se asignó la subred del nodo 10.244.4.1/24:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
Es más seguro permitir que flanneld. exe genere este archivo por usted.

### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>La conectividad de Pod-to-Pod entre los hosts está dañada en mi clúster de Kubernetes que se ejecuta en vSphere 
Puesto que tanto vSphere como flannel reservan el puerto 4789 (puerto de VXLAN predeterminado) para las redes de superposición, los paquetes pueden acabar de interceptarse. Si se usa vSphere para las redes de superposición, debe configurarse para que use un puerto diferente para liberar 4789.  


### <a name="my-endpointsips-are-leaking"></a>Mis puntos de conexión/IPs están perdiendo ###
Existen 2 problemas conocidos actualmente que pueden provocar la pérdida de puntos de conexión. 
1.  El primer [problema conocido](https://github.com/kubernetes/kubernetes/issues/68511) es un problema en la versión 1,11 de Kubernetes. Evita usar la versión de Kubernetes 1.11.0-1.11.2.
2. El segundo [problema conocido](https://github.com/docker/libnetwork/issues/1950) que puede provocar la pérdida de los puntos de conexión es un problema de simultaneidad en el almacenamiento de los puntos de conexión. Para recibir la corrección, debe usar el acoplador EE 18,09 o una versión posterior.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>Mis pods no pueden iniciarse debido a errores "red: no se pudieron asignar para el intervalo" ###
Esto indica que se usa el espacio de direcciones IP en el nodo. Para limpiar los [puntos de conexión](#my-endpointsips-are-leaking)que se hayan perdido, migre los recursos de los nodos afectados & ejecute los siguientes comandos:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>El nodo de Windows no puede acceder a mis servicios mediante la dirección IP de servicio ###
Se trata de una limitación conocida de la actual pila de redes de Windows. Sin embargo, los *pods* **** de Windows pueden acceder a la IP del servicio.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>No se encuentra ningún adaptador de red al iniciar Kubelet ###
La pila de redes de Windows necesita un adaptador virtual para que funcionen las redes de Kubernetes. Si los siguientes comandos no devuelven ningún resultado (en un shell de administrador), se ha producido un error en la creación de la red virtual, un requisito previo necesario para que funcione Kubelet:

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

A menudo es conveniente modificar el parámetro [interfacename](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) del script Start. ps1, en aquellos casos en los que el adaptador de red del host no es "Ethernet". En caso contrario, consulte el resultado `start-kubelet.ps1` de la secuencia de comandos para ver si hay errores durante la creación de la red virtual. 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Los pods dejan de resolver correctamente consultas DNS después de un tiempo en vivo ###
Existe un problema conocido de almacenamiento en caché de DNS en la pila de red de Windows Server, versión 1803 y versiones anteriores, que a veces pueden provocar errores en las solicitudes DNS. Para evitar este problema, puede establecer los valores de caché de Max TTL en cero con las siguientes claves del registro:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>Sigo viendo problemas. ¿Qué debo hacer? ### 
Pueden existir restricciones adicionales en la red o hosts que evitan determinados tipos de comunicación entre los nodos. Asegúrate de lo siguiente:
  - ha configurado correctamente la topología de [red](./network-topologies.md) elegida.
  - Se permite el tráfico que parece que procede de los pods
  - Se permite el tráfico HTTP, en el caso de implementar servicios web
  - Los paquetes de diferentes protocolos (ICMP de IE frente a TCP/UDP) no se quitan


## <a name="common-windows-errors"></a>Errores comunes de Windows ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Mis pods de Kubernetes están bloqueados en "ContainerCreating" ###
Este problema puede tener varias causas, pero una de las más comunes es que la imagen de pausa se ha configurado de forma incorrecta. Se trata de un síntoma de alto nivel del siguiente problema.


### <a name="when-deploying-docker-containers-keep-restarting"></a>Al realizar la implementación, los contenedores de Docker siguen reiniciándose ###
Comprueba que la imagen de pausa sea compatible con la versión del sistema operativo. En las [instrucciones](./deploying-resources.md) se da por hecho que tanto el sistema operativo como los contenedores son de la versión 1803. Si tienes una versión posterior de Windows, como por ejemplo, una compilación de Insider, tendrás que ajustar las imágenes según corresponda. Consulta el [Repositorio de Docker](https://hub.docker.com/u/microsoft/) de Microsoft para obtener información sobre las imágenes. De todos modos, el Dockerfile de imagen de pausa y el servicio de muestra esperarán que la imagen se etiquete como `:latest`.


## <a name="common-kubernetes-master-errors"></a>Errores comunes de los maestros de Kubernetes ##
La depuración del maestro de Kubernetes recae en tres categorías principales (en función de la probabilidad):

  - Algo no funciona con los contenedores del sistema Kubernetes.
  - Algo no funciona con la forma en que `kubelet` se ejecuta.
  - Algo no funciona con el sistema.

Ejecuta `kubectl get pods -n kube-system` para ver los pods que Kubernetes crea; esto puede ofrecer información sobre cuáles se bloquean o se inician correctamente. Después ejecuta `docker ps -a` para ver todos los contenedores sin procesar que realizan copias de seguridad de estos pods. Por último, ejecuta `docker logs [ID]` en los contenedores que son sospechosos de provocar el problema para ver el resultado sin procesar de los procesos.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>No se puede conectar con el servidor de API en `https://[address]:[port]`. ###
A menudo, este error indica problemas de certificado. Asegúrate de que se ha generado correctamente el archivo de configuración, de que las direcciones IP en él coinciden con las del host y de que lo has copiado en el directorio que el servidor de la API ha montado.

Si sigues [nuestras instrucciones](./creating-a-linux-master.md), los buenos lugares en los que encontrarás esto es:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 de lo contrario, consulta el archivo de manifiesto del servidor de la API para comprobar los puntos de montaje.