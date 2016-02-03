# Implementar un host de contenedor de Windows en una nueva máquina virtual de Hyper-V

En este documento se detallan los pasos para usar un script de PowerShell a fin de implementar una nueva máquina virtual de Hyper-V, que se configura después como un host de contenedor de Windows.

Para conocer paso a paso la implementación con script de un host de contenedor de Windows en un sistema físico o virtual existente, consulte [Implementación de un host de contenedor de Windows en contexto](./inplace_setup.md).

**POR FAVOR, LEA ESTO ANTES DE INSTALAR LA IMAGEN DEL SISTEMA OPERATIVO DEL CONTENEDOR:** los términos de licencia del software de versión preliminar de Microsoft Windows Server ("términos de licencia") se aplican al uso del complemento de la imagen del sistema operativo del contenedor Microsoft Windows (el "software complementario"). Al descargar y usar el software complementario, acepta los términos de licencia y no podrá usarlo si no ha aceptado los términos de licencia. El software de versión preliminar de Windows Server y el software complementario cuentan con licencia de Microsoft Corporation.

Se necesita lo siguiente para completar los ejercicios de **contenedores de Hyper-V** y **Windows Server** de esta guía de inicio rápido.

* Un sistema que ejecute la versión de compilación 10586 de Windows 10 o una versión posterior y Windows Server Technical Preview 4 o una versión posterior.
* Disponer del rol de Hyper-V habilitado ([consulte las instrucciones](https://msdn.microsoft.com/virtualization/hyperv_on_windows/quick_start/walkthrough_install#UsingPowerShell))).
* Disponer de 20 GB de almacenamiento disponibles para la imagen del host de contenedor, la imagen base del sistema operativo y los scripts de instalación.
* Disponer de permisos de administrador en el host de Hyper-V.

> Un host de contenedor virtualizado, que ejecute contenedores de Hyper-V, requerirá virtualización anidada. El host físico y el host virtual deberán ejecutar un sistema operativo que admita virtualización anidada. Para más información, consulte Novedades de Hyper-V en [Windows Server 2016 Technical Preview](https://technet.microsoft.com/library/dn765471.aspx#BKMK_nested).

## Instalar un nuevo host de contenedor en una nueva máquina virtual

Los contenedores de Windows constan de varios componentes, como el host de contenedor de Windows y las imágenes base del sistema operativo del contenedor. Hemos creado un script que se ocupará de la descarga y configuración de estos elementos. Siga estos pasos para implementar una nueva máquina virtual de Hyper-V y configurar este sistema como un host de contenedor de Windows.

Inicie una sesión de PowerShell como administrador. Esto se puede realizar con un clic derecho en el icono de PowerShell, seleccionando "Ejecutar como administrador", o bien ejecutando el siguiente comando desde cualquier sesión de PowerShell.

``` powershell
PS C:\> start-process powershell -Verb runAs
```

Antes de descargar y ejecutar el script, asegúrese de que se ha creado un conmutador virtual de Hyper-V externo. Este script provocará un error si no hay uno.

Ejecute lo siguiente para devolver una lista de conmutadores virtuales externos. Si no se devuelve nada, cree un nuevo conmutador virtual externo y luego continúe con el paso siguiente de esta guía.

```powershell
PS C:\> Get-VMSwitch | where {$_.SwitchType –eq “External”}
```

Utilice el comando siguiente para descargar el script de configuración. El script también se puede descargar manualmente desde esta ubicación: [script de configuración](https://aka.ms/tp4/New-ContainerHost).

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/New-ContainerHost -OutFile c:\New-ContainerHost.ps1
```

Ejecute el siguiente comando para crear y configurar el host de contenedor, donde `&lt;containerhost&gt;` será el nombre de la máquina virtual.

``` powershell
PS C:\> c:\New-ContainerHost.ps1 –VmName <containerhost> -WindowsImage ServerDatacenterCore -Hyperv
```

Cuando se inicie el script, le pedirá una contraseña. Será de la contraseña asignada a la cuenta de administrador.

A continuación, se le pedirá que lea y acepte los términos de licencia.

```
Before installing and using the Windows Server Technical Preview 4 with Containers virtual machine you must:
    1. Review the license terms by navigating to this link: http://aka.ms/tp4/containerseula
    2. Print and retain a copy of the license terms for your records.
By downloading and using the Windows Server Technical Preview 4 with Containers virtual machine you agree to such
license terms. Please confirm you have accepted and agree to the license terms.
[N] No  [Y] Yes  [?] Help (default is "N"):
```

El script comenzará entonces a descargar y configurar los componentes del contenedor de Windows. Este proceso puede tardar bastante tiempo debido a la descarga de gran tamaño. Cuando termine, la máquina virtual estará configurada y lista para que cree y administre contenedores de Windows e imágenes de contenedores de Windows con PowerShell y Docker.

Cuando el script de configuración haya finalizado, inicie sesión en la máquina virtual con la contraseña especificada durante el proceso de configuración y asegúrese de que la máquina virtual tenga una dirección IP válida. Con estos elementos completados, el sistema debería estar preparado para los contenedores de Windows.

## Pasos siguientes: empezar a usar contenedores

Ahora que tiene un sistema de Windows Server 2016 ejecutando la característica de contenedor de Windows, continúe con las guías siguientes para empezar a trabajar con contenedores de Hyper-V y Windows Server.

[Inicio rápido: contenedores de Windows y PowerShell](./manage_powershell.md)  
[Inicio rápido: contenedores de Windows y Docker](./manage_docker.md)




<!--HONumber=Jan16_HO2-->
