---
title: Red de contenedores de Windows
description: Controladores de red y topologías para contenedores de Windows.
keywords: docker, contenedores
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 3fb36008d9304e6fbdd318c55e012ff3f096d58b
ms.sourcegitcommit: ec186664e76d413d3bf75f2056d5acb556f4205d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2018
ms.locfileid: "1876164"
---
# <a name="windows-container-network-drivers"></a>Controladores de red de contenedores Windows  

Además de aprovechar la red 'nat' predeterminada creada por Docker en Windows, los usuarios pueden definir redes de contenedores personalizadas. Pueden crearse redes definidas por los usuarios mediante la CLI de Docker, con el comando [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/). En Windows, están disponibles los siguientes tipos de controladores de red:

- **nat**: los contenedores conectados a una red que se haya creado con el controlador 'nat' se conectarán a un conmutador Hyper-V *interno* y recibirán una dirección IP desde el prefijo de IP (``--subnet``) especificado por el usuario. Se admite la asignación o el enrutamiento de puertos desde el host del contenedor a los puntos de conexión del contenedor.
  > Las redes NAT múltiples se admiten si se tiene instalada al actualización Windows 10 Creators Update.

- **transparent**: los contenedores conectados a una red que se haya creado con el controlador 'transparente' se conectarán directamente a la red física, a través de un conmutador Hyper-V *externo*. Las direcciones IP de la red física se pueden asignar de manera estática (requiere la opción ``--subnet`` especificada por el usuario) o dinámica mediante un servidor DHCP externo. 

- **overlay**: cuando el motor docker se ejecuta en [modo enjambre](../manage-containers/swarm-mode.md), los contenedores conectados a una red de superposición ('overlay') pueden comunicarse con otros contenedores conectados a la misma red en varios hosts de contenedor. Cada red superpuesta que se crea en un clúster de enjambre se crea con su propia subred IP, definida por un prefijo de IP privado. El controlador de red de superposición usa encapsulado VXLAN. **Puede usarse con Kubernetes al usar los planos de control de red apropiados (Flannel o OVN).**
  > Nota: asegúrate de que el entorno cumpla estos *necesarios* [requisitos previos](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) para crear redes de superposición.

  > Nota: requiere Windows Server 2016 con [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217), Windows 10 Creators Update o una versión posterior.

- **l2bridge**: los contenedores conectados a una red creada con el controlador 'l2bridge' estarán en la misma subred IP que el host de contenedor y estarán conectados a la red física a través de un conmutador Hyper-V *externo*. Las direcciones IP deben asignarse estáticamente desde el mismo prefijo que el host del contenedor. Todos los puntos de conexión de contenedor del host tendrán la misma dirección MAC que el host debido a la operación de traducción de direcciones de capa 2 (reescritura de MAC) en la entrada y la salida.
  > Nota: cuando este modo se usa en un escenario de virtualización (el host contenedor es una VM), _se requiere suplantación de direcciones MAC_.
  
  > Nota: requiere Windows Server 2016, Windows 10 Creators Update o una versión posterior.

- **l2tunnel**: similar a l2bridge; sin embargo _este controlador solo se debe usar en una pila de Microsoft Cloud_. Los paquetes que provienen de un contenedor se envían al host de virtualización, donde se aplica la directiva SDN.

> Para obtener información sobre cómo conectar puntos de conexión de contenedor a una red virtual de inquilino existente con la pila de redes definidas por software (SDN) de Microsoft, consulte el tema [Asociar contenedores a una red virtual](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network).

> Windows 10 Creators Update (o posterior) ya dispone de compatibilidad de plataformas para agregar un nuevo punto de conexión de contenedor a un contenedor en ejecución (es decir, 'agregado en caliente').


## <a name="network-topologies-and-ipam"></a>Topologías de red e IPAM
En la tabla siguiente se muestra cómo se proporciona conectividad de red para las conexiones internas (de contenedor a contenedor) y externas para cada controlador de red.

### <a name="networking-modes--docker-drivers"></a>Modos de redes/Controladores Docker

  | Controlador de red de Windows Docker | Usos típicos | Contenedor a contenedor (nodo único) | Contenedor a externo (nodo único + nodo múltiple) | Contenedor a contenedor (nodo múltiple) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (predeterminado)** | Adecuado para desarrolladores | <ul><li>Misma subred: conexión con puente a través del conmutador virtual de Hyper-V</li><li> Subred cruzada: no se admite en WS2016 (solo un prefijo interno NAT)</li></ul> | Enrutado a través de vNIC de administración (unido a WinNAT) | No se admite directamente: requiere exponer los puertos a través del host |
  | **Transparente** | Adecuado para desarrolladores o pequeñas implementaciones | <ul><li>Misma subred: conexión con puente a través del conmutador virtual de Hyper-V</li><li>Subred cruzada: enrutado a través de host de contenedor</li></ul> | Enrutado a través de host de contenedor con acceso directo al adaptador de red (físico) | Enrutado a través de host de contenedor con acceso directo al adaptador de red (físico) |
  | **Superposición** | Necesario para enjambre Docker, varios nodos | <ul><li>Misma subred: conexión con puente a través del conmutador virtual de Hyper-V</li><li>Subred cruzada: el tráfico de red se encapsula y enruta a través de administración vNIC</li></ul> | No admitido directamente: requiere un segundo punto de conexión unido a la red NAT | Misma subred/subred cruzada: el tráfico de red se encapsula usando VXLAN y se enruta a través de administración vNIC |
  | **L2Bridge** | Se usa para Kubernetes y Microsoft SDN | <ul><li>Misma subred: conexión con puente a través del conmutador virtual de Hyper-V</li><li> Subred cruzada: la dirección MAC del contenedor se reescribe al entrar y salir y se enruta.</li></ul> | La dirección MAC del contenedor se reescribe al entrar y salir. | <ul><li>Misma subred: conexión con puente</li><li>Subred cruzada: no admitida en WS2016.</li></ul> |
  | **L2Tunnel**| Solo Azure | Misma subred/subred cruzada: enlazado con el conmutador virtual Hyper-V del host al que se aplica la directiva | El tráfico debe sir a través de la puerta de enlace de red virtual de Azure | Misma subred/subred cruzada: enlazado con el conmutador virtual Hyper-V del host al que se aplica la directiva |

### <a name="ipam"></a>IPAM 
Las direcciones IP se asignan de forma diferente para cada controlador de red. Windows usa el servicio de redes de host (SNP) para proporcionar IPAM para el controlador nat y funciona con el modo enjambre de Docker (KVS interno) para proporcionar IPAM para la superposición. Todos los demás controladores de red usan IPAM externo.

| Modo/controlador de redes | IPAM |
| -------------------------|:----:|
| NAT | Asignación dinámica de IP y asignación por el servicio de redes de host de red (SNP) desde el prefijo de subred NAT interno |
| Transparente | Asignación de direcciones IP estática o dinámica (con servidor DHCP externo) en el prefijo de red del host de contenedor |
| Superposición | Asignación IP dinámica desde los prefijos gestionados por el modo de enjambre del motor de Docker y asignación mediante SNP |
| L2Bridge | Asignación de direcciones IP estática y asignación desde direcciones IP en el prefijo de red del host(también podría asignarse a través del complemento HNS) |
| L2Tunnel | Solo Azure: asignación de IP dinámica y asignación desde complemento |

### <a name="service-discovery"></a>Detección de servicios
La detección de servicios solo se admite para determinados controladores de red de Windows.

|  | Detección de servicios locales  | Detección de servicios globales |
| :---: | :---------------     |  :---                |
| nat | SÍ | N/A |  
| overlay | SÍ | SÍ, con Docker EE |
| transparente | NO | NO |
| l2bridge | NO | SÍ, con Kube DNS |
