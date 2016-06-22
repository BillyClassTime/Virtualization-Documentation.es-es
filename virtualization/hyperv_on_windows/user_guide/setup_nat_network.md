---
title: Configurar una red NAT
description: Configurar una red NAT
keywords: windows 10, hyper-v
author: jmesser81
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1f8a691c-ca75-42da-8ad8-a35611ad70ec
---

# Configurar una red NAT

Windows 10 Hyper-V permite la traducción de direcciones de red (NAT) nativa de una red virtual.

Esta guía le orientará a lo largo de:
* la creación de una red NAT
* la conexión de una máquina virtual existente a la nueva red
* la confirmación de que la máquina virtual está conectada correctamente

Requisitos:
* Compilación 14295 o posterior de Windows
* Que el rol de Hyper-V esté habilitado (instrucciones [aquí](../quick_start/walkthrough_create_vm.md))

> **Nota:** Actualmente, Hyper-V solo permite crear una red NAT.

## Información general sobre NAT
NAT proporciona a una máquina virtual acceso a los recursos de red con la dirección IP y un puerto del equipo host.

La traducción de direcciones de red (NAT) es un modo de red diseñado para conservar las direcciones IP mediante la asignación de una dirección IP externa y un puerto a un conjunto mucho mayor de direcciones IP internas.  Básicamente, un conmutador NAT usa una tabla de asignación NAT para enrutar el tráfico de una dirección IP y un número de puerto a la dirección IP interna correcta asociada a un dispositivo de la red (máquina virtual, equipo, contenedor, etc.).

Además, NAT permite que varias máquinas virtuales hospeden aplicaciones que necesitan puertos de comunicación idénticos (internos) asignándolos a puertos externos únicos.

Por todas estas razones, la red NAT es muy común para la tecnología de contenedores (consulte [Container Networking (Red de contenedores)](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/management/container_networking)).


## Creación de una red virtual NAT
Pasemos a la configuración de una nueva red NAT.

1.  Abra una consola de PowerShell como administrador.  

2. Creación de un conmutador interno  

  ``` PowerShell
  New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
  ```

3. Configure la puerta de enlace NAT con [New-NetIPAddress](https://technet.microsoft.com/en-us/library/hh826150.aspx).  

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

  * **InterfaceIndex**: ifIndex es el índice de interfaz del conmutador virtual creado anteriormente.

    Puede encontrar el índice de interfaz si ejecuta `Get-NetAdapter`

    El resultado debe ser similar al siguiente:

    ```
    PS C:\> Get-NetAdapter

    Name                  InterfaceDescription               ifIndex Status       MacAddress           LinkSpeed
    ----                  --------------------               ------- ------       ----------           ---------
    vEthernet (intSwitch) Hyper-V Virtual Ethernet Adapter        24 Up           00-15-5D-00-6A-01      10 Gbps
    Wi-Fi                 Marvell AVASTAR Wireless-AC Net...      18 Up           98-5F-D3-34-0C-D3     300 Mbps
    Bluetooth Network ... Bluetooth Device (Personal Area...      21 Disconnected 98-5F-D3-34-0C-D4       3 Mbps

    ```

    El conmutador interno tendrá un nombre como `vEthernet (SwitchName)` y una descripción de interfaz de `Hyper-V Virtual Ethernet Adapter`.

  Ejecute lo siguiente para crear la puerta de enlace NAT:

  ``` PowerShell
  New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
  ```

4. Configure la red NAT con [New-NetNat](https://technet.microsoft.com/en-us/library/dn283361(v=wps.630).aspx).  

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

Enhorabuena.  Ahora tiene una red virtual NAT.  Para agregar una máquina virtual a la red NAT, siga [estas instrucciones](setup_nat_network.md#connect-a-virtual-machine).

## Conectar una máquina virtual

Para conectar una máquina virtual a la nueva red NAT, conecte el conmutador interno que creó en el primer paso de la sección [Configuración de red NAT](setup_nat_network.md#create-a-nat-virtual-network) a la máquina virtual mediante el menú de configuración de máquina virtual.


## Solucionar problemas

Este flujo de trabajo asume que no hay otras redes NAT en el host. Aunque a veces varias aplicaciones o servicios exigirán el uso de una red NAT. Puesto que Windows (WinNAT) solo admite un prefijo de subred interno NAT, si intenta crear varias NAT, dejará al sistema en un estado desconocido.

### Pasos para la solución de problemas
1. Asegúrese de que solo tiene una NAT

  ``` PowerShell
  Get-NetNat
  ```
2. Si ya existe una NAT, elimínela

  ``` PowerShell
  Get-NetNat | Remove-NetNat
  ```

3. Asegúrese de que solo tiene un vmSwitch "interno" para NAT. Registre el nombre del vSwitch del paso 4

  ``` PowerShell
  Get-VMSwitch
  ```

4. Compruebe si hay direcciones IP privadas (por ejemplo, dirección IP de puerta de enlace predeterminada NAT, normalmente *.1) de la antigua NAT todavía asignadas a un adaptador

  ``` PowerShell
  Get-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)"
  ```

5. Si alguna dirección IP privada antigua está en uso, elimínela  
   ``` PowerShell
  Remove-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)" -IPAddress <IPAddress>
  ```

## Varias aplicaciones que usan la misma NAT

Algunos escenarios exigen que varias aplicaciones o servicios usen la misma NAT. En este caso, debe seguir el siguiente flujo de trabajo para que varias aplicaciones o servicios puedan usar un prefijo de subred interna NAT mayor

**_Detallaremos la coexistencia de Docker 4 Windows - Docker Beta - Linux VM con la característica de contenedor de Windows en el mismo host como ejemplo. Este flujo de trabajo está sujeto a cambios_**

1. C:\> net stop docker
2. Detenga Docker4Windows MobyLinux VM
3. PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
4. PS C:\> Get-NetNat | Remove-NetNat  
   *Quita las redes de contenedor existentes (es decir, elimina vSwitch, elimina NetNat, limpia)*  

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


## Referencias
Obtenga más información sobre [redes NAT](https://en.wikipedia.org/wiki/Network_address_translation)


<!--HONumber=May16_HO5-->


