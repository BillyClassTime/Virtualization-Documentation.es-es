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
ms.openlocfilehash: 9acb433e0165d0ca97012dc73363036804298d2d
ms.sourcegitcommit: c00fe5752f45378177f1927f10cb09da7cae402c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/22/2017
---
# <a name="getting-started-with-swarm-mode"></a>Introducción al modo enjambre 

## <a name="what-is-swarm-mode"></a>¿Qué es el “modo enjambre”?
El modo enjambre es una característica de Docker que proporciona funciones de orquestación de contenedores integradas, lo que incluye la agrupación en clústeres de hosts de Docker y la programación de cargas de trabajo de contenedores. Un grupo de hosts de Docker forman un clúster “enjambre” cuando sus motores de Docker se ejecutan juntos en “modo enjambre”. Para obtener contexto adicional respecto al modo enjambre, consulta [el sitio de documentación principal del Docker](https://docs.docker.com/engine/swarm/).

## <a name="manager-nodes-and-worker-nodes"></a>Nodos de administrador y nodos de trabajo
Un enjambre se compone de dos tipos de hosts de contenedor: *nodos de administrador* y *nodos de trabajo*. Cada enjambre se inicializa a través de un nodo de administrador y todos los comandos de la CLI de Docker para el control y la supervisión de un enjambre se deben ejecutar desde uno de sus nodos de administrador. Los nodos de administrador se pueden ver como “guardianes” del estado del enjambre; en conjunto, forman un grupo de consenso que mantiene el conocimiento del estado de los servicios que se ejecutan en el enjambre, y su trabajo es garantizar que el estado real del enjambre siempre coincide con el estado pretendido, según lo que hayan definido el desarrollador o el administrador. 

>    **Nota:** Cualquier enjambre puede tener varios nodos de administrador, pero siempre debe tener *al menos uno*. 

Los nodos de trabajo los organiza el enjambre de Docker a través de los nodos de administrador. Para unirse a un enjambre, un nodo de trabajo debe usar un “token de unión” que genera el nodo de administrador al inicializa el enjambre. Los nodos de trabajo simplemente reciben tareas desde los nodos de administrador y las ejecutan, por lo que no requieren (ni poseen) ningún conocimiento del estado del enjambre.

## <a name="swarm-mode-system-requirements"></a>Requisitos del sistema para el modo enjambre

Al menos un sistema de equipo físico o virtual (para usar la funcionalidad completa de enjambre, se recomienda al menos dos nodos) que ejecute **Windows 10 Creators Update** o **Windows Server 2016** *con todas las últimas actualizaciones\ **, configurado como un host de contenedor (consulta el tema [Contenedores de Windows en Windows 10](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10) o [Contenedores de Windows en Windows Server](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-server) para obtener más información sobre cómo empezar a usar contenedores de Docker en Windows 10).

\***Nota**: El enjambre de Docker en Windows Server 2016 requiere [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217).

**Docker Engine v1.13.0 o posterior**

Puertos abiertos: Los siguientes puertos deben estar disponibles en todos los host. En algunos sistemas, estos puertos están abiertos de manera predeterminada.
- Puerto TCP 2377 para las comunicaciones de administración de clústeres.
- Puerto TCP y UDP 7946 para la comunicación entre los nodos.
- Puerto UDP 4789 para el tráfico de redes superpuestas

## <a name="initializing-a-swarm-cluster"></a>Inicialización de un clúster enjambre

Para inicializar un enjambre, solo tienes que ejecutar el comando siguiente desde uno de los hosts de contenedor (sustituyendo \<HOSTIPADDRESS\> por la dirección IPv4 local del equipo host):

```none
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```
Cuando este comando se ejecuta desde un host de contenedor, el Docker Engine de dicho host comienza a ejecutarse en modo enjambre como un nodo de administrador.

## <a name="adding-nodes-to-a-swarm"></a>Agregar nodos a un enjambre

> **Nota:** *No* son necesarios varios nodos para aprovechar las características del modo enjambre y las redes superpuestas. Todas las características de enjambre/superposición pueden usarse con un único host que se ejecute en modo enjambre (es decir, un nodo de administrador situado en modo enjambre con el comando `docker swarm init`).

### <a name="adding-workers-to-a-swarm"></a>Agregar nodos de trabajo a un enjambre
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

### <a name="adding-managers-to-a-swarm"></a>Agregar nodos de administrador a un enjambre
Pueden agregarse nodos de administrador adicionales a un clúster enjambre con el siguiente comando:

```none
C:\> docker swarm join --token <MANAGERJOINTOKEN> <MANAGERIPADDRESS>
```

De nuevo, \<MANAGERIPADDRESS\> es la dirección IP local de un nodo de administrador enjambre. El token de unión \<MANAGERJOINTOKEN\> es un token de unión de nodo de *administrador* para el enjambre, que puede obtenerse mediante la ejecución de uno de los siguientes comandos desde un nodo de administrador existente:

```none
# Get the full command required to join a **manager** node to the swarm
C:\> docker swarm join-token manager

# Get only the join-token needed to join a **manager** node to the swarm
C:\> docker swarm join-token manager -q
```

## <a name="creating-an-overlay-network"></a>Creación de una red superpuesta

Una vez que se ha configurado un clúster enjambre, se pueden crear redes superpuestas en el enjambre. Es posible crear una red superpuesta mediante la ejecución del siguiente comando desde un nodo de administrador enjambre:

```none
# Create an overlay network 
C:\> docker network create --driver=overlay <NETWORKNAME>
```

Aquí, \<NETWORKNAME\> es el nombre que quieras dar a la red.

## <a name="deploying-services-to-a-swarm"></a>Implementación de servicios para un enjambre
Una vez que se ha creado una red superpuesta, pueden crearse servicios y adjuntarlos a dicha red. Para crear un servicio, se emplea la sintaxis siguiente:

```none
# Deploy a service to the swarm
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

Aquí, \<SERVICENAME\> es el nombre que quieres darle al servicio; es el nombre que usarás para hacer referencia al servicio a través de la detección de servicios (que usa el servidor DNS nativo de Docker). \<NETWORKNAME\> es el nombre de la red a la que quieres conectar este servicio (por ejemplo, "myOverlayNet"). \<CONTAINERIMAGE\> es el nombre de la imagen de contenedor que definirá el servicio.

> **Nota:** El segundo argumento de este comando, `--endpoint-mode dnsrr`, es necesario especificarle a Docker Engine que se usará la directiva de round robin de DNS para equilibrar el tráfico de red en los puntos de conexión del contenedor de servicios. Actualmente, Round robin de DNS es la única estrategia de equilibrio de carga compatible en Windows . La [malla de enrutamiento](https://docs.docker.com/engine/swarm/ingress/) para hosts de Windows Docker aún no se admite, pero pronto estará disponible. Los usuarios que deseen una estrategia de equilibrio de carga alternativa en estos momentos pueden configurar un equilibrador de carga externo (como NGINX) y usar el [modo de publicar puerto](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) del enjambre para exponer los puertos de host del contenedor para los que equilibrar la carga.

## <a name="scaling-a-service"></a>Escalado de un servicio
Una vez que un servicio se ha implementado en un clúster enjambre, se implementan en todo el clúster las instancias de contenedor que componen ese servicio. De manera predeterminada, el número de instancias de contenedor que respaldan un servicio (el número de “réplicas” o “tareas” para un servicio) es uno. Sin embargo, se puede crear un servicio con varias tareas mediante la opción `--replicas` para el comando `docker service create` o mediante el escalado del servicio después de haberlo creado.

La escalabilidad del servicio es una de las ventajas clave que ofrece el modo enjambre de Docker y, además, se puede aprovechar con un solo comando de Docker:

```none
C:\> docker service scale <SERVICENAME>=<REPLICAS>
```

Aquí, \<SERVICENAME\> es el nombre del servicio que se escala, y \<REPLICAS\> es el número de tareas, o instancias del contenedor, al que se escala el servicio.


## <a name="viewing-the-swarm-state"></a>Visualización el estado del enjambre

Existen varios comandos útiles para ver el estado de un enjambre y los servicios que se ejecutan en él.

### <a name="list-swarm-nodes"></a>Mostrar lista de nodos enjambre
Usa el siguiente comando para ver una lista de los nodos unidos a un enjambre en estos momentos, incluida información sobre el estado de cada uno de dichos nodos. Este comando debe ejecutarse desde un **nodo de administrador**.

```none
C:\> docker node ls
```

En el resultado de este comando, verás que uno de los nodos está marcado con un asterisco (*); este asterisco indica simplemente el nodo actual: el nodo desde el que el se ha ejecutado el comando `docker node ls`.

### <a name="list-networks"></a>Mostrar lista de redes
Usa el siguiente comando para ver una lista de las redes que se encuentran en un nodo determinado. Para ver las redes superpuestas, este comando debe ejecutarse desde un **nodo de administrador** que se ejecute en modo enjambre.

```none
C:\> docker network ls
```

### <a name="list-services"></a>Mostrar lista de servicios
Usa el siguiente comando para ver una lista de los servicios que se están ejecutando actualmente en un enjambre, incluida información sobre su estado.

```none
C:\> docker service ls
```

### <a name="list-the-container-instances-that-define-a-service"></a>Mostrar lista de las instancias de contenedor que definen un servicio
Usa el siguiente comando para ver los detalles de las instancias de contenedor que se ejecutan para un servicio determinado. El resultado de este comando incluye los identificadores y los nodos en los que se ejecuta cada contenedor, así como información sobre el estado de los contenedores.  

```none
C:\> docker service ps <SERVICENAME>
```
## <a name="linuxwindows-mixed-os-clusters"></a>Clústeres de sistemas operativos combinados Windows + Linux

Recientemente, un miembro de nuestro equipo publicó una demo breve de tres partes sobre cómo configurar una aplicación con sistemas operativos combinados Windows y Linux con el enjambre de Docker. Es un lugar excelente para empezar si no estás familiarizado con el enjambre de Docker o para utilizarlo para ejecutar aplicaciones con sistemas operativos combinados. Echa un vistazo ahora:
- [Usar enjambre de Docker para ejecutar una aplicación en contenedores de Windows + Linux (parte 1/3)](https://www.youtube.com/watch?v=ZfMV5JmkWCY&t=170s)
- [Usar enjambre de Docker para ejecutar una aplicación en contenedores de Windows + Linux (parte 2/3)](https://www.youtube.com/watch?v=VbzwKbcC_Mg&t=406s)
- [Usar enjambre de Docker para ejecutar una aplicación en contenedores de Windows + Linux (parte 3/3)](https://www.youtube.com/watch?v=I9oDD78E_1E&t=354s)

### <a name="initializing-a-linuxwindows-mixed-os-cluster"></a>Inicialización de un clúster con sistemas operativos combinados Linux+Windows
La inicialización de un clúster enjambre con sistemas operativos combinados es fácil: siempre y cuando las reglas de firewall estén correctamente configuradas y los hosts tengan acceso unos a otros, lo único que necesitas para agregar un host de Linux a un conjunto es el comando `docker swarm join` estándar:
```none
C:\> docker swarm join --token <JOINTOKEN> <MANAGERIPADDRESS>
```
También se puede inicializar un enjambre desde un host de Linux con el mismo comando que se ejecutaría para inicializar el enjambre desde un host de Windows:
```none
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```

### <a name="adding-labels-to-swarm-nodes"></a>Agregar etiquetas a nodos enjambre
Para iniciar un servicio de Docker para un clúster enjambre de sistemas operativos combinados, debe haber una forma de distinguir qué nodos enjambre ejecutan el sistema operativo para el que se ha diseñado ese servicio y cuáles no. Las [etiquetas de objeto de Docker](https://docs.docker.com/engine/userguide/labels-custom-metadata/) proporcionan una forma útil de etiquetar nodos, de forma que los servicios pueden crearse y configurarse para ejecutarse solamente en los nodos que coincidan con su sistema operativo. 

> Nota: Las [etiquetas de objeto de Docker](https://docs.docker.com/engine/userguide/labels-custom-metadata/) puede usarse para aplicar los metadatos a diversos objetos de Docker (incluidas imágenes de contenedor, contenedores, volúmenes y redes) y para diversos fines (por ejemplo, pueden usarse etiquetas para separar componentes de 'front-end' y 'back-end' de una aplicación, al permitir la programación de microservicios front-end únicamente en nodos etiquetados como 'front-end' y programar microservicios back-end solo en nodos etiquetados como 'back-end'). En este caso, usamos etiquetas en nodos para distinguir entre nodos con sistema operativo Windows y nodos con sistema operativo Linux.

Para etiquetar los nodos enjambre existente, usa la sintaxis siguiente:

```none
C:\> docker node update --label-add <LABELNAME>=<LABELVALUE> <NODENAME>
```

Aquí, `<LABELNAME>` es el nombre de la etiqueta que se crea: por ejemplo, en este caso distinguimos nodos por su sistema operativo, por lo que un nombre lógico de la etiqueta podría ser "so". `<LABELVALUE>` es el valor de la etiqueta: en este caso, puedes usar los valores "windows" y "linux". (Por supuesto, puedes elegir los nombres que quieras para la etiqueta y los valores de la etiqueta, siempre que mantengas la coherencia). `<NODENAME>` es el nombre del nodo que se etiqueta; puedes recordarte los nombres de los nodos ejecutando `docker node ls`. 

**Por ejemplo**, si tienes cuatro nodos enjambre en el clúster, incluidos dos nodos Windows y dos nodos Linux, los comandos de actualización de la etiquetas pueden tener el siguiente aspecto:

```none
# Example -- labeling 2 Windows nodes and 2 Linux nodes in a cluster...
C:\> docker node update --label-add os=windows Windows-SwarmMaster
C:\> docker node update --label-add os=windows Windows-SwarmWorker1
C:\> docker node update --label-add os=linux Linux-SwarmNode1
C:\> docker node update --label-add os=linux Linux-SwarmNode2
```

### <a name="deploying-services-to-a-mixed-os-swarm"></a>Implementación de servicios para un enjambre con sistemas operativos combinados
Con etiquetas para los nodos enjambre, implementar servicios en el clúster es fácil; solo tienes que usar la opción `--constraint` para el comando [`docker service create`](https://docs.docker.com/engine/reference/commandline/service_create/):

```none
# Deploy a service with swarm node constraint
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> --constraint node.labels.<LABELNAME>=<LABELVALUE> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

Por ejemplo, si usas la nomenclatura de etiqueta y valor de etiqueta del ejemplo anterior, un conjunto de comandos de creación de servicios (uno para un servicio basado en Windows y otro para un servicio basado en Linux) podría tener el siguiente aspecto:

```none
# Example -- using the 'os' label and 'windows'/'linux' label values, service creation commands might look like these...

# A Windows service
C:\> docker service create --name=win_s1 --endpoint-mode dnsrr --network testoverlay --constraint 'node.labels.os==windows' microsoft/nanoserver:latest powershell -command { sleep 3600 }

# A Linux service
C:\> docker service create --name=linux_s1 --endpoint-mode dnsrr --network testoverlay --constraint 'node.labels.os==linux' redis
```

## <a name="limitations"></a>Limitaciones
Actualmente, el modo enjambre en Windows tiene las siguientes limitaciones:
- No se admite el cifrado del plano de datos (es decir, el tráfico de contenedor a contenedor mediante la opción `--opt encrypted`).
- La [malla de enrutamiento](https://docs.docker.com/engine/swarm/ingress/) para hosts de Windows Docker aún no se admite, pero pronto estará disponible. Los usuarios que deseen una estrategia de equilibrio de carga alternativa en estos momentos pueden configurar un equilibrador de carga externo (como NGINX) y usar el [modo de publicar puerto](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) del enjambre para exponer los puertos de host del contenedor para los que equilibrar la carga. Encontrarás más información sobre este tema más adelante.

## <a name="publish-ports-for-service-endpoints"></a>Publicar puertos para los puntos de conexión de servicio
La característica de [malla enrutamiento](https://docs.docker.com/engine/swarm/ingress/) del enjambre de Docker aún no se admite en Windows, pero los usuarios que quieran publicar los puertos para los puntos de conexión de servicios pueden hacerlo ya con el modo de publicar puerto. 

Para hacer que los puertos de host se publiquen para cada uno de los puntos de conexión de tareas o contenedores que definen un servicio, usa el argumento `--publish mode=host,target=<CONTAINERPORT>` para el comando `docker service create`:

```none
# Create a service for which tasks are exposed via host port
C:\ > docker service create --name=<SERVICENAME> --publish mode=host,target=<CONTAINERPORT> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

Por ejemplo, el siguiente comando crear un servicio, 's1', para el que cada tarea se mostrará a través del puerto 80 del contenedor y un puerto del host seleccionado de forma aleatoria.

```none
C:\ > docker service create --name=s1 --publish mode=host,target=80 --endpoint-mode dnsrr web_1 powershell -command {echo sleep; sleep 360000;}
```

Tras crear un servicio con el modo de publicar puerto, el servicio puede consultarse para ver la asignación de puertos correspondiente a cada tarea de servicio:

```none
C:\ > docker service ps <SERVICENAME>
```
El comando anterior devolverá los detalles sobre cada instancia de contenedor en ejecución para el servicio (en todos los hosts enjambre). Una columna de la salida, la columna "ports", incluye información de puerto para cada host con la forma \<HOSTPORT\>->\<CONTAINERPORT\>/tcp. Los valores de \<HOSTPORT\> será diferente para cada instancia de contenedor, ya que cada contenedor se publica en su propio puerto del host.


## <a name="tips--insights"></a>Sugerencias e información útil 

#### *<a name="existing-transparent-network-can-block-swarm-initializationoverlay-network-creation"></a>Una red transparente existente puede bloquear inicialización de un enjambre o la creación de una red superpuesta* 
En Windows, los controladores de red overlay (para redes superpuestas) y transparent (para redes transparentes) requieren que se enlace un vSwitch externo con un adaptador de red de host (virtual). Cuando se crea una red superpuesta, también se crea un nuevo conmutador y después se conecta a un adaptador de red abierto. El modo de redes transparentes también usa un adaptador de red de host. Al mismo tiempo, cualquier adaptador de red solo se puede enlazar a un único conmutador a la vez; si un host tiene un solo adaptador de red, puede conectarse a un único vSwitch externo en cada momento, tanto si ese vSwitch es para una red superpuesta como si es para una red transparente. 

Por lo tanto, si un host de contenedor tiene solamente un adaptador de red, es posible encontrarse el problema de que una red transparente bloquee la creación de una red superpuesta (o viceversa), porque la red transparente está ocupando actualmente la única interfaz de red virtual del host.

Existen dos maneras de solventar este problema:
- *Opción 1: Eliminar la red transparente existente.* Antes de inicializar un enjambre, asegúrate de que no haya ninguna red transparente existente en el host del contenedor. Elimina las redes transparentes para asegurarte de que hay un adaptador de red virtual libre en el host para usarlo para la creación de la red superpuesta.
- *Opción 2: Crear un adaptador de red (virtual) adicional en el host.* En lugar de quitar las redes transparentes que haya en el host, puedes crear un adaptador de red adicional en el host y utilizarlo para la creación de la red superpuesta. Para ello, simplemente crea un nuevo adaptador de red externo (mediante PowerShell o el administrador de Hyper-V); con la nueva interfaz creada, cuando se inicialice el enjambre, el servicio de red de host (HNS) la reconocerá automáticamente en el host y la usará para enlazar el vSwitch externo para la creación de la red superpuesta.



