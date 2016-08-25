---
title: Red de contenedores de Windows
description: "Configuración de la red para contenedores de Windows."
keywords: docker, contenedores
author: jmesser81
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
translationtype: Human Translation
ms.sourcegitcommit: 7b5cf299109a967b7e6aac839476d95c625479cd
ms.openlocfilehash: 2e26177f3e653e9102dc91070b987e28ef713bed

---

# Red de contenedores

Los contenedores de Windows funcionan de forma similar a las máquinas virtuales en lo que respecta a las redes. Cada contenedor tiene un adaptador de red virtual, que está conectado a un conmutador virtual, a través del cual se reenvía el tráfico entrante y saliente. Para aplicar el aislamiento entre contenedores del mismo host, se crea un compartimento de red para cada Windows Server y contenedor de Hyper-V en el que esté instalado el adaptador de red para el contenedor. Los contenedores de Windows Server usan una vNIC de host para conectarse al conmutador virtual. Los contenedores de Hyper-V usan una NIC de máquina virtual sintética (no expuesta a la máquina virtual de utilidad) para conectarse al conmutador virtual.

Los contenedores de Windows admiten cuatro modos o controladores de red diferentes: *nat*, *transparent*, *l2bridge* y *l2tunnel*. En función de su infraestructura de red física y de sus requisitos de red de un solo host o de varios hosts, debe elegir el modo de red que mejor se adapte a sus necesidades. 

El motor de Docker crea una red NAT de forma predeterminada cuando se ejecuta por primera vez el servicio dockerd. El prefijo IP interno predeterminado que se crea es 172.16.0.0/12. 

> Nota: Si su dirección IP del host de contenedor está en este mismo prefijo, deberá cambiar el prefijo IP interno de NAT como se indica a continuación.

Los puntos de conexión del contenedor se adjuntarán a esta red predeterminada y se les asignará una dirección IP desde el prefijo interno. Actualmente solo se admite una red NAT en Windows (aunque una [solicitud de incorporación de cambios](https://github.com/docker/docker/pull/25097) pendiente podría solucionar esta restricción). 

Pueden crearse en el mismo host de contenedor redes adicionales mediante un controlador diferente (por ejemplo, transparent, l2bridge). En la tabla siguiente se muestra cómo se proporciona conectividad de red para las conexiones internas (de contenedor a contenedor) y externas para cada modo.

- **Traducción de direcciones de red**: cada contenedor recibirá una dirección IP de un prefijo IP interno y privado (por ejemplo, 172.16.0.0/12). Se admite la asignación o el reenvío de puerto del host de contenedor a los puntos de conexión del contenedor.

- **Transparente**: cada punto de conexión de contenedor está conectado directamente a la red física. Las direcciones IP de la red física se pueden asignar de manera estática o dinámica mediante un servidor DHCP externo.

- **Puente de nivel 2**: cada punto de conexión de contenedor estará en la misma subred IP que el host de contenedor. Las direcciones IP deben asignarse estáticamente desde el mismo prefijo que el host de contenedor. Todos los puntos de conexión de contenedor del host tendrán la misma dirección MAC debido a la traducción de direcciones de nivel 2.

- **Túnel de nivel 2**:_este modo solo se debe usar en una pila en la nube de Microsoft.

> Para obtener información sobre cómo conectar puntos de conexión de contenedor a una red virtual superpuesta con la pila de redes definidas por software (SDN) de Microsoft, consulte el tema [Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network) (Asociar contenedores a una red virtual).

## Nodo único

|  | De contenedor a contenedor | De contenedor a externo |
| :---: | :---------------     |  :---                |
| nat | Conexión con puente a través del conmutador virtual de Hyper-V | Enrutado a través de WinNAT con traducciones de direcciones aplicadas | 
| transparent | Conexión con puente a través del conmutador virtual de Hyper-V | Acceso directo a la red física | 
| l2bridge | Conexión con puente a través del conmutador virtual de Hyper-V|  Acceso a la red física con traducción de direcciones MAC|  



## Múltiples nodos

|  | De contenedor a contenedor | De contenedor a externo |
| :---: | :----       | :---------- |
| nat | Debe hacer referencia al puerto y dirección IP del host de contenedor externo; enrutado a través de WinNAT con traducciones de direcciones aplicadas | Debe hacer referencia al host externo; enrutado a través de WinNAT con traducciones de direcciones aplicadas | 
| transparent | Debe hacer referencia directamente al punto de conexión IP del contenedor | Acceso directo a la red física | 
| l2bridge | Debe hacer referencia directamente al punto de conexión IP del contenedor| Acceso a la red física con traducción de direcciones MAC| 


## Creación de redes 

### Red NAT (predeterminada)

El motor de Docker de Windows crea una red "nat" predeterminada con el prefijo IP 172.16.0.0/12. Actualmente solo se permite una red NAT en un host de contenedor de Windows. Si un usuario quiere crear una red NAT con un prefijo IP específico, puede hacer una de estas dos cosas cambiando las opciones en el archivo daemon.json de configuración de docker (ubicado en C:\ProgramData\Docker\config\daemon.json; si no existe, créelo).
 1. Usar la opción _"fixed-cidr": "< prefijo IP > / Mask"_, que creará la red NAT predeterminada con el prefijo IP y la máscara especificada.
 2. Usar la opción _"bridge": "none"_, que no creará una red predeterminada. El usuario puede crear una red definida por el usuario con cualquier controlador mediante el comando *docker network create -d<driver>*.

Antes de llevar a cabo una de estas opciones de configuración, debe detener el servicio Docker y eliminar las redes NAT que existan.

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

Si el usuario agrega la opción "fixed-cidr" al archivo daemon.json, el motor de Docker creará una red NAT definida por el usuario con el prefijo IP personalizado y la máscara especificada. En cambio, si agregó la opción "bridge:none", tendrá que crear una red de forma manual.

```none
# Create a user-defined nat network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

De forma predeterminada, los puntos de conexión de contenedor se conectarán a la red NAT predeterminada. Si no se ha creado la red NAT predeterminada (porque se ha especificado "bridge:none" en el archivo daemon.json) o si se requiere acceso a una red diferente definida por el usuario, los usuarios pueden especificar el parámetro *--network* con el comando de ejecución de Docker.

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### Asignación de puertos

Para tener acceso a las aplicaciones que se ejecutan dentro de un contenedor conectado a una red NAT, deben crearse asignaciones de puertos entre el host de contenedor y el punto de conexión de contenedor. Estas asignaciones se deben especificar al crear el contenedor o mientras el contenedor se encuentra en estado detenido.

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Create a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

También se admiten las asignaciones de puertos dinámicos mediante el parámetro -p o el comando EXPOSE en un archivo Dockerfile con el parámetro -P. Se seleccionará un puerto efímero aleatorio en el host de contenedor, que se puede inspeccionar al ejecutar Docker ps.

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
 1. Solo se admite una red NAT (un prefijo IP interno) por host de contenedor.
 2. Solo se puede tener acceso a los extremos de conexión de contenedor desde el host de contenedor mediante el puerto y la dirección IP interna.

Se pueden crear redes adicionales mediante controladores diferentes. 

> Los controladores de red de Docker usan solo letras minúsculas.

### Red transparente

Para usar el modo de red transparente, cree una red de contenedor con el nombre de controlador "transparent". 

```none
C:\> docker network create -d transparent MyTransparentNetwork
```

Si el host de contenedor está virtualizado y quiere usar DHCP para la asignación de direcciones IP, debe habilitar MACAddressSpoofing en el adaptador de red de las máquinas virtuales. En caso contrario, el host de Hyper-V bloqueará el tráfico de red procedente de los contenedores en la máquina virtual con varias direcciones MAC.

```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> Si quiere crear más de una red transparente, debe especificar a qué adaptador de red (virtual) se debe enlazar el conmutador virtual externo de Hyper-V (creado automáticamente).

Para enlazar una red (asociada a través del conmutador virtual de Hyper-V) a una interfaz de red específica, use la opción *-o com.docker.network.windowsshim.interface=<Interface>*.

```none
# Create a transparent network which is attached to the "Ethernet 2" network interface
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

El valor de *com.docker.network.windowsshim.interface* es el *nombre* del adaptador de: 
```none
Get-NetAdapter
```

También se pueden asignar direcciones IP de manera estática o dinámica desde un servidor DHCP externo para los puntos de conexión de contenedor conectados a una red transparente.

Cuando use una asignación de IP estática, debe asegurarse de que se especifican los parámetros *--subnet* y *--gateway* al crear la red. La dirección IP de puerta de enlace y subred debe ser la misma que la configuración de red del host de contenedor, es decir, la red física.

```none
# Create a transparent network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 TransparentNet3
```
Especifique una dirección IP con la opción *--ip* en el comando de ejecución de Docker.

```none
C:\> docker run -it --network=TransparentNet3 --ip 10.123.174.105 <image> <cmd>
```

> Asegúrese de que esta dirección IP no está asignada a otro dispositivo de red de la red física.

Dado que los puntos de conexión de contenedor tienen acceso directo a la red física, no es necesario especificar asignaciones de puerto.

### Puente de nivel 2 

Para usar el modo de redes de puente de nivel 2, cree una red de contenedor con el nombre de controlador "l2bridge". Es necesario especificar de nuevo una subred y una puerta de enlace, correspondientes a la red física.

```none
C:\> docker network create -d l2bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyBridgeNetwork
```

Solo se admite la asignación de IP estática con redes l2bridge. 

> Cuando se usa una red l2bridge en un tejido de SDN, solo se admite la asignación de IP dinámica. Consulte el tema [Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network) (Asociar contenedores a una red virtual) para obtener más información.

## Otras operaciones y configuraciones

### Mostrar en una lista las redes disponibles

```none
# list container networks
C:\> docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
0a297065f06a        nat                 nat                 local
d42516aa0250        none                null                local
```

### Quitar una red

Use `docker network rm` para eliminar una red de contenedor.

```none
C:\> docker network rm "<network name>"
```

De este modo se limpian los conmutadores virtuales de Hyper-V que haya usado la red de contenedor, así como la traducción de direcciones de red creada (instancias de red NAT de WinNAT).

### Inspección de redes 

Para ver qué contenedores están conectados a una red específica y las direcciones IP asociadas con estos puntos de conexión del contenedor, puede ejecutar lo siguiente.

```none
C:\> docker network inspect <network name>
```

### Varias redes de contenedor

Pueden crearse varias redes de contenedor en un host de contenedor único, pero deben tenerse en cuenta las advertencias siguientes:
* Solo se puede crear una red NAT por host contenedor.
* Las redes que usen un vSwitch externo para conectividad (por ejemplo, transparente, puente de nivel 2, transparente de nivel 2) deben usar su propio adaptador de red.

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


## Advertencias y problemas comunes

### Firewall

El host de contenedor requiere la creación de reglas de firewall específicas para habilitar ICMP (Ping) y DHCP. Los contenedores de Windows Server requieren que ICMP y DHCP hagan ping entre dos contenedores del mismo host y que reciban direcciones IP asignadas dinámicamente mediante DHCP. En TP5, estas reglas se crearán mediante el script Install-ContainerHost.ps1. En versiones posteriores a TP5, estas reglas se crearán automáticamente. Todas las reglas de firewall correspondientes a las reglas de reenvío de puerto NAT se crean automáticamente y se limpian cuando se detiene el contenedor.

### Características no admitidas

En la actualidad no se admiten las siguientes características de red a través de la CLI de Docker:
 * vinculación de contenedores (por ejemplo, --link)
 * resolución de IP basada en nombres o servicios para los contenedores. _Esto se admitirá pronto con una actualización de servicios_
 * controlador de red superpuesta predeterminada

En este momento, no se admiten en Windows Docker las siguientes opciones de red:
 * --add-host
 * --dns
 * --dns-opt
 * --dns-search
 * -h, --hostname
 * --net-alias
 * --aux-address
 * --internal
 * --ip-range

 > Hay un problema conocido en Windows Server 2016 Technical Preview 5 y en las compilaciones recientes no finales de Windows Insider Preview (WIP) donde, después de la actualización a una nueva compilación da como resultado una red de contenedor duplicada (es decir, "perdida") y un conmutador virtual. Para solucionar este problema, ejecute el script siguiente.
```none
$KeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
$keys = get-childitem $KeyPath
foreach($key in $keys)
{
   if ($key.GetValue("FriendlyName") -eq 'nat')
   {
      $newKeyPath = $KeyPath+"\"+$key.PSChildName
      Remove-Item -Path $newKeyPath -Recurse
   }
}
remove-netnat -Confirm:$false
Get-ContainerNetwork | Remove-ContainerNetwork
Get-VmSwitch -Name nat | Remove-VmSwitch # Note: failure is expected
Stop-Service docker
Set-Service docker -StartupType Disabled
```
> Reinicie el host y ejecute los pasos restantes:
```none
Get-NetNat | Remove-NetNat -Confirm $false
Set-Service docker -StartupType automatic
Start-Service docker 
```



<!--HONumber=Aug16_HO4-->


