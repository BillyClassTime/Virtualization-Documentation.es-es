---
title: Red de contenedor de Windows
description: Controladores de red y topologías para contenedores de Windows.
keywords: docker, contenedores
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: f044cf6f9d0457dd4cc9b444dcbeebc97f22f17b
ms.sourcegitcommit: bea2c90f31a38fc7fda356619f0dd812f79d008f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/31/2019
ms.locfileid: "9685292"
---
# <a name="windows-container-network-drivers"></a>Controladores de red de contenedor de Windows  

Además de aprovechar la red 'nat' predeterminada creada por Docker en Windows, los usuarios pueden definir redes de contenedores personalizadas. Las redes definidas por el usuario se pueden crear con el comando [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) de la CLI del Dock. En Windows, están disponibles los siguientes tipos de controladores de red:

- **nat**: los contenedores conectados a una red que se haya creado con el controlador 'nat' se conectarán a un conmutador Hyper-V *interno* y recibirán una dirección IP desde el prefijo de IP (``--subnet``) especificado por el usuario. Se admite la asignación o el enrutamiento de puertos desde el host del contenedor a los puntos de conexión del contenedor.
  
  >[!NOTE]
  > Las redes NAT creadas en Windows Server 2019 (o versiones posteriores) ya no se conservan después del reinicio.

  > Se admiten varias redes NAT si tiene instalada la versión Windows 10 Creators Update (o una versión posterior).
  
- **transparent**: los contenedores conectados a una red que se haya creado con el controlador 'transparente' se conectarán directamente a la red física, a través de un conmutador Hyper-V *externo*. Las direcciones IP de la red física se pueden asignar de manera estática (requiere la opción ``--subnet`` especificada por el usuario) o dinámica mediante un servidor DHCP externo.
  
  >[!NOTE]
  >Debido al siguiente requisito, las máquinas virtuales de Azure no admiten la conexión de los hosts de contenedor a través de una red transparente.
  
  > Requiere: cuando este modo se usa en un escenario de virtualización (el host de contenedor es una VM) se requiere la suplantación de _direcciones MAC_.

- **overlay**: cuando el motor docker se ejecuta en [modo enjambre](../manage-containers/swarm-mode.md), los contenedores conectados a una red de superposición ('overlay') pueden comunicarse con otros contenedores conectados a la misma red en varios hosts de contenedor. Cada red superpuesta que se crea en un clúster de enjambre se crea con su propia subred IP, definida por un prefijo de IP privado. El controlador de red de superposición usa encapsulado VXLAN. **Puede usarse con Kubernetes al usar los planos de control de red apropiados (Flannel o OVN).**
  > Requiere: Asegúrese de que su entorno cumple con los [requisitos previos](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) necesarios para crear redes de superposición.

  > Requiere: requiere Windows Server 2016 con [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217), Windows 10 Creators Update o una versión posterior.

  >[!NOTE]
  >En Windows Server 2019 que ejecuta Docko EE 18,03 y versiones posteriores, superponer redes creadas por Dock Swarm aprovechar las reglas NAT de VFP para conectividad saliente. Esto significa que un contenedor dado recibe una dirección IP. También significa que las herramientas basadas en ICMP, como `ping` o `Test-NetConnection` deben configurarse con sus opciones TCP/UDP en situaciones de depuración.

- **l2bridge**: los contenedores conectados a una red creada con el controlador 'l2bridge' estarán en la misma subred IP que el host de contenedor y estarán conectados a la red física a través de un conmutador Hyper-V *externo*. Las direcciones IP deben asignarse estáticamente desde el mismo prefijo que el host del contenedor. Todos los puntos de conexión de contenedor del host tendrán la misma dirección MAC que el host debido a la operación de traducción de direcciones de capa 2 (reescritura de MAC) en la entrada y la salida.
  > Requiere: requiere Windows Server 2016, Windows 10 Creators Update o una versión posterior.

  > Requiere: [Directiva OutboundNAT](./advanced.md#specify-outboundnat-policy-for-a-network) para conectividad externa.

- **l2tunnel**: similar a l2bridge; sin embargo _este controlador solo se debe usar en una pila de Microsoft Cloud_. Los paquetes que provienen de un contenedor se envían al host de virtualización, donde se aplica la directiva SDN.


## <a name="network-topologies-and-ipam"></a>Topologías de red y IPAM

En la tabla siguiente se muestra cómo se proporciona conectividad de red para las conexiones internas (de contenedor a contenedor) y externas para cada controlador de red.

### <a name="networking-modesdocker-drivers"></a>Modos de red/drivers de acoplamiento

  | Controlador de red de Windows Docker | Sistemas operativos típicos | Contenedor a contenedor (un solo nodo) | Contenedor a externo (+ nodo de un solo nodo) | Contenedor a contenedor (varios nodos) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (predeterminado)** | Adecuado para desarrolladores | <ul><li>Misma subred: conexión con puente a través del conmutador virtual de Hyper-V</li><li> Cruce de subred: no compatible (solo un prefijo NAT interno)</li></ul> | Enrutado a través de vNIC de administración (unido a WinNAT) | No se admite directamente: requiere exponer los puertos a través del host |
  | **Transparente** | Adecuado para desarrolladores o pequeñas implementaciones | <ul><li>Misma subred: conexión con puente a través del conmutador virtual de Hyper-V</li><li>Subred cruzada: enrutado a través de host de contenedor</li></ul> | Enrutado a través de host de contenedor con acceso directo al adaptador de red (físico) | Enrutado a través de host de contenedor con acceso directo al adaptador de red (físico) |
  | **Superposición** | Es bueno para varios nodos; necesario para la Swarm de acoplamiento, disponible en Kubernetes | <ul><li>Misma subred: conexión con puente a través del conmutador virtual de Hyper-V</li><li>Subred cruzada: el tráfico de red se encapsula y enruta a través de administración vNIC</li></ul> | No admitido directamente: requiere un segundo punto de conexión unido a la red NAT | Misma subred/subred cruzada: el tráfico de red se encapsula usando VXLAN y se enruta a través de administración vNIC |
  | **L2Bridge** | Se usa para Kubernetes y Microsoft SDN | <ul><li>Misma subred: conexión con puente a través del conmutador virtual de Hyper-V</li><li> Subred cruzada: la dirección MAC del contenedor se reescribe al entrar y salir y se enruta.</li></ul> | La dirección MAC del contenedor se reescribe al entrar y salir. | <ul><li>Misma subred: conexión con puente</li><li>Subred cruzada: enrutar a través de la administración de vNIC en WSv1709 y versiones posteriores</li></ul> |
  | **L2Tunnel**| Solo Azure | Misma subred/subred cruzada: enlazado con el conmutador virtual Hyper-V del host al que se aplica la directiva | El tráfico debe sir a través de la puerta de enlace de red virtual de Azure | Misma subred/subred cruzada: enlazado con el conmutador virtual Hyper-V del host al que se aplica la directiva |

### <a name="ipam"></a>IPAM

Las direcciones IP se asignan de forma diferente para cada controlador de red. Windows usa el servicio de redes de host (SNP) para proporcionar IPAM para el controlador nat y funciona con el modo enjambre de Docker (KVS interno) para proporcionar IPAM para la superposición. Todos los demás controladores de red usan IPAM externo.

| Modo/controlador de redes | IPAM |
| -------------------------|:----:|
| NAT | Asignación y asignación IP dinámicas mediante el servicio de redes de host (SNP) desde el prefijo de subred NAT interna |
| Transparente | Asignación de direcciones IP estática o dinámica (con servidor DHCP externo) en el prefijo de red del host de contenedor |
| Superposición | Asignación IP dinámica desde los prefijos gestionados por el modo de enjambre del motor de Docker y asignación mediante SNP |
| L2Bridge | Asignación de IP estática y asignación de direcciones IP en el prefijo de red del host de contenedor (también se puede asignar a través de SNP) |
| L2Tunnel | Solo Azure: asignación de IP dinámica y asignación desde complemento |

### <a name="service-discovery"></a>Detección de servicios

La detección de servicios solo se admite para determinados controladores de red de Windows.

|  | Detección de servicios locales  | Detección de servicios globales |
| :---: | :---------------     |  :---                |
| nat | SÍ | SÍ, con Docker EE |  
| overlay | SÍ | SÍ, con acoplador EE o Kube-DNS |
| transparente | NO | NO |
| l2bridge | NO | SÍ, con Kube-DNS |
