---
title: Administrar máquinas virtuales de Windows con PowerShell Direct
description: Administrar máquinas virtuales de Windows con PowerShell Direct
keywords: windows 10, hyper-v, powershell, servicios de integración, componentes de integración, automatización, powershell direct
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: fb228e06-e284-45c0-b6e6-e7b0217c3a49
ms.openlocfilehash: 779dcf51d4903c9467cc52dbadb865beb9929bd2
ms.sourcegitcommit: e7fa38bcb7744a34e7a58978b55af1fbf6353247
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/20/2018
---
# <a name="virtual-machine-automation-and-management-using-powershell"></a>Administración y automatización de máquinas virtuales con PowerShell
 
Puedes usar PowerShell Direct para ejecutar PowerShell arbitrario en una máquina virtual con Windows 10 o Windows Server 2016 desde el host de Hyper-V, independientemente de la configuración de administración remota o la configuración de red.

**Formas de ejecutar PowerShell Direct:**  
* Como una sesión interactiva: [haga clic aquí](#create-and-exit-an-interactive-powershell-session) para crear una sesión interactiva de PowerShell con Enter-PSSession y luego cerrarla.
* Como una sesión de un solo uso para ejecutar un único comando o script: [haga clic aquí](#run-a-script-or-command-with-invoke-command) para ejecutar un script o comando mediante Invoke-Command.
* Como una sesión persistente (compilación 14280 y posteriores): [haga clic aquí](#copy-files-with-new-pssession-and-copy-item) para crear una sesión persistente con New-PSSession.  
Luego copie un archivo en la máquina virtual y desde ella mediante Copy-Item y desconecte con Remove-PSSession.

## <a name="requirements"></a>Requisitos
**Requisitos del sistema operativo:**
* Host: Windows 10, Windows Server 2016 o posteriores con Hyper-V.
* Invitado o máquina virtual: Windows 10, Windows Server 2016 o posteriores.

Si está administrando máquinas virtuales antiguas, use la opción Conexión a máquina virtual (VMConnect) o [configure una red virtual para la máquina virtual](http://technet.microsoft.com/library/cc816585.aspx). 

**Requisitos de configuración:**    
* La máquina virtual debe ejecutarse localmente en el host.
* La máquina virtual debe estar activada y en funcionamiento con al menos un perfil de usuario configurado.
* Debe iniciar sesión en el equipo host como un administrador de Hyper-V.
* Debe facilitar credenciales de usuario válidas para la máquina virtual.

-------------

## <a name="create-and-exit-an-interactive-powershell-session"></a>Crear y salir de una sesión interactiva de PowerShell

La manera más sencilla de ejecutar comandos de PowerShell en una máquina virtual es iniciar una sesión interactiva.

Cuando se inicia la sesión, los comandos que escribe se ejecutan en la máquina virtual, igual que si los hubiera escrito directamente en una sesión de PowerShell en la propia máquina virtual.

**Para iniciar una sesión interactiva:**

1. En el host de Hyper-V, abra PowerShell como administrador.

2. Ejecute uno de los siguientes comandos para crear una sesión interactiva mediante el GUID o el nombre de la máquina virtual:  
  
  ``` PowerShell
  Enter-PSSession -VMName <VMName>
  Enter-PSSession -VMId <VMId>
  ```
  
  Proporcione las credenciales de la máquina virtual cuando se le pidan.

3. Ejecute comandos en la máquina virtual.
  
  Debería ver VMName como prefijo del símbolo de PowerShell, tal como se muestra:
  
  ``` 
  [VMName]: PS C:\>
  ```
  
  Cualquier ejecución de comandos se ejecutará en la máquina virtual. Para probar, puede ejecutar `ipconfig` o `hostname` para asegurarse de que estos comandos se ejecutan en la máquina virtual.
  
4. Cuando haya terminado, ejecute el siguiente comando para cerrar la sesión:  
  
   ``` PowerShell
   Exit-PSSession 
   ``` 

> Nota: Si la sesión no se conecta, consulte [Solucionar problemas](#troubleshooting) para ver las posibles causas. 

Para más información sobre estos cmdlets, consulte [Enter-PSSession](http://technet.microsoft.com/library/hh849707.aspx) y [Exit-PSSession](http://technet.microsoft.com/library/hh849743.aspx). 

-------------

## <a name="run-a-script-or-command-with-invoke-command"></a>Ejecutar un script o un comando con Invoke-Command

PowerShell Direct con Invoke-Command es perfecto para aquellas situaciones en que necesita ejecutar un comando o un script en una máquina virtual pero no necesita seguir interactuando con ella después.

**Para ejecutar un solo comando:**

1. En el host de Hyper-V, abra PowerShell como administrador.

2. Ejecute uno de los siguientes comandos para crear una sesión mediante el GUID o el nombre de la máquina virtual:  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -ScriptBlock { command } 
   Invoke-Command -VMId <VMId> -ScriptBlock { command }
   ```
   
   Proporcione las credenciales de la máquina virtual cuando se le pidan.
   
   El comando se ejecutará en la máquina virtual. Si se enviaran resultados a la consola, se imprimirían en ella.  La conexión se cerrará automáticamente en cuanto se ejecute el comando.
   
   
**Para ejecutar un script:**

1. En el host de Hyper-V, abra PowerShell como administrador.

2. Ejecute uno de los siguientes comandos para crear una sesión mediante el GUID o el nombre de la máquina virtual:  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -FilePath C:\host\script_path\script.ps1 
   Invoke-Command -VMId <VMId> -FilePath C:\host\script_path\script.ps1 
   ```
   
   Proporcione las credenciales de la máquina virtual cuando se le pidan.
   
   El script se ejecutará en la máquina virtual.  La conexión se cerrará automáticamente en cuanto se ejecute el comando.

Para obtener más información sobre este cmdlet, consulte [Invoke-Command](http://technet.microsoft.com/library/hh849719.aspx). 

-------------

## <a name="copy-files-with-new-pssession-and-copy-item"></a>Copiar archivos con New-PSSession y Copy-Item

> **Nota:** PowerShell Direct solo admite las sesiones persistentes en las compilaciones 14280 y posteriores de Windows

Las sesiones persistentes de PowerShell son increíblemente útiles a la hora de escribir scripts que coordinen acciones entre uno o más equipos remotos.  Una vez creadas, las sesiones persistentes existen en segundo plano hasta que se decide eliminarlas.  Esto significa que puede hacer referencia a la misma sesión una y otra vez con `Invoke-Command` o `Enter-PSSession` sin pasar credenciales.

Del mismo modo, las sesiones conservan el estado.  Puesto que las sesiones persistentes se almacenan, se conservarán las variables creadas en una sesión o pasadas a ella entre varias llamadas. Hay varias herramientas disponibles para trabajar con sesiones persistentes.  En este ejemplo se usarán [New-PSSession](https://technet.microsoft.com/en-us/library/hh849717.aspx) y [Copy-Item](https://technet.microsoft.com/en-us/library/hh849793.aspx) para mover datos desde el host a una máquina virtual y desde una máquina virtual al host.

**Para crear una sesión, copie archivos:**  

1. En el host de Hyper-V, abra PowerShell como administrador.

2. Ejecute uno de los siguientes comandos para crear una sesión de PowerShell persistente en la máquina virtual mediante `New-PSSession`.
  
  ``` PowerShell
  $s = New-PSSession -VMName <VMName> -Credential (Get-Credential)
  $s = New-PSSession -VMId <VMId> -Credential (Get-Credential)
  ```
  
  Proporcione las credenciales de la máquina virtual cuando se le pidan.
  
  > **Advertencia:**  
   Hay un error en las compilaciones anteriores a la 14500.  Si las credenciales no se especifican de forma explícita con la marca `-Credential`, el servicio del invitado se bloqueará y deberá reiniciarse.  Si se produce este problema, existen instrucciones para solucionarlo [aquí](#error-a-remote-session-might-have-ended).
  
3. Copie un archivo en la máquina virtual.
  
  Para copiar `C:\host_path\data.txt` en la máquina virtual desde el equipo host, ejecute:
  
  ``` PowerShell
  Copy-Item -ToSession $s -Path C:\host_path\data.txt -Destination C:\guest_path\
  ```
  
4.  Copie un archivo desde la máquina virtual (en el host). 
   
   Para copiar `C:\guest_path\data.txt` en el host desde la máquina virtual, ejecute:
  
  ``` PowerShell
  Copy-Item -FromSession $s -Path C:\guest_path\data.txt -Destination C:\host_path\
  ```

5. Detenga la sesión persistente mediante `Remove-PSSession`.
  
  ``` PowerShell 
  Remove-PSSession $s
  ```
  
-------------

## <a name="troubleshooting"></a>Solucionar problemas

Hay un pequeño conjunto de mensajes de error comunes que aparecen mediante PowerShell Direct.  Aquí se muestran los más comunes, algunas de las causas y herramientas para diagnosticar problemas.

### <a name="-vmname-or--vmid-parameters-dont-exist"></a>No existen los parámetros -VMName o -VMID
**Problema:**  
`Enter-PSSession`, `Invoke-Command` o `New-PSSession` no tienen un parámetro `-VMName` o `-VMId`.

**Causas posibles:**  
El problema más probable es que PowerShell Direct no sea compatible con el sistema operativo host.

Puede comprobar la compilación de Windows si ejecuta el comando siguiente:

``` PowerShell
[System.Environment]::OSVersion.Version
```

Si está ejecutando una compilación compatible, también es posible que la versión de PowerShell no ejecute PowerShell Direct.  La versión principal de PowerShell Direct y JEA debe ser 5 o posterior.

Puede comprobar la versión de PowerShell si ejecuta el comando siguiente:

``` PowerShell
$PSVersionTable.PSVersion
```


### <a name="error-a-remote-session-might-have-ended"></a>Error: Puede que haya finalizado una sesión remota
> **Nota:**  
En el caso de Enter-PSSession entre las compilaciones de host 10240 y 12400, todos los errores siguientes se notifican como "Puede que haya finalizado una sesión remota".

**Mensaje de error:**
```
Enter-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**Causas posibles:**
* La máquina virtual existe pero no se está ejecutando.
* El sistema operativo invitado no admite PowerShell Direct (consulte los [requisitos](#Requirements)).
* PowerShell aún no está disponible en el invitado
  * El sistema operativo no ha terminado de iniciarse
  * El sistema operativo no se puede iniciar correctamente
  * Algunos eventos de tiempo de arranque necesitan entrada del usuario

Puede usar el cmdlet [Get-VM](http://technet.microsoft.com/library/hh848479.aspx) para comprobar qué máquinas virtuales se están ejecutando en el host.

**Mensaje de error:**  
```
New-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**Causas posibles:**
* Uno de los motivos mencionados anteriormente: todos pueden aplicarse `New-PSSession`  
* Un error en las compilaciones actuales en las que las credenciales deben pasarse de forma explícita con `-Credential`.  Cuando esto sucede, todo el servicio se bloquea en el sistema operativo invitado y debe reiniciarse.  Puede comprobar si la sesión sigue estando disponible con Enter-PSSession.

Para solucionar el problema de las credenciales, inicie sesión en la máquina virtual con VMConnect, abra PowerShell y reinicie el servicio vmicvmsession con el PowerShell siguiente:

``` PowerShell
Restart-Service -Name vmicvmsession
```

### <a name="error-parameter-set-cannot-be-resolved"></a>Error: No se puede resolver el conjunto de parámetros
**Mensaje de error:**  
``` 
Enter-PSSession : Parameter set cannot be resolved using the specified named parameters.
```

**Causas posibles:**  
* `-RunAsAdministrator` no se admite al conectar con máquinas virtuales.
     
  Cuando se conecta a un contenedor de Windows, el indicador `-RunAsAdministrator` permite conexiones de administrador sin credenciales explícitas.  Dado que las máquinas virtuales no proporcionan al host implícito acceso de administrador, debe especificar explícitamente las credenciales.

Las credenciales de administrador se pueden pasar a la máquina virtual con el parámetro `-Credential` o especificarse manualmente cuando se le solicite.


### <a name="error-the-credential-is-invalid"></a>Error: La credencial no es válida.

**Mensaje de error:**  
```
Enter-PSSession : The credential is invalid.
```

**Causas posibles:** 
* No se pudieron validar las credenciales de invitado
  * Las credenciales proporcionadas son incorrectas.
  * No hay ninguna cuenta de usuario en el invitado (el sistema operativo no arrancó antes)
  * Si se conecta como administrador, tenga en cuenta que el administrador no se estableció como un usuario activo.  Obtenga más información [aquí](https://technet.microsoft.com/en-us/library/hh825104.aspx).
  
### <a name="error-the-input-vmname-parameter-does-not-resolve-to-any-virtual-machine"></a>Error: El parámetro VMName no se resuelve en ninguna máquina virtual.

**Mensaje de error:**  
```
Enter-PSSession : The input VMName parameter does not resolve to any virtual machine.
```

**Causas posibles:**  
* No es administrador de Hyper-V.  
* La máquina virtual no existe.

Puede usar el cmdlet [Get-VM](http://technet.microsoft.com/library/hh848479.aspx) para comprobar que las credenciales que usa tengan el rol de administrador de Hyper-V y para ver qué máquinas virtuales se ejecutan localmente en el host y están iniciadas.


-------------

## <a name="samples-and-user-guides"></a>Ejemplos y guías de usuario

PowerShell Direct admite JEA (Just Enough Administration).  Consulte esta guía de usuario para probarlo.

Consulta los ejemplos de [GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93).
