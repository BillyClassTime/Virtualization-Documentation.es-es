---
title: "Introducción al modo enjambre"
description: "Inicialización de un clúster enjambre, creación de una red superpuesta y adjuntar un servicio a la red."
keywords: "docker, contenedores, enjambre, orquestación"
author: kallie-b
ms.date: 02/9/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5ceb9626-7c48-4d42-81f8-9c936595ad85
translationtype: Human Translation
ms.sourcegitcommit: f615c6dd268932a2ff99ac12c4e9ffdcf2cc217e
ms.openlocfilehash: ee6053003b31f226d2cfba8566f274ccc19d97ec
ms.lasthandoff: 03/02/2017

---

# Introducción al modo enjambre 

**Nota importante:** *Actualmente, la compatibilidad con el modo enjambre y las redes superpuestas solo está disponible para los usuarios de [Windows Insider](https://insider.windows.com/) como parte de la actualización Windows 10 Creators Update, que se publicará muy pronto. La compatibilidad con otras plataformas de Windows estará disponible próximamente.*

## ¿Qué es el “modo enjambre”?
El modo enjambre es una característica de Docker que proporciona funciones de orquestación de contenedores integradas, lo que incluye la agrupación en clústeres de hosts de Docker y la programación de cargas de trabajo de contenedores. Un grupo de hosts de Docker forman un clúster “enjambre” cuando sus motores de Docker se ejecutan juntos en “modo enjambre”. Para obtener contexto adicional respecto al modo enjambre, consulta [el sitio de documentación principal del Docker](https://docs.docker.com/engine/swarm/).

## Nodos de administrador y nodos de trabajo
Un enjambre se compone de dos tipos de hosts de contenedor: *nodos de administrador* y *nodos de trabajo*. Cada enjambre se inicializa a través de un nodo de administrador y todos los comandos de la CLI de Docker para el control y la supervisión de un enjambre se deben ejecutar desde uno de sus nodos de administrador. Los nodos de administrador se pueden ver como “guardianes” del estado del enjambre; en conjunto, forman un grupo de consenso que mantiene el conocimiento del estado de los servicios que se ejecutan en el enjambre, y su trabajo es garantizar que el estado real del enjambre siempre coincide con el estado pretendido, según lo que hayan definido el desarrollador o el administrador. 

>    **Nota:** Cualquier enjambre puede tener varios nodos de administrador, pero siempre debe tener *al menos uno*. 

Los nodos de trabajo los organiza el enjambre de Docker a través de los nodos de administrador. Para unirse a un enjambre, un nodo de trabajo debe usar un “token de unión” que genera el nodo de administrador al inicializa el enjambre. Los nodos de trabajo simplemente reciben tareas desde los nodos de administrador y las ejecutan, por lo que no requieren (ni poseen) ningún conocimiento del estado del enjambre.

## Requisitos del sistema para el modo enjambre

Al menos un sistema de equipo físico o virtual (para usar la funcionalidad completa del enjambre, se recomiendan al menos dos nodos) que ejecuten **Windows 10 Creators Update** (disponible para los miembros del programa [Windows Insider](https://insider.windows.com/)), configurado como host de contenedor (consulta el tema, [contenedores de Windows en Windows 10](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10) para obtener más información sobre cómo empezar a usar contenedores de Docker en Windows 10)

**Docker Engine v1.13.0 o posterior**

Puertos abiertos: Los siguientes puertos deben estar disponibles en todos los host. En algunos sistemas, estos puertos están abiertos de manera predeterminada.
- Puerto TCP 2377 para las comunicaciones de administración de clústeres.
- Puerto TCP y UDP 7946 para la comunicación entre los nodos.
- Puerto TCP y UDP 4789 para el tráfico de redes superpuestas.

## Inicialización de un clúster de enjambre
Para inicializar un enjambre, solo tienes que ejecutar el comando siguiente desde uno de los hosts de contenedor (sustituyendo \<HOSTIPADDRESS\> por la dirección IPv4 local del equipo host):

```none
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```
Cuando este comando se ejecuta desde un host de contenedor, el Docker Engine de dicho host comienza a ejecutarse en modo enjambre como un nodo de administrador.

## Agregar nodos a un enjambre

> **Nota:** *No* son necesarios varios nodos para aprovechar las características del modo enjambre y las redes superpuestas. Todas las características de enjambre/superposición pueden usarse con un único host que se ejecute en modo enjambre (es decir, un nodo de administrador situado en modo enjambre con el comando `docker swarm init`).

### Agregar nodos de trabajo a un enjambre
Una vez que se ha inicializado un enjambre desde un nodo de administrador, pueden agregarse otros hosts al enjambre como nodos de trabajo con otro sencillo comando:

```none
C:\> docker swarm join --token <WORKERJOINTOKEN> <MANAGERIPADDRESS>
```

Aquí, \<MANAGERIPADDRESS\> es la dirección IP local de un nodo de administrador enjambre, y \<WORKERJOINTOKEN\> es el token de unión de nodo de trabajo proporcionado como salida por el comando `docker swarm init` ejecutado desde el nodo de administrador. El token de unión también se puede obtener si se ejecuta uno de los siguientes comandos desde el nodo de administrador después de inicializar el enjambre:

```none
# Get the full command required to join a worker node to the swarm
C:\> docker swarm join-token worker

# Get only the join-token needed to join a worker node to the swarm
C:\> docker swarm join-token worker -q
```

### Agregar nodos de administrador a un enjambre
Pueden agregarse nodos de administrador adicionales a un clúster enjambre con el siguiente comando:

```none
C:\> docker swarm join --token <MANAGERJOINTOKEN> <MANAGERIPADDRESS>
```

De nuevo, \<MANAGERIPADDRESS\> es la dirección IP local de un nodo de administrador enjambre. El token de unión \<MANAGERJOINTOKEN\>, es un token de unión de nodo de *administrador* para el enjambre, que puede obtenerse mediante la ejecución de uno de los siguientes comandos desde un nodo de administrador existente:

```none
# Get the full command required to join a **manager** node to the swarm
C:\> docker swarm join-token manager

# Get only the join-token needed to join a **manager** node to the swarm
C:\> docker swarm join-token manager -q
```

## Creación de una red superpuesta

Una vez que se ha configurado un clúster enjambre, se pueden crear redes superpuestas en el enjambre. Es posible crear una red superpuesta mediante la ejecución del siguiente comando desde un nodo de administrador enjambre:

```none
# Create an overlay network 
C:\> docker network create --driver=overlay <NETWORKNAME>
```

Aquí, \<NETWORKNAME\> es el nombre que quieras dar a la red.

## Implementación de servicios para un enjambre
Una vez que se ha creado una red superpuesta, pueden crearse servicios y adjuntarlos a dicha red. Para crear una red, se emplea la sintaxis siguiente:

```none
# Deploy a service to the swarm
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

Aquí, \<SERVICENAME\> es el nombre que quieres darle al servicio; es el nombre que usarás para hacer referencia al servicio a través de la detección de servicios (que usa el servidor DNS nativo de Docker). \<NETWORKNAME\> es el nombre de la red a la que quieres conectar este servicio (por ejemplo, "myOverlayNet"). \<CONTAINERIMAGE\> es el nombre de la imagen de contenedor que definirá el servicio.

> **Nota:** El segundo argumento de este comando, `--endpoint-mode dnsrr`, es necesario especificarle a Docker Engine que se usará la directiva de round robin de DNS para equilibrar el tráfico de red en los puntos de conexión del contenedor de servicios. Actualmente, Round robin de DNS es la única estrategia de equilibrio de carga compatible en Windows . La [malla de enrutamiento](https://docs.docker.com/engine/swarm/ingress/) para hosts de Windows Docker aún no se admite, pero pronto estará disponible. Los usuarios que deseen una estrategia de equilibrio de carga alternativa en estos momentos pueden configurar un equilibrador de carga externo (como NGINX) y usar el [modo de publicar puerto](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) del enjambre para exponer los puertos de host del contenedor para los que equilibrar la carga.

## Escalado de un servicio
Una vez que un servicio se ha implementado en un clúster enjambre, se implementan en todo el clúster las instancias de contenedor que componen ese servicio. De manera predeterminada, el número de instancias de contenedor que respaldan un servicio (el número de “réplicas” o “tareas” para un servicio) es uno. Sin embargo, se puede crear un servicio con varias tareas mediante la opción `--replicas` para el comando `docker service create` o mediante el escalado del servicio después de haberlo creado.

La escalabilidad del servicio es una de las ventajas clave que ofrece el modo enjambre de Docker y, además, se puede aprovechar con un solo comando de Docker:

```none
C:\> docker service scale <SERVICENAME>=<REPLICAS>
```

Aquí, \<SERVICENAME\> es el nombre del servicio que se escala, y \<REPLICAS\> es el número de tareas, o instancias del contenedor, al que se escala el servicio.

## Visualización el estado del enjambre

Existen varios comandos útiles para ver el estado de un enjambre y los servicios que se ejecutan en él.

### Mostrar lista de nodos enjambre
Usa el siguiente comando para ver una lista de los nodos unidos a un enjambre en estos momentos, incluida información sobre el estado de cada uno de dichos nodos. Este comando debe ejecutarse desde un **nodo de administrador**.

```none
C:\ docker node ls
```

En el resultado de este comando, verás que uno de los nodos está marcado con un asterisco (*); este asterisco indica simplemente el nodo actual: el nodo desde el que el se ha ejecutado el comando `docker node ls`.

### Mostrar lista de redes
Usa el siguiente comando para ver una lista de las redes que se encuentran en un nodo determinado. Para ver las redes superpuestas, este comando debe ejecutarse desde un **nodo de administrador** que se ejecute en modo enjambre.

```none
C:\ docker network ls
```

### Mostrar lista de servicios
Usa el siguiente comando para ver una lista de los servicios que se están ejecutando actualmente en un enjambre, incluida información sobre su estado.

```none
C:\ docker service ls
```

### Mostrar lista de las instancias de contenedor que definen un servicio
Usa el siguiente comando para ver los detalles de las instancias de contenedor que se ejecutan para un servicio determinado. El resultado de este comando incluye los identificadores y los nodos en los que se ejecuta cada contenedor, así como información sobre el estado de los contenedores.  

```none
C:\ docker service ps <SERVICENAME>
```

## Limitaciones
Actualmente, el modo enjambre en Windows tiene las siguientes limitaciones:
- PROBLEMA CONOCIDO: Actualmente las redes superpuestas y el modo enjambre se admiten únicamente en hosts de contenedor conectados por Ethernet; **no funcionarán con hosts conectados por Wi-Fi.** Estamos trabajando para solucionar esta situación.
- No se admite el cifrado del plano de datos (es decir, el tráfico de contenedor a contenedor mediante la opción `--opt encrypted`).
- La [malla de enrutamiento](https://docs.docker.com/engine/swarm/ingress/) para hosts de Windows Docker aún no se admite, pero pronto estará disponible. Los usuarios que deseen una estrategia de equilibrio de carga alternativa en estos momentos pueden configurar un equilibrador de carga externo (como NGINX) y usar el [modo de publicar puerto](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) del enjambre para exponer los puertos de host del contenedor para los que equilibrar la carga.  



