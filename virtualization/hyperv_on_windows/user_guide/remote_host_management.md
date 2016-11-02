---
title: Administrar hosts remotos de Hyper-V con el Administrador de Hyper-V
description: Administrar hosts remotos de Hyper-V con el Administrador de Hyper-V
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 2d34e98c-6134-479b-8000-3eb360b8b8a3
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 7d7aee3d87237b93887ac7303c70cf6c9a02285d

---

# Administrar hosts remotos de Hyper-V con el Administrador de Hyper-V

El Administrador de Hyper-V es una herramienta incluida de forma predeterminada para diagnosticar y administrar el host de Hyper-V local y un pequeño número de hosts remotos.  En este artículo se documentan los pasos de configuración para conectarse a los hosts de Hyper-V con el Administrador de Hyper-V en todas las configuraciones admitidas.

> El Administrador de Hyper-V está disponible en **Programas y características** como **Herramientas de administración de Hyper-V** en [cualquier sistema operativo de Windows que incluya Hyper-V](../quick_start/walkthrough_compatibility.md#OperatingSystemRequirements).  No es necesario habilitar la plataforma de Hyper-V para administrar los hosts remotos.

Para conectarse a un host de Hyper-V en el **Administrador de Hyper-V**, asegúrese de que se ha seleccionado Administrador de Hyper-V en el panel izquierdo y, luego, seleccione **Conectarse al servidor...** en el panel derecho.

![](media/HyperVManager-ConnectToHost.png)

## Combinaciones de hosts de Hyper-V compatibles con el Administrador de Hyper-V
El Administrador de Hyper-V en Windows 10 le permite administrar los siguientes hosts de Hyper-V:
* Windows 10
* Windows 8.1
* Windows 8
* Windows Server 2016: todas las ediciones y opciones de instalación, incluidas Nano Server y la versión correspondiente de Hyper-V Server
* Windows Server 2012 R2: todas las ediciones y opciones de instalación, así como la versión correspondiente de Hyper-V Server
* Windows Server 2012: todas las ediciones y opciones de instalación, así como la versión correspondiente de Hyper-V Server

El Administrador de Hyper-V en Windows 8.1 y Windows Server 2012 R2 le permite administrar:
* Windows 8.1
* Windows 8
* Windows Server 2012 R2: todas las ediciones y opciones de instalación, así como la versión correspondiente de Hyper-V Server
* Windows Server 2012: todas las ediciones y opciones de instalación, así como la versión correspondiente de Hyper-V Server

El Administrador de Hyper-V en Windows 8 y Windows Server 2012 le permite administrar:
* Windows 8
* Windows Server 2012: todas las ediciones y opciones de instalación, así como la versión correspondiente de Hyper-V Server

El Administrador de Hyper-V en Windows 7 y Windows Server 2008 R2 le permite administrar:
* Windows Server 2008 R2: todas las ediciones y opciones de instalación, así como la versión correspondiente de Hyper-V Server

El Administrador de Hyper-V en Windows Vista y Windows Server 2008 le permite administrar:
* Windows Server 2008: todas las ediciones y opciones de instalación, así como la versión correspondiente de Hyper-V Server

> **Nota**: La funcionalidad del Administrador de Hyper-V coincide con la funcionalidad disponible para la versión que se está administrando. En otras palabras, si está administrando un host remoto de Windows Server 2012 desde Windows Server 2012 R2, las nuevas características del Administrador de Hyper-V desde Windows Server 2012 R2 no estarán disponibles.

## Administrar host local ##
Para agregar un host local al Administrador de Hyper-V como un host de Hyper-V, seleccione **Equipo local** en el cuadro de diálogo **Seleccionar equipo**.

![](media/HyperVManager-ConnectToLocalHost.png)

Si no se puede establecer una conexión:
*  Asegúrese de que el rol de la plataforma Hyper-V está habilitado.  
  Consulte la [sección del tutorial para la comprobación de la compatibilidad](../quick_start/walkthrough_compatibility.md) a fin de ver si se admite Hyper-V.
*  Confirme que su cuenta de usuario forma parte del grupo Administradores de Hyper-V.


## Administrar otro host de Hyper-V en el mismo dominio ##

Para agregar un host remoto de Hyper-V al Administrador de Hyper-V, seleccione **Otro equipo** en el cuadro de diálogo **Seleccionar equipo** y escriba el nombre de host, NetBIOS o FQDN del host remoto en el campo de texto.

![](media/HyperVManager-ConnectToRemoteHost.png)

Para administrar los hosts remotos de Hyper-V, debe habilitar la administración remota en el equipo local y el host remoto.

Puede hacerlo mediante `Server Manager -> Remote management` o ejecutando el siguiente comando de PowerShell como administrador: 

``` PowerShell
Enable-PSRemoting
```

Si su cuenta de usuario actual coincide con una cuenta de administrador de Hyper-V en el host remoto, continúe y pulse **Aceptar** para conectarse.  

> Esta es la única manera admitida para administrar un host remoto en el Administrador de Hyper-V en Windows 8 o Windows 8.1.


Windows 10 amplió enormemente las combinaciones posibles de tipos de conexiones remotas.  
Ahora puede conectarse a un sistema operativo Windows 10 remoto o una versión posterior con el nombre de host o la dirección IP.  El Administrador de Hyper-V ahora también admite credenciales de usuario alternativas.  


### Conectarse al host remoto como un usuario diferente
> Solo está disponible cuando se conecte a un host remoto de Windows 10 o Windows Server 2016 Technical Preview 3 o una versión posterior.

En Windows 10, si no se ejecuta con la cuenta de usuario correcta para el host remoto, puede conectarse como otro usuario con credenciales alternativas.

Para especificar las credenciales del host remoto de Hyper-V, seleccione **Conectarse como otro usuario:** en el cuadro de diálogo **Seleccionar equipo** y luego seleccione **Establecer usuario...**.

![](media/HyperVManager-ConnectToRemoteHostAltCreds.png)


### Conectarse al host remoto mediante la dirección IP
> Solo está disponible cuando se conecte a un host remoto de Windows 10 o Windows Server 2016 Technical Preview 3 o una versión posterior.

A veces es más fácil conectarse mediante la dirección IP en lugar del nombre del host. Windows 10 le permite hacer exactamente eso.

Para conectarse mediante la dirección IP, escriba la dirección IP en el campo de texto **Otro equipo**.


## Administrar un host de Hyper-V fuera de su dominio (o sin dominio) ##
> Solo está disponible cuando se conecte a un host remoto de Windows 10 o Windows Server 2016 Technical Preview 3 o una versión posterior.

En el host de Hyper-V que se va a administrar, ejecute lo siguiente como administrador:

1.  [Enable-PSRemoting](https://technet.microsoft.com/en-us/library/hh849694.aspx)
  * [Enable-PSRemoting](https://technet.microsoft.com/en-us/library/hh849694.aspx) creará las reglas de firewall necesarias para las zonas de red *privadas*. Para permitir este acceso en zonas públicas, deberá habilitar las reglas para CredSSP y WinRM.
2.  [Enable-WSManCredSSP](https://technet.microsoft.com/en-us/library/hh849872.aspx) -Role server

En el equipo de administración, ejecute lo siguiente como administrador:

1. Set-Item WSMan:\localhost\Client\TrustedHosts -Value "fqdn-of-hyper-v-host"
2. [Enable-WSManCredSSP](https://technet.microsoft.com/en-us/library/hh849872.aspx) -Role client -DelegateComputer "fqdn-of-hyper-v-host"
3. Además puede que necesite configurar la directiva de grupo siguiente: ** Configuración del equipo | Plantillas administrativas | Sistema | Delegación de credenciales | Permitir la delegación de credenciales nuevas con autenticación solo NTLM de servidor **
    * Haga clic en **Habilitar** y agregue *wsman/fqdn-of-hyper-v-host*



<!--HONumber=Oct16_HO4-->


