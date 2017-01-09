---
title: "Virtualización anidada"
description: "Virtualización anidada"
keywords: Windows 10, Hyper-V
author: theodthompson
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: b0903a3e00b92bc60e26282dde072b66030ada09

---

# Ejecución de Hyper-V en una máquina virtual con la virtualización anidada

La virtualización anidada es una característica que le permite ejecutar Hyper-V dentro de una máquina virtual de Hyper-V. En otras palabras: un host de Hyper-V puede virtualizarse con la virtualización anidada. Algunos casos de uso de virtualización anidada serían la ejecución de un contenedor de Hyper-V en un host de contenedor virtualizado, la configuración de un laboratorio de Hyper-V en un entorno virtualizado o la prueba de los escenarios de varios equipos sin tener hardware individual. En este documento se detallan los requisitos previos de hardware y software, los pasos de configuración y las limitaciones. 

## Requisitos previos

- Un host de Hyper-V que ejecuta Windows Server 2016 o la Actualización de aniversario de Windows 10.
- Una VM de Hyper-V que ejecuta Windows Server 2016 o la Actualización de aniversario de Windows 10.
- Una VM de Hyper-V con la versión de configuración 8.0 o posterior.
- Un procesador Intel con tecnología VT-x y EPT.

## Configurar la virtualización anidada

1. Cree una máquina virtual. Vea los requisitos previos anteriores para las versiones requeridas de máquina virtual y sistema operativo.
2. Mientras la máquina virtual esté con el estado desactivado, ejecute el siguiente comando en el host físico de Hyper-V. Esto permite la virtualización anidada de la máquina virtual.

```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. Inicie la máquina virtual.
4. Instale Hyper-V en la máquina virtual, como lo haría en un servidor físico. Para obtener más información sobre la instalación de Hyper-V, consulte [Instalar Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install).

## Deshabilitar la virtualización anidada
Puede deshabilitar la virtualización anidada para una máquina virtual detenida mediante el siguiente comando de PowerShell:
```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## Cambio de tamaño de memoria en tiempo de ejecución y memoria dinámica
Cuando Hyper-V se está ejecutando en una máquina virtual, esta debe desactivarse para ajustar su memoria. Esto significa que, aunque la memoria dinámica esté habilitada, la cantidad de memoria no fluctuará. Para las máquinas virtuales sin memoria dinámica habilitada, cualquier intento que se produzca para ajustar la cantidad de memoria mientras esté activada provocará un error. 

Tenga en cuenta que habilitar simplemente la virtualización anidada no tendrá ningún efecto en la memoria dinámica ni en el cambio de tamaño de la memoria en tiempo de ejecución. La incompatibilidad solo se produce mientras Hyper-V se está ejecutando en la VM.

## Opciones de red
Hay dos opciones para las redes con las máquinas virtuales anidadas: suplantación de direcciones MAC y modo NAT.

### Suplantación de direcciones MAC
Para que los paquetes de red se enruten a través de dos conmutadores virtuales, debe habilitarse la suplantación de direcciones MAC en el primer nivel de conmutador virtual. Esto se completa con el siguiente comando de PowerShell.

```none
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```
### Traducción de direcciones de red
La segunda opción se basa en la traducción de direcciones de red (NAT). Este enfoque es más adecuado para aquellos casos en donde no es posible la suplantación de direcciones MAC, como en un entorno de nube pública.

En primer lugar, debe crearse un conmutador virtual de NAT en la máquina virtual host (la máquina virtual "central"). Tenga en cuenta que las direcciones IP son solo un ejemplo y varían en los distintos entornos:
```none
new-vmswitch -name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```
Después, asigne una dirección IP al adaptador de red:
```none
get-netadapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```
Cada máquina virtual anidada debe tener una dirección IP y una puerta de enlace asignadas. Tenga en cuenta que la dirección IP de la puerta de enlace debe apuntar al adaptador de NAT en el paso anterior. También puede asignar un servidor DNS:
```none
get-netadapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## Aplicaciones de virtualización de terceros
Las aplicaciones de virtualización que no sean Hyper-V no se admiten en las máquinas virtuales de Hyper-V y suelen producir errores. Esto incluye cualquier software que requiera extensiones de virtualización de hardware.



<!--HONumber=Oct16_HO4-->


