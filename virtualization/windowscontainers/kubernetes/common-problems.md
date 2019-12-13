---
title: Solución de problemas de Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Soluciones para problemas comunes al implementar Kubernetes y unirse a nodos de Windows.
keywords: kubernetes, 1,14, Linux, compilación
ms.openlocfilehash: 471731ec50c7c03816a956bd7aae859ad218be6d
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910455"
---
# <a name="troubleshooting-kubernetes"></a>Solución de problemas de Kubernetes #
Esta página te guía a través de varios problemas comunes con las implementaciones, redes y configuración de Kubernetes.

> [!tip]
> Recomienda un elemento de Preguntas frecuentes enviando una PR a [nuestro repositorio de documentación](https://github.com/MicrosoftDocs/Virtualization-Documentation/).

Esta página se divide en las siguientes categorías:
1. [Preguntas generales](#general-questions)
2. [Errores comunes de redes](#common-networking-errors)
3. [Errores comunes de Windows](#common-windows-errors)
4. [Errores comunes del maestro de Kubernetes](#common-kubernetes-master-errors)

## <a name="general-questions"></a>Preguntas generales ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>Cómo sabe que Start. PS1 en Windows se completó correctamente? ###
Debería ver kubelet, Kube-proxy y (si elige flannel como solución de red) flanneld procesos de agente host que se ejecutan en el nodo, y los registros en ejecución se muestran en ventanas de PoSh independientes. Además, el nodo de Windows debe aparecer como "listo" en el clúster de Kubernetes.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>¿Puedo configurar para ejecutar todo esto en segundo plano en lugar de en ventanas de PoSh? ###
A partir de la versión 1,11 de Kubernetes, kubelet & Kube-proxy se pueden ejecutar como servicios nativos de [Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services). También puede usar administradores de servicios alternativos, como [NSSM. exe](https://nssm.cc/) , para ejecutar siempre estos procesos (flanneld, kubelet & Kube-proxy) en segundo plano. Consulte los [servicios de Windows en Kubernetes](./kube-windows-services.md) para obtener pasos de ejemplo.

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>Tengo problemas para ejecutar procesos de Kubernetes como servicios de Windows ###
Para solucionar problemas iniciales, puede usar las siguientes marcas en [NSSM. exe](https://nssm.cc/) para redirigir stdout y stderr a un archivo de salida:
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
Para obtener más información, consulte documentos oficiales de [uso de NSSM](https://nssm.cc/usage) .

## <a name="common-networking-errors"></a>Errores comunes de redes ##

### <a name="load-balancers-are-plumbed-inconsistently-across-the-cluster-nodes"></a>Los equilibradores de carga se conectan de forma incoherente entre los nodos del clúster ###
En Windows, Kube-proxy crea un equilibrador de carga SNP para cada servicio de Kubernetes en el clúster. En la configuración de Kube-proxy (predeterminada), los nodos de clústeres que contienen muchos equilibradores de carga (normalmente 100 +) pueden quedarse sin puertos TCP efímeros disponibles (también conocidos como intervalo de puertos dinámicos, que de forma predeterminada cubre los puertos de 49152 a 65535). Esto se debe al elevado número de puertos reservados en cada nodo para cada equilibrador de carga (no DSR). Este problema puede manifestarse a través de errores en Kube-proxy, como:
```
Policy creation failed: hcnCreateLoadBalancer failed in Win32: The specified port already exists.
```

Los usuarios pueden identificar este problema ejecutando el script [CollectLogs. PS1](https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/debug/collectlogs.ps1) y consultando los archivos `*portrange.txt`.

La `CollectLogs.ps1` también imitará la lógica de asignación de SNP para probar la disponibilidad de asignación del grupo de puertos en el intervalo de puertos TCP efímeros y notificará el éxito o el error en `reservedports.txt`. El script reserva 10 intervalos de 64 puertos efímeros TCP (para emular el comportamiento SNP), cuenta los éxitos de reserva & errores y, a continuación, libera los intervalos de puertos asignados. Un número de éxito inferior a 10 indica que el grupo efímero se está quedando sin espacio disponible. También se generará un resumen heurístico del número de reservas de puerto de bloque 64 aproximadamente en `reservedports.txt`.

Para resolver este problema, se pueden realizar algunos pasos:
1.  Para una solución permanente, el equilibrio de carga de Kube-proxy debe establecerse en [modo DSR](https://techcommunity.microsoft.com/t5/Networking-Blog/Direct-Server-Return-DSR-in-a-nutshell/ba-p/693710). El modo DSR está totalmente implementado y disponible solo en [Windows Server Insider build 18945](https://blogs.windows.com/windowsexperience/2019/07/30/announcing-windows-server-vnext-insider-preview-build-18945/#o1bs7T2DGPFpf7HM.97) (o posterior).
2. Como solución alternativa, los usuarios también pueden aumentar la configuración predeterminada de Windows de los puertos efímeros disponibles con un comando como `netsh int ipv4 set dynamicportrange TCP <start_port> <port_count>`. *ADVERTENCIA:* Reemplazar el intervalo de puertos dinámicos predeterminado puede tener consecuencias en otros procesos o servicios del host que dependen de los puertos TCP disponibles del intervalo no efímero, por lo que este intervalo debe seleccionarse con cuidado.
3. Existe una mejora en la escalabilidad de los equilibradores de carga en modo no DSR mediante el uso compartido inteligente de grupos de puertos, que está programado para su lanzamiento a través de una actualización acumulativa en el primer trimestre de 2020.

### <a name="hostport-publishing-is-not-working"></a>La publicación de denominaba hostport no funciona ###
Actualmente no es posible publicar puertos mediante el campo Kubernetes `containers.ports.hostPort` porque los complementos de Windows CNI no respetan este campo. Use la publicación de NodePort para el tiempo de publicación de puertos en el nodo.

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>Veo errores como "error de hnsCall en Win32: el disco equivocado está en la unidad". ###
Este error puede producirse al realizar modificaciones personalizadas en objetos SNP o al instalar nuevas Windows Update que introducen cambios en SNP sin eliminar los objetos SNP antiguos. Indica que un objeto SNP creado previamente antes de una actualización es incompatible con la versión de SNP actualmente instalada.

En Windows Server 2019 (y versiones anteriores), los usuarios pueden eliminar objetos SNP eliminando el archivo SNP. Data 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

Los usuarios deben poder eliminar directamente cualquier punto de conexión o red de SNP no compatible:
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Los usuarios de Windows Server, versión 1903 pueden ir a la siguiente ubicación del registro y eliminar las NIC que empiecen por el nombre de red (por ejemplo, `vxlan0` o `cbr0`):
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```

### <a name="containers-on-my-flannel-host-gw-deployment-on-azure-cannot-reach-the-internet"></a>Contenedores en mi host de flannel: la implementación de GW en Azure no puede tener acceso a Internet ###
Al implementar flannel en el modo host-GW en Azure, los paquetes tienen que pasar por el vSwitch de host físico de Azure. Los usuarios deben programar [rutas definidas por el usuario](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined) de tipo "aplicación virtual" para cada subred asignada a un nodo. Esto puede realizarse a través del Azure Portal (vea un ejemplo [aquí](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)) o a través de `az` CLI de Azure. Este es un ejemplo UDR con el nombre "ruta" mediante los comandos AZ para un nodo con IP 10.0.0.4 y la subred Pod correspondiente 10.244.0.0/24:
```
az network route-table create --resource-group <my_resource_group> --name BridgeRoute 
az network route-table route create  --resource-group <my_resource_group> --address-prefix 10.244.0.0/24 --route-table-name BridgeRoute  --name MyRoute --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4 
```

>[!TIP]
> Si va a implementar Kubernetes en máquinas virtuales de Azure o IaaS desde otros proveedores de nube, también puede usar [redes de superposición](./network-topologies.md#flannel-in-vxlan-mode) en su lugar.

### <a name="my-windows-pods-cannot-ping-external-resources"></a>Mis pods de Windows no pueden hacer ping a recursos externos ###
Los pods de Windows no tienen reglas de salida programadas para el protocolo ICMP hoy en día. Sin embargo, se admite TCP/UDP. Cuando intente demostrar la conectividad a recursos fuera del clúster, sustituya `ping <IP>` por los comandos de `curl <IP>` correspondientes.

Si sigue teniendo problemas, lo más probable es que la configuración de red en [CNI. conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) merece una atención adicional. Siempre puede editar este archivo estático, la configuración se aplicará a los recursos de Kubernetes recién creados.

¿Por qué?
Uno de los requisitos de red de Kubernetes (consulte el [modelo Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) es para que la comunicación del clúster se produzca sin NAT internamente. Para cumplir este requisito, disponemos [de una excepción](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) para toda la comunicación donde no se desea que se produzca NAT de salida. Sin embargo, esto también significa que debe excluir la dirección IP externa que está intentando consultar desde la siguiente. Solo entonces el tráfico que se origina en los pods de Windows se SNAT'ed correctamente para recibir una respuesta del mundo exterior. En este sentido, la siguiente `cni.conf` debe tener el aspecto siguiente:
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>Mi nodo Windows no puede tener acceso a un servicio NodePort ###
Se producirá un error en el acceso local de NodePort desde el propio nodo. Se trata de una limitación conocida. El acceso a NodePort funcionará desde otros nodos o clientes externos.

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>Transcurrido un tiempo, los puntos de conexión de VNIC y SNP de los contenedores se están eliminando ###
Este problema puede deberse a que el parámetro `hostname-override` no se pasa a [Kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/). Para resolverlo, los usuarios deben pasar el nombre de host a Kube-proxy de la siguiente manera:
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>En el modo flannel (vxlan), mis pods tienen problemas de conectividad después de volver a unirse al nodo ###
Cada vez que se vuelve a unir un nodo previamente eliminado al clúster, flannelD intentará asignar una nueva subred Pod al nodo. Los usuarios deben quitar los archivos de configuración de la subred Pod antigua en las siguientes rutas de acceso:
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Después de iniciar Start. ps1, Flanneld está atascado en "esperando a que se cree la red" ###
Hay numerosos informes de este problema que se están investigando; lo más probable es que se produzca un problema de sincronización cuando se establece la dirección IP de administración de la red flannel. Una solución alternativa consiste en volver a iniciar Start. PS1 o reiniciarlo manualmente de la siguiente manera:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

También [hay una solicitud de incorporación de problemas](https://github.com/coreos/flannel/pull/1042) que soluciona este problema en el momento de la revisión.


### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Mis pods de Windows no se pueden iniciar porque falta/Run/flannel/subnet.env ###
Esto indica que flannel no se inició correctamente. Puede intentar reiniciar flanneld. exe o copiar los archivos manualmente desde `/run/flannel/subnet.env` en el maestro de Kubernetes para `C:\run\flannel\subnet.env` en el nodo de trabajo de Windows y modificar la fila de `FLANNEL_SUBNET` a la subred que se asignó. Por ejemplo, si se asignó la subred de nodo 10.244.4.1/24:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
Es más seguro permitir que flanneld. exe genere este archivo automáticamente.


### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>La conectividad Pod-to-Pod entre hosts está interrumpida en el clúster de Kubernetes que se ejecuta en vSphere 
Puesto que vSphere y flannel reservan el puerto 4789 (puerto de VXLAN predeterminado) para redes de superposición, los paquetes pueden acabar de interceptarse. Si vSphere se usa para la red de superposición, debe configurarse para usar un puerto diferente con el fin de liberar 4789.  


### <a name="my-endpointsips-are-leaking"></a>Mis extremos/direcciones IP están perdiendo ###
Existen dos problemas conocidos actualmente que pueden provocar la pérdida de los extremos. 
1.  El primer [problema conocido](https://github.com/kubernetes/kubernetes/issues/68511) es un problema en la versión 1,11 de Kubernetes. Evite el uso de la versión de Kubernetes 1.11.0-1.11.2.
2. El segundo [problema conocido](https://github.com/docker/libnetwork/issues/1950) que puede provocar la pérdida de los puntos de conexión es un problema de simultaneidad en el almacenamiento de puntos de conexión. Para recibir la corrección, debe usar Docker EE 18,09 o superior.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>Mis pods no se pueden iniciar debido a errores de "red: no se pudo asignar para el intervalo" ###
Esto indica que se ha usado el espacio de direcciones IP en el nodo. Para limpiar los [puntos de conexión perdidos](#my-endpointsips-are-leaking), migre los recursos de los nodos afectados & ejecute los siguientes comandos:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>El nodo de Windows no puede acceder a mis servicios mediante la dirección IP de servicio ###
Se trata de una limitación conocida de la actual pila de redes de Windows. Sin embargo, los *pods* de **Windows pueden tener** acceso a la dirección IP del servicio.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>No se encuentra ningún adaptador de red al iniciar Kubelet ###
La pila de redes de Windows necesita un adaptador virtual para que funcionen las redes de Kubernetes. Si los siguientes comandos no devuelven ningún resultado (en un shell de administrador), se ha producido un error en la creación de la red virtual, un requisito previo necesario para que funcione Kubelet:

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

A menudo, merece la pena modificar el parámetro [interfacename](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) del script Start. ps1, en los casos en los que el adaptador de red del host no sea "Ethernet". En caso contrario, consulte la salida del script `start-kubelet.ps1` para ver si hay errores durante la creación de la red virtual. 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Los pods dejan de resolver correctamente consultas DNS después de un tiempo en vivo ###
Hay un problema conocido de almacenamiento en caché DNS en la pila de red de Windows Server, versión 1803 y versiones anteriores, que a veces pueden provocar errores en las solicitudes DNS. Para solucionar este problema, puede establecer los valores de caché Max TTL en cero con las siguientes claves del registro:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>Todavía veo problemas. ¿Qué debería hacer? ### 
Pueden existir restricciones adicionales en la red o hosts que evitan determinados tipos de comunicación entre los nodos. Asegúrate de lo siguiente:
  - ha configurado correctamente la topología de [red](./network-topologies.md) elegida
  - Se permite el tráfico que parece que procede de los pods
  - Se permite el tráfico HTTP, en el caso de implementar servicios web
  - No se quitan los paquetes de diferentes protocolos (por ej. ICMP y TCP/UDP)

>[!TIP]
> Para otros recursos de autoayuda, también hay una guía de solución de problemas de Kubernetes para Windows [disponible aquí](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648).

## <a name="common-windows-errors"></a>Errores comunes de Windows ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Mis pods de Kubernetes están bloqueados en "ContainerCreating" ###
Este problema puede tener varias causas, pero una de las más comunes es que la imagen de pausa se ha configurado de forma incorrecta. Se trata de un síntoma de alto nivel del siguiente problema.


### <a name="when-deploying-docker-containers-keep-restarting"></a>Al realizar la implementación, los contenedores de Docker siguen reiniciándose ###
Comprueba que la imagen de pausa sea compatible con la versión del sistema operativo. Las [instrucciones](./deploying-resources.md) suponen que el sistema operativo y los contenedores son de la versión 1803. Si tienes una versión posterior de Windows, como por ejemplo, una compilación de Insider, tendrás que ajustar las imágenes según corresponda. Consulta el [Repositorio de Docker](https://hub.docker.com/u/microsoft/) de Microsoft para obtener información sobre las imágenes. De todos modos, el Dockerfile de imagen de pausa y el servicio de muestra esperarán que la imagen se etiquete como `:latest`.


## <a name="common-kubernetes-master-errors"></a>Errores comunes del maestro de Kubernetes ##
La depuración del maestro de Kubernetes recae en tres categorías principales (en función de la probabilidad):

  - Algo no funciona con los contenedores del sistema Kubernetes.
  - Algo no funciona con la forma en que `kubelet` se ejecuta.
  - Algo no funciona con el sistema.

Ejecuta `kubectl get pods -n kube-system` para ver los pods que Kubernetes crea; esto puede ofrecer información sobre cuáles se bloquean o se inician correctamente. Después ejecuta `docker ps -a` para ver todos los contenedores sin procesar que realizan copias de seguridad de estos pods. Por último, ejecuta `docker logs [ID]` en los contenedores que son sospechosos de provocar el problema para ver el resultado sin procesar de los procesos.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>No se puede conectar con el servidor de API en `https://[address]:[port]`. ###
A menudo, este error indica problemas de certificado. Asegúrate de que se ha generado correctamente el archivo de configuración, de que las direcciones IP en él coinciden con las del host y de que lo has copiado en el directorio que el servidor de la API ha montado.

Si sigue las [instrucciones](./creating-a-linux-master.md), los buenos lugares para encontrar esto son:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 en caso contrario, consulte el archivo de manifiesto del servidor de la API para comprobar los puntos de montaje.
