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
translationtype: Human Translation
ms.sourcegitcommit: 26a8adb426a7cf859e1a9813da2033e145ead965
ms.openlocfilehash: d17413fc572e59ec21ff513ef5de994c6716aa08

---

# Ejecución de Hyper-V en una máquina virtual con la virtualización anidada

La virtualización anidada es una característica que le permite ejecutar Hyper-V dentro de una máquina virtual de Hyper-V. En otras palabras: un host de Hyper-V puede virtualizarse con la virtualización anidada. Algunos casos de uso de virtualización anidada serían la ejecución de un contenedor de Hyper-V en un host de contenedor virtualizado, la configuración de un laboratorio de Hyper-V en un entorno virtualizado o la prueba de los escenarios de varios equipos sin tener hardware individual. En este documento se detallan los requisitos previos de hardware y software, los pasos de configuración y la información para solucionar errores. Si ejecuta Hyper-V en una vista previa de Windows Insider, compilación 14361 o posterior, consulte [Versión preliminar de la virtualización anidada para Windows Insider: compilaciones 14361 y posteriores](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting#nested-virtualization-preview-for-windows-insiders-builds-14361-).

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

##Versión preliminar de la virtualización anidada para Windows Insider: compilaciones 14361 y posteriores
Hace unos meses, anunciamos un anticipo de la virtualización anidada de Hyper-V con la compilación 10565. Nos encantó ver el entusiasmo que esta característica genera y nos alegra compartir una actualización con Windows Insider.

###Una nueva versión de la máquina virtual necesaria para la virtualización anidada
A partir de compilación 14361, versión 8.0, es necesaria para las máquinas virtuales con la virtualización anidada habilitada. Esto requerirá una actualización de la versión para las máquinas virtuales con la anidación habilitada que se crearon en hosts antiguos. 

####Actualizar la versión de la máquina virtual
Para continuar usando la virtualización anidada, debe actualizar la versión de la máquina virtual a 8.0. Esto significa que se debe quitar este estado guardado y la máquina virtual debe apagarse. El siguiente cmdlet de PowerShell actualizará la versión de la máquina virtual:
```none
Update-VMVersion -Name <VMName>
```
####Deshabilitar la virtualización anidada
Si no desea actualizar la máquina virtual, puede deshabilitar la virtualización anidada para que pueda arrancarse la máquina virtual:
```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

###Nuevo comportamiento de la versión 8.0 de la máquina virtual 
Hay varios cambios en cómo las máquinas virtuales con la anidación habilitada funcionan en esta versión preliminar:
-   La creación y aplicación de los puntos de control ahora funciona para la máquina virtual con la virtualización anidada habilitada.
-   Ahora puede guardar e iniciar las máquinas virtuales con la anidación habilitada.
-   Ahora las máquinas virtuales con la virtualización anidada habilitada se pueden ejecutar en hosts con la seguridad basada en la virtualización (incluida la protección de dispositivos y la protección de credenciales).
-   Hemos mejorado los mensajes de error sobre las limitaciones existentes.

###Limitaciones funcionales
-   La virtualización anidada está diseñada para ejecutar Hyper-V en una máquina virtual de Hyper-V. Las aplicaciones de virtualización de terceros no son compatibles y es probable que tengan errores en las máquinas virtuales de Hyper-V.
-   La memoria dinámica no es compatible con la virtualización anidada. Cuando se está ejecutando Hyper-V dentro de una máquina virtual, la máquina virtual no puede cambiar su memoria en tiempo de ejecución. 
-   El cambio del tamaño de la memoria en tiempo de ejecución no es compatible con la virtualización anidada. El cambio del tamaño de la memoria de una máquina virtual mientras Hyper-V se está ejecutando en ella, se producirá un error. 
-   La virtualización anidada solo se admite en sistemas de Intel.

###Problema conocido
Hay un problema conocido en la compilación 14361, por el que las máquinas virtuales de generación 2 no arrancarán y generarán el siguiente error:
```none
“Cannot modify property without enabling VirtualizationBasedSecurityOptOut”
```
Se puede corregir temporalmente deshabilitando la virtualización anidada o no optar por la seguridad basada en la virtualización:
```none
Set-VMSecurity -VMName <vmname> -VirtualizationBasedSecurityOptOut $true
```

###Estamos escuchando
Como siempre, continúe enviándonos comentarios con la aplicación de comentarios de Windows. Si tiene alguna pregunta, registre un problema en la página de documentación de [GitHub](https://github.com/Microsoft/Virtualization-Documentation). 



<!--HONumber=Jul16_HO1-->


