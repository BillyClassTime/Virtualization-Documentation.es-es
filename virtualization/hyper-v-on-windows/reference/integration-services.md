---
title: "Servicios de integración de Hyper-V"
description: "Referencia de los servicios de integración de Hyper-V"
keywords: "windows 10, hyper-v, servicios de integración, componentes de integración"
author: scooley
ms.date: 05/25/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 18930864-476a-40db-aa21-b03dfb4fda98
ms.openlocfilehash: c98ab9c32dfcd6e9b3a0258d0282b28d78726f6a
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/21/2017
---
# Servicios de integración de Hyper-V

Los servicios de integración (también denominados componentes de integración) son servicios que permiten que la máquina virtual se comunique con el host de Hyper-V. Muchos de estos servicios son comodidades, mientras que otros pueden ser bastante importantes para la capacidad de la máquina virtual de funcionar correctamente.

Este artículo es una referencia para cada servicio de integración disponible en Windows.  También sirve como punto de partida para cualquier información relacionada con servicios de integración específicos o su historial.

**Guías de usuario:**  
* [Administración de los servicios de integración](https://technet.microsoft.com/windows-server-docs/compute/hyper-v/manage/manage-Hyper-V-integration-services)


## Referencia rápida

| Nombre | Nombre del servicio de Windows | Nombre del demonio de Linux |  Descripción | Efecto en la máquina virtual cuando se deshabilita |
|:---------|:---------|:---------|:---------|:---------|
| [Servicio de latido de Hyper-V](#hyper-v-heartbeat-service) |  vmicheartbeat | hv_utils | Informa de que la máquina virtual se está ejecutando correctamente. | Variable |
| [Servicio de cierre de invitado de Hyper-V](#hyper-v-guest-shutdown-service) | vmicshutdown | hv_utils |  Permite al host activar el apagado de máquinas virtuales. | **Alto** |
| [Servicio de sincronización de hora de Hyper-V](#hyper-v-time-synchronization-service) | vmictimesync | hv_utils | Sincroniza el reloj de la máquina virtual con el reloj del equipo host. | **Alto** |
| [Servicio de intercambio de datos de Hyper-V (KVP)](#hyper-v-data-exchange-service-kvp) | vmickvpexchange | hv_kvp_daemon | Proporciona una forma de intercambiar metadatos básicos entre la máquina virtual y el host. | Mediana |
| [Solicitante de instantáneas de volumen de Hyper-V](#hyper-v-volume-shadow-copy-requestor) | vmicvss | hv_vss_daemon | Permite al Servicio de instantáneas de volumen hacer una copia de seguridad de la máquina virtual sin apagarla. | Variable |
| [Interfaz de servicio de invitado de Hyper-V](#hyper-v-powershell-direct-service) | vmicguestinterface | hv_fcopy_daemon | Proporciona una interfaz para que el host de Hyper-V copie archivos en o desde la máquina virtual. | Bajo |
| [Servicio directo de PowerShell de Hyper-V](#hyper-v-powershell-direct-service) | vmicvmsession | no está disponible | Proporciona una forma de administrar la máquina virtual con PowerShell sin una conexión de red. | Bajo |  


## Servicio de latido de Hyper-V

**Nombre del servicio de Windows:** vmicheartbeat  
**Nombre del demonio de Linux:** hv_utils  
**Descripción:** indica al host de Hyper-V que la máquina virtual tiene un sistema operativo instalado y que se inició correctamente.  
**Incluido en:** Windows Server 2012, Windows 8  
**Efecto:** cuando está deshabilitado, la máquina virtual no puede informar de que el sistema operativo en la máquina virtual está funcionando correctamente.  Esto puede afectar a algunos tipos de supervisión y diagnósticos del lado host.  

El servicio de latido permite responder a cuestiones básicas como "¿Se inició la máquina virtual?".  

Cuando Hyper-V informa de que el estado de una máquina virtual es "Running" (vea el siguiente ejemplo), significa que Hyper-V tiene reservados recursos para una máquina virtual, y no que hay un sistema operativo instalado o en funcionamiento.  Ahí es donde el servicio de latido resulta útil.  El servicio de latido indica a Hyper-V que el sistema operativo de la máquina virtual se ha iniciado.  

### Comprobar el latido con PowerShell

Ejecute [Get-VM](https://technet.microsoft.com/en-us/library/hh848479.aspx) como administrador para ver el latido de una máquina virtual:
``` PowerShell
Get-VM -VMName $VMName | select Name, State, Status
```

El resultado debe ser similar al siguiente:
```
Name    State    Status
----    -----    ------
DemoVM  Running  Operating normally
```

El campo `Status` viene determinado por el servicio de latido.



## Servicio de cierre de invitado de Hyper-V

**Nombre del servicio de Windows:** vmicshutdown  
**Nombre del demonio de Linux:** hv_utils  
**Descripción:** permite al host de Hyper-V solicitar el apagado de una máquina virtual.  El host siempre puede forzar la desconexión de la máquina virtual, pero sería apagarlo con el interruptor, en vez de seleccionar el apagado.  
**Incluido en:** Windows Server 2012, Windows 8  
**Efecto:** **Alto** cuando está deshabilitado, el host no puede activar un apagado en condiciones dentro de la máquina virtual.  Todos los apagados se producirán como desconexiones de hardware, lo que podría causar pérdidas o daños en los datos.  


## Servicio de sincronización de hora de Hyper-V

**Nombre del servicio de Windows:** vmictimesync  
**Nombre del demonio de Linux:** hv_utils  
**Descripción:** sincroniza el reloj del sistema de la máquina virtual con el reloj del sistema del equipo físico.  
**Incluido en:** Windows Server 2012, Windows 8  
**Efecto:** **Alto** cuando está deshabilitado, el reloj de la máquina virtual se desviará de forma errática.  


## Servicio de intercambio de datos de Hyper-V (KVP)

**Nombre del servicio de Windows:** vmickvpexchange  
**Nombre del demonio de Linux:** hv_kvp_daemon  
**Descripción:** proporciona un mecanismo para intercambiar metadatos básicos entre la máquina virtual y el host.  
**Incluido en:** Windows Server 2012, Windows 8  
**Efecto:** cuando está deshabilitado, las máquinas virtuales que ejecutan Windows 8 o Windows Server 2012 (o anterior) no recibirán actualizaciones de los servicios de integración de Hyper-V.  La desactivación del intercambio de datos también puede afectar a algunos tipos de supervisión y diagnósticos del lado host.  

El servicio de intercambio de datos (denominado a veces KVP) comparte pequeñas cantidades de información sobre el equipo entre la máquina virtual y el host de Hyper-V, usando para ello pares de clave-valor (KVP) a través del Registro de Windows.  Este mismo mecanismo también puede servir para compartir datos personalizados entre la máquina virtual y el host.

Los pares clave-valor constan de una “clave” y un “valor”. Tanto la clave como el valor son cadenas. No se admite ningún otro tipo de dato. Cuando un par clave-valor se crea o se modifica, es visible para el invitado y el host. La información sobre el par clave-valor se transfiere a través de VMbus de Hyper-V y no precisa de ningún tipo de conexión de red entre el invitado y el host de Hyper-V. 

El servicio de intercambio de datos es una herramienta excelente para conservar la información sobre la máquina virtual; para transferir datos o compartir datos de forma interactiva, use [PowerShell Direct](#hyper-v-powershell-direct-service). 


**Guías de usuario:**  
* [Utilizar pares clave-valor para compartir información entre el host y el invitado en Hyper-V](https://technet.microsoft.com/en-us/library/dn798287.aspx).  


## Solicitante de instantáneas de volumen de Hyper-V

**Nombre del servicio de Windows:** vmicvss  
**Nombre del demonio de Linux:** hv_vss_daemon  
**Descripción:** permite al Servicio de instantáneas de volumen hacer una copia de seguridad de las aplicaciones y los datos en la máquina virtual.  
**Incluido en:** Windows Server 2012, Windows 8  
**Efecto:** cuando está deshabilitado, no se puede hacer una copia de seguridad de la máquina virtual mientras se esté ejecutando (con VSS).  

El servicio de integración Solicitante de instantáneas de volumen es necesario en el Servicio de instantáneas de volumen ([VSS](https://msdn.microsoft.com/en-us/library/aa384589.aspx)).  El Servicio de instantáneas de volumen (VSS) captura y copia imágenes para la copia de seguridad en los sistemas en ejecución, especialmente los servidores, sin degradar innecesariamente el rendimiento y la estabilidad de los servicios que estos prestan.  Este servicio de integración hace esto posible al coordinar las cargas de trabajo de la máquina virtual con el proceso de copia de seguridad del host.

Encontrará más información sobre las instantáneas de volumen [aquí](https://msdn.microsoft.com/en-us/library/dd405549.aspx).


## Interfaz de servicio de invitado de Hyper-V

**Nombre del servicio de Windows:** vmicguestinterface  
**Nombre del demonio de Linux:** hv_fcopy_daemon  
**Descripción:** proporciona una interfaz para que el host de Hyper-V copie archivos bidireccionalmente en o desde la máquina virtual.  
**Incluido en:** Windows Server 2012 R2, Windows 8.1  
**Efecto:** cuando está deshabilitado, el host no puede copiar archivos en y desde el invitado con `Copy-VMFile`.  Obtenga más información sobre el [cmdlet Copy-VMFile](https://technet.microsoft.com/library/dn464282.aspx).  

**Notas:**  
Deshabilitada de forma predeterminada.  Vea [Copiar archivos con New-PSSession y Copy-Item](../user-guide/powershell-direct.md#copy-files-with-new-pssession-and-copy-item). 


## Servicio directo de PowerShell de Hyper-V

**Nombre del servicio de Windows:** vmicvmsession  
**Nombre del demonio de Linux:** no disponible  
**Descripción:** proporciona un mecanismo para administrar la máquina virtual con PowerShell a través de una sesión de máquina virtual sin una red virtual.    
**Incluido en:** Windows Server TP3, Windows 10  
**Efecto:** cuando está deshabilitado, este servicio impide que el host pueda conectarse a la máquina virtual con PowerShell Direct.  

**Notas:**  
Originalmente, el nombre del servicio era Servicio de sesión de máquina virtual de Hyper-V.  
PowerShell Direct se encuentra en fase activa de desarrollo y únicamente está disponible en Windows 10, en Windows Server Technical Preview 3 o en hosts o invitados con versiones posteriores.

PowerShell Direct permite la administración de PowerShell dentro de una máquina virtual desde el host de Hyper-V, independientemente de la configuración de la red o la configuración de administración remota en el host de Hyper-V o la máquina virtual. Esto facilita a los administradores de Hyper-V la automatización y las tareas de administración y configuración de scripts.

[Más información sobre PowerShell Direct](../user-guide/powershell-direct.md).  

**Guías de usuario:**  
* [Ejecución de un script en una máquina virtual](../user-guide/powershell-direct.md#run-a-script-or-command-with-invoke-command)
* [Copiar archivos en y desde una máquina virtual](../user-guide/powershell-direct.md#copy-files-with-new-pssession-and-copy-item)
