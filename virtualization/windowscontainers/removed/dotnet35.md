---
author: enderb-ms
redirect_url: https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples
---


# Crear una imagen de contenedor de .NET 3.5 Server Core

En esta guía se detalla la creación de un contenedor de Windows Server Core que incluye .NET 3.5 Framework. Antes de comenzar este ejercicio, necesitará el archivo .iso de Windows Server 2016 o tener acceso a los elementos multimedia de Windows Server 2016.

## Preparar los medios

Antes de crear una imagen de contenedor habilitada de .NET 3.5, el paquete de .NET 3.5 debe estar almacenado provisionalmente para su uso dentro de un contenedor. En este ejemplo, el archivo `Microsoft-windows-netfx3-ondemand-package.cab` se copiará desde los elementos multimedia de Windows Server 2016 al host del contenedor.

Cree un directorio denominado `dotnet3.5\source` en el host del contenedor.

```powershell
New-Item -ItemType Directory c:\dotnet3.5\source
```

Copie el archivo `Microsoft-windows-netfx3-ondemand-package.cab` en este directorio. Este archivo puede encontrarse en la carpeta sources\sxs de los elementos multimedia de Windows Server 2016.

```powershell
$file = "d:\sources\sxs\Microsoft-windows-netfx3-ondemand-package.cab"
Copy-Item -Path $file -Destination c:\dotnet3.5\source
``` 
    
Como alternativa, si el host del contenedor se ejecuta en una máquina virtual de Hyper-V y se implementó con un script de inicio rápido, puede ejecutarse lo siguiente. Tenga en cuenta que esto se ejecuta en el host de Hyper-V y no en el host del contenedor. 

```powershell
$vm = "<Container Host VM Name>"
$iso = "$((Get-VMHost).VirtualHardDiskPath)".TrimEnd("\") + "\WindowsServerTP4.iso"
Mount-DiskImage -ImagePath $iso
$ISOImage = Get-DiskImage -ImagePath $iso | Get-Volume
$ISODrive = "$([string]$iSOImage.DriveLetter):"
Get-VM -Name $vm | Enable-VMIntegrationService -Name "Guest Service Interface"
Copy-VMFile -Name $vm -SourcePath "$iSODrive\sources\sxs\microsoft-windows-netfx3-ondemand-package.cab" -DestinationPath "c:\dotnet3.5\source\microsoft-windows-netfx3-ondemand-package.cab" -FileSource Host -CreateFullPath
Dismount-DiskImage -ImagePath $iso
```

Ahora se puede crear una imagen del contenedor que incluirá el .NET 3.5 Framework. Esto se puede completar con PowerShell o Docker. A continuación se incluyen ejemplo de ambos.

## Crear imagen - PowerShell

Para crear una nueva imagen con PowerShell, se creará un contenedor, modificado con todos los cambios que quiera y después se captura en una imagen nueva.

Cree un nuevo formulario de contenedor en la imagen base de Windows Server Core.

```powershell
New-Container -Name dotnet35 -ContainerImageName windowsservercore -SwitchName "Virtual Switch"
```

Cree una carpeta compartida con el nuevo contenedor. Se usará para hacer que el archivo cab de .NET 3.5 sea accesible dentro del nuevo contenedor.  Tenga en cuenta que el contenedor debe detenerse cuando se ejecute el comando siguiente.

```powershell
Add-ContainerSharedFolder -ContainerName dotnet35 -SourcePath C:\dotnet3.5\source -DestinationPath c:\sxs
```

Inicie el contenedor y ejecute el comando siguiente para instalar .NET 3.5.

```powershell
Start-Container dotnet35
Invoke-Command -ContainerName dotnet35 -ScriptBlock {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs} -RunAsAdministrator
```

Una vez finalizada la instalación, detenga el contenedor.

```powershell
Stop-Container dotnet35
```

Para crear una imagen de este contenedor, ejecute lo siguiente en el host del contenedor.

```powershell
New-ContainerImage -ContainerName dotnet35 -Name dotnet35 -Publisher Demo -Version 1.0
```

Ejecute `Get-ContainerImages` para ver la imagen nueva. Esta imagen puede usarse para ejecutar un contenedor con .NET 3.5 Framework preinstalado.

```powershell
Get-ContainerImages
```

## Crear imagen - Docker
 
Para crear una imagen nueva con Docker, se creará un dockerfile con instrucciones sobre cómo crear la nueva imagen. A continuación este dockerfile se ejecutará, lo que producirá una nueva imagen del contenedor. Tenga en cuenta que los siguientes comandos se ejecutan en la máquina virtual del host del contenedor.

Cree un dockerfile y ábralo en el Bloc de notas.

```powershell
New-Item C:\dotnet3.5\dockerfile -Force
Notepad C:\dotnet3.5\dockerfile
```

Copie este texto en el dockerfile y guárdelo.

```powershell
FROM windowsservercore
ADD source /sxs
RUN powershell -Command "& {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs}"
```

Ejecute `docker build`, que consumirá el dockerfile y creará la nueva imagen del contenedor.

```powershell
Docker build -t dotnet35 C:\dotnet3.5\
```

Ejecute `docker images` para ver la imagen nueva. Esta imagen puede usarse para ejecutar un contenedor con .NET 3.5 Framework preinstalado.

```powershell
docker images
```


<!--HONumber=May16_HO3-->


