---
title: "Solución de problemas de Kubernetes"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: troubleshooting
ms.prod: containers
description: Soluciones para problemas comunes al implementar Kubernetes y unirse a nodos de Windows.
keywords: kubernetes, 1.9, linux, compilar
ms.openlocfilehash: 4fb7ac312b08c63564beb0f40889ff6a050c7166
ms.sourcegitcommit: b0e21468f880a902df63ea6bc589dfcff1530d6e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/17/2018
---
# <a name="troubleshooting-kubernetes"></a>Solución de problemas de Kubernetes #
Esta página te guía a través de varios problemas comunes con las implementaciones, redes y configuración de Kubernetes.

> [!tip]
> Recomienda un elemento de Preguntas frecuentes enviando una PR a [nuestro repositorio de documentación](https://github.com/MicrosoftDocs/Virtualization-Documentation/).


## <a name="common-deployment-errors"></a>Errores comunes de implementación ##
La depuración del maestro de Kubernetes recae en tres categorías principales (en función de la probabilidad):

  - Algo no funciona con los contenedores del sistema Kubernetes.
  - Algo no funciona con la forma en que `kubelet` se ejecuta.
  - Algo no funciona con el sistema.


Ejecuta `kubectl get pods -n kube-system` para ver los pods que Kubernetes crea; esto puede ofrecer información sobre cuáles se bloquean o se inician correctamente. Después ejecuta `docker ps -a` para ver todos los contenedores sin procesar que realizan copias de seguridad de estos pods. Por último, ejecuta `docker logs [ID]` en los contenedores que son sospechosos de provocar el problema para ver el resultado sin procesar de los procesos.


### <a name="permission-denied-errors"></a>Errores de _"Permission Denied"_ ###
Asegúrate de que los scripts tienen permisos ejecutables:

```bash
chmod +x [script name]
```

Además, algunos scripts deben ejecutarse con privilegios de superusuario (como `kubelet`) y deben llevar el prefijo `sudo`.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>No se puede conectar con el servidor de API en `https://[address]:[port]`. ###
A menudo, este error indica problemas de certificado. Asegúrate de que se ha generado correctamente el archivo de configuración, de que las direcciones IP en él coinciden con las del host y de que lo has copiado en el directorio que el servidor de la API ha montado.

Si sigues [nuestras instrucciones](./creating-a-linux-master), este paso se indicará en `~/kube/kubelet/`; de lo contrario, consulta el archivo de manifiesto del servidor de la API para comprobar los puntos de montaje.


## <a name="common-networking-errors"></a>Errores comunes de redes ##
Pueden existir restricciones adicionales en la red o hosts que evitan determinados tipos de comunicación entre los nodos. Asegúrate de lo siguiente:

  - Se permite el tráfico que parece que procede de los pods
  - Se permite el tráfico HTTP, en el caso de implementar servicios web
  - No se colocan paquetes ICMP


<!-- ### My Linux node cannot ping my Windows pods ### -->

## <a name="common-windows-errors"></a>Errores comunes de Windows ##

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Los pods dejan de resolver correctamente consultas DNS después de un tiempo en vivo ###
Este es un problema conocido de la pila de redes que afecta a algunas configuraciones; se está realizando un seguimiento rápido de este problema a través del servicio de mantenimiento de Windows.


### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Mis pods de Kubernetes están bloqueados en "ContainerCreating" ###
Este problema puede tener varias causas, pero una de las más comunes es que la imagen de pausa se ha configurado de forma incorrecta. Se trata de un síntoma de alto nivel del siguiente problema.


### <a name="when-deploying-docker-containers-keep-restarting"></a>Al realizar la implementación, los contenedores de Docker siguen reiniciándose ###
Comprueba que la imagen de pausa sea compatible con la versión del sistema operativo. Las [instrucciones](./getting-started-kubernetes-windows.md) dan por hecho que la versión tanto del sistema operativo como de los contenedores es la versión 1709. Si tienes una versión posterior de Windows, como por ejemplo, una compilación de Insider, tendrás que ajustar las imágenes según corresponda. Consulta el [Repositorio de Docker](https://hub.docker.com/u/microsoft/) de Microsoft para obtener información sobre las imágenes. De todos modos, el Dockerfile de imagen de pausa y el servicio de muestra esperarán que la imagen se etiquete como `microsoft/windowsservercore:latest`.


### <a name="my-windows-pods-cannot-access-the-linux-master-or-vice-versa"></a>Mis pods de Windows no tienen acceso al maestro de Linux o viceversa ###
Si estás usando una máquina virtual de Hyper-V, asegúrate de que la suplantación de identidad de MAC está habilitada en los adaptadores de red.


### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>El nodo de Windows no puede acceder a mis servicios mediante la dirección IP de servicio ###
Se trata de una limitación conocida de la actual pila de redes de Windows. Solo los pods pueden hacer referencia a la dirección IP de servicio.


### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>No se encuentra ningún adaptador de red al iniciar Kubelet ###
La pila de redes de Windows necesita un adaptador virtual para que funcionen las redes de Kubernetes. Si los siguientes comandos no devuelven ningún resultado (en un shell de administrador), se ha producido un error en la creación de la red virtual, un requisito previo necesario para que funcione Kubelet:

```powershell
Get-HnsNetwork | ? Name -Like "l2bridge"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

Consulta el resultado del script `start-kubelet.ps1` para comprobar si hay errores durante la creación de la red virtual.

