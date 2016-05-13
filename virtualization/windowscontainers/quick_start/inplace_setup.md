



# Implementar un host de contenedor de Windows en un sistema físico o virtual existentes

En este documento se explican los pasos del uso de un script de PowerShell para implementar y configurar el rol de contenedor de Windows en un sistema físico o virtual existente.

Para conocer los pasos de la implementación con scripts de una nueva máquina virtual de Hyper-V configurada como un host de contenedor de Windows, consulte [Nuevo host de contenedor de Windows de Hyper-V](./container_setup.md).

**POR FAVOR, LEA ESTO ANTES DE INSTALAR LA IMAGEN DEL SISTEMA OPERATIVO DEL CONTENEDOR:** los términos de licencia del software de versión preliminar de Microsoft Windows Server ("términos de licencia") se aplican al uso del complemento de la imagen del sistema operativo del contenedor Microsoft Windows (el "software complementario"). Al descargar y usar el software complementario, acepta los términos de licencia y no podrá usarlo si no ha aceptado los términos de licencia. El software de versión preliminar de Windows Server y el software complementario cuentan con licencia de Microsoft Corporation.

Se necesita lo siguiente para completar los ejercicios de contenedores de Hyper-V y Windows Server de esta guía de inicio rápido.

* Sistema que ejecuta Windows Server Technical Preview 4 o posterior.
* Disponer de 10 GB de almacenamiento disponibles para la imagen del host de contenedor, la imagen base del sistema operativo y los scripts de instalación.
* Disponer de permisos de administrador en el sistema.

## Configuración de una máquina virtual o un host sin sistema operativo existentes para contenedores

Los contenedores de Windows requieren las imágenes base del sistema operativo del contenedor. Hemos creado un script que se ocupará de la descarga y la instalación. Siga estos pasos para configurar el sistema como un host de contenedor de Windows. Para más información, consulte Novedades de Hyper-V en [Windows Server 2016 Technical Preview](https://tnstage.redmond.corp.microsoft.com/en-US/library/dn765471.aspx#BKMK_nested).

Inicie una sesión de PowerShell como administrador. Puede hacerlo ejecutando el siguiente comando desde la línea de comandos.

``` powershell
PS C:\> powershell.exe
```

Asegúrese de que el título de las ventanas sea "Administrador: Windows PowerShell". Si no se indica administrador, ejecute este comando para ejecutar con privilegios de administrador:

``` powershell
PS C:\> start-process powershell -Verb runas
```

Utilice el comando siguiente para descargar el script de instalación. El script también se puede descargar manualmente desde esta ubicación: [script de configuración](https://aka.ms/tp4/Install-ContainerHost).

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/Install-ContainerHost -OutFile C:\Install-ContainerHost.ps1
```

 Cuando finalice la descarga, ejecute el script.
``` PowerShell
PS C:\> powershell.exe -NoProfile C:\Install-ContainerHost.ps1 -HyperV
```

El script comenzará entonces a descargar y configurar los componentes del contenedor de Windows. Este proceso puede tardar bastante tiempo debido a la descarga de gran tamaño. Es posible que la máquina se reinicie durante el proceso. Cuando termine, la máquina estará configurada y lista para que cree y administre contenedores de Windows e imágenes de contenedores de Windows con PowerShell y Docker.

 Con estos elementos completados, el sistema debería estar preparado para los contenedores de Windows.

## Pasos siguientes: empezar a usar contenedores

Ahora que tiene un sistema de Windows Server 2016 ejecutando la característica de contenedor de Windows, continúe con las guías siguientes para empezar a trabajar con contenedores de Hyper-V y Windows Server.

[Inicio rápido: contenedores de Windows y Docker](./manage_docker.md)

[Inicio rápido: contenedores de Windows y PowerShell](./manage_powershell.md)






<!--HONumber=Feb16_HO3-->


