---
title: "Virtualización anidada"
description: "Virtualización anidada"
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.sourcegitcommit: ef18b63c454b3c12a7067d3604ba142d55403226
ms.openlocfilehash: 2d1679ffe4876ddd4eefe1b457098e8797899492

---

# Ejecución de Hyper-V en una máquina virtual con la virtualización anidada

La virtualización anidada es una característica que le permite ejecutar Hyper-V dentro de una máquina virtual de Hyper-V. En otras palabras: un host de Hyper-V puede virtualizarse con la virtualización anidada. Algunos casos de uso de virtualización anidada serían la ejecución de un contenedor de Hyper-V en un host de contenedor virtualizado, la configuración de un laboratorio de Hyper-V en un entorno virtualizado o la prueba de los escenarios de varios equipos sin tener hardware individual. En este documento se detallan los requisitos previos de hardware y software, los pasos de configuración y la información para solucionar errores.

## Requisitos previos

- Un host de Hyper-V que ejecute una compilación de Windows Insider (Windows Server 2016 o Windows 10) que ejecute la compilación 10565 o una versión posterior.
- Los dos hipervisores (principal y secundario) deben ejecutar la misma versión de Windows (10565 o una posterior).
- Un procesador Intel con tecnología Intel VT-x y EPT.

## Configurar la virtualización anidada

Primero cree una máquina virtual; **no active la máquina virtual**. Para obtener más información, consulte [Crear una máquina virtual](../quick_start/walkthrough_create_vm.md).

Una vez creada la máquina virtual, ejecute el siguiente comando en el host de Hyper-V físico. Esto permite la virtualización anidada en la máquina virtual.

```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
Al ejecutar un host de Hyper-V anidado, debe deshabilitarse la memoria dinámica en la máquina virtual. Esto puede configurarse en las propiedades de la máquina virtual o usando el siguiente comando de PowerShell.
```none
Set-VMMemory -VMName <VMName> -DynamicMemoryEnabled $false
```

Cuando haya completado estos pasos, se podrá iniciar una máquina virtual y se podrá instalar Hyper-V. Para obtener más información sobre la instalación de Hyper-V, consulte [Instalar Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install).

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


## Problemas conocidos

- Los hosts con Device Guard habilitado no pueden exponer las extensiones de virtualización a los invitados.
- Las máquinas virtuales con la seguridad basada en la virtualización (VBS) habilitada no pueden tener habilitada la anidación al mismo tiempo. Primero debe deshabilitar VBS para usar la virtualización anidada.
- Cuando se habilite la virtualización anidada en una máquina virtual, las siguientes características ya no serán compatibles con esa máquina virtual.  
  * Cambio de tamaño de memoria en tiempo de ejecución y memoria dinámica
  * Puntos de comprobación
  * Una máquina virtual con Hyper-V habilitado no puede migrar en vivo.

## Preguntas más frecuentes y solución de problemas

Mi máquina virtual no se inicia, ¿qué debo hacer?

1. Asegúrese de que la memoria dinámica esté desactivada.
2. Asegúrese de que tiene un procesador compatible con Intel.
3. Ejecute [este script de PowerShell](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1) en el equipo host desde un símbolo de sistema con privilegios elevados.

## Comentarios

Informe de otros problemas a través de la aplicación de comentarios de Windows, los [foros de virtualización](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv) o a través de [GitHub](https://github.com/Microsoft/Virtualization-Documentation).



<!--HONumber=Jun16_HO3-->


