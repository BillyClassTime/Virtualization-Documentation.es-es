---
title: Administración remota de un host de Windows Docker
description: Cómo administrar de forma segura un host de Docker remoto que ejecute Windows Server.
keywords: docker, contenedores
author: taylorb-microsoft
ms.date: 02/14/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 0cc1b621-1a92-4512-8716-956d7a8fe495
ms.openlocfilehash: b975c593bd5c736ec3e7e1e21b76b2f6a2c8f8a4
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9575366"
---
# <a name="remote-management-of-a-windows-docker-host"></a>Administración remota de un host de Windows Docker

Incluso aunque falte `docker-machine`, es posible crear un host de Docker accesible remotamente en una VM de Windows Server 2016.

Los pasos son muy simples:

* Crea los certificados en el servidor mediante [dockertls](https://hub.docker.com/r/stefanscherer/dockertls-windows/). Si vas a crear los certificados con una dirección IP, quizá quieras usar una dirección IP estática para evitar tener que volver a crear los certificados cuando cambie la dirección IP.

* Reinicia el servicio de Docker `Restart-Service Docker`
* Haz que los puertos TLS de Docker 2375 y 2376 estén disponibles mediante la creación de una regla de NSG que permita el tráfico entrante. Ten en cuenta que, en el caso de las conexiones seguras, solo debes permitir el 2376.  
  El portal debería mostrar una configuración de NSG como esta:  
  ![NGS](media/nsg.png)  
  
* Permite las conexiones entrantes a través del Firewall de Windows. 
```
New-NetFirewallRule -DisplayName 'Docker SSL Inbound' -Profile @('Domain', 'Public', 'Private') -Direction Inbound -Action Allow -Protocol TCP -LocalPort 2376
```
* Copia los archivos `ca.pem`, 'cert.pem' y 'key.pem' desde carpeta de Docker del usuario del equipo; por ejemplo, `c:\users\chris\.docker` en el equipo local. Por ejemplo, puedes usar ctrl-c y ctrl-v para copiar y pegar los archivos desde una sesión de RDP. 
* Confirma que puedes conectarte al host de Docker remoto. Ejecución
```
docker -D -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify --tlscacert=c:\
users\foo\.docker\client\ca.pem --tlscert=c:\users\foo\.docker\client\cert.pem --tlskey=c:\users\foo\.doc
ker\client\key.pem ps
```


## <a name="troubleshooting"></a>Solución de problemas
### <a name="try-connecting-without-tls-to-determine-your-nsg-firewall-settings-are-correct"></a>Intenta conectarte sin TLS para determinar si la configuración del firewall de NSG es correcta
Los errores de conectividad normalmente se manifiestan en errores como los siguientes:
```
error during connect: Get https://wsdockerhost.southcentralus.cloudapp.azure.com:2376/v1.25/version: dial tcp 13.85.27.177:2376: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.
```

Para permitir las conexiones sin cifrar, agrega 
```
{
    "tlsverify":  false,
}
```
a `c"\programdata\docker\config\daemon.json` y luego reinicia el servicio.

Conéctate al host remoto con una línea de comandos similar a esta:
```
docker -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify=0 version
```

### <a name="cert-problems"></a>Problemas de certificado
Si se accede al host de Docker con un certificado que no se haya creado para la dirección IP o el nombre DNS, se producirá un error:
```
error during connect: Get https://w.x.y.c.z:2376/v1.25/containers/json: x509: certificate is valid for 127.0.0.1, a.b.c.d, not w.x.y.z
```
Asegúrate de que w.x.y.z es el nombre de DNS para la dirección IP pública del host y que el nombre de DNS coincide con el [nombre común](https://www.ssl.com/faqs/common-name/) del certificado, que era la variable de entorno `SERVER_NAME` o una de las direcciones IP de la variable `IP_ADDRESSES` suministrada a dockertls

### <a name="cryptox509-warning"></a>Advertencia crypto/x509
Puede que recibas una advertencia. 
```
level=warning msg="Unable to use system certificate pool: crypto/x509: system root pool is not available on Windows"
```
Esta advertencia no es peligrosa.
