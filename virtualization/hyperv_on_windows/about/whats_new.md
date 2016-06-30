---
redirect_url: ../windows_welcome
translationtype: Human Translation
ms.sourcegitcommit: 3491d21a31a92f0a97de572afafc29ae8e661c12
ms.openlocfilehash: 54b496f535b94f0b9aa83cce3ae5504830faee65

---

## Agregar y quitar adaptadores de red y memoria en caliente

Ahora puede agregar o quitar un adaptador de red mientras la máquina virtual está en ejecución, sin tiempo de inactividad. Esto funciona para las máquinas virtuales de generación 2 con sistemas operativos Windows y Linux. 

También puede ajustar la cantidad de memoria asignada a una máquina virtual mientras se está ejecutando, incluso aunque no haya habilitado Memoria dinámica. Esto funciona para las máquinas virtuales de generación 1 y generación 2.

## Puntos de control de producción

Los puntos de control de producción permiten crear fácilmente imágenes de "un momento específico" de una máquina virtual, que se pueden restaurar más adelante de forma que se admitan por completo para todas las cargas de trabajo de producción. Esto se consigue mediante el uso de tecnología de copia de seguridad dentro del invitado para crear el punto de control, en lugar de usar tecnología de estado guardado. Para los puntos de control de producción, se utiliza el Servicio de instantáneas de volumen (VSS) dentro de máquinas virtuales de Windows. Las máquinas virtuales de Linux vacían sus búferes de sistema de archivos para crear un punto de control coherente de sistema de archivos. Si quiere crear puntos de control mediante la tecnología de estado guardado, sigue teniendo la posibilidad de usar puntos de control estándares para la máquina virtual. 


> **Importante:** El valor predeterminado de las nuevas máquinas virtuales será la creación de puntos de control de producción con una reserva de los puntos de control estándares. 
 

## Mejoras del administrador de Hyper-V

- **Compatibilidad con credenciales alternativas**: ahora puede usar un conjunto diferente de credenciales del Administrador de Hyper-V cuando se conecte a otro host remoto de Windows 10 Technical Preview. También puede guardar dichas credenciales, por lo que será más fácil iniciar sesión más tarde. 

- **Administración de nivel inferior**: ahora puede usar el Administrador de Hyper-V para administrar más versiones de Hyper-V. Con el Administrador de Hyper-V en Windows 10 Technical Preview, puede administrar equipos con Hyper-V en Windows Server 2012, Windows 8, Windows Server 2012 R2 y Windows 8.1.

- **Protocolo de administración actualizado**: el Administrador de Hyper-V se actualizó para comunicarse con los hosts remotos de Hyper-V mediante el protocolo WS-MAN, que permite la autenticación CredSSP, Kerberos o NTLM. Si usa CredSSP para conectarse a un host de Hyper-V remoto, podrá realizar una migración en vivo sin que primero deba habilitar la delegación restringida en Active Directory. La infraestructura basada en WS-MAN también simplifica la configuración necesaria para habilitar un host para la administración remota. WS-MAN se conecta a través del puerto 80, que está abierto de forma predeterminada.


## Compatibilidad con modo de espera conectado 

Si se habilita Hyper-V en un equipo que utiliza el modelo de energía siempre activo o conectado (AOAC), el estado de energía del modo de espera conectado ahora está disponible.

En Windows 8 y 8.1, Hyper-V provocaba que los equipos que usaban el modelo de energía siempre activo o conectado (AOAC) (también conocido como InstantON) nunca entraran en modo de suspensión. Consulte este [artículo de KB](
https://support.microsoft.com/en-us/kb/2973536) para obtener una descripción completa.


## Arranque seguro de Linux 

Ahora es posible arrancar más sistemas operativos Linux que se ejecuten en máquinas virtuales de generación 2 con la opción de arranque seguro habilitada.  Ubuntu 14.04 y las versiones posteriores y SUSE Linux Enterprise Server 12 están habilitados para el arranque seguro en hosts que ejecuten la edición Technical Preview. Antes de arrancar la máquina virtual por primera vez, debe especificar que la máquina virtual debe utilizar la entidad de certificación de Microsoft UEFI.  En un símbolo de Windows PowerShell con privilegios elevados, escriba:

    Set-VMFirmware [-VMName] <VMName> [-SecureBootTemplate] <MicrosoftUEFICertificateAuthority>

Para más información sobre la ejecución de máquinas virtuales Linux en Hyper-V, consulte [Máquinas virtuales Linux y FreeBSD en Hyper-V](http://technet.microsoft.com/library/dn531030.aspx).
 
 
## Versión de configuración de máquina virtual

Al mover o importar una máquina virtual a un host que ejecuta Hyper-V en Windows 10 desde un host que ejecuta Windows 8.1, el archivo de configuración de la máquina virtual no se actualiza automáticamente. Esto permite que la máquina virtual se mueva de vuelta a un host que ejecute Windows 8.1. No podrá usar las nuevas características de Hyper-V con la máquina virtual hasta que actualice manualmente la versión de configuración de la máquina virtual. 

La versión de configuración de la máquina virtual representa la versión de Hyper-V con la que son compatibles la configuración, el estado guardado y los archivos de instantáneas de la máquina virtual. Las máquinas virtuales con la versión 5 de configuración son compatibles con Windows 8.1 y puede ejecutarse en Windows 8.1 y Windows 10. Las máquinas virtuales con la versión 6 de configuración son compatibles con Windows 10 y no se ejecutarán en Windows 8.1.

### Comprobar la versión de configuración

Desde un símbolo del sistema con privilegios elevados, ejecute el siguiente comando:

``` PowerShell
Get-VM * | Format-Table Name, Version
```

### Actualizar la versión de configuración 

Desde un símbolo de Windows PowerShell con privilegios elevados, ejecute uno de los siguientes comandos:

``` 
Update-VmConfigurationVersion <VMName>
```

O bien,

``` 
Update-VmConfigurationVersion <VMObject>
```

> **Importante:**
>
- Después de actualizar la versión de configuración de máquina virtual, no se podrá mover la máquina virtual a un host que ejecute Windows 8.1.
- No se puede cambiar la versión de configuración de máquina virtual de la versión 6 a la 5.
- Debe desactivar la máquina virtual para actualizar la configuración de máquina virtual.
- Después de la actualización, la máquina virtual usará el nuevo formato de archivo de configuración. Para más información, consulte [Formato de archivos de configuración](#configuration-file-format).


## <a name="configuration-file-format"></a>Formato de archivo de configuración

Las máquinas virtuales tienen ahora un nuevo formato de archivo de configuración que está diseñado para aumentar la eficacia de lectura y escritura de datos de configuración de máquina virtual. También se ha diseñado para reducir la posibilidad de daños en los datos si se produce un error de almacenamiento. Los nuevos archivos de configuración usan la extensión .VMCX para los datos de configuración de máquina virtual y la extensión .VMRS para los datos de estado de tiempo de ejecución. 

> **Importante:** El archivo .VMCX es un formato binario. No se admite la edición de los archivos .VMCX o .VMRS directamente.


<!--HONumber=Jun16_HO4-->


