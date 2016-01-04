# Novedades de Hyper-V en Windows 10

En este tema se explica la funcionalidad nueva y modificada en Hyper-V en Windows 10®.

## Windows PowerShell Direct

Ahora hay una manera fácil y confiable de ejecutar comandos de Windows PowerShell en una máquina virtual desde el sistema operativo host. No existen requisitos de red o firewall ni de configuración especial. 
Funciona independientemente de la configuración de administración remota. Para usarlo, debe ejecutar Windows 10 o Windows Server Technical Preview en el host y en el sistema operativo invitado de la máquina virtual.

Para crear una sesión de PowerShell Direct, utilice uno de los siguientes comandos:

``` PowerShell
Enter-PSSession -VMName VMName
Invoke-Command -VMName VMName -ScriptBlock { commands }
```

Actualmente, los administradores de Hyper-V dependen de dos categorías de herramientas para conectarse a una máquina virtual en su host de Hyper-V:
- Herramientas de administración remota como PowerShell o Escritorio remoto.
- Conexión a máquina virtual de Hyper-V (VM Connect).

Las dos tecnologías funcionan bien, pero cada una de ellas tiene ventajas e inconvenientes a medida que crece la implementación de Hyper-V. VMConnect es confiable, pero puede ser difícil de automatizar. PowerShell remoto es eficaz, pero puede ser difícil de instalar y mantener.

Windows PowerShell Direct ofrece una experiencia de scripting y automatización eficaz con la simplicidad de VMConnect. Como Windows PowerShell Direct se ejecuta entre el host y la máquina virtual, no hay necesidad de una conexión de red ni de habilitar la administración remota. Necesita credenciales de invitado para iniciar sesión en la máquina virtual.

### Requisitos

- Debe estar conectado a un host de Windows 10 o de Windows Server Technical Preview con máquinas virtuales que ejecuten Windows 10 o Windows Server Technical Preview como invitados.
- Debe haber iniciado sesión con credenciales de administrador de Hyper-V en el host.
- Se necesitan credenciales de usuario para la máquina virtual.
- La máquina virtual a la que quiere conectarse debe estar en ejecución y arrancada.


## Agregar y quitar adaptadores de red y memoria en caliente

Ahora puede agregar o quitar un adaptador de red mientras la máquina virtual está en ejecución, sin tiempo de inactividad. Esto funciona para las máquinas virtuales de generación 2 con sistemas operativos Windows y Linux.

También puede ajustar la cantidad de memoria asignada a una máquina virtual mientras se está ejecutando, incluso aunque no haya habilitado Memoria dinámica. Esto funciona para las máquinas virtuales de generación 1 y generación 2.

## Puntos de control de producción

Los puntos de control de producción permiten crear fácilmente imágenes de "un momento específico" de una máquina virtual, que se pueden restaurar más adelante de forma que se admitan por completo para todas las cargas de trabajo de producción. Esto se consigue mediante el uso de tecnología de copia de seguridad dentro del invitado para crear el punto de control, en lugar de usar tecnología de estado guardado. Para los puntos de control de producción, se utiliza el Servicio de instantáneas de volumen (VSS) dentro de máquinas virtuales de Windows. Las máquinas virtuales de Linux vacían sus búferes de sistema de archivos para crear un punto de control coherente de sistema de archivos. Si quiere crear puntos de control mediante la tecnología de estado guardado, sigue teniendo la posibilidad de usar puntos de control estándares para la máquina virtual.


> **Importante:** El valor predeterminado de las nuevas máquinas virtuales será la creación de puntos de control de producción con una reserva de los puntos de control estándares.


## Mejoras del administrador de Hyper-V

- **Compatibilidad con credenciales alternativas**: ahora puede usar un conjunto diferente de credenciales de administrador de Hyper-V cuando se conecte a otro host remoto de Windows 10 Technical Preview. También puede guardar dichas credenciales, por lo que será más fácil iniciar sesión más tarde.

- **Administración de nivel inferior**: ahora puede usar el administrador de Hyper-V para administrar más versiones de Hyper-V. Con el administrador de Hyper-V en Windows 10 Technical Preview, puede administrar equipos que ejecuten Hyper-V en Windows Server 2012, Windows 8, Windows Server 2012 R2 y Windows 8.1.

- **Protocolo de administración actualizado**: el administrador de Hyper-V se actualizó para comunicarse con los hosts de Hyper-V remotos mediante el protocolo WS-MAN, que permite la autenticación CredSSP, Kerberos o NTLM. Si usa CredSSP para conectarse a un host de Hyper-V remoto, podrá realizar una migración en vivo sin que primero deba habilitar la delegación restringida en Active Directory. La infraestructura basada en WS-MAN también simplifica la configuración necesaria para habilitar un host para la administración remota. WS-MAN se conecta a través del puerto 80, que está abierto de forma predeterminada.


## Modo de espera conectado funciona

Si se habilita Hyper-V en un equipo que utiliza el modelo de energía siempre activo o conectado (AOAC), el estado de energía del modo de espera conectado ahora está disponible.

En Windows 8 y 8.1, Hyper-V provocaba que los equipos que usaban el modelo de energía siempre activo o conectado (AOAC) (también conocido como InstantON) nunca entraran en modo de suspensión. Consulte este [artículo de Knowledge Base](
https://support.microsoft.com/es-es/kb/2973536) para ver una descripción completa.


## Arranque seguro de Linux

Ahora es posible arrancar más sistemas operativos Linux, que se ejecutan en máquinas virtuales de generación 2, con la opción de arranque seguro habilitada. Ubuntu 14.04 y las versiones posteriores y SUSE Linux Enterprise Server 12 están habilitados para el arranque seguro en hosts que ejecuten la edición Technical Preview. Antes de arrancar la máquina virtual por primera vez, debe especificar que la máquina virtual debe utilizar la entidad de certificación de Microsoft UEFI. En un símbolo de Windows PowerShell con privilegios elevados, escriba:

    Set-VMFirmware vmname -SecureBootTemplate MicrosoftUEFICertificateAuthority

Para obtener más información sobre cómo ejecutar máquinas virtuales de Linux en Hyper-V, consulte [Máquinas virtuales de Linux y FreeBSD en Hyper-V](http://technet.microsoft.com/library/dn531030.aspx).


## Versión de configuración de máquina virtual

Al mover o importar una máquina virtual en un host que ejecuta Hyper-V en Windows 10 desde un host que ejecuta Windows 8.1, el archivo de configuración de la máquina virtual no se actualiza automáticamente. Esto permite que la máquina virtual se mueva de vuelta a un host que ejecute Windows 8.1. No tendrá acceso a las nuevas características de la máquina virtual hasta que actualice manualmente la versión de configuración de la máquina virtual.

La versión de configuración de la máquina virtual representa la versión de Hyper-V con la que son compatibles la configuración de la máquina virtual, el estado guardado y los archivos de instantáneas. Las máquinas virtuales con la versión 5 de configuración son compatibles con Windows 8.1 y puede ejecutarse en Windows 8.1 y Windows 10. Las máquinas virtuales con la versión 6 de configuración son compatibles con Windows 10 y no se ejecutarán en Windows 8.1.

### Comprobar la versión de configuración

Desde un símbolo del sistema con privilegios elevados, ejecute el siguiente comando:

``` PowerShell
Get-VM * | Format-Table Name, Version
```

### Actualizar la versión de configuración

Desde un símbolo de sistema de Windows PowerShell con privilegios elevados, ejecute uno de los siguientes comandos:

``` PowerShell
Update-VmConfigurationVersion <vmname>
```

O bien

``` PowerShell
Update-VmConfigurationVersion <vmobject>
```

**Importante: **
- Después de actualizar la versión de configuración de máquina virtual, no se podrá mover la máquina virtual a un host que ejecute Windows 8.1.
- No se puede cambiar la versión de configuración de máquina virtual de la versión 6 a la 5.
- Debe desactivar la máquina virtual para actualizar la configuración de máquina virtual.
- Después de la actualización, la máquina virtual usará el nuevo formato de archivo de configuración. Para más información, consulte el nuevo formato de archivo de configuración de máquina virtual.


## Formato de archivo de configuración

Las máquinas virtuales tienen ahora un nuevo formato de archivo de configuración que está diseñado para aumentar la eficacia de lectura y escritura de datos de configuración de máquina virtual. También se ha diseñado para reducir la posibilidad de daños en los datos si se produce un error de almacenamiento. Los nuevos archivos de configuración usan la extensión .VMCX para los datos de configuración de máquina virtual y la extensión .VMRS para los datos de estado de tiempo de ejecución.


> **Importante:** El archivo .VMCX es un formato binario. No se admite la edición de los archivos .VMCX o .VMRS directamente.

## Servicios de integración mediante Windows Update

Las actualizaciones de servicios de integración para los invitados de Windows ahora se distribuyen a través de Windows Update.

Los componentes de integración (también conocidos como servicios de integración) son el conjunto de controladores sintéticos que permiten que una máquina virtual se comunique con el sistema operativo host. Controlan los servicios que van desde la sincronización de hora a la copia de archivos de invitado. Hemos estado hablando con los clientes sobre la instalación y actualización de componentes de integración durante el año pasado y descubrimos que suponen una gran dificultad durante el proceso de actualización.


Históricamente, todas las nuevas versiones de Hyper-V incorporan nuevos componentes de integración. La actualización del host de Hyper-V requiere actualizar los componentes de integración también en las máquinas virtuales. Los nuevos componentes de integración se incluyeron con el host de Hyper-V y después se instalaron en las máquinas virtuales mediante vmguest.iso. Este proceso requería reiniciar la máquina virtual y no se pudo procesar por lotes con otras actualizaciones de Windows. Como el administrador de Hyper-V tenía que ofrecer vmguest.iso y el administrador de la máquina virtual debía instalarlos, la actualización del componente de integración requería que el administrador de Hyper-V tuviera credenciales de administrador en las máquinas virtuales, lo que no siempre es el caso.
　　


A partir de Windows 10, todos los componentes de integración se entregarán a las máquinas virtuales mediante Windows Update junto con otras actualizaciones importantes.


Hay actualizaciones disponibles actualmente para las máquinas virtuales que ejecutan:
*  Windows Server 2012
*  Windows Server 2008 R2
*  Windows 8
*  Windows 7

La máquina virtual debe conectarse a Windows Update o a un servidor de WSUS. En el futuro, las actualizaciones de componentes de integración tendrán un identificador de categoría; en esta versión, se muestran como artículos de Knowledge Base.

Para obtener más información sobre cómo se determina la aplicabilidad, consulte esta [entrada de blog](http://blogs.technet.com/b/virtualization/archive/2014/11/24/integration-components-how-we-determine-windows-update-applicability.aspx).


Consulte [esta entrada de blog](http://blogs.msdn.com/b/virtual_pc_guy/archive/2014/11/12/updating-integration-components-over-windows-update.aspx) para ver un tutorial detallado acerca de la instalación de servicios de integración.


> **Importante:** El archivo de imagen ISO vmguest.iso ya no es necesario para actualizar los componentes de integración. No se incluye con Hyper-V en Windows 10.


## Paso siguiente

[Tutorial de Hyper-V en Windows 10](..\quick_start\walkthrough.md)



