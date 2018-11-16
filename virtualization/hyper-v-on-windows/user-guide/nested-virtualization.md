---
title: Virtualización anidada
description: Virtualización anidada
keywords: Windows 10, Hyper-V
author: johncslack
ms.date: 12/18/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.openlocfilehash: 625a9b36ff782c86065ef3d9124708e5716e066f
ms.sourcegitcommit: 941a82f463684b893488cd998f79b539c506105b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/16/2018
ms.locfileid: "7012578"
---
# <a name="run-hyper-v-in-a-virtual-machine-with-nested-virtualization"></a>Ejecución de Hyper-V en una máquina virtual con la virtualización anidada

La virtualización anidada es una característica que te permite ejecutar Hyper-V dentro de una máquina virtual (VM) de Hyper-V. Esto es útil para ejecutar un emulador de teléfono de VisualStudio en una máquina virtual o probar configuraciones que normalmente requieren varios hosts.

![](./media/HyperVNesting.png)

## <a name="prerequisites"></a>Requisitos previos

* El host de Hyper-V y el invitado deben tener la Actualización de aniversario de Windows10/WindowsServer2016 o posterior.
* Versión de configuración de máquina virtual 8.0 o superior.
* Un procesador Intel con tecnología VT-x y EPT: el anidamiento es actualmente **solo para Intel**.
* Existen algunas diferencias con redes virtuales para máquinas virtuales de segundo nivel. Consulta "Redes de máquinas virtuales anidadas".


## <a name="configure-nested-virtualization"></a>Configurar la virtualización anidada

1. Cree una máquina virtual. Vea los requisitos previos anteriores para las versiones requeridas de máquina virtual y sistema operativo.
2. Mientras la máquina virtual esté con el estado desactivado, ejecute el siguiente comando en el host físico de Hyper-V. Esto permite la virtualización anidada de la máquina virtual.

```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. Inicie la máquina virtual.
4. Instale Hyper-V en la máquina virtual, como lo haría en un servidor físico. Para obtener más información sobre la instalación de Hyper-V, consulte [Instalar Hyper-V](../quick-start/enable-hyper-v.md).

## <a name="disable-nested-virtualization"></a>Deshabilitar la virtualización anidada
Puede deshabilitar la virtualización anidada para una máquina virtual detenida mediante el siguiente comando de PowerShell:
```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## <a name="dynamic-memory-and-runtime-memory-resize"></a>Cambio de tamaño de memoria en tiempo de ejecución y memoria dinámica
Cuando Hyper-V se está ejecutando en una máquina virtual, esta debe desactivarse para ajustar su memoria. Esto significa que, aunque la memoria dinámica esté habilitada, la cantidad de memoria no fluctuará. Para las máquinas virtuales sin memoria dinámica habilitada, cualquier intento que se produzca para ajustar la cantidad de memoria mientras esté activada provocará un error. 

Tenga en cuenta que habilitar simplemente la virtualización anidada no tendrá ningún efecto en la memoria dinámica ni en el cambio de tamaño de la memoria en tiempo de ejecución. La incompatibilidad solo se produce mientras Hyper-V se está ejecutando en la VM.

## <a name="networking-options"></a>Opciones de red

Hay dos opciones para las redes con las máquinas virtuales anidadas: 

1. Suplantación de direcciones MAC
2. Redes NAT

### <a name="mac-address-spoofing"></a>Suplantación de direcciones MAC
Para que los paquetes de red se enruten a través de dos conmutadores virtuales, debe habilitarse la suplantación de direcciones MAC en el primer nivel (L1) de conmutador virtual. Esto se completa con el siguiente comando de PowerShell.

``` PowerShell
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="network-address-translation-nat"></a>Traducción de direcciones de red (NAT)
La segunda opción se basa en la traducción de direcciones de red (NAT). Este enfoque es más adecuado para aquellos casos en donde no es posible la suplantación de direcciones MAC, como en un entorno de nube pública.

En primer lugar, debe crearse un conmutador virtual de NAT en la máquina virtual host (la máquina virtual "central"). Tenga en cuenta que las direcciones IP son solo un ejemplo y varían en los distintos entornos:

``` PowerShell
New-VMSwitch -Name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```

Después, asigne una dirección IP al adaptador de red:

``` PowerShell
Get-NetAdapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```

Cada máquina virtual anidada debe tener una dirección IP y una puerta de enlace asignadas. Tenga en cuenta que la dirección IP de la puerta de enlace debe apuntar al adaptador de NAT en el paso anterior. También puedes asignar un servidor DNS:

``` PowerShell
Get-NetAdapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## <a name="how-nested-virtualization-works"></a>Cómo funciona la virtualización anidada

Los procesadores modernos incluyen características de hardware que hacen que la virtualización sea más rápida y segura. Hyper-V se basa en estas extensiones de procesador para ejecutar máquinas virtuales (por ejemplo, Intel VT-x y AMD-V). Por lo general, una vez que se inicie Hyper-V, esto impide que otro software use estas funcionalidades del procesador.  Esto impide que las máquinas virtuales invitadas se ejecuten en Hyper-V.

La virtualización anidada hace que la compatibilidad de este hardware esté disponible para las máquinas virtuales invitadas.

En el diagrama siguiente se muestra Hyper-V sin anidamiento.  El hipervisor de Hyper-V toma el control completo de las funcionalidades de virtualización de hardware (flecha naranja) y no las expone al sistema operativo invitado.

![](./media/HVNoNesting.PNG)

En cambio, en el diagrama siguiente se muestra Hyper-V con la virtualización anidada habilitada. En este caso, Hyper-V expone las extensiones de virtualización de hardware a sus máquinas virtuales. Con el anidamiento habilitado, una máquina virtual invitada puede instalar su propio hipervisor y ejecutar su propia máquina virtual invitada.

![](./media/HVNesting.png)

## <a name="3rd-party-virtualization-apps"></a>Aplicaciones de virtualización de terceros

Las aplicaciones de virtualización que no sean Hyper-V no se admiten en las máquinas virtuales de Hyper-V y suelen producir errores. Esto incluye cualquier software que requiera extensiones de virtualización de hardware.
