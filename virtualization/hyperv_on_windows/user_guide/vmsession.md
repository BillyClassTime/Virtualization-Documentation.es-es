# Administrar Windows con PowerShell Direct

Puede usar PowerShell Direct para administrar de forma remota una máquina virtual de Windows 10 o Windows Server Technical Preview desde un host de Hyper-V de Windows 10 o Windows Server Technical Preview. PowerShell Direct permite la administración de PowerShell dentro de una máquina virtual, independientemente de la configuración de la red o la configuración de administración remota en el host de Hyper-V o la máquina virtual. Esto facilita a los administradores de Hyper-V la automatización y creación de scripts de configuración y administración de las máquinas virtuales.

Hay dos maneras de ejecutar PowerShell Direct:
* Como si fuera una sesión interactiva: [vaya a esta sección](vmsession.md#create-and-exit-an-interactive-powershell-session) para crear y salir de una sesión de PowerShell Direct mediante los cmdlets de PSSession.
* Mediante la ejecución de un conjunto de comandos o un script: [vaya a esta sección](vmsession.md#run-a-script-or-command-with-invoke-command) para ejecutar un script o un comando con el cmdlet Invoke-Command.


## Requisitos

**Requisitos del sistema operativo:**
* El sistema operativo host debe ejecutar Windows 10, Windows Server Technical Preview o una versión posterior.
* La máquina virtual debe ejecutar Windows 10, Windows Server Technical Preview o una versión posterior.

Si está administrando máquinas virtuales antiguas, use la opción Conexión a máquina virtual (VMConnect) o [configure una red virtual para la máquina virtual](http://technet.microsoft.com/library/cc816585.aspx).

Para crear una sesión de PowerShell Direct en una máquina virtual,
* La máquina virtual debe ejecutarse localmente en el host y estar iniciada.
* Debe iniciar sesión en el equipo host como un administrador de Hyper-V.
* Debe facilitar credenciales de usuario válidas para la máquina virtual.

## Crear y salir de una sesión interactiva de PowerShell

1. En el host de Hyper-V, abra Windows PowerShell como administrador.

3. Ejecute uno de los siguientes comandos para crear una sesión mediante el GUID o el nombre de la máquina virtual:
``` PowerShell
Enter-PSSession -VMName <VMName>
Enter-PSSession -VMGUID <VMGUID>
```

4. Ejecute los comandos que necesite. Estos comandos se ejecutan en la máquina virtual con la que creó la sesión.
5. Cuando haya terminado, ejecute el siguiente comando para cerrar la sesión:
``` PowerShell
Exit-PSSession 
```

>Nota: Si la sesión no se conecta, asegúrese de que utiliza las credenciales de la máquina virtual a la que se está conectando, no del host de Hyper-V.

Para más información sobre estos cmdlets, consulte [Enter-PSSession](http://technet.microsoft.com/library/hh849707.aspx) y [Exit-PSSession](http://technet.microsoft.com/library/hh849743.aspx).

## Ejecutar un script o un comando con Invoke-Command

Puede usar el cmdlet [Invoke-Command](http://technet.microsoft.com/library/hh849719.aspx) para ejecutar un conjunto predeterminado de comandos en la máquina virtual. Este es un ejemplo de cómo puede usar el cmdlet Invoke-Command, donde PSTest es el nombre de la máquina virtual y el script que se ejecutará (foo.ps1) está en la carpeta script de la unidad C:/:

 ``` PowerShell
 Invoke-Command -VMName PSTest -FilePath C:\script\foo.ps1 
 ```

Para ejecutar un único comando, use el parámetro **-ScriptBlock**:

 ``` PowerShell
 Invoke-Command -VMName PSTest -ScriptBlock { cmdlet } 
 ```

## Solucionar problemas

Hay un pequeño conjunto de mensajes de error comunes que aparecen mediante PowerShell Direct. Aquí se muestran los más comunes, algunas de las causas y herramientas para diagnosticar problemas.

### Error: Puede que haya finalizado una sesión remota

Mensaje de error:
```
Enter-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

Causas posibles:
* No se está ejecutando la máquina virtual
* El sistema operativo invitado no admite PowerShell Direct (consulte los [requisitos](#Requirements))
* PowerShell aún no está disponible en el invitado
* El sistema operativo no ha terminado de iniciarse
* El sistema operativo no se puede iniciar correctamente
* Algunos eventos de tiempo de arranque necesitan entrada del usuario
* No se pudieron validar las credenciales de invitado
* Las credenciales proporcionadas son incorrectas
* No hay ninguna cuenta de usuario en el invitado (el sistema operativo no arrancó antes)
* Si se conecta como administrador, tenga en cuenta que el administrador no se estableció como un usuario activo. Más información [aquí](https://technet.microsoft.com/en-us/library/hh825104.aspx).

Puede usar el cmdlet [Get-VM](http://technet.microsoft.com/library/hh848479.aspx) para comprobar que las credenciales que usa tienen el rol de administrador de Hyper-V y para ver qué máquinas virtuales se ejecutan localmente en el host y están iniciadas.

## Ejemplos

Consulte los ejemplos en [GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93).

Consulte los [fragmentos de código de PowerShell Direct](../develop/powershell_snippets.md) para ver numerosos ejemplos de cómo usar PowerShell Direct en su entorno, así como sugerencias y trucos para escribir scripts de Hyper-V con PowerShell.



<!--HONumber=Jan16_HO1-->
