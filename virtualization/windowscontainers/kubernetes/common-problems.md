---
title: Solución de problemas de Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Soluciones para problemas comunes al implementar Kubernetes y unirse a nodos de Windows.
keywords: kubernetes, 1.12, linux, compilar
ms.openlocfilehash: a5e9369b000aa83aa7ec6ec9bb147f0fd844c820
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178918"
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
Deberías ver kubelet, kube proxy, y (si eliges Flannel como solución de red) que se ejecutan en el nodo con la ejecución de registros que se muestran en los procesos de agente de host de flanneld separan PoSh windows. Además, el nodo de Windows debe aparecer como "Listo" en el clúster de Kubernetes.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>¿Se puede configurar para ejecutar todo esto en segundo plano en lugar de windows PoSh? ###
A partir de la versión 1.11 de Kubernetes, kubelet & kube proxy se pueden ejecutar como nativo de [Servicios de Windows](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services). También siempre puede utilizar los administradores de servicio alternativo como [nssm.exe](https://nssm.cc/) siempre ejecutar estos procesos (flanneld, kubelet y kube proxy) en segundo plano para TI.


## <a name="common-networking-errors"></a>Errores comunes de redes ##

### <a name="my-windows-pods-do-not-have-network-connectivity"></a>Mis pods de Windows no tienen conectividad de red ###
Si estás usando las máquinas virtuales, asegúrate de que la suplantación de identidad de MAC está habilitada en los adaptadores de red de máquina virtual. Consulta [el anti-spoofing protección](./getting-started-kubernetes-windows.md#disable-anti-spoofing-protection) para obtener más detalles.


### <a name="my-windows-pods-cannot-ping-external-resources"></a>Mis pods de Windows no pueden hacer ping a recursos externos ###
Pods de Windows no tienen reglas de salida programadas para el protocolo ICMP hoy en día. Sin embargo, se admite TCP/UDP. Al intentar demostrar la conectividad a los recursos fuera del clúster, sustituir `ping <IP>` con correspondiente `curl <IP>` comandos.

Si todavía se enfrentan problemas, más probable es que la configuración de red en [cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) merece una atención extra. Siempre puedes editar este archivo estático, la configuración se aplicará a todos los recursos de Kubernetes recién creados.

¿Por qué?
Uno de los requisitos de red de Kubernetes (consulta el [modelo de Kubernetes](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) es para que la comunicación de clúster que se produzca sin NAT internamente. Para cumplir este requisito, tenemos un [ExceptionList](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) para toda la comunicación donde que no queremos que saliente NAT que se produzca. Sin embargo, esto también significa que debes excluir la dirección IP externa que estás intentando consultar desde el ExceptionList. Solo el tráfico procedente de los pods de Windows estará SNAT'ed correctamente para recibir una respuesta desde el exterior. En esta vista, tu ExceptionList en `cni.conf` debe tener el siguiente aspecto:
```
                "ExceptionList": [
                    "10.244.0.0/16",  # Cluster subnet
                    "10.96.0.0/12",   # Service subnet
                    "10.127.130.0/24" # Management (host) subnet
                ]
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Después de iniciar start.ps1, Flanneld está atascada en "A la espera de la red to be created" ###
Hay varios informes de este problema, que se está investigando; probablemente es un problema de temporización para cuando se establece la dirección IP de administración de la red flannel. Una solución alternativa es simplemente volver a iniciar start.ps1 o iniciarse de forma manual como sigue:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

También hay un [PR](https://github.com/coreos/flannel/pull/1042) que resuelve este problema revisados actualmente.

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Mis pods de Windows no se pueden iniciar debido a falta /run/flannel/subnet.env ###
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
Esto indica que se usa el espacio de direcciones IP en el nodo de. Para limpiar las [pérdidas de los puntos de conexión](#my-endpointsips-are-leaking), migrar todos los recursos en nodos afectados y ejecute los siguientes comandos:
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

Consulta el resultado del script `start-kubelet.ps1` para comprobar si hay errores durante la creación de la red virtual.

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Los pods dejan de resolver correctamente consultas DNS después de un tiempo en vivo ###
Hay un DNS conocido almacenamiento en caché de problema en la pila de redes de Windows Server, versión 1803 y, a continuación a veces, puede causar las solicitudes DNS para un error. Para evitar este problema, puedes establecer los valores de caché máximo de TTL a cero mediante las siguientes claves del registro:

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

Si sigue [nuestras instrucciones](./creating-a-linux-master.md), coloca buena para buscar que esto es:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 de lo contrario, hacer referencia al archivo de manifiesto del servidor de API para comprobar los puntos de montaje.