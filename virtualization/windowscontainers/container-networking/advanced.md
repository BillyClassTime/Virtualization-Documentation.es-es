---
title: Redes de contenedores de Windows
description: Redes de contenedores de Windows avanzada.
keywords: docker, contenedores
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: cf5173a98032820e1ad72e99e9b6e874dedbed83
ms.sourcegitcommit: 1715411ac2768159cd9c9f14484a1cad5e7f2a5f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 03/26/2019
ms.locfileid: "9263522"
---
# <a name="advanced-network-options-in-windows"></a>Opciones de red avanzadas en Windows
Se admiten varias opciones de controladores de red para aprovechar las características y funcionalidades específicas de Windows. 

## <a name="switch-embedded-teaming-with-docker-networks"></a>Switch Embedded Teaming con redes de Docker

> Se aplica a todos los controladores de red. 

Puedes aprovechar [Switch Embedded Teaming](https://technet.microsoft.com/en-us/windows-server-docs/networking/technologies/hyper-v-virtual-switch/rdma-and-switch-embedded-teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set) al crear redes de host de contenedor para que las use Docker si especifica varios adaptadores de red (separados por comas) con la opción `-o com.docker.network.windowsshim.interface`. 

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2", "Ethernet 3" TeamedNet
```

## <a name="set-the-vlan-id-for-a-network"></a>Establecer el ID de VLAN para una red

> Se aplica a los controladores de red transparent y l2bridge 

Para establecer un ID de VLAN una red, usa la opción `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` para el comando `docker network create`. Por ejemplo, es posible usar el siguiente comando para crear una red transparente con un ID de VLAN de 11:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
Cuando se establece el identificador de VLAN de una red, se establece el aislamiento de VLAN para los puntos de conexión de contenedor que se adjuntarán a esa red.

> Asegúrate de que el adaptador de red de host (físico) está en modo de tronco para permitir que todo el tráfico etiquetado lo procese el vSwitch con el puerto vNIC (punto de conexión de contenedor) en el modo de acceso de la VLAN correcta.

## <a name="specify-the-name-of-a-network-to-the-hns-service"></a>Especificar el nombre de una red con el servicio SNP

> Se aplica a todos los controladores de red. 

Normalmente, cuando se crea un contenedor de red mediante `docker network create`, el servicio de Docker usa el nombre de red que tú proporcionas, pero el servicio SNP no lo hace. Si vas a crear una red, puedes especificar el nombre que proporciona el servicio SNP con la opción `-o com.docker.network.windowsshim.networkname=<network name>` en el comando `docker network create`. Por ejemplo, podrías usar el siguiente comando para crear una red transparente con un nombre especificado para el servicio SNP:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

## <a name="bind-a-network-to-a-specific-network-interface"></a>Enlazar una red con una interfaz de red específica

> Se aplica a todos los controladores de red excepto 'nat'  

Para enlazar una red (conectada a través del conmutador virtual de Hyper-V) con una interfaz de red específica, usa la opción `-o com.docker.network.windowsshim.interface=<Interface>` para el comando `docker network create`. Por ejemplo, podrías usar el siguiente comando para crear una red transparente que se adjunte a la interfaz de red "Ethernet 2":

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> Nota: El valor de *com.docker.network.windowsshim.interface* es el adaptador de red *Name*, que se puede encontrar de esta manera:

```
PS C:\> Get-NetAdapter
```

## <a name="specify-the-dns-suffix-andor-the-dns-servers-of-a-network"></a>Especificar el sufijo DNS y/o los servidores DNS de una red

> Se aplica a todos los controladores de red. 

Usa la opción `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` para especificar el sufijo DNS de una red y la opción `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` para especificar los servidores DNS de una red. Por ejemplo, podrías usar el siguiente comando para establecer el sufijo DNS de una red en "example.com" y los servidores DNS de una red en 4.4.4.4 y 8.8.8.8:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## <a name="vfp"></a>VFP

Consulta [este artículo](https://www.microsoft.com/en-us/research/project/azure-virtual-filtering-platform/) para obtener más información.

## <a name="tips--insights"></a>Sugerencias y detalles
A continuación hay una lista de sugerencias e información útil inspiradas por preguntas frecuentes sobre redes de contenedores de Windows que recibimos de la comunidad...

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>SNP requiere que IPv6 esté habilitado en los equipos host de contenedor 
Como parte de [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217), SNP requiere que IPv6 esté habilitado en los hosts del contenedor de Windows. Si te encuentras un error como el siguiente, es posible que IPv6 está deshabilitado en el equipo host.
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
Estamos trabajando en cambios en la plataforma para detectar y evitar automáticamente este problema. Actualmente, puede usarse la siguiente solución alternativa para garantizar que IPv6 está habilitado en el equipo host:

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```


#### <a name="linux-containers-on-windows"></a>Contenedores de Linux en Windows

**NUEVO:** estamos trabajando para que sea posible ejecutar los contenedores de Windows y Linux paralelamente _sin la VM Linux de Moby_. Consulta esta [entrada de blog sobre contenedores Linux en Windows (LCOW)](https://blog.docker.com/2017/11/docker-for-windows-17-11/) para obtener más información. Aquí es cómo [empezar a trabajar](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10-linux).
> NOTA: LCOW está haciendo que la VM Linux de Moby esté quedando obsoleta y utilizará el vSwitch interno predeterminado SNP "nat".

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-cehttpswwwdockercomcommunity-edition"></a>Las VM Linux de Moby usan el conmutador DockerNAT con Docker para Windows (un producto de [Docker CE](https://www.docker.com/community-edition))

Docker para Windows (el controlador de Windows para el motor CE de Docker) en Windows 10 usará un vSwitch interno llamado 'DockerNAT' para conectar VM Linux de Moby al host del contenedor. Los desarrolladores que usen VM Linux de Moby en Windows deben tener en cuenta que sus hosts usan el vSwitch DockerNAT en lugar del vSwitch "nat" creado por el servicio SNP (que es el conmutador predeterminado usado para los contenedores de Windows).



#### <a name="to-use-dhcp-for-ip-assignment-on-a-virtual-container-host-enable-macaddressspoofing"></a>Para usar DHCP para la asignación de direcciones IP en un host de contenedor virtual, habilita MACAddressSpoofing

Si el host de contenedor está virtualizado y quieres usar DHCP para la asignación de direcciones IP, debes habilitar MACAddressSpoofing en el adaptador de red de las máquinas virtuales. En caso contrario, el host de Hyper-V bloqueará el tráfico de red procedente de los contenedores que estén en la máquina virtual y posean varias direcciones MAC. Puedes habilitar MACAddressSpoofing con este comando de PowerShell:
```
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
Si estás ejecutando VMware como el hipervisor, tendrás que habilitar el modo promiscuo para que funcione. Puedes encontrar más información [aquí](https://kb.vmware.com/s/article/1004099)


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
La asignación de dirección IP estática se lleva a cabo directamente en el adaptador de red del contenedor y solo debe realizarse cuando el contenedor se encuentre en estado detenido. En Windows Server 2016 no se admiten ni el "agregado en caliente" de los adaptadores de redes de contenedor ni los cambios en la pila de red mientras se esté ejecutando el contenedor.
> Nota: Este comportamiento cambiará en Windows 10 Creators Update, ya que la plataforma ahora sí admite el "agregado en caliente". Esta funcionalidad activará E2E después que esta [solicitud de extracción de Docker pendiente](https://github.com/docker/libnetwork/pull/1661) se combine

#### <a name="existing-vswitch-not-visible-to-docker-can-block-transparent-network-creation"></a>Un VSwitch existente (que no sea visible para Docker) puede bloquear la creación de una red transparente
Si se produce un error al crear una red transparente, es posible que haya un vSwitch externo en el sistema que Docker no haya detectado de forma automática y, por tanto, que impida que la red transparente se enlace con el adaptador de red externo del host de contenedor. 

Al crear una red transparente, Docker crea un vSwitch externo para la red y después intenta enlazar el conmutador a un adaptador de red (externo); el adaptador podría ser un adaptador de red de máquina virtual o el adaptador de red físico. Si ya ha creado un vSwitch en el host de contenedor *y está visible para Docker*, el motor de Docker de Windows usará ese conmutador en lugar de crear uno nuevo. En cambio, si el vSwitch que se ha creado fuera de banda (es decir, se ha creado en el host de contenedor mediante el Administrador de Hyper-V o PowerShell) aún no está visible para Docker, el motor de Docker de Windows intentará crear un vSwitch nuevo y después no podrá conectar el nuevo conmutador con el adaptador de red externo del host de contenedor (porque el adaptador de red ya estará conectado al conmutador que se ha creado fuera de banda).

Por ejemplo, este problema podría aparecer si ha creado un vSwitch nuevo en el host mientras se estaba ejecutando el servicio Docker y después intenta crear una red transparente. En este caso, Docker no reconocería el conmutador que ha creado y crearía un vSwitch nuevo para la red transparente.

Hay tres formas de resolver este problema:

* Por supuesto, puede eliminar el vSwitch que se ha creado fuera de banda, lo que permitirá a Docker crear un vSwitch nuevo y conectarlo al adaptador de red de host sin ningún problema. Antes de elegir este enfoque, asegúrese de que ningún otro servicio (por ejemplo, Hyper-V) usa el vSwitch fuera de banda.
* Como alternativa, si decide usar un vSwitch externo que se ha creado fuera de banda, reinicie los servicios Docker y SNP para que *el conmutador esté visible para Docker.*
```
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* Otra opción es usar la opción "-o com.docker.network.windowsshim.interface" para enlazar el vSwitch externo de la red transparente a un adaptador de red específico que no esté en uso en el host de contenedor (es decir, un adaptador de red distinto del que usa el vSwitch que se ha creado fuera de banda). La opción '-o' se describe con más detalle más adelante, en la sección [Red transparente](https://msdn.microsoft.com/virtualization/windowscontainers/management/container_networking#transparent-network) de este documento.


## <a name="windows-server-2016-work-arounds"></a>Soluciones para Windows Server 2016 

Aunque seguimos agregando nuevas características y desarrollo de controladores, algunas de estas características no se incorporarán respectivamente a las plataformas anteriores. En su lugar, la mejor solución es "subirse al tren" y obtener las actualizaciones más recientes para Windows 10 y Windows Server.  La sección siguiente indica algunas soluciones e inconvenientes referentes a Windows Server 2016 y a versiones anteriores de Windows 10 (es decir, antes de la versión 1704, Creators Update)

### <a name="multiple-nat-networks-on-ws2016-container-host"></a>Varias redes NAT en un host de contenedor WS2016

Las particiones de las nuevas redes NAT deben crearse en el prefijo de red NAT interno más grande. El prefijo puede encontrarse ejecutando el siguiente comando de PowerShell y al hacer referencia al campo "InternalIPInterfaceAddressPrefix".

```
PS C:\> Get-NetNAT
```

Por ejemplo, el prefijo interno de red NAT del host podría ser 172.16.0.0/16. En este caso, se puede usar Docker para crear redes NAT adicionales *siempre que sean un subconjunto del prefijo 172.16.0.0/16.* Por ejemplo, se podrían crear dos redes NAT con los prefijos IP 172.16.1.0/24 (puerta de enlace, 172.16.1.1) y 172.16.2.0/24 (puerta de enlace, 172.16.2.1).

```
C:\> docker network create -d nat --subnet=172.16.1.0/24 --gateway=172.16.1.1 CustomNat1
C:\> docker network create -d nat --subnet=172.16.2.0/24 --gateway=172.16.1.1 CustomNat2
```

Las redes recién creadas se pueden mostrar mediante:
```
C:\> docker network ls
```

### <a name="docker-compose"></a>Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) puede usarse para definir y configurar redes de contenedor junto con los contenedores y servicios que usarán esas redes. La clave de Compose "networks" se usa como clave de nivel superior en la definición de las redes a las que se conectarán los contenedores. Por ejemplo, la siguiente sintaxis define la red NAT preexistente creada por Docker como la red "default" (predeterminada) para todos los contenedores y servicios definidos en un archivo determinado de Compose.

```
networks:
 default:
  external:
   name: "nat"
```

De forma similar, puede usarse la sintaxis siguiente para definir una red NAT personalizada.

> Nota: La "red NAT personalizada" definida en el ejemplo siguiente se define como una partición del prefijo interno de NAT preexistente del host de contenedor. Para obtener más contexto, consulte la sección anterior "Varias redes NAT".

```
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.16.3.0/24
```

Para obtener más información sobre la definición y configuración de redes de contenedor mediante Docker Compose, consulte la [referencia de archivos de Compose](https://docs.docker.com/compose/compose-file/).