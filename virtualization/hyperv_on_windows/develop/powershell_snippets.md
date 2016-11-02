---
title: "Fragmentos de código de PowerShell"
description: "Fragmentos de código de PowerShell"
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: dc33c703-c5bc-434e-893b-0c0976b7cb88
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: ff356c8dad96b5705c6d698480c6f435cedd7bd9

---

# Fragmentos de código de PowerShell

PowerShell es una maravillosa herramienta de creación de scripts, automatización y administración para Hyper-V.  Aquí puede encontrar un cuadro de herramientas para explorar algunas de las interesantes cosas que puede hacer.

Toda la administración de Hyper-V debe ejecutarse como administrador, por lo que asuma que todos los scripts y fragmentos de código se deben ejecutar como administrador desde una cuenta de administrador de Hyper-V.

Si no sabe si tiene los permisos adecuados, escriba `Get-VM` y, si se ejecuta sin errores, todo está listo.


## Herramientas de PowerShell Direct
Todos los scripts y fragmentos de código de esta sección se basarán en los siguientes datos básicos.

**Requisitos**:  
*  PowerShell Direct.  Sistemas operativos host e invitado de Windows 10.

**Variables comunes**:  
`$VMName` una cadena con el valor de VMName.  Consulte una lista de máquinas virtuales disponibles con `Get-VM`  
`$cred` credencial para el sistema operativo invitado.  Se pueden rellenar con `$cred = Get-Credential`  

### Compruebe si se ha iniciado el invitado

El Administrador de Hyper-V no le ofrece visibilidad en el sistema operativo invitado, lo que a menudo dificulta saber si se ha iniciado el sistema operativo invitado.

A continuación se incluyen dos vistas de la misma funcionalidad, como un fragmento de código y como una función de PowerShell.

Fragmento de código:  
``` PowerShell
if((Invoke-Command -VMName $VMName -Credential $cred {"Test"}) -ne "Test"){Write-Host "Not Booted"} else {Write-Host "Booted"}
```  

Function:  
``` PowerShell
function waitForPSDirect([string]$VMName, $cred){
   Write-Output "[$($VMName)]:: Waiting for PowerShell Direct (using $($cred.username))"
   while ((Invoke-Command -VMName $VMName -Credential $cred {"Test"} -ea SilentlyContinue) -ne "Test") {Sleep -Seconds 1}}
```

**Resultado**  
Imprime un mensaje descriptivo y se bloquea hasta que la conexión con la máquina virtual sea correcta.  
Se ejecuta correctamente en modo silencioso.

### Bloqueo del script hasta que el invitado tenga una red
Con PowerShell Direct es posible conectarse a una sesión de PowerShell dentro de una máquina virtual antes de que la máquina virtual haya recibido una dirección IP.

``` PowerShell
# Wait for DHCP
while ((Get-NetIPAddress | ? AddressFamily -eq IPv4 | ? IPAddress -ne 127.0.0.1).SuffixOrigin -ne "Dhcp") {sleep -Milliseconds 10}
```

** Resultado ** Se bloquea hasta que se recibe una concesión de DHCP.  Como este script no está buscando una subred específica ni una dirección IP, funciona sin importar la configuración de red que esté utilizando.  
Se ejecuta correctamente en modo silencioso.

## Administración de credenciales con PowerShell
A menudo, los scripts de Hyper-V necesitan controlar las credenciales de una o varias máquinas virtuales, el host de Hyper-V o todos ellos.

Hay varias formas de lograr esto al trabajar con PowerShell Direct o con la comunicación remota estándar de PowerShell:

1. La forma primera (y más sencilla) es que las mismas credenciales de usuario sean válidas en el host y en el invitado, o en los hosts local y remoto.  
  Esto es bastante fácil si inicia sesión con su cuenta de Microsoft (o si se encuentra en un entorno de dominio).  
  En este escenario simplemente se puede ejecutar `Invoke-Command -VMName "test" {get-process}`.

2. Permiten que PowerShell solicite credenciales  
  Si las credenciales no coinciden, automáticamente obtendrá una petición de credenciales que le permitirá facilitar las credenciales adecuadas para la máquina virtual.

3. Almacenar las credenciales en una variable para su reutilización.
  Ejecute un comando simple similar al siguiente:  
  ``` PowerShell
  $localCred = Get-Credential
   ```
  Y después ejecute algo parecido a esto:
  ``` PowerShell
  Invoke-Command -VMName "test" -Credential $localCred  {get-process} 
  ```
  Que significará que las credenciales solo se le solicitarán una vez por cada sesión de PowerShell o script.

4. Codifique las credenciales en los scripts.  **No haga esto en ningún sistema ni carga de trabajo real**
 > Advertencia: _No haga esto en un sistema de producción.  No lo haga con contraseñas reales._
  
  Puede crear manualmente un objeto de PSCredential con código similar al siguiente:  
  ``` PowerShell
  $localCred = New-Object -typename System.Management.Automation.PSCredential -argumentlist "Administrator", (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) 
  ```
  Absolutamente inseguro, pero útil para realizar pruebas.  Ahora no obtendrá ningún mensaje en esta sesión. 




<!--HONumber=Oct16_HO4-->


