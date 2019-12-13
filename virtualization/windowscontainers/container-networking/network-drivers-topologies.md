---
title: Redes de contenedores de Windows
description: Controladores de red y topologías para contenedores de Windows.
keywords: docker, contenedores
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 46eefb03f8f5a53333f5e7eca7074ab34e72a767
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910075"
---
# <a name="windows-container-network-drivers"></a>Controladores de red de contenedor de Windows  

Además de aprovechar la red 'nat' predeterminada creada por Docker en Windows, los usuarios pueden definir redes de contenedores personalizadas. Las redes definidas por el usuario se pueden crear mediante el comando Docker CLI [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) . En Windows, están disponibles los siguientes tipos de controladores de red:

- **nat**: los contenedores conectados a una red que se haya creado con el controlador 'nat' se conectarán a un conmutador Hyper-V *interno* y recibirán una dirección IP desde el prefijo de IP (``--subnet``) especificado por el usuario. Se admite la asignación o el reenvío de puertos desde el host del contenedor a los puntos de conexión del contenedor.
  
  >[!NOTE]
  > Las redes NAT creadas en Windows Server 2019 (o superior) ya no se conservan después del reinicio.

  > Se admiten varias redes NAT si tiene instalado Windows 10 Creators Update (o superior).
  
- **transparent**: los contenedores conectados a una red que se haya creado con el controlador 'transparente' se conectarán directamente a la red física, a través de un conmutador Hyper-V *externo*. Las direcciones IP de la red física se pueden asignar de manera estática (requiere la opción ``--subnet`` especificada por el usuario) o dinámica mediante un servidor DHCP externo.
  
  >[!NOTE]
  >Debido al siguiente requisito, la conexión de los hosts de contenedor a través de una red transparente no se admite en las máquinas virtuales de Azure.
  
  > Requiere: cuando este modo se usa en un escenario de virtualización (el host de contenedor es una máquina virtual), _se requiere la suplantación de direcciones MAC_.

- **overlay**: cuando el motor docker se ejecuta en [modo enjambre](../manage-containers/swarm-mode.md), los contenedores conectados a una red de superposición ('overlay') pueden comunicarse con otros contenedores conectados a la misma red en varios hosts de contenedor. Cada red superpuesta que se crea en un clúster enjambre se crea con su propia subred IP, definida por un prefijo de IP privado. El controlador de red overlay usa encapsulación VXLAN. **Se puede usar con Kubernetes cuando se usan los planos de control de red adecuados (por ejemplo, flannel).**
  > Requiere: Asegúrese de que el entorno cumple los [requisitos previos](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) necesarios para crear redes de superposición.

  > Requiere: en Windows Server 2019, requiere [KB4489899](https://support.microsoft.com/help/4489899).

  > Requiere: en Windows Server 2016, requiere [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217).

  >[!NOTE]
  >En Windows Server 2019, las redes superpuestas creadas por Docker Swarm aprovechan las reglas NAT de VFP para la conectividad saliente. Esto significa que un contenedor determinado recibe una dirección IP. También significa que las herramientas basadas en ICMP, como `ping` o `Test-NetConnection`, deben configurarse con sus opciones de TCP/UDP en situaciones de depuración.

- **l2bridge** : similar a `transparent` modo de red, los contenedores conectados a una red creada con el controlador "l2bridge" se conectarán a la red física a través de un conmutador de Hyper-V *externo* . La diferencia en l2bridge es que los extremos del contenedor tendrán la misma dirección MAC que el host debido a la operación de traducción de direcciones de nivel 2 (reescritura de MAC) en la entrada y salida. En escenarios de agrupación en clústeres, esto ayuda a mitigar el esfuerzo de los conmutadores que tienen que aprender direcciones MAC de contenedores de corta duración. Las redes L2bridge se pueden configurar de dos maneras diferentes:
  1. La red L2bridge está configurada con la misma subred IP que el host de contenedor
  2. La red L2bridge está configurada con una nueva subred IP personalizada
  
  En Configuration 2, los usuarios deberán agregar un punto de conexión en el compartimiento de red del host que actúe como puerta de enlace y configurar las capacidades de enrutamiento para el prefijo designado. 
  > Requiere: requiere Windows Server 2016, Windows 10 Creators Update o una versión posterior.

  > Requiere: [Directiva de OutboundNAT](./advanced.md#specify-outboundnat-policy-for-a-network) para la conectividad externa.

- **l2tunnel** : similar a l2bridge, sin embargo, _este controlador solo debe usarse en una pila de Microsoft Cloud (Azure)_ . Los paquetes que provienen de un contenedor se envían al host de virtualización, donde se aplica la directiva SDN.


## <a name="network-topologies-and-ipam"></a>Topologías de red y IPAM

En la tabla siguiente se muestra cómo se proporciona conectividad de red para las conexiones internas (de contenedor a contenedor) y externas para cada controlador de red.

### <a name="networking-modesdocker-drivers"></a>Modos de red/controladores de Docker

  | Controlador de red de Windows Docker | Usos típicos | Contenedor a contenedor (nodo único) | Contenedor a externo (nodo único + varios nodos) | De contenedor a contenedor (varios nodos) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (valor predeterminado)** | Adecuado para desarrolladores | <ul><li>Misma subred: conexión con puente a través del conmutador virtual de Hyper-V</li><li> Cross Subnet: no compatible (solo un prefijo interno de NAT)</li></ul> | Enrutado a través de vNIC de administración (unido a WinNAT) | No se admite directamente: requiere exponer los puertos a través del host |
  | **Partes** | Adecuado para desarrolladores o pequeñas implementaciones | <ul><li>Misma subred: conexión con puente a través del conmutador virtual de Hyper-V</li><li>Subred cruzada: enrutado a través de host de contenedor</li></ul> | Enrutado a través de host de contenedor con acceso directo al adaptador de red (físico) | Enrutado a través de host de contenedor con acceso directo al adaptador de red (físico) |
  | **Overlay** | Buena para varios nodos; necesario para Docker Swarm, disponible en Kubernetes | <ul><li>Misma subred: conexión con puente a través del conmutador virtual de Hyper-V</li><li>Subred cruzada: el tráfico de red se encapsula y enruta a través de administración vNIC</li></ul> | No compatible directamente: requiere el segundo punto de conexión de contenedor conectado a la red NAT en Windows Server 2016 o la regla de NAT de VFP en Windows Server 2019.  | Misma subred/subred cruzada: el tráfico de red se encapsula usando VXLAN y se enruta a través de administración vNIC |
  | **L2Bridge** | Se usa para Kubernetes y Microsoft SDN | <ul><li>Misma subred: conexión con puente a través del conmutador virtual de Hyper-V</li><li> Subred cruzada: la dirección MAC del contenedor se reescribe al entrar y salir y se enruta.</li></ul> | La dirección MAC del contenedor se reescribe al entrar y salir. | <ul><li>Misma subred: conexión con puente</li><li>Cross Subnet: enrutado a través de MGMT vNIC en WSv1809 y versiones posteriores</li></ul> |
  | **L2Tunnel**| Solo Azure | Misma subred/subred cruzada: enlazado con el conmutador virtual Hyper-V del host al que se aplica la directiva | El tráfico debe sir a través de la puerta de enlace de red virtual de Azure | Misma subred/subred cruzada: enlazado con el conmutador virtual Hyper-V del host al que se aplica la directiva |

### <a name="ipam"></a>IPAM

Las direcciones IP se asignan de forma diferente para cada controlador de red. Windows usa el servicio de red de host (HNS) para proporcionar IPAM para el controlador nat y funciona con el modo enjambre de Docker (KVS interno) para proporcionar IPAM para la superposición. Todos los demás controladores de red usan IPAM externo.

| Modo/controlador de redes | IPAM |
| -------------------------|:----:|
| NAT | Asignación y asignación de IP dinámicas mediante el servicio de red de host (SNP) desde el prefijo de subred NAT interna |
| Transparente | Asignación de direcciones IP estática o dinámica (con servidor DHCP externo) en el prefijo de red del host de contenedor |
| Overlay | Asignación IP dinámica desde los prefijos gestionados por el modo de enjambre del motor de Docker y asignación mediante SNP |
| L2Bridge | Asignación de IP estática y asignación de direcciones IP dentro del prefijo de red del host de contenedor (también se puede asignar a través de SNP) |
| L2Tunnel | Solo Azure: asignación de IP dinámica y asignación desde complemento |

### <a name="service-discovery"></a>Detección de servicios

La detección de servicios solo se admite para determinados controladores de red de Windows.

|  | Detección de servicios locales  | Detección de servicios globales |
| :---: | :---------------     |  :---                |
| nat | SÍ | SÍ, con Docker EE |  
| overlay | SÍ | SÍ, con Docker EE o Kube-DNS |
| transparent | NO | NO |
| l2bridge | NO | SÍ, con Kube-DNS |
