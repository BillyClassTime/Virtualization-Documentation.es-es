---
title: "Solución de problemas de Kubernetes"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: troubleshooting
ms.prod: containers
description: Soluciones para problemas comunes al implementar Kubernetes y unirse a nodos de Windows.
keywords: kubernetes, 1.9, linux, compilar
ms.openlocfilehash: 73b44ffd12fba58ac4ef38352c012061a6817945
ms.sourcegitcommit: ad5f6344230c7c4977adf3769fb7b01a5eca7bb9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/05/2017
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

Además, algunos scripts deben ejecutarse con privilegios de administrador (como `kubelet`) y deben llevar el prefijo `sudo`.


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


### <a name="my-windows-pods-cannot-access-the-linux-master-or-vice-versa"></a>Mis pods de Windows no tienen acceso al maestro de Linux o viceversa. ###
Si estás usando una máquina virtual de Hyper-V, asegúrate de que la suplantación de identidad de MAC está habilitada en los adaptadores de red.


### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>El nodo de Windows no puede acceder a mis servicios mediante la dirección IP de servicio. ###
Se trata de una limitación conocida de la actual pila de redes de Windows.
