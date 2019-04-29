---
title: Configurar una red NAT
description: Configurar una red NAT
keywords: Windows 10, Hyper-V
author: jmesser81
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1f8a691c-ca75-42da-8ad8-a35611ad70ec
ms.openlocfilehash: 0c365b9351ee09c946e1711f3a3a5e82eb71c785
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9577316"
---
# <a name="set-up-a-nat-network"></a>Configurar una red NAT

Windows 10 Hyper-V permite la traducción de direcciones de red (NAT) nativa de una red virtual.

Esta guía le orientará a lo largo de:
* la creación de una red NAT
* la conexión de una máquina virtual existente a la nueva red
* la confirmación de que la máquina virtual está conectada correctamente

Requisitos:
* Actualización de aniversario de Windows10 o posterior
* Hyper-V debe estar habilitado (instrucciones [aquí](../quick-start/enable-hyper-v.md))

> **Nota:** Actualmente, hay una limitación de una red NAT por cada host. Para obtener más detalles sobre la implementación de Windows NAT (WinNAT), sus capacidades y sus limitaciones, consulta el [blog de capacidades y limitaciones de WinNAT](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations/)

## <a name="nat-overview"></a>Información general sobre NAT
NAT proporciona a una máquina virtual acceso a los recursos de red con la dirección IP y un puerto del equipo host a través de un conmutador virtual de Hyper-V.

La traducción de direcciones de red (NAT) es un modo de red diseñado para conservar las direcciones IP mediante la asignación de una dirección IP externa y un puerto a un conjunto mucho mayor de direcciones IP internas.  Básicamente, una red NAT usa una tabla de flujo para enrutar el tráfico de una dirección IP (host) externa y un número de puerto a la dirección IP interna correcta asociada a un punto de conexión de la red (máquina virtual, equipo, contenedor, etc.).

Además, NAT permite que varias máquinas virtuales hospeden aplicaciones que necesitan puertos de comunicación idénticos (internos) asignándolos a puertos externos únicos.

Por todas estas razones, la red NAT es muy común para la tecnología de contenedores (consulte [Container Networking (Red de contenedores)](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/management/container_networking)).


## <a name="create-a-nat-virtual-network"></a>Creación de una red virtual NAT
Pasemos a la configuración de una nueva red NAT.

1.  Abra una consola de PowerShell como administrador.  

2. Crea un conmutador interno.

  ``` PowerShell
  New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
  ```

3. Busca el índice de interfaz del conmutador virtual que acabas de crear.

    Puedes encontrar el índice de interfaz si ejecutas `Get-NetAdapter`

    El resultado debe ser similar al siguiente:

    ```
    PS C:\> Get-NetAdapter

    Name                  InterfaceDescription               ifIndex Status       MacAddress           LinkSpeed
    ----                  --------------------               ------- ------       ----------           ---------
    vEthernet (intSwitch) Hyper-V Virtual Ethernet Adapter        24 Up           00-15-5D-00-6A-01      10 Gbps
    Wi-Fi                 Marvell AVASTAR Wireless-AC Net...      18 Up           98-5F-D3-34-0C-D3     300 Mbps
    Bluetooth Network ... Bluetooth Device (Personal Area...      21 Disconnected 98-5F-D3-34-0C-D4       3 Mbps

    ```

    El conmutador interno tendrá un nombre como `vEthernet (SwitchName)` y una descripción de interfaz de `Hyper-V Virtual Ethernet Adapter`. Anota su `ifIndex` para usarlo en el siguiente paso.

4. Configura la puerta de enlace NAT con [New-NetIPAddress](https://docs.microsoft.com/powershell/module/nettcpip/New-NetIPAddress).  

  Este es el comando genérico:
  ``` PowerShell
  New-NetIPAddress -IPAddress <NAT Gateway IP> -PrefixLength <NAT Subnet Prefix Length> -InterfaceIndex <ifIndex>
  ```

  Para configurar la puerta de enlace, necesitará un poco de información sobre la red:  
  * **IPAddress**: la dirección IP de puerta de enlace NAT especifica la dirección IPv4 o IPv6 que se usará como dirección IP de puerta de enlace NAT.  
    El formato genérico será a.b.c.1 (por ejemplo, 172.16.0.1).  Aunque la posición final no tiene que ser .1, normalmente lo es (según la longitud del prefijo)

    Una dirección IP de puerta de enlace común es 192.168.0.1  

  * **PrefixLength**: la longitud del prefijo de subred NAT define el tamaño de la subred local NAT (máscara de subred).
    La longitud del prefijo de subred será un valor entero entre 0 y 32.

    0 asignaría todo Internet, 32 solo permitiría una dirección IP asignada.  Los valores comunes oscilan entre 24 y 12, según el número de direcciones IP que haya que asociar a NAT.

    Un valor PrefixLength común es 24, es decir, una máscara de subred 255.255.255.0

  * **InterfaceIndex**: ifIndex es el índice de interfaz del conmutador virtual que has determinado en el paso anterior.

  Ejecuta lo siguiente para crear la puerta de enlace NAT:

  ``` PowerShell
  New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
  ```

5. Configure la red NAT con [New-NetNat](https://docs.microsoft.com/powershell/module/netnat/New-NetNat).  

  Este es el comando genérico:

  ``` PowerShell
  New-NetNat -Name <NATOutsideName> -InternalIPInterfaceAddressPrefix <NAT subnet prefix>
  ```

  Para configurar la puerta de enlace, debe proporcionar información sobre la red y la puerta de enlace NAT:  
  * **Name**: NATOutsideName describe el nombre de la red NAT.  Se usará para quitar la red NAT.

  * **InternalIPInterfaceAddressPrefix**: el prefijo de subred NAT describe el prefijo IP de puerta de enlace NAT anterior, así como la longitud del prefijo de subred NAT de arriba.

    El formato genérico será a.b.c.0/longitud de prefijo de subred NAT

    De los anteriores, vamos a usar 192.168.0.0/24 para este ejemplo

  En nuestro ejemplo, ejecute lo siguiente para configurar la red NAT:

  ``` PowerShell
  New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.0.0/24
  ```

Enhorabuena.  Ahora tiene una red virtual NAT.  Para agregar una máquina virtual a la red NAT, siga [estas instrucciones](#connect-a-virtual-machine).

## <a name="connect-a-virtual-machine"></a>Conectar una máquina virtual

Para conectar una máquina virtual a la nueva red NAT, conecte el conmutador interno que creó en el primer paso de la sección [Configuración de red NAT](#create-a-nat-virtual-network) a la máquina virtual mediante el menú de configuración de máquina virtual.

Como WinNAT por sí mismo no asigna direcciones IP a un punto de conexión (por ejemplo, una máquina virtual), debe hacerlo manualmente desde la propia máquina virtual; es decir, establecer la dirección de IP dentro del intervalo del prefijo interno de NAT, establecer la dirección IP de puerta de enlace predeterminada y establecer la información del servidor DNS. El único inconveniente es cuando el punto de conexión está asociado a un contenedor. En este caso, el servicio de red de host (HNP) asigna y utiliza el servicio de proceso de host (HCS) para asignar la dirección IP, la IP de puerta de enlace y la información de DNS directamente en el contenedor.


## <a name="configuration-example-attaching-vms-and-containers-to-a-nat-network"></a>Ejemplo de configuración: conexión de máquinas virtuales y contenedores a una red NAT

_Si necesita conectar varias máquinas virtuales y contenedores a una única red NAT, debe asegurarse de que el prefijo de subred interna NAT es lo suficientemente grande como para abarcar los intervalos IP que se asignan mediante distintas aplicaciones o servicios (por ejemplo, Docker para Windows y Windows Container - SNP). Esto requiere una asignación de nivel de aplicación de direcciones IP y de configuración de red o una configuración manual que debe realizarla un administrador y no se garantiza que se vuelvan a usar las asignaciones de IP existentes en el mismo host._

### <a name="docker-for-windows-linux-vm-and-windows-containers"></a>Docker para Windows (máquina virtual Linux) y contenedores de Windows
La siguiente solución permitirá tanto a Docker para Windows (máquina virtual Linux que ejecuta contenedores Linux) como a los contenedores Windows compartir la misma instancia de WinNAT con conmutadores virtuales internos independientes. La conectividad entre contenedores Linux y Windows funcionará.

El usuario ha conectado máquinas virtuales a una red NAT a través de un conmutador virtual interno denominado "VMNAT" y ahora quiere instalar la característica de contenedor de Windows con el motor de Docker.
```
PS C:\> Get-NetNat “VMNAT”| Remove-NetNat (this will remove the NAT but keep the internal vSwitch).
Install Windows Container Feature
DO NOT START Docker Service (daemon)
Edit the arguments passed to the docker daemon (dockerd) by adding –fixed-cidr=<container prefix> parameter. This tells docker to create a default nat network with the IP subnet <container prefix> (e.g. 192.168.1.0/24) so that HNS can allocate IPs from this prefix.
PS C:\> Start-Service Docker; Stop-Service Docker
PS C:\> Get-NetNat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> Start-Service docker
```
Docker/SNP asignará direcciones IP a los contenedores de Windows y administrador asignará direcciones IP a las máquinas virtuales del conjunto de diferencia de las dos.

El usuario ha instalado la característica de contenedor de Windows con el motor de Docker en ejecución y ahora quiere conectar máquinas virtuales a la red NAT.
```
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
PS C:\> Get-NetNat | Remove-NetNat (this will remove the NAT but keep the internal vSwitch)
Edit the arguments passed to the docker daemon (dockerd) by adding -b “none” option to the end of docker daemon (dockerd) command to tell docker not to create a default NAT network.
PS C:\> New-ContainerNetwork –name nat –Mode NAT –subnetprefix <container prefix> (create a new NAT and internal vSwitch – HNS will allocate IPs to container endpoints attached to this network from the <container prefix>)
PS C:\> Get-Netnat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> New-VirtualSwitch -Type internal (attach VMs to this new vSwitch)
PS C:\> Start-Service docker
```
Docker/SNP asignará direcciones IP a los contenedores de Windows y administrador asignará direcciones IP a las máquinas virtuales del conjunto de diferencia de las dos.

Al final, debe tener dos conmutadores internos de máquina virtual y una red NAT compartida entre ellos.

## <a name="multiple-applications-using-the-same-nat"></a>Varias aplicaciones que usan la misma NAT

Algunos escenarios exigen que varias aplicaciones o servicios usen la misma NAT. En este caso, debe seguir el siguiente flujo de trabajo para que varias aplicaciones o servicios puedan usar un prefijo de subred interna NAT mayor

**_Detallaremos la coexistencia de Docker 4 Windows - Docker Beta - Linux VM con la característica de contenedor de Windows en el mismo host como ejemplo. Este flujo de trabajo está sujeto a cambios_**

1. C:\> net stop docker
2. Stop Docker4Windows MobyLinux VM
3. PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
4. PS C:\> Get-NetNat | Remove-NetNat  
   *Quita las redes de contenedor existentes (es decir, elimina vSwitch, elimina NetNat y limpia)*  

5. New-ContainerNetwork -Name nat -Mode NAT –subnetprefix 10.0.76.0/24 (esta subred se usará para la característica de contenedores de Windows) *Crea un vSwitch interno denominado nat*  
   *Crea una red NAT denominada "nat" con un prefijo IP 10.0.76.0/24*  

6. Remove-NetNAT  
   *Quita DockerNAT y las redes NAT nat (mantiene los vSwitches internos)*  

7. New-NetNat -Name DockerNAT -InternalIPInterfaceAddressPrefix 10.0.0.0/17 (esto creará una red NAT más grande para que la compartan D4W y los contenedores)  
   *Crea una red NAT denominada DockerNAT con un prefijo mayor 10.0.0.0/17*  

8. Ejecute Docker4Windows (MobyLinux.ps1)  
   *Crea el vSwitch interno DockerNAT*  
   *Crea una red NAT denominada "DockerNAT" con un prefijo IP 10.0.75.0/24*  

9. Net start docker  
   *Docker usará la red NAT definida por el usuario como valor predeterminado para conectar contenedores de Windows*  

Al final, debería tener dos vSwitches internos: uno denominado DockerNAT y el otro nat. Si ejecuta Get-NetNat, solo tendrá una red NAT (10.0.0.0/17) confirmada. Las direcciones IP de los contenedores de Windows serán asignadas por el servicio de red de host (HNS) de Windows desde la subred 10.0.76.0/24. Según el script MobyLinux.ps1 existente, las direcciones IP de Docker 4 Windows se asignarán desde la subred 10.0.75.0/24.


## <a name="troubleshooting"></a>Solucionar problemas

### <a name="multiple-nat-networks-are-not-supported"></a>No se admiten varias redes NAT  
En esta guía se da por supuesto que no hay otras redes NAT en el host. Sin embargo, las aplicaciones o los servicios requerirán el uso de una NAT y podrían crear una como parte de la instalación. Puesto que Windows (WinNAT) solo admite un prefijo de subred interno NAT, si intenta crear varias NAT, dejará al sistema en un estado desconocido.

Para comprobar si es este el problema, asegúrese de que solo tiene una NAT:
``` PowerShell
Get-NetNat
```

Si ya existe una NAT, elimínela.
``` PowerShell
Get-NetNat | Remove-NetNat
```
Asegúrese de que solo tiene un conmutador de máquina virtual "interno" para la aplicación o característica (por ejemplo, contenedores Windows). Registro del nombre del conmutador virtual
``` PowerShell
Get-VMSwitch
```

Compruebe si hay direcciones IP privadas (por ejemplo, dirección IP de puerta de enlace predeterminada NAT, normalmente *.1) de la antigua NAT todavía asignadas a un adaptador.
``` PowerShell
Get-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)"
```

Si alguna dirección IP privada antigua está en uso, elimínela
``` PowerShell
Remove-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)" -IPAddress <IPAddress>
```

**Eliminar varias redes NAT**  
Hemos visto informes de varias redes NAT creadas inadvertidamente. Esto es debido a un error en las compilaciones recientes (incluidas las compilaciones de Windows Server 2016 Technical Preview 5 y Windows 10 Insider Preview). Si ve varias redes NAT, después de la ejecución de docker network ls o de Get-ContainerNetwork, lleve a cabo lo siguiente desde un PowerShell con privilegios elevados:

```
PS> $KeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
PS> $keys = get-childitem $KeyPath
PS> foreach($key in $keys)
PS> {
PS>    if ($key.GetValue("FriendlyName") -eq 'nat')
PS>    {
PS>       $newKeyPath = $KeyPath+"\"+$key.PSChildName
PS>       Remove-Item -Path $newKeyPath -Recurse
PS>    }
PS> }
PS> remove-netnat -Confirm:$false
PS> Get-ContainerNetwork | Remove-ContainerNetwork
PS> Get-VmSwitch -Name nat | Remove-VmSwitch (_failure is expected_)
PS> Stop-Service docker
PS> Set-Service docker -StartupType Disabled
Reboot Host
PS> Get-NetNat | Remove-NetNat
PS> Set-Service docker -StartupType automaticac
PS> Start-Service docker 
```

Consulte esta [guía de configuración para varias aplicaciones que usan la misma NAT](#multiple-applications-using-the-same-nat) para volver a generar el entorno de NAT, si es necesario. 

## <a name="references"></a>Referencias
Obtenga más información sobre [redes NAT](https://en.wikipedia.org/wiki/Network_address_translation)
