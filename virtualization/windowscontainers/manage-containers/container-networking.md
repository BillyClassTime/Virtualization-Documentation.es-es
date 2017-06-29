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
ms.openlocfilehash: cfab87efeb04f701f405739d53e6028f841712ce
ms.sourcegitcommit: ca64c1aceccd97c6315b28ff814ec7ac91fba9da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2017
---
# <a name="windows-container-networking"></a>Red de contenedores de Windows
> ***Consulta [Docker Container Networking](https://docs.docker.com/engine/userguide/networking/) (Redes de contenedores de Docker) para conocer los comandos, las opciones y sintaxis generales para las redes de Docker.*** Con excepción de los casos descritos en este documento, todos los comandos de red de Docker se admiten en Windows con la misma sintaxis que en Linux. De todas formas, ten en cuenta que las pilas de red de Windows y Linux son diferentes y, por tanto, encontrarás que algunos comandos de red de Linux (por ejemplo, ifconfig) no se admiten en Windows.

## <a name="basic-networking-architecture"></a>Arquitectura de red básica
Este tema proporciona una visión general del modo en que Docker crea y administra redes en Windows. Los contenedores de Windows funcionan de forma similar a las máquinas virtuales en lo que respecta a las redes. Cada contenedor tiene un adaptador de red virtual (vNIC) que se conecta a un conmutador virtual Hyper-V (vSwitch). Windows admite cinco controladores de red o modos diferentes que pueden crearse mediante Docker: *nat*, *overlay*, *transparent*, *l2bridge* y *l2tunnel*. En función de la infraestructura de red física y de los requisitos de red de un solo host o de varios hosts, debes elegir el controlador de red que mejor se adapte a tus necesidades.

<figure>
  <img src="media/windowsnetworkstack-simple.png">
</figure>  

La primera vez que se ejecuta Docker Engine, crea una red NAT predeterminada, 'nat', que usa un vSwitch interno y un componente de Windows denominado `WinNAT`. Si hay cualquier vSwitch externo preexistente en el host que se creara mediante PowerShell o el administrador de Hyper-V, también estarán disponibles para Docker con el controlador de red *transparent* y podrán verse al ejecutar el comando ``docker network ls``.  

<figure>
  <img src="media/docker-network-ls.png">
</figure>

> - Un vSwitch ***interno*** es aquel que no está conectado directamente a un adaptador de red en el host contenedor. 

> - Un vSwitch ***externo*** es aquel que _está_ conectado directamente a un adaptador de red en el host contenedor.  

<figure>
  <img src="media/get-vmswitch.png">
</figure>

La red 'nat' es la red predeterminada para los contenedores que se ejecutan en Windows. Los contenedores que se ejecuten en Windows sin marcas ni argumentos para implementar configuraciones de red específicas se adjuntarán a la red de 'nat' predeterminada y se les asignará automáticamente una dirección IP desde el intervalo de IP de prefijo interno de la red 'nat'. El prefijo IP interno de predeterminado usado para 'nat' es 172.16.0.0/16. 


## <a name="windows-container-network-drivers"></a>Controladores de red de contenedores Windows  

Además de aprovechar la red 'nat' predeterminada creada por Docker en Windows, los usuarios pueden definir redes de contenedores personalizadas. Pueden crearse redes definidas por el usuario mediante la CLI de Docker, con el comando [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/). En Windows, están disponibles los siguientes tipos de controladores de red:

- **nat**: Los contenedores conectados a una red que se haya creado con el controlador 'nat' recibirán una dirección IP desde el prefijo de IP especificado por el usuario (``--subnet``). Se admite la asignación o el reenvío de puertos desde el host del contenedor a los puntos de conexión del contenedor.
> Nota: Con Windows 10 Creators Update se admiten varias redes NAT. 

- **transparent**: Los contenedores conectados a una red que se haya creado con el controlador 'transparent' se conectarán directamente a la red física. Las direcciones IP de la red física se pueden asignar de manera estática (requiere la opción ``--subnet`` especificada por el usuario) o dinámica mediante un servidor DHCP externo. 

- **overlay** - __Novedad.__  Cuando Docker Engine se ejecuta en [modo enjambre](./swarm-mode.md), los contenedores conectados a una red de superposición ('overlay') pueden comunicarse con otros contenedores conectados a la misma red entre varios hosts de contenedor. Cada red superpuesta que se crea en un clúster enjambre se crea con su propia subred IP, definida por un prefijo de IP privado. El controlador de red overlay usa encapsulación VXLAN.
> Requiere Windows Server 2016 con [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217) o Windows 10 Creators Update. 

- **l2bridge**: Los contenedores conectados a una red que se haya creado con el controlador 'l2bridge' estarán en la misma subred IP que el host del contenedor. Las direcciones IP deben asignarse estáticamente desde el mismo prefijo que el host del contenedor. Todos los puntos de conexión de contenedor del host tendrán la misma dirección MAC debido a la operación de traducción de direcciones de capa 2 (reescritura de MAC) en la entrada y la salida.
> Requiere Windows Server 2016 o Windows 10 Creators Update.

- **l2tunnel** - _Este controlador solo se debe usar en una pila de Microsoft Cloud._

> Para obtener información sobre cómo conectar puntos de conexión de contenedor a una red virtual superpuesta con la pila de redes definidas por software (SDN) de Microsoft, consulte el tema [Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network) (Asociar contenedores a una red virtual).

> Windows 10 Creators Update ha introducido la compatibilidad de plataformas para agregar un nuevo punto de conexión de contenedor a un contenedor en ejecución (es decir, el "agregado en caliente"). Se activará de un extremo a otro pendiente de una [solicitud de extracción de Docker pendiente](https://github.com/docker/libnetwork/pull/1661)

## <a name="network-topologies-and-ipam"></a>Topologías de red e IPAM
En la tabla siguiente se muestra cómo se proporciona conectividad de red para las conexiones internas (de contenedor a contenedor) y externas para cada controlador de red.

<figure>
  <img src="media/network-modes-table.png">
</figure>

### <a name="ipam"></a>IPAM 
Las direcciones IP se asignan de forma diferente para cada controlador de red. Windows usa el servicio de red de host (HNS) para proporcionar IPAM para el controlador nat y funciona con el modo enjambre de Docker (KVS interno) para proporcionar IPAM para la superposición. Todos los demás controladores de red usan IPAM externo.

<figure>
  <img src="media/ipam.png">
</figure>

# <a name="details-on-windows-container-networking"></a>Información sobre las redes de contenedores de Windows

## <a name="isolation-namespace-with-network-compartments"></a>Aislamiento (espacio de nombres) con compartimentos de red
Cada punto de conexión de contenedor se coloca en su propio __compartimento de red__, que equivale a un espacio de nombres de red en Linux. El vNIC del host de administración y la pila de red de host se sitúan en el compartimento de red predeterminado. Para aplicar el aislamiento de red entre contenedores del mismo host, se crea un compartimento de red para cada Windows Server y cada contenedor de Hyper-V en el que esté instalado el adaptador de red para el contenedor. Los contenedores de Windows Server usan una vNIC de host para conectarse al conmutador virtual. Los contenedores de Hyper-V usan una NIC de máquina virtual sintética (no expuesta a la máquina virtual de utilidad) para conectarse al conmutador virtual. 

<figure>
  <img src="media/network-compartment-visual.png">
</figure>

```powershell 
Get-NetCompartment
```

## <a name="windows-firewall-security"></a>Seguridad del Firewall de Windows

El Firewall de Windows se usa para reforzar la seguridad de red a través de las ACL de puerto.

> Nota: De manera predeterminada, para todos los puntos de conexión de contenedor conectados a una red de superposición se crea una regla ALLOW ALL.   

<figure>
  <img src="media/windows-firewall-containers.png">
</figure>

## <a name="container-network-management-with-host-network-service"></a>Administración de redes de contenedores con el servicio de red de host

La siguiente imagen muestra cómo colaboran el servicio de red de host (HNS) y el servicio de proceso de host (HCS) para crear contenedores y conectar puntos de conexión a una red. 

<figure>
  <img src="media/HNS-Management-Stack.png">
</figure>

# <a name="advanced-network-options-in-windows"></a>Opciones de red avanzadas en Windows
Se admiten varias opciones de controladores de red para aprovechar las características y funcionalidades específicas de Windows. 

## <a name="switch-embedded-teaming-with-docker-networks"></a>Switch Embedded Teaming con redes de Docker

> Se aplica a todos los controladores de red. 

Puedes aprovechar [Switch Embedded Teaming](https://technet.microsoft.com/en-us/windows-server-docs/networking/technologies/hyper-v-virtual-switch/rdma-and-switch-embedded-teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set) al crear redes de host de contenedor para que las use Docker si especifica varios adaptadores de red (separados por comas) con la opción `-o com.docker.network.windowsshim.interface`. 

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2", "Ethernet 3" TeamedNet
```

## <a name="set-the-vlan-id-for-a-network"></a>Establecer el ID de VLAN para una red

> Se aplica a los controladores de red transparent y l2bridge 

Para establecer un ID de VLAN una red, usa la opción `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` para el comando `docker network create`. Por ejemplo, es posible usar el siguiente comando para crear una red transparente con un ID de VLAN de 11:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
Cuando se establece el identificador de VLAN de una red, se establece el aislamiento de VLAN para los puntos de conexión de contenedor que se adjuntarán a esa red.

> Asegúrate de que el adaptador de red de host (físico) está en modo de tronco para permitir que todo el tráfico etiquetado lo procese el vSwitch con el puerto vNIC (punto de conexión de contenedor) en el modo de acceso de la VLAN correcta.

## <a name="specify-the-name-of-a-network-to-the-hns-service"></a>Especificar el nombre de una red con el servicio HNS

> Se aplica a todos los controladores de red. 

Normalmente, cuando se crea un contenedor de red mediante `docker network create`, el servicio de Docker usa el nombre de red que tú proporcionas, pero el servicio SNP no lo hace. Si vas a crear una red, puedes especificar el nombre que proporciona el servicio SNP con la opción `-o com.docker.network.windowsshim.networkname=<network name>` en el comando `docker network create`. Por ejemplo, podrías usar el siguiente comando para crear una red transparente con un nombre especificado para el servicio HNS:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

## <a name="bind-a-network-to-a-specific-network-interface"></a>Enlazar una red con una interfaz de red específica

> Se aplica a todos los controladores de red excepto 'nat'  

Para enlazar una red (conectada a través del conmutador virtual de Hyper-V) con una interfaz de red específica, usa la opción `-o com.docker.network.windowsshim.interface=<Interface>` para el comando `docker network create`. Por ejemplo, podrías usar el siguiente comando para crear una red transparente que se adjunte a la interfaz de red "Ethernet 2":

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> Nota: El valor de *com.docker.network.windowsshim.interface* es el *nombre* del adaptador de red, que se puede encontrar con lo siguiente:

>```none
PS C:\> Get-NetAdapter
```
## Specify the DNS Suffix and/or the DNS Servers of a Network

> Applies to all network drivers 

Use the option, `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` to specify the DNS suffix of a network, and the option, `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` to specify the DNS servers of a network. For example, you might use the following command to set the DNS suffix of a network to "example.com" and the DNS servers of a network to 4.4.4.4 and 8.8.8.8:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## <a name="vfp"></a>VFP

La extensión de plataforma de filtrado virtual (VFP) es un conmutador virtual de Hyper-V que reenvía la extensión usada para imponer el uso de la directiva de red y manipular los paquetes. Por ejemplo, VFP lo usan el controlador de red 'overlay' para realizar la encapsulación VXLAN y el controlador 'l2bridge' para realizar reescritura de MAC en la entrada y la salida. La extensión VFP solo está presente en Windows Server 2016 y Windows 10 Creators Update. Para comprobar si se ejecuta correctamente, un usuario debe ejecutar dos comandos:

```powershell
Get-Service vfpext

# This should indicate the extension is Running: True 
Get-VMSwitchExtension  -VMSwitchName <vSwitch Name> -Name "Microsoft Azure VFP Switch Extension"
```

## <a name="tips--insights"></a>Sugerencias e información útil
A continuación hay una lista de sugerencias e información útil inspiradas por preguntas frecuentes sobre redes de contenedores de Windows que recibimos de la comunidad...

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>HNS requiere que IPv6 esté habilitado en los equipos host de contenedor 
Como parte de [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217), HNS requiere que IPv6 esté habilitado en los hosts del contenedor de Windows. Si te encuentras un error como el siguiente, es posible que IPv6 está deshabilitado en el equipo host.
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
Estamos trabajando en cambios en la plataforma para detectar y evitar automáticamente este problema. Actualmente, puede usarse la siguiente solución para garantizar que IPv6 está habilitado en el equipo host:

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-cehttpswwwdockercomcommunity-edition-instead-of-hns-internal-vswitch"></a>Las máquinas virtuales de Linux Moby usan el conmutador DockerNAT con Docker para Windows (un producto de [Docker CE](https://www.docker.com/community-edition)) en lugar del vSwitch interno de HNS 
Docker para Windows (el controlador de Windows para el motor de Docker CE) en Windows 10 usará un vSwitch interno llamado 'DockerNAT' para conectar máquinas virtuales de Linux Moby al host de contenedor. Los desarrolladores que usen máquinas virtuales de Linux Moby en Windows tener en cuenta que los hosts utilizan el vSwitch DockerNAT en lugar del vSwitch creado por el servicio HNS (que es el conmutador predeterminado usado para los contenedores de Windows). 

#### <a name="to-use-dhcp-for-ip-assignment-on-a-virtual-container-host-enable-macaddressspoofing"></a>Para usar DHCP para la asignación de direcciones IP en un host de contenedor virtual, habilita MACAddressSpoofing 
Si el host de contenedor está virtualizado y quieres usar DHCP para la asignación de direcciones IP, debes habilitar MACAddressSpoofing en el adaptador de red de las máquinas virtuales. En caso contrario, el host de Hyper-V bloqueará el tráfico de red procedente de los contenedores que estén en la máquina virtual y posean varias direcciones MAC. Puedes habilitar MACAddressSpoofing con este comando de PowerShell:
```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
#### <a name="creating-multiple-transparent-networks-on-a-single-container-host"></a>Creación de varias redes transparentes en un solo host de contenedor
Si quieres crear más de una red transparente, debes especificar con qué adaptador de red (virtual) se debe enlazar el conmutador virtual externo de Hyper-V. Para especificar la interfaz de una red, usa la sintaxis siguiente:
```
# General syntax:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface=<INTERFACE NAME> <NETWORK NAME> 

# Example:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" myTransparent2
```

#### <a name="remember-to-specify---subnet-and---gateway-when-using-static-ip-assignment"></a>Recuerda especificar *--subnet* y *--gateway* al usar la asignación de IP estática
Cuando se usa la asignación de IP estática, es necesario asegurarse antes de que se especifican los parámetros *--subnet* y *--gateway* al crear la red. La dirección IP de la puerta de enlace y la subred debe ser la misma que la configuración de red del host de contenedor; es decir, la red física. Por ejemplo, así se puede crear una red transparente y luego ejecutar un punto de conexión en dicha red usando la asignación de IP estática:

```
# Example: Create a transparent network using static IP assignment
# A network create command for a transparent container network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 MyTransparentNet
# Run a container attached to MyTransparentNet
C:\> docker run -it --network=MyTransparentNet --ip=10.123.174.105 windowsservercore cmd
```

#### <a name="dhcp-ip-assignment-not-supported-with-l2bridge-networks"></a>La asignación de IP de DHCP no se admite con las redes L2Bridge
Con las redes de contenedor creadas usando el controlador l2bridge solo se admite la asignación de IP estática. Como se indica anteriormente, recuerda que tienes que usar los parámetros *--subnet* y *--gateway* para crear una red que esté configurada para la asignación de IP estática.

#### <a name="networks-that-leverage-external-vswitch-must-each-have-their-own-network-adapter"></a>Las redes que aprovechan un vSwitch externo deben tener su propio adaptador de red
Ten en cuenta que si se crean varias redes que usan un vSwitch externo para la conectividad (por ejemplo, Transparent, L2 Bridge, L2 Transparent) en el mismo host de contenedor, cada una de ellas requiere su propio adaptador de red. 

#### <a name="ip-assignment-on-stopped-vs-running-containers"></a>Asignación de IP en contenedores detenidos y contenedores en ejecución
La asignación de dirección IP estática se lleva a cabo directamente en el adaptador de red del contenedor y solo debe realizarse cuando el contenedor se encuentre en estado detenido. En Windows Server 2016 no se admiten ni el "agregado en caliente" de los adaptadores de red de contenedor ni los cambios en la pila de red mientras se esté ejecutando el contenedor.
> Nota: Este comportamiento cambiará en Windows 10 Creators Update, ya que la plataforma ahora sí admite el "agregado en caliente". Esta funcionalidad activará E2E después que esta [solicitud de extracción de Docker pendiente](https://github.com/docker/libnetwork/pull/1661) se combine

#### <a name="existing-vswitch-not-visible-to-docker-can-block-transparent-network-creation"></a>Un VSwitch existente (que no sea visible para Docker) puede bloquear la creación de una red transparente
Si se produce un error al crear una red transparente, es posible que haya un vSwitch externo en el sistema que Docker no haya detectado de forma automática y, por tanto, que impida que la red transparente se enlace con el adaptador de red externo del host de contenedor. 

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


## <a name="unsupported-features-and-network-options"></a>Características y opciones de red no compatibles 

Las siguientes opciones de red no se admiten en Windows y no pueden pasarse a ``docker run``:
 * Vinculación de contenedor (por ejemplo, ``--link``): _La alternativa es depender de la detección de servicios._
 * Direcciones IPv6 (por ejemplo, ``--ip6``)
 * Opciones DNS (por ejemplo, ``--dns-option``)
 * Dominios de búsqueda de DNS múltiples (por ejemplo, ``--dns-search``)
 
Las siguientes características y opciones de red no se admiten en Windows y no pueden pasarse a ``docker network create``:
 * --aux-address
 * --internal
 * --ip-range
 * --ipam-driver
 * --ipam-opt
 * --ipv6 

Las siguientes opciones de red no se admiten en los servicios de Docker
* Cifrado del plano de datos (por ejemplo, ``--opt encrypted``) 


## <a name="windows-server-2016-work-arounds"></a>Soluciones para Windows Server 2016 

Aunque seguimos agregando nuevas características y desarrollo de controladores, algunas de estas características no se incorporarán respectivamente a las plataformas anteriores. En su lugar, la mejor solución es "subirse al tren" y obtener las actualizaciones más recientes para Windows 10 y Windows Server.  La sección siguiente indica algunas soluciones e inconvenientes referentes a Windows Server 2016 y a versiones anteriores de Windows 10 (es decir, antes de la versión 1704, Creators Update)

### <a name="multiple-nat-networks-on-ws2016-container-host"></a>Varias redes NAT en un host de contenedor WS2016

Las particiones de las nuevas redes NAT deben crearse en el prefijo de red NAT interno más grande. El prefijo puede encontrarse ejecutando el siguiente comando de PowerShell y al hacer referencia al campo "InternalIPInterfaceAddressPrefix".

```none
PS C:\> Get-NetNAT
```

Por ejemplo, el prefijo interno de red NAT del host podría ser 172.16.0.0/16. En este caso, se puede usar Docker para crear redes NAT adicionales *siempre que sean un subconjunto del prefijo 172.16.0.0/16.* Por ejemplo, se podrían crear dos redes NAT con los prefijos IP 172.16.1.0/24 (puerta de enlace, 172.16.1.1) y 172.16.2.0/24 (puerta de enlace, 172.16.2.1).

```none
C:\> docker network create -d nat --subnet=172.16.1.0/24 --gateway=172.16.1.1 CustomNat1
C:\> docker network create -d nat --subnet=172.16.2.0/24 --gateway=172.16.1.1 CustomNat2
```

Las redes recién creadas se pueden mostrar mediante:
```none
C:\> docker network ls
```

### <a name="docker-compose"></a>Docker Compose

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
      - subnet: 172.16.3.0/24
```

Para obtener más información sobre la definición y configuración de redes de contenedor mediante Docker Compose, consulte la [referencia del archivo de Compose](https://docs.docker.com/compose/compose-file/).

### <a name="service-discovery"></a>Detección de servicios
La detección de servicios solo se admite para determinados controladores de red de Windows.

|  | Detección de servicios locales  | Detección de servicios globales |
| :---: | :---------------     |  :---                |
| nat | SÍ | N/A |  
| overlay | SÍ | SÍ |
| transparent | NO | NO |
| l2bridge | NO | NO |


