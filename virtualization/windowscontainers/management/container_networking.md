# Red de contenedores

**Esto es contenido preliminar y está sujeto a cambios.**

Los contenedores de Windows funcionan de forma similar a las máquinas virtuales en lo que respecta a las redes. Cada contenedor tiene un adaptador de red virtual, que está conectado a un conmutador virtual, a través del cual se reenvía el tráfico entrante y saliente. Existen dos tipos de configuración de red.

- **Modo de traducción de direcciones de red**: cada contenedor está conectado a un conmutador virtual interno y recibirá una dirección IP interna. Una configuración de NAT traducirá esta dirección interna para la dirección externa del host del contenedor.

- **Modo transparente**: cada contenedor está conectado a un conmutador virtual externo y recibirá una dirección IP de un servidor DHCP.

En este documento encontrará información detallada de los beneficios y la configuración de cada uno de estos tipos de configuración.

## Modo de red NAT

**Traducción de direcciones de red**: esta configuración se compone de un conmutador de red interno con un tipo de NAT, y WinNat. En esta configuración, el host de contenedor tiene una dirección IP "externa" con la que se puede establecer comunicación en una red. Todos los contenedores tienen asignadas direcciones "internas" no accesibles desde una red. Para que un contenedor sea accesible en esta configuración, se asigna un puerto externo del host a un puerto interno del contenedor. Estas asignaciones se almacenan en una tabla de asignaciones de puertos NAT. El contenedor es accesible a través de la dirección IP y del puerto externo del host, que reenvía el tráfico a la dirección IP interna y al puerto del contenedor. La ventaja de NAT es que el host del contenedor puede escalar a cientos de contenedores, mientras se usa solo una dirección IP disponible externamente. Además, NAT permite que varios contenedores hospeden aplicaciones que requieran puertos de comunicación idénticos.

### Configuración del host

Para configurar el host del contenedor para la traducción de direcciones de red, siga estos pasos.

Cree un conmutador virtual con un tipo de "NAT" y configúrelo con una subred interna. Para más información sobre el comando **New-VMSwitch**, consulte la [referencia New-VMSwitch](https://technet.microsoft.com/en-us/library/hh848455.aspx).

```powershell
New-VMSwitch -Name "NAT" -SwitchType NAT -NATSubnetAddress "172.16.0.0/12"
```
Cree el objeto de traducción de direcciones de red. Este objeto es responsable de la traducción de direcciones NAT. Para más información sobre el comando **New-NetNat**, consulte la [referencia New-NetNat](https://technet.microsoft.com/en-us/library/dn283361.aspx).

```powershell
New-NetNat -Name NAT -InternalIPInterfaceAddressPrefix "172.16.0.0/12" 
```

### Configuración del contenedor

Cuando crea un contenedor de Windows, puede seleccionar un conmutador virtual para el contenedor. Cuando el contenedor está conectado a un conmutador virtual configurado para usar NAT, el contenedor recibirá una dirección traducida.

En este ejemplo se crea un contenedor conectado a un conmutador virtual de NAT habilitado.

```powershell
New-Container -Name DemoNAT -ContainerImageName WindowsServerCore -SwitchName "NAT"
```

Cuando el contenedor se inicia, la dirección IP se puede ver desde dentro del contenedor.

```powershell
[DemoNAT]: PS C:\> ipconfig

Windows IP Configuration
Ethernet adapter vEthernet (Virtual Switch-527ED2FB-D56D-4852-AD7B-E83732A032F5-0):
   Connection-specific DNS Suffix  . : contoso.com
   Link-local IPv6 Address . . . . . : fe80::384e:a23d:3c4b:a227%16
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.240.0.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

Para más información sobre cómo iniciar un contenedor de Windows y conectarse a él, consulte [Administración de contenedores](./manage_containers.md).

### Asignación de puertos

Para tener acceso a las aplicaciones dentro de un contenedor "habilitado para NAT", las asignaciones de puertos deben crearse entre el contenedor y el host del contenedor. Para crear la asignación, necesita la dirección IP del contenedor, el puerto del contenedor "interno" y el puerto del host "externo".

En este ejemplo se crea una asignación entre el puerto **80** del host y el puerto **80** de un contenedor con una dirección IP de **172.16.0.2**.

```powershell
Add-NetNatStaticMapping -NatName "Nat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
```

En este ejemplo se crea una asignación entre el puerto **82** del host del contenedor y el puerto **80** de un contenedor con una dirección IP de **172.16.0.3**.

```powershell
Add-NetNatStaticMapping -NatName "Nat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.3 -InternalPort 80 -ExternalPort 82
```
>Una regla de firewall correspondiente será necesaria para cada puerto externo. Puede crearse con el comando `New-NetFirewallRule`. Para más información, consulte la [referencia New-NetFirewallRule](https://technet.microsoft.com/en-us/library/jj554908.aspx).

Una vez creada la asignación de puertos, se puede obtener acceso a una aplicación de contenedores a través de la dirección IP del host del contenedor (física o virtual) y del puerto externo expuesto. Por ejemplo, el diagrama siguiente representa una configuración de NAT con una solicitud dirigida al puerto externo **82** del host del contenedor. Según la asignación de puertos, esta solicitud devolvería la aplicación hospedada en el contenedor 2.

![](./media/nat1.png)

Una vista de la solicitud de un explorador de Internet.

![](./media/portmapping.png)

## Modo de red transparente

**Red transparente**: esta configuración se compone de un conmutador de red externo. En esta configuración cada contenedor recibe una dirección IP de un servidor DHCP y es accesible desde esta dirección IP. Aquí la ventaja es que no se mantiene una tabla de asignaciones de puertos.

### Configuración del host

Para configurar el sistema de contenedores para que los contenedores puedan recibir una dirección IP de un servidor DHCP, cree un conmutador virtual que esté conectado a un adaptador de red física o virtual.

En el ejemplo siguiente se crea un conmutador virtual con el nombre de DHCP, mediante un adaptador de red denominado Ethernet.

```powershell
New-VMSwitch -Name DHCP -NetAdapterName Ethernet
```

Si el host del contenedor es una máquina virtual, debe habilitar MacAddressSpoofing en el adaptador de red utilizado con el conmutador del contenedor. En el ejemplo siguiente se realiza esta operación en una máquina virtual denominada `DemoVm`.

```powershell
Get-VMNetworkAdapter -VMName DemoVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
Ahora se puede conectar el conmutador virtual externo a un contenedor, que es capaz de recibir una dirección IP de un servidor DHCP. En esta configuración, las aplicaciones hospedadas dentro del contenedor serán accesibles en la dirección IP asignada al contenedor.

## Configuración de Docker

Al iniciar el demonio de Docker, puede seleccionarse un puente de red. Cuando ejecute Docker en Windows, este es el conmutador virtual externo o NAT. En el ejemplo siguiente se inicia el demonio de Docker, que especifica un conmutador virtual denominado `Virtual Switch`.

```powershell
Docker daemon –D –b “Virtual Switch” -H 0.0.0.0:2375
```

Si implementó el host del contenedor y Docker con los scripts que se proporcionan en el Inicio rápido del contenedor de Windows, se crea un conmutador virtual interno con un tipo de NAT y se crea un servicio Docker que se preconfigura para usar este conmutador. Para cambiar el conmutador virtual que está usando el servicio Docker, debe detener el servicio Docker, modificar un archivo de configuración y volver a iniciar el servicio.

Para detener el servicio, use el comando de PowerShell siguiente.

```powershell
Stop-Service docker
```

El archivo de configuración puede encontrarse en 'c:\programdata\docker\runDockerDaemon.cmd'. Edite la siguiente línea reemplazando `Virtual Switch` por el nombre del conmutador virtual que va a usar el servicio Docker.

```powershell
docker daemon -D -b “New Switch Name"
```
Por último, inicie el servicio.

```powershell
Start-Service docker
```

## Administración de adaptadores de red

Independientemente de la configuración de red (NAT o transparente), hay varios comandos de PowerShell disponibles para administrar el adaptador de red del contenedor y las conexiones del conmutador virtual.

Administrar un adaptador de red de contenedores

- Add-ContainerNetworkAdapter: agrega un adaptador de red a un contenedor.
- Set-ContainerNetworkAdapter: modifica un adaptador de red de contenedores.
- Remove-ContainerNetworkAdapter: elimina un adaptador de red de contenedores.
- Get-ContainerNetworkAdapter: devuelve datos acerca de un adaptador de red de contenedores.

Administre la conexión entre un adaptador de red de contenedores y un conmutador virtual.

- Connect-ContainerNetworkAdapter: conecta un contenedor a un conmutador virtual.
- Disconect-ContainerNetworkAdapter: desconecta un contenedor de un conmutador virtual.

Para más información sobre cada uno de estos comandos, consulte la [referencia de PowerShell sobre contenedores](https://technet.microsoft.com/en-us/library/mt433069.aspx).




<!--HONumber=Jan16_HO1-->
