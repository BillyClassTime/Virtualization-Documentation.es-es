---
title: Red de contenedores de Windows
description: "Configuración de la red para contenedores de Windows."
keywords: docker, contenedores
author: jmesser81
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
translationtype: Human Translation
ms.sourcegitcommit: 23d4b665da627f35cf5fce49c3c9974d0ef287dd
ms.openlocfilehash: e56a5b984cc1c42e27628d00a5cd532788aef11c
ms.lasthandoff: 02/10/2017

---

# Red de contenedores

Los contenedores de Windows funcionan de forma similar a las máquinas virtuales en lo que respecta a las redes. Cada contenedor tiene un adaptador de red virtual (vNIC), que está conectado a un conmutador virtual (vSwitch), a través del cual se reenvía el tráfico de entrada y salida. Para aplicar el aislamiento entre contenedores del mismo host, se crea un compartimento de red para cada Windows Server y contenedor de Hyper-V en el que esté instalado el adaptador de red para el contenedor. Los contenedores de Windows Server usan una vNIC de host para conectarse al conmutador virtual. Los contenedores de Hyper-V usan una NIC de máquina virtual sintética (no expuesta a la máquina virtual de utilidad) para conectarse al conmutador virtual.

Los contenedores de Windows admiten cuatro modos o controladores de red diferentes: *nat*, *transparent*, *l2bridge* y *l2tunnel*. En función de su infraestructura de red física y de sus requisitos de red de un solo host o de varios hosts, debe elegir el modo de red que mejor se adapte a sus necesidades.

El motor de Docker crea una red NAT de manera predeterminada cuando se ejecuta por primera vez el servicio dockerd. El prefijo IP interno predeterminado que se crea es 172.16.0.0/12. Los puntos de conexión del contenedor se adjuntarán de forma automática a esta red predeterminada y se les asignará una dirección IP desde su prefijo interno.

> Nota: Si su dirección IP del host de contenedor está en este mismo prefijo, deberá cambiar el prefijo IP interno de NAT como se indica a continuación.

Pueden crearse en el mismo host de contenedor redes adicionales mediante un controlador diferente (por ejemplo, transparent, l2bridge). En la tabla siguiente se muestra cómo se proporciona conectividad de red para las conexiones internas (de contenedor a contenedor) y externas para cada modo.

- **Traducción de direcciones de red (NAT)**: Cada contenedor recibirá una dirección IP desde un prefijo IP interno y privado (por ejemplo, 172.16.0.0/12). Se admite la asignación o el reenvío de puerto del host de contenedor a los puntos de conexión del contenedor.

- **Transparente**: cada punto de conexión de contenedor está conectado directamente a la red física. Las direcciones IP de la red física se pueden asignar de manera estática o dinámica mediante un servidor DHCP externo.

- **[¡Novedad!] Superposición** : Cuando Docker Engine se está ejecutando en [modo enjambre](./swarm-mode.md), se pueden usar redes superpuestas, que se basan en la tecnología VXLAN, para conectar los puntos de conexión de contenedor entre varios hosts de contenedor. Cada red de superpuesta que se crea en un clúster enjambre se crea con su propia subred IP, definida por un prefijo de IP privado.

- **Puente de nivel 2**: Cada punto de conexión de contenedor estará en la misma subred IP que el host de contenedor. Las direcciones IP deben asignarse estáticamente desde el mismo prefijo que el host de contenedor. Todos los puntos de conexión de contenedor del host tendrán la misma dirección MAC debido a la traducción de direcciones de nivel 2.

- **Túnel de nivel 2** - _este modo solo se debe usar en una pila en la nube de Microsoft._

> Para obtener información sobre cómo conectar puntos de conexión de contenedor a una red virtual superpuesta con la pila de redes definidas por software (SDN) de Microsoft, consulte el tema [Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network) (Asociar contenedores a una red virtual).

## Nodo único

|  | De contenedor a contenedor | De contenedor a externo |
| :---: | :---------------     |  :---                |
| nat | Conexión con puente a través del conmutador virtual de Hyper-V | Enrutado a través de WinNAT con traducciones de direcciones aplicadas |
| transparent | Conexión con puente a través del conmutador virtual de Hyper-V | Acceso directo a la red física |
| Superposición | Se produce encapsulación de VXLAN en la extensión de reenvío de VFP en el conmutador virtual de Hyper-V; se produce comunicación *dentro del host* mediante una conexión con puente a través del conmutador virtual de Hyper-V | Enrutado a través de WinNAT con traducciones de direcciones aplicadas
| l2bridge | Conexión con puente a través del conmutador virtual de Hyper-V|  Acceso a la red física con traducción de direcciones MAC|  



## Múltiples nodos

|  | De contenedor a contenedor | De contenedor a externo |
| :---: | :----       | :---------- |
| nat | Debe hacer referencia al puerto y dirección IP del host de contenedor externo; enrutado a través de WinNAT con traducciones de direcciones aplicadas | Debe hacer referencia al host externo; enrutado a través de WinNAT con traducciones de direcciones aplicadas |
| transparent | Debe hacer referencia directamente al punto de conexión IP del contenedor | Acceso directo a la red física |
| Superposición | Se produce encapsulación de VXLAN en la extensión de reenvío de VFP en el conmutador virtual de Hyper-V; las comunicaciones *entre hosts* hacen referencia a puntos de conexión IP directamente. | Enrutado a través de WinNAT con traducciones de direcciones aplicadas| 
| l2bridge | Debe hacer referencia directamente al punto de conexión IP del contenedor| Acceso a la red física con traducción de direcciones MAC|


## Creación de redes

### Red NAT (predeterminada)

El motor de Docker de Windows crea una red NAT predeterminada (que Docker denomina "nat") con el prefijo IP 172.16.0.0/12. Si un usuario quiere crear una red NAT con un prefijo IP específico, puede realizar una de estas dos acciones cambiando las opciones en el archivo daemon.json de configuración de Docker (ubicado en C:\ProgramData\Docker\config\daemon.json; si no existe, créelo).
 1. Usar la opción _"fixed-cidr": "< prefijo IP > / Mask"_, que creará la red NAT predeterminada con el prefijo IP y la máscara especificada.
 2. Usar la opción _"bridge": "none"_, que no creará una red predeterminada. El usuario puede crear una red definida por el usuario con cualquier controlador mediante el comando *docker network create -d<driver>*.

Antes de llevar a cabo una de estas opciones de configuración, debe detener el servicio Docker y eliminar las redes NAT que existan.

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

Si la opción "fixed-cidr" se agrega al archivo daemon.json, el motor de Docker creará una red NAT definida por el usuario con el prefijo IP personalizado y la máscara especificada. Si en su lugar se agrega la opción "bridge:none", la red debe crearse de forma manual.

```none
# Create a user-defined NAT network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

De manera predeterminada, los puntos de conexión de contenedor se conectarán a la red "nat" predeterminada. Si no se ha creado la red "nat" (porque se ha especificado "bridge:none" en el archivo daemon.json) o si se requiere acceso a una red diferente definida por el usuario, los usuarios pueden especificar el parámetro *--network* con el comando de ejecución de Docker.

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### Asignación de puertos

Para tener acceso a las aplicaciones que se ejecutan dentro de un contenedor conectado a una red NAT, deben crearse asignaciones de puertos entre el host de contenedor y el punto de conexión de contenedor. Estas asignaciones se deben especificar al crear el contenedor o mientras el contenedor se encuentra en estado detenido.

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Creates a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

También se admiten las asignaciones de puertos dinámicos mediante el parámetro -p o el comando EXPOSE en un archivo Dockerfile con el parámetro -P. Si no se especifica, se seleccionará un puerto efímero aleatorio en el host de contenedor, que se puede inspeccionar al ejecutar "docker ps".

```none
C:\> docker run -itd -p 80 windowsservercore cmd

# Network services running on port TCP:80 in this container can be accessed externally on port TCP:14824
C:\> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
bbf72109b1fc        windowsservercore   "cmd"               6 seconds ago       Up 2 seconds        *0.0.0.0:14824->80/tcp*   drunk_stonebraker

# Container image specified EXPOSE 80 in Dockerfile - publish this port mapping
C:\> docker network
```
> A partir de WS2016 TP5 y compilaciones de Windows Insider superiores a 14300, se creará automáticamente una regla de firewall para todas las asignaciones de puertos NAT. Esta regla de firewall será global para el host de contenedor y no estará localizada en un adaptador de red o punto de conexión de contenedor específico.

La implementación de Windows NAT (WinNAT) tiene algunas brechas de funcionalidad que se comentan en la entrada de blog [WinNAT capabilities and limitations](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations/) (Funcionalidades y limitaciones de WinNAT).
 1. Solo se admite un prefijo IP interno de NAT por host de contenedor; para definir "varias" redes NAT, se deben crear particiones del prefijo (consulte la sección "Varias redes NAT" de este documento).
 2. Los puntos de conexión de contenedor solo son accesibles desde el host de contenedor mediante los puertos y direcciones IP internos del contenedor (puede encontrar esta información mediante "docker network inspect <CONTAINER ID>").

Se pueden crear redes adicionales mediante controladores diferentes.

> Los controladores de red de Docker usan solo letras minúsculas.

### Red transparente

Para usar el modo de red transparente, cree una red de contenedor con el nombre de controlador "transparent".

```none
C:\> docker network create -d transparent MyTransparentNetwork
```
> Nota: Si se produce un error al crear la red transparente, es posible que haya un vSwitch externo en el sistema que no ha detectado Docker de forma automática y, por tanto, impide que la red transparente se enlace con el adaptador de red externo del host de contenedor. Consulte la siguiente sección, "Un vSwitch existente bloquea la creación de redes transparentes" en "Advertencias y problemas comunes" para obtener más información.

Si el host de contenedor está virtualizado y quiere usar DHCP para la asignación de direcciones IP, debe habilitar MACAddressSpoofing en el adaptador de red de las máquinas virtuales. En caso contrario, el host de Hyper-V bloqueará el tráfico de red procedente de los contenedores en la máquina virtual con varias direcciones MAC.

```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> Si quieres crear más de una red transparente, debes especificar a qué adaptador de red (virtual) se debe enlazar el conmutador virtual externo de Hyper-V (creado automáticamente).

También se pueden asignar direcciones IP de manera estática o dinámica desde un servidor DHCP externo para los puntos de conexión de contenedor conectados a una red transparente.

Cuando use una asignación de IP estática, debe asegurarse de que se especifican los parámetros *--subnet* y *--gateway* al crear la red. La dirección IP de la puerta de enlace y la subred debe ser la misma que la configuración de red del host de contenedor; es decir, la red física.

```none
# Create a transparent network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 TransparentNet3
```
Especifica una dirección IP con la opción *--ip* en el comando `docker run`:

```none
C:\> docker run -it --network=TransparentNet3 --ip 10.123.174.105 <image> <cmd>
```

> Asegúrate de que esta dirección IP no está asignada a otro dispositivo de red de la red física.

Dado que los puntos de conexión de contenedor tienen acceso directo a la red física, no es necesario especificar asignaciones de puerto.

### Red superpuesta

*Para usar el modo de red superpuesta, debes estar usando un host de Docker que se ejecute en modo enjambre como un nodo de administrador.* Para obtener más información sobre el modo enjambre y cómo inicializar un administrador de enjambre, consulta el tema, [Introducción al modo enjambre](./swarm-mode.md).

Para crear una red superpuesta, ejecuta el siguiente comando desde un **nodo de administrador de enjambre**:

```none
# Create an overlay network from a swarm manager node, called "myOverlayNet"
C:\> docker network create --driver=overlay myOverlayNet
```

### Puente de nivel 2

Para usar el modo de redes de puente de nivel 2, crea una red de contenedor con el nombre de controlador "l2bridge". Es necesario especificar de nuevo una subred y una puerta de enlace, correspondientes a la red física.

```none
C:\> docker network create -d l2bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyBridgeNetwork
```

Solo se admite la asignación de IP estática con redes l2bridge.

> Cuando se usa una red l2bridge en un tejido de SDN, solo se admite la asignación de IP dinámica. Consulta el tema [Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network) (Asociar contenedores a una red virtual) para obtener más información.

## Otras operaciones y configuraciones

> Estamos trabajando constantemente para mejorar Docker en Windows. Para asegurarte de que tienes acceso a las últimas funcionalidades confirma que estás usando la versión más reciente del motor Docker. Puedes comprobar la versión de Docker usando `docker -v`. Consulta el tema [Docker Engine en Windows](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon) para ver las instrucciones sobre cómo configurar Docker.

### Mostrar en una lista las redes disponibles

```none
# list container networks
C:\> docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
0a297065f06a        nat                 nat                 local
d42516aa0250        none                null                local
```

### Quitar una red

Usa `docker network rm` para eliminar una red de contenedor.

```none
C:\> docker network rm <network name>
```

De este modo se limpian los conmutadores virtuales de Hyper-V que haya usado la red de contenedor, así como la traducción de direcciones de red creada (instancias de red NAT de WinNAT).

### Inspección de redes

Para ver qué contenedores están conectados a una red específica y las direcciones IP asociadas con estos puntos de conexión del contenedor, puedes ejecutar lo siguiente.

```none
C:\> docker network inspect <network name>
```

### Especifica el nombre de una red con el servicio SNP

**Normalmente, cuando se crea un contenedor de red mediante `docker network create`, el servicio de Docker usa el nombre de red que tú proporcionas, pero el servicio SNP no lo hace.**

Si vas a crear una red, puedes especificar el nombre que proporciona el servicio SNP con la opción `-o com.docker.network.windowsshim.networkname=<network name>` en el comando `docker network create`. Por ejemplo, podrías usar el siguiente comando para crear una red transparente con un nombre especificado para el servicio SNP:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

#### Ejemplo: Comportamiento predeterminado de los nombres de SNP

Para poner en contexto el comportamiento de esta opción de asignación de nombres, la captura de pantalla siguiente muestra cómo el servicio SNP da nombre a una red cuando *no* se usa esta opción de asignación de nombres. En este ejemplo, el nombre de la red, "MyTransparentNetwork" es visible para Docker, como se muestra en el comando `docker network ls`. Sin embargo, el nombre de la red no es visible para el servicio SNP, como se muestra mediante el comando `Get-ContainerNetwork` de Windows PowerShell; en su lugar, SNP ha generado automáticamente un nombre alfanumérico grande para la red.

><figure>
  <img src="media/SpecifyName_Capture.PNG">
  <figcaption>Ejemplo: El nombre de una red <i>no</i> se especifica para el servicio SNP. </figcaption>
</figure>

#### Ejemplo: Especificación del nombre de una red para el servicio SNP

Por otra parte, cuando *sí* se usa `-o com.docker.network.windowsshim.networkname=<network name>`, el servicio SNP emplea el nombre especificado en lugar de un nombre generado. Este comportamiento se muestra en la siguiente captura de pantalla.

><figure>
  <img src="media/SpecifyName_Capture_2.PNG">
  <figcaption>Ejemplo: Se especifica el nombre de una red para el servicio de SNP mediante la opción '-o com.docker.network.windowsshim.networkname=<network name>'.</figcaption>
</figure>


### Enlace de una red a una interfaz de red específica

Para enlazar una red (conectada a través del conmutador virtual de Hyper-V) a una interfaz de red específica, usa la opción `-o com.docker.network.windowsshim.interface=<Interface>` para el comando `docker network create`. Por ejemplo, podrías usar el siguiente comando para crear una red transparente que se adjunte a la interfaz de red "Ethernet 2":

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> Nota: El valor de *com.docker.network.windowsshim.interface* es el *nombre* del adaptador de red, que se puede encontrar con lo siguiente:

>```none
PS C:\> Get-NetAdapter
```

### Set the VLAN ID for a Network

To set a VLAN ID for a network, use the option, `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` to the `docker network create` command. For instance, you might use the following command to create a transparent network with a VLAN ID of 11:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
Cuando se establece el identificador de VLAN de una red, se establece el aislamiento de VLAN para los puntos de conexión de contenedor que se adjuntarán a esa red.

**Nota:** Asegúrate de que el adaptador de red de host (físico) está en modo de tronco para permitir que todo el tráfico etiquetado lo procese el vSwitch con el puerto vNIC (punto de conexión de contenedor) en el modo de acceso de la VLAN correcta.


### Especificación del sufijo DNS o de los servidores DNS de una red

Usa la opción `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` para especificar el sufijo DNS de una red y la opción `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` para especificar los servidores DNS de una red. Por ejemplo, podrías usar el siguiente comando para establecer el sufijo DNS de una red en "example.com" y los servidores DNS de una red en 4.4.4.4 y 8.8.8.8:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

### Varias redes de contenedor
Pueden crearse varias redes de contenedor en un host de contenedor único, pero deben tenerse en cuenta las advertencias siguientes:

* Las redes que usen un vSwitch externo para conectividad (por ejemplo, transparente, puente de nivel 2, transparente de nivel 2) deben usar su propio adaptador de red.
* Actualmente, la solución para crear varias redes NAT en un host de contenedor único es realizar particiones del prefijo interno de la red NAT existente. Consulte la siguiente sección "Varias redes NAT" para obtener más información.

### Varias redes NAT
Es posible definir varias redes NAT en un host de contenedor único al realizar particiones del prefijo interno de red NAT del host.

Las particiones de las nuevas redes NAT deben crearse en el prefijo de red NAT interno más grande. El prefijo puede encontrarse ejecutando el siguiente comando de PowerShell y al hacer referencia al campo "InternalIPInterfaceAddressPrefix".

```none
PS C:\> Get-NetNAT
```

Por ejemplo, el prefijo interno de red NAT del host podría ser 172.16.0.0/12. En este caso, se puede usar Docker para crear redes NAT adicionales *siempre que estén en el prefijo 172.16.0.0/12.* Por ejemplo, se podrían crear dos redes NAT con los prefijos IP 172.16.0.0/16 (puerta de enlace, 172.16.0.1) y 172.17.0.0/16 (puerta de enlace, 172.17.0.1).

```none
C:\> docker network create -d nat --subnet=172.16.0.0/16 --gateway=172.16.0.1 CustomNat1
C:\> docker network create -d nat --subnet=172.17.0.0/16 --gateway=172.17.0.1 CustomNat2
```

Las redes recién creadas se pueden enumerar mediante:
```none
C:\> docker network ls
```


### Selección de red

Al crear un contenedor de Windows, puede especificarse una red a la que se conectará el adaptador de red del contenedor. Si no se especifica ninguna red, se usará la red NAT predeterminada.

Para conectar un contenedor a una red NAT que no sea la predeterminada, use la opción --network con el comando de ejecución de Docker.

```none
C:\> docker run -it --network=MyTransparentNet windowsservercore cmd
```

### Dirección IP estática

```none
C:\> docker run -it --network=MyTransparentNet --ip=10.80.123.32 windowsservercore cmd
```

La asignación de dirección IP estática se lleva a cabo directamente en el adaptador de red del contenedor y solo debe realizarse cuando el contenedor se encuentre en estado detenido. No se admite el "agregado en caliente" de los adaptadores de red de contenedor ni los cambios en la pila de red mientras se esté ejecutando el contenedor.

## Docker Compose y detección de servicios

> Para obtener un ejemplo práctico de cómo se pueden usar Docker Compose y la detección de servicios para definir aplicaciones escaladas horizontalmente de varios servicios, visite [esta publicación](https://blogs.technet.microsoft.com/virtualization/2016/10/18/use-docker-compose-and-service-discovery-on-windows-to-scale-out-your-multi-service-container-application/) en nuestro [Blog Virtualization](https://blogs.technet.microsoft.com/virtualization/).

### Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) puede usarse para definir y configurar redes de contenedor junto con los contenedores y servicios que usarán esas redes. La clave de Compose "networks" se usa como clave de nivel superior en la definición de las redes a las que se conectarán los contenedores. Por ejemplo, la siguiente sintaxis define la red NAT preexistente creada por Docker como la red "default" (predeterminada) para todos los contenedores y servicios definidos en un archivo determinado de Compose.

```none
networks:
 default:
  external:
   name: "nat"
```

De forma similar, puede usarse la sintaxis siguiente para definir una red NAT personalizada.

> Nota: La "red NAT personalizada" definida en el ejemplo siguiente se define como una partición del prefijo interno de NAT preexistente del host de contenedor. Para obtener más contexto, consulte la sección anterior "Varias redes NAT".

```none
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.17.0.0/16
```

Para obtener más información sobre la definición y configuración de redes de contenedor mediante Docker Compose, consulte la [referencia del archivo de Compose](https://docs.docker.com/compose/compose-file/).

### Detección de servicios
La detección de servicios, que controla el registro de servicios y las asignaciones de nombre a IP (DNS) para contenedores y servicios, está integrada en Docker. Con la detección de servicios, todos los puntos de conexión de contenedor pueden detectarse entre ellos por nombre (ya sea por nombre del contenedor o nombre del servicio). Esto resulta especialmente útil en escenarios de escalado horizontal, en los que se usan varios puntos de conexión de contenedor para definir un servicio único. En tales casos, la detección de servicios facilita que un servicio se considere una entidad única, independientemente de cuántos contenedores se ejecuten en segundo plano. En el caso de los servicios de varios contenedores, el tráfico de red de entrada se administra con un enfoque round-robin, mediante el que se usa el equilibrio de carga de DNS para distribuir el tráfico de manera uniforme en todas las instancias de contenedor que implementan un servicio determinado.

## Redes superpuestas y el modo enjambre de Docker (redes de contenedor de varios nodos)
El controlador nativo de redes superpuestas y el modo enjambre de Docker se combinan para proporcionar compatibilidad para escenarios de varios nodos (clústeres) en Windows. Para obtener más información acerca de la superposición y del modo enjambre, visita la [entrada de blog](https://blogs.technet.microsoft.com/virtualization/2017/02/09/overlay-network-driver-with-support-for-docker-swarm-mode-now-available-to-windows-insiders-on-windows-10/) que acompañó al lanzamiento de la superposición y el enjambre para los usuarios de Windows Insider en Windows 10, o consulta el tema, [Introducción al modo enjambre](./swarm-mode.md).

## Advertencias y problemas comunes

### Un vSwitch existente bloquea la creación de redes transparentes

Al crear una red transparente, Docker crea un vSwitch externo para la red y después intenta enlazar el conmutador a un adaptador de red (externo); el adaptador podría ser un adaptador de red de máquina virtual o el adaptador de red físico. Si ya ha creado un vSwitch en el host de contenedor *y está visible para Docker*, el motor de Docker de Windows usará ese conmutador en lugar de crear uno nuevo. En cambio, si el vSwitch que se ha creado fuera de banda (es decir, se ha creado en el host de contenedor mediante el Administrador de Hyper-V o PowerShell) aún no está visible para Docker, el motor de Docker de Windows intentará crear un vSwitch nuevo y después no podrá conectar el nuevo conmutador con el adaptador de red externo del host de contenedor (porque el adaptador de red ya estará conectado al conmutador que se ha creado fuera de banda).

Por ejemplo, este problema podría aparecer si ha creado un vSwitch nuevo en el host mientras se estaba ejecutando el servicio Docker y después intenta crear una red transparente. En este caso, Docker no reconocería el conmutador que ha creado y crearía un vSwitch nuevo para la red transparente.

Hay tres formas de resolver este problema:

* Por supuesto, puede eliminar el vSwitch que se ha creado fuera de banda, lo que permitirá a Docker crear un vSwitch nuevo y conectarlo al adaptador de red de host sin ningún problema. Antes de elegir este enfoque, asegúrese de que ningún otro servicio (por ejemplo, Hyper-V) usa el vSwitch fuera de banda.
* Como alternativa, si decide usar un vSwitch externo que se ha creado fuera de banda, reinicie los servicios Docker y SNP para que *el conmutador esté visible para Docker.*
```none
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* Otra opción es usar la opción "-o com.docker.network.windowsshim.interface" para enlazar el vSwitch externo de la red transparente a un adaptador de red específico que no esté en uso en el host de contenedor (es decir, un adaptador de red distinto del que usa el vSwitch que se ha creado fuera de banda). La opción "-o" se describe con más detalle más adelante, en la sección [Red transparente](https://msdn.microsoft.com/virtualization/windowscontainers/management/container_networking#transparent-network) de este documento.

### Características no admitidas

En la actualidad, a través de la CLI de Docker no se admiten las siguientes características de red:
 * Vinculación de contenedores (por ejemplo, --link)

En este momento, en Windows Docker no se admiten las siguientes opciones de red:
 * --add-host
 * --dns-opt
 * --dns-search
 * --aux-address
 * --internal
 * --ip-range

