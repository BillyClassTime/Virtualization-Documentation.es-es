---
title: Redes de contenedores de Windows
description: Sencilla introducción a la arquitectura de redes de contenedores de Windows.
keywords: docker, contenedores
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: e9d4a9ac88c6853ce019a2469ee80688490b8fdf
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910705"
---
# <a name="windows-container-networking"></a>Redes de contenedores de Windows

>[!IMPORTANT]
>Haga referencia a las [redes de contenedor de Docker](https://docs.docker.com/engine/userguide/networking/) para los comandos, opciones y sintaxis de redes de Docker generales. * * * con la excepción de los casos descritos en [características no admitidas y opciones de red](#unsupported-features-and-network-options), todos los comandos de red de Docker se admiten en Windows con la misma sintaxis que en Linux. Sin embargo, las pilas de red de Windows y Linux son diferentes y, por lo tanto, verá que algunos comandos de red de Linux (por ejemplo, ifconfig) no se admiten en Windows.

## <a name="basic-networking-architecture"></a>Arquitectura de red básica

Este tema proporciona una visión general del modo en que Docker crea y administra redes de host en Windows. Los contenedores de Windows funcionan de forma similar a las máquinas virtuales en lo que respecta a las redes. Cada contenedor tiene un adaptador de red virtual (vNIC) que se conecta a un conmutador virtual Hyper-V (vSwitch). Windows admite cinco [controladores o modos de redes](./network-drivers-topologies.md) diferentes que pueden crearse mediante Docker: *nat*, *overlay*, *transparent*, *l2bridge*, e *l2tunnel*. En función de la infraestructura de red física y de los requisitos de red de un solo host o de varios hosts, debes elegir el controlador de red que mejor se adapte a tus necesidades.

![texto](media/windowsnetworkstack-simple.png)

La primera vez que se ejecuta Docker Engine, crea una red NAT predeterminada, 'nat', que usa un vSwitch interno y un componente de Windows denominado `WinNAT`. Si existen algunos vSwitch externo preexistente en el host, que se crearon mediante PowerShell o el administrador de Hyper-V, también estarán disponibles para Docker con el controlador de red *transparent* y podrán verse al ejecutar el comando ``docker network ls``.  

![texto](media/docker-network-ls.png)

- Un vSwitch **interno** es aquél que no está conectado directamente a un adaptador de red en el host del contenedor.
- Un vSwitch **externo** es aquel que está conectado directamente a un adaptador de red en el host del contenedor.

![texto](media/get-vmswitch.png)

La red 'nat' es la red predeterminada para los contenedores que se ejecutan en Windows. Los contenedores que se ejecuten en Windows sin marcas ni argumentos para implementar configuraciones de red específicas se adjuntarán a la red de 'nat' predeterminada y se les asignará automáticamente una dirección IP desde el intervalo de IP de prefijo interno de la red 'nat'. El prefijo IP interno de predeterminado usado para 'nat' es 172.16.0.0/16. 

## <a name="container-network-management-with-host-network-service"></a>Administración de redes de contenedores con el servicio de red de host

El servicio de redes de host (HNS) y el servicio de cálculo de host (HCS) trabajan conjuntamente para crear contenedores y conectar puntos de conexión a una red.

### <a name="network-creation"></a>Creación de redes

- HNS crea un nuevo conmutador virtual Hyper-V para cada red.
- SNP crea grupos de NAT e IP según sea necesario

### <a name="endpoint-creation"></a>Creación de punto de conexión

- SNP crea el espacio de nombres de red para cada punto de conexión de contenedor
- Lugares SNP/HCS v(m)NIC dentro del espacio de nombres de red
- SNP crea puertos (vSwitch)
- SNP asigna la dirección IP, la información de DNS, rutas, etc. (según el modo de redes) al punto de conexión

### <a name="policy-creation"></a>Creación de directivas

- Red NAT predeterminada: SNP crea reglas/asignaciones de enrutamiento de puerto WinNAT con las correspondientes reglas PERMITIR de Firewall de Windows
- Resto de redes: SNP utiliza la plataforma de filtrado Virtual (VFP) para la creación de directivas
    - Esto incluye: equilibrio de carga, ACL encapsulación, etc.
    - Busque nuestras API y el esquema de SNP publicados [aquí](https://docs.microsoft.com/en-us/windows-server/networking/technologies/hcn/hcn-top)

![texto](media/HNS-Management-Stack.png)

## <a name="unsupported-features-and-network-options"></a>Características y opciones de red no compatibles

Las siguientes opciones de red **no** se admiten actualmente en Windows:

- Los contenedores de Windows conectados a las redes l2bridge, NAT y overlay no admiten la comunicación a través de la pila IPv6.
- Comunicación de contenedor cifrada a través de IPsec.
- Compatibilidad con el proxy HTTP para contenedores.
- Redes de [modo host](https://docs.docker.com/ee/ucp/interlock/config/host-mode-networking/) 
- Redes en la infraestructura virtualizada de Azure a través del controlador de red transparente.

| Comando        | Opción no admitida   |
|---------------|:--------------------:|
| ``docker run``|   ``--ip6``, ``--dns-option`` |
| ``docker network create``| ``--aux-address``, ``--internal``, ``--ip-range``, ``--ipam-driver``, ``--ipam-opt``, ``--ipv6``, ``--opt encrypted`` |
