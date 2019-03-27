---
title: Solución de problemas de Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Soluciones para problemas comunes al implementar Kubernetes y unirse a nodos de Windows.
keywords: kubernetes, 1.12, linux, compilar
ms.openlocfilehash: 1c5a5ec90b828a4f2430508f02cb9b9afb1c4d53
ms.sourcegitcommit: 1715411ac2768159cd9c9f14484a1cad5e7f2a5f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 03/26/2019
ms.locfileid: "9263512"
---
# <a name="troubleshooting-kubernetes"></a>Solución de problemas de Kubernetes #
Esta página te guía a través de varios problemas comunes con las implementaciones, redes y configuración de Kubernetes.

> [!tip]
> Recomienda un elemento de Preguntas frecuentes enviando una PR a [nuestro repositorio de documentación](https://github.com/MicrosoftDocs/Virtualization-Documentation/).

Esta página se subdivide en las siguientes categorías:
1. [Preguntas generales](#general-questions)
2. [Errores comunes de redes](#common-networking-errors)
3. [Errores comunes de Windows](#common-windows-errors)
4. [Errores comunes de maestro de Kubernetes](#common-kubernetes-master-errors)

## <a name="general-questions"></a>Preguntas generales ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>¿Cómo sé start.ps1 en Windows se ha completado correctamente? ###
Deberías ver kubelet, kube proxy, y (si se ha elegido Flannel como solución de red) que se ejecutan en el nodo con la ejecución de registros que se muestran en los procesos de agente de host de flanneld separan PoSh windows. Además, el nodo de Windows debe aparecer como "Listo" en el clúster de Kubernetes.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>¿Se puede configurar para ejecutar todo esto en segundo plano en lugar de windows PoSh? ###
A partir de la versión 1.11 de Kubernetes, se puede ejecutar kubelet & kube-proxy como nativo de [Servicios de Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services). También siempre puede utilizar los administradores de servicio alternativo como [nssm.exe](https://nssm.cc/) siempre ejecutar estos procesos (flanneld, kubelet & kube-proxy) en segundo plano para TI. Consulta por ejemplo, los [Servicios de Windows en Kubernetes](./kube-windows-services.md) pasos.

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>Tengo problemas al ejecutar procesos de Kubernetes como servicios de Windows ###
Para solucionar problemas de inicial, puedes usar los siguientes indicadores en [nssm.exe](https://nssm.cc/) para redirigir stdout y stderr a un archivo de salida:
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
Para obtener más información, consulta a oficial [nssm uso](https://nssm.cc/usage) documentos.

## <a name="common-networking-errors"></a>Errores comunes de redes ##

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>Veo errores como "no se pudo hnsCall en Win32: el disco incorrecto está en la unidad." ###
Este error puede producirse al realizar modificaciones personalizadas en objetos de SNP o realizar una instalación nueva actualización de Windows que presentan cambios SNP sin destruir objetos SNP antiguos. Indica que un objeto de SNP que se ha creado previamente antes de una actualización no es compatible con la versión de SNP instalada actualmente.

En Windows Server 2019 (y a continuación), los usuarios pueden eliminar objetos de SNP eliminando el archivo HNS.data 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

Los usuarios deben poder eliminar directamente los puntos de conexión de SNP incompatibles o redes:
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Los usuarios de Windows Server, versión 1903 puede ir a la siguiente ubicación del registro y eliminar cualquier NIC comenzando por el nombre de red (por ejemplo, `vxlan0` o `cbr0`):
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```


### <a name="my-windows-pods-cannot-ping-external-resources"></a>Mis pods de Windows no pueden hacer ping a recursos externos ###
Pods de Windows no tienen reglas de salida programadas para el protocolo ICMP hoy en día. Sin embargo, se admite TCP/UDP. Al intentar demostrar la conectividad a los recursos fuera del clúster, sustituir `ping <IP>` con correspondiente `curl <IP>` comandos.

Si todavía se enfrentan problemas, es muy probable que la configuración de red en [cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) merece una atención extra. Siempre puedes editar este archivo estático, la configuración se aplicará a todos los recursos de Kubernetes recién creados.

¿Por qué?
Uno de los requisitos de red de Kubernetes (consulta el [modelo de Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) es para que la comunicación de clúster que se produzca sin NAT internamente. Para cumplir este requisito, tenemos un [ExceptionList](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) para toda la comunicación donde que no queremos que NAT saliente a producirse. Sin embargo, esto también significa que debes excluir la dirección IP externa que estás intentando consultar desde el ExceptionList. Solo el tráfico procedente de los pods de Windows estará SNAT'ed correctamente para recibir una respuesta desde el exterior. En esta vista, tu ExceptionList en `cni.conf` debe tener el siguiente aspecto:
```
                "ExceptionList": [
                    "10.244.0.0/16",  # Cluster subnet
                    "10.96.0.0/12",   # Service subnet
                    "10.127.130.0/24" # Management (host) subnet
                ]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>El nodo de Windows no puede acceder a un servicio de NodePort ###
Se producirá un error de acceso local de NodePort desde el nodo en Sí. Se trata de una limitación conocida. Acceso NodePort funcionará desde otros nodos o los clientes externos.

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>Después de algún tiempo, se eliminan las vNICs y puntos de conexión de SNP de contenedores ###
Este problema puede deberse al `hostname-override` parámetro no se pasa a [kube proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/). Para resolverlo, los usuarios deben pasar el nombre de host a kube proxy como sigue:
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>En el modo de Flannel (vxlan), Mis pods están teniendo problemas de conectividad después de volver a unir el nodo ###
Siempre que un nodo eliminado anteriormente es que se va a unirse al clúster, flannelD intentará asignar una nueva subred de pod al nodo. Los usuarios deben quitar los archivos de configuración de subred de pod antiguo en las siguientes rutas:
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Después de iniciar start.ps1, Flanneld está atascada en "A la espera de la red to be created" ###
Hay varios informes de este problema que se está investigando; probablemente es un problema de temporización para cuando se establece la dirección IP de administración de la red flannel. Una solución alternativa es simplemente volver a iniciar start.ps1 o iniciarse manualmente como sigue:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

También hay un [PR](https://github.com/coreos/flannel/pull/1042) que resuelve este problema revisados actualmente.


### <a name="on-flannel-host-gw-my-windows-pods-do-not-have-network-connectivity"></a>En Flannel (host gw), Mis pods de Windows no tienen conectividad de red ###
Si desea utilizar l2bridge para redes (también conocido como [flannel host-gateway](./network-topologies.md#flannel-in-host-gateway-mode)), debes asegurarte de suplantación de direcciones MAC está habilitada para el host de contenedor de Windows (invitados) de las máquinas virtuales. Para lograr esto, debes ejecutar lo siguiente como administrador en el equipo que hospeda las máquinas virtuales (ejemplo de Hyper-V):

```powershell
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> [!TIP]
> Si estás usando un producto basado en VMware para satisfacer sus necesidades de virtualización, busque en habilitar el [modo promiscuo](https://kb.vmware.com/s/article/1004099) para el requisito de suplantación de identidad de MAC.

>[!TIP]
> Si vas a implementar Kubernetes en Azure o máquinas virtuales de IaaS de otros proveedores de nube tú mismo, también puedes usar [las redes de superposición](./network-topologies.md#flannel-in-vxlan-mode) en su lugar.

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Mis pods de Windows no se pueden iniciar porque falta /run/flannel/subnet.env ###
Esto indica que Flannel no se inicia correctamente. Puedes intentar reiniciar flanneld.exe o puedes copiar manualmente los archivos a través de `/run/flannel/subnet.env` en el maestro de Kubernetes para `C:\run\flannel\subnet.env` en el nodo de trabajo de Windows y modificar el `FLANNEL_SUBNET` fila a un número diferente. Por ejemplo, si se desea nodo subred 10.244.4.1/24:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```

### <a name="my-endpointsips-are-leaking"></a>Se pierdan mi IPs de puntos de conexión ###
Existen 2 problemas detectados que pueden provocar la pérdida de los puntos de conexión. 
1.  El primer [problema conocido](https://github.com/kubernetes/kubernetes/issues/68511) es un problema en la versión 1.11 de Kubernetes. Por favor, evita usar Kubernetes versión 1.11.0 - 1.11.2.
2. El segundo [problema conocido](https://github.com/docker/libnetwork/issues/1950) que puede provocar la pérdida de los puntos de conexión es un problema de simultaneidad en el almacenamiento de los puntos de conexión. Para recibir la corrección, debes usar Docker EE 18.09 o superior.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>No se pueden iniciar Mis pods debido a "red: no se pudo asignar de intervalo" errores ###
Esto indica que se usa el espacio de direcciones IP en el nodo de. Para limpiar las [pérdidas de los puntos de conexión](#my-endpointsips-are-leaking), por favor, migrar todos los recursos en & nodos afectados ejecuta los siguientes comandos:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>El nodo de Windows no puede acceder a mis servicios mediante la dirección IP de servicio ###
Se trata de una limitación conocida de la actual pila de redes de Windows. Windows *pods* **son** capaces de obtener acceso a la dirección IP de servicio sin embargo.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>No se encuentra ningún adaptador de red al iniciar Kubelet ###
La pila de redes de Windows necesita un adaptador virtual para que funcionen las redes de Kubernetes. Si los siguientes comandos no devuelven ningún resultado (en un shell de administrador), se ha producido un error en la creación de la red virtual, un requisito previo necesario para que funcione Kubelet:

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

A menudo vale la pena modificar el parámetro de la [interfaz](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) de la secuencia de start.ps1, en los casos donde el adaptador de red del host no "Ethernet". De lo contrario, consulta el resultado de la `start-kubelet.ps1` script para ver si hay errores durante la creación de una red virtual. 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Los pods dejan de resolver correctamente consultas DNS después de un tiempo en vivo ###
Hay un problema en la pila de redes de Windows Server de caché de DNS conocidos, versión 1803 y, a continuación a veces puede provocar errores en las solicitudes DNS. Para evitar este problema, puedes establecer los valores de caché máximo de TTL a cero mediante las siguientes claves del registro:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>Aún veo problemas. ¿Qué debo hacer? ### 
Pueden existir restricciones adicionales en la red o hosts que evitan determinados tipos de comunicación entre los nodos. Asegúrate de lo siguiente:
  - has configurado correctamente la [topología de red](./network-topologies.md) de elegido
  - Se permite el tráfico que parece que procede de los pods
  - Se permite el tráfico HTTP, en el caso de implementar servicios web
  - No se se descartan los paquetes de protocolos diferentes (ie ICMP frente a TCP/UDP)


## <a name="common-windows-errors"></a>Errores comunes de Windows ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Mis pods de Kubernetes están bloqueados en "ContainerCreating" ###
Este problema puede tener varias causas, pero una de las más comunes es que la imagen de pausa se ha configurado de forma incorrecta. Se trata de un síntoma de alto nivel del siguiente problema.


### <a name="when-deploying-docker-containers-keep-restarting"></a>Al realizar la implementación, los contenedores de Docker siguen reiniciándose ###
Comprueba que la imagen de pausa sea compatible con la versión del sistema operativo. Las [instrucciones](./deploying-resources.md) se supone que el sistema operativo y los contenedores están la versión 1803. Si tienes una versión posterior de Windows, como por ejemplo, una compilación de Insider, tendrás que ajustar las imágenes según corresponda. Consulta el [Repositorio de Docker](https://hub.docker.com/u/microsoft/) de Microsoft para obtener información sobre las imágenes. De todos modos, el Dockerfile de imagen de pausa y el servicio de muestra esperarán que la imagen se etiquete como `:latest`.


## <a name="common-kubernetes-master-errors"></a>Errores comunes de maestro de Kubernetes ##
La depuración del maestro de Kubernetes recae en tres categorías principales (en función de la probabilidad):

  - Algo no funciona con los contenedores del sistema Kubernetes.
  - Algo no funciona con la forma en que `kubelet` se ejecuta.
  - Algo no funciona con el sistema.

Ejecuta `kubectl get pods -n kube-system` para ver los pods que Kubernetes crea; esto puede ofrecer información sobre cuáles se bloquean o se inician correctamente. Después ejecuta `docker ps -a` para ver todos los contenedores sin procesar que realizan copias de seguridad de estos pods. Por último, ejecuta `docker logs [ID]` en los contenedores que son sospechosos de provocar el problema para ver el resultado sin procesar de los procesos.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>No se puede conectar con el servidor de API en `https://[address]:[port]`. ###
A menudo, este error indica problemas de certificado. Asegúrate de que se ha generado correctamente el archivo de configuración, de que las direcciones IP en él coinciden con las del host y de que lo has copiado en el directorio que el servidor de la API ha montado.

Si sigues [nuestras instrucciones](./creating-a-linux-master.md), coloca adecuado para encontrar que esto es:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 de lo contrario, hacer referencia al archivo de manifiesto del servidor de API para comprobar los puntos de montaje.