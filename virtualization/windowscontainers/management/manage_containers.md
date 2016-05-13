---
autor: neilpeterson
---

# Administración de contenedores de Windows Server

**Esto es contenido preliminar y está sujeto a cambios.** 

El ciclo de vida del contenedor incluye acciones tales como iniciar, detener y quitar contenedores. Al realizar estas acciones, puede que también tenga que recuperar una lista de imágenes de contenedores, administrar redes de contenedores y limitar recursos de contenedores. En este documento se detallarán las tareas básicas de administración de contenedores mediante PowerShell.

Para obtener documentación sobre la administración de contenedores de Windows con Docker, consulte el documento de Docker [Working with Containers]( https://docs.docker.com/userguide/usingdocker/) (Trabajar con contenedores).

## PowerShell

### Crear un contenedor

Al crear un nuevo contenedor, necesita el nombre de una imagen de contenedor que servirá como la base del contenedor. Se puede encontrar mediante el comando `Get-ContainerImage`.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version         IsOSImage
----              ---------    -------         ---------
NanoServer        CN=Microsoft 10.0.10584.1000 True
WindowsServerCore CN=Microsoft 10.0.10584.1000 True
```

Use el comando `New-Container` para crear un nuevo contenedor. También se puede asignar un nombre NetBIOS al contenedor con el parámetro `-ContainerComputerName`.

```powershell
PS C:\> New-Container -ContainerImageName WindowsServerCore -Name demo -ContainerComputerName demo

Name State Uptime   ParentImageName
---- ----- ------   ---------------
demo  Off   00:00:00 WindowsServerCore
```

Cuando el contenedor se haya creado, agregue un adaptador de red al contenedor.

```powershell
PS C:\> Add-ContainerNetworkAdapter -ContainerName demo
```

Para conectar el adaptador de red de los contenedores a un conmutador virtual, se necesita el nombre del conmutador. Use `Get-VMSwitch` para devolver una lista de conmutadores virtuales. 

```powershell
PS C:\> Get-VMSwitch

Name SwitchType NetAdapterInterfaceDescription
---- ---------- ------------------------------
DHCP External   Microsoft Hyper-V Network Adapter
NAT  NAT
```

Conecte el adaptador de red al conmutador virtual con `Connect-ContainerNetworkAdapter`. **NOTA**: Esto también se puede realizar cuando el contenedor se crea mediante el parámetro –SwitchName.

```powershell
PS C:\> Connect-ContainerNetworkAdapter -ContainerName demo -SwitchName NAT
```

### Iniciar un contenedor
Para iniciar el contenedor, se enumerará un objeto de PowerShell que represente el contenedor. Esto puede realizarse colocando la salida de `Get-Container` en una variable de PowerShell.

```powershell
PS C:\> $container = Get-Container -Name demo
```

Estos datos pueden usarse a continuación con el comando `Start-Container` para iniciar el contenedor.

```powershell
PS C:\> Start-Container $container
```

El siguiente script iniciará todos los contenedores en el host.

```powershell
PS C:\> Get-Container | Start-Container
```

### Conectarse al contenedor

PowerShell Direct puede usarse para conectarse a un contenedor. Esto puede resultar útil si debe realizar manualmente una tarea, como instalar software, iniciar un proceso o solucionar problemas de un contenedor. Dado que se está usando PowerShell Direct, se puede crear una sesión de PowerShell con el contenedor, independientemente de la configuración de red. Para obtener más información sobre PowerShell Direct, consulte el [Manual del usuario de PowerShell Direct](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/vmsession)

Para crear una sesión interactiva con el contenedor, use el comando `Enter-PSSession`.

 ```powershell
PS C:\> Enter-PSSession -ContainerName demo -RunAsAdministrator
```

Observe que cuando se haya creado la sesión remota de PowerShell, el símbolo del shell cambia para reflejar el nombre del contenedor.

```powershell
[demo]: PS C:\>
```

Los comandos también se pueden ejecutar en un contenedor sin crear una sesión de PowerShell persistente. Para ello, use `Invoke-Command`.

En el ejemplo siguiente se crea una carpeta denominada "Application" en el contenedor.

```powershell

PS C:\> Invoke-Command -ContainerName demo -ScriptBlock {New-Item -ItemType Directory -Path c:\application }

Directory: C:\
Mode                LastWriteTime         Length Name                                                 PSComputerName
----                -------------         ------ ----                                                 --------------
d-----       10/28/2015   3:31 PM                application                                          demo
```

### Detener un contenedor

Para detener el contenedor, se necesitará un objeto de PowerShell que represente ese contenedor. Esto puede realizarse colocando la salida de `Get-Container` en una variable de PowerShell.

```powershell
PS C:\> $container = Get-Container -Name demo
```

Después, se puede usar con el comando `Stop-Container` para detener el contenedor.

```powershell
PS C:\> Stop-Container $container
```

Lo siguiente detendrá todos los contenedores del host.

```powershell
PS C:\> Get-Container | Stop-Container
```

### Quitar un contenedor

Es posible quitar un contenedor cuando ya no sea necesario. Para quitar un contenedor, debe tener un estado detenido y se debe crear un objeto de PowerShell que represente el contenedor.

```powershell
PS C:\> $container = Get-Container -Name demo
```

Para quitar el contenedor, use el comando `Remove-Container`.

```powershell
PS C:\> Remove-Container $container -Force
```

Lo siguiente eliminará todos los contenedores del host.

```powershell
PS C:\> Get-Container | Remove-Container -Force
```

## Docker

### Crear un contenedor <!--docker-->

Use `docker run` para crear un contenedor con Docker.

```powershell
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Para más información sobre el comando docker run, consulte la [referencia de docker run]( https://docs.docker.com/engine/reference/run/).

### Detener un contenedor <!--docker-->

Use el comando `docker stop` para detener un contenedor con Docker.

```powershell
PS C:\> docker stop tender_panini

tender_panini
```

Este ejemplo detiene todos los contenedores en ejecución con Docker.

```powershell
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### Quitar contenedor <!--docker-->

Para quitar un contenedor con Docker, use el comando `docker rm`.

```powershell
PS C:\> docker rm prickly_pike

prickly_pike
``` 

Para quitar todos los contenedores con Docker.

```powershell
PS C:\> docker rm $(docker ps -a -q)

dc3e282c064d
2230b0433370
```

Para más información sobre el comando docker rm, consulte la [referencia de docker rm](https://docs.docker.com/engine/reference/commandline/rm/).


<!--HONumber=Mar16_HO1-->


