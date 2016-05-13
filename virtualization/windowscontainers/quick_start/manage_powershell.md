



# Inicio rápido de contenedores de Windows: PowerShell

Los contenedores de Windows se pueden utilizar para implementar rápidamente varias aplicaciones aisladas en un único equipo. En este inicio rápido se muestra la implementación y administración de contenedores de Windows Server y Hyper-V mediante PowerShell. En este ejercicio se creará desde cero una aplicación "hello world" muy simple, que se ejecuta tanto en un contenedor de Windows Server como de Hyper-V. Durante este proceso, se crearán imágenes del contenedor, se trabajará con carpetas compartidas de contenedor y se administrará el ciclo de vida del contenedor. Cuando lo complete, tendrá un conocimiento básico de la administración e implementación de contenedores de Windows.

En este tutorial se detallan los contenedores de Windows Server y Hyper-V. Cada tipo de contenedor tiene sus propios requisitos básicos. Con la documentación de contenedores de Windows se incluye un procedimiento para implementar rápidamente un host de contenedor. Se trata de la manera más fácil para empezar a trabajar rápidamente con contenedores de Windows. Si aún no dispone de un host de contenedor, consulte [Implementar un host de contenedor de Windows en una nueva máquina virtual de Hyper-V](./container_setup.md).

Para cada uno de los ejercicios se requieren los elementos siguientes.

**Contenedores de Windows Server:**

- Un host de contenedor de Windows que ejecute Windows Server 2016 Core, ya sea de forma local o en Azure.

**Contenedores de Hyper-V:**

- Un host de contenedor de Windows habilitado con virtualización anidada.
- Elementos multimedia de Windows Server 2016: [descargar](https://aka.ms/tp4/serveriso).

> Microsoft Azure no admite contenedores de Hyper-V. Para completar los ejercicios de Hyper-V, necesita un host de contenedor local.

## Contenedor de Windows Server

Los contenedores de Windows Server ofrecen un entorno operativo aislado, portátil y controlado por recursos para ejecutar aplicaciones y procesos de hospedaje. Los contenedores de Windows Server ofrecen aislamiento entre el contenedor y el host, y entre los contenedores que se ejecutan en el host, mediante el aislamiento del espacio de nombres y el proceso.

### Crear contenedor

En el momento de la publicación de TP4, los contenedores de Windows Server que se ejecutan en un equipo con Windows Server 2016 o Windows Server 2016 core requieren la imagen del sistema operativo Windows Server 2016 Core.

Inicie una sesión de PowerShell escribiendo `powershell`.

```powershell
C:\> powershell
Windows PowerShell
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\>
```

Para comprobar si se ha instalado la imagen del sistema operativo Windows Server Core, use el comando `Get-ContainerImage`. Puede ver varias imágenes de sistema operativo, lo cual es normal.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

Para crear un contenedor de Windows Server, use el comando `New-Container`. En el ejemplo siguiente, se crea un contenedor denominado `TP4Demo` a partir de la imagen de sistema operativo `WindowsServerCore`. A continuación, se conecta el contenedor a un conmutador de máquina virtual denominado `Virtual Switch`.

```powershell
PS C:\> New-Container -Name TP4Demo -ContainerImageName WindowsServerCore -SwitchName "Virtual Switch"

Name    State Uptime   ParentImageName
----    ----- ------   ---------------
TP4Demo Off   00:00:00 WindowsServerCore
```

Para ver los contenedores existentes, use el comando `Get Container`.

```powershell
PS C:\> Get-Container

Name    State Uptime   ParentImageName
----    ----- ------   ---------------
TP4Demo Off   00:00:00 WindowsServerCore
```

Inicie el contenedor mediante el comando `Start-Container`.

```powershell
PS C:\> Start-Container -Name TP4Demo
```

Conéctese al contenedor mediante el comando `Enter-PSSession`. Observe que cuando se crea la sesión de PowerShell con el contenedor, el símbolo de PowerShell cambia para reflejar el nombre del contenedor.

```powershell
PS C:\> Enter-PSSession -ContainerName TP4Demo -RunAsAdministrator

[TP4Demo]: PS C:\Windows\system32>
```

### Crear imagen de IIS

Ahora se puede modificar el contenedor y se pueden capturar estas modificaciones para crear una nueva imagen de contenedor. En este ejemplo, IIS está instalado.

Para instalar el rol de IIS en el contenedor, use el comando `Install-WindowsFeature`.

```powershell
[TP4Demo]: PS C:\> Install-WindowsFeature web-server

Success Restart Needed Exit Code      Feature Result
------- -------------- ---------      --------------
True    No             Success        {Common HTTP Features, Default Document, D...
```

Una vez finalice la instalación de IIS, salga del contenedor escribiendo `exit`. Esto devuelve la sesión de PowerShell a la del host de contenedor.

```powershell
[TP4Demo]: PS C:\> exit
PS C:\>
```

Por último, detenga el contenedor con el comando `Stop-Container`.

```powershell
PS C:\> Stop-Container -Name TP4Demo
```

Ahora se puede capturar el estado de este contenedor en una nueva imagen de contenedor. Hágalo mediante el comando `New-ContainerImage`.

En este ejemplo, se crea una nueva imagen de contenedor denominada `WindowsServerCoreIIS` con un publicador de `Demo` y una versión `1.0`.

```powershell
PS C:\> New-ContainerImage -ContainerName TP4Demo -Name WindowsServerCoreIIS -Publisher Demo -Version 1.0

Name                 Publisher Version IsOSImage
----                 --------- ------- ---------
WindowsServerCoreIIS CN=Demo   1.0.0.0 False
```

Ahora que el contenedor está capturado en la nueva imagen, ya no es necesario. Puede quitarlo mediante el comando `Remove-Container`.

```powershell
PS C:\> Remove-Container -Name TP4Demo -Force
```


### Crear contenedor de IIS

Cree un contenedor nuevo a partir de la imagen de contenedor `WindowsServerCoreIIS`.

```powershell
PS C:\> New-Container -Name IIS -ContainerImageName WindowsServerCoreIIS -SwitchName "Virtual Switch"

Name State Uptime   ParentImageName
---- ----- ------   ---------------
IIS  Off   00:00:00 WindowsServerCoreIIS
```
Inicie el contenedor.

```powershell
PS C:\> Start-Container -Name IIS
```

### Configurar redes

La configuración de red predeterminada de los inicios rápidos de contenedores de Windows es que los contenedores estén conectados a un conmutador virtual configurado con traducción de direcciones de red (NAT). Debido a ello, para conectarse a una aplicación que se ejecute dentro de un contenedor, debe asignarse un puerto del host de contenedor a un puerto del contenedor. Para obtener más información sobre redes de contenedor, consulte [Red de contenedores](../management/container_networking.md).

Para este ejercicio, se hospeda un sitio web en IIS, que se ejecuta dentro de un contenedor. Para acceder al sitio web en el puerto 80, asigne el puerto 80 de la dirección IP del host de contenedor al puerto 80 de la dirección IP del contenedor.

Ejecute lo siguiente para devolver la dirección IP del contenedor.

```powershell
PS C:\> Invoke-Command -ContainerName IIS {ipconfig}

Windows IP Configuration


Ethernet adapter vEthernet (Virtual Switch-7570F6B1-E1CA-41F1-B47D-F3CA73121654-0):

   Connection-specific DNS Suffix  . : DNS
   Link-local IPv6 Address . . . . . : fe80::ed23:c1c6:310a:5c10%16
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

Para crear la asignación de puertos de NAT, use el comando `Add-NetNatStaticMapping`. En el ejemplo siguiente se comprueba si hay una regla de asignación de puertos existente y, si no existe, se crea. Tenga en cuenta que `-InternalIPAddress` debe coincidir con la dirección IP del contenedor.

```powershell
if (!(Get-NetNatStaticMapping | where {$_.ExternalPort -eq 80})) {
Add-NetNatStaticMapping -NatName "ContainerNat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
}
```

Cuando se ha creado la asignación de puertos, debe configurar una regla de firewall de entrada para el puerto configurado. Para hacerlo con el puerto 80, ejecute el siguiente script. Tenga en cuenta que si ha creado una regla NAT para un puerto externo distinto del 80, la regla del firewall debe crearse de forma que coincida.

```powershell
if (!(Get-NetFirewallRule | where {$_.Name -eq "TCP80"})) {
    New-NetFirewallRule -Name "TCP80" -DisplayName "HTTP on TCP/80" -Protocol tcp -LocalPort 80 -Action Allow -Enabled True
}
```

Si está trabajando en Azure y aún no ha creado un grupo de seguridad de red, debe crear uno ahora. Para obtener más información sobre los grupos de seguridad de red, consulte el artículo [¿Qué es un grupo de seguridad de red?](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-nsg/).

### Crear aplicación

Ahora que se ha creado un contenedor a partir de la imagen de IIS y se han configurado las redes, abra un explorador y vaya a la dirección IP del host de contenedor. Debería ver la pantalla de presentación de IIS.

![](media/iis1.png)

Una vez que se comprueba que las instancias de IIS están en ejecución, se puede crear una aplicación "Hello World" y hospedarla en la instancia de IIS. Para ello, cree una sesión de PowerShell con el contenedor.

```powershell
PS C:\> Enter-PSSession -ContainerName IIS -RunAsAdministrator
[IIS]: PS C:\Windows\system32>
```

Ejecute el siguiente comando para quitar la pantalla de presentación de IIS.

```powershell
[IIS]: PS C:\> del C:\inetpub\wwwroot\iisstart.htm
```
Ejecute el comando siguiente para reemplazar el sitio de IIS predeterminado por un nuevo sitio estático.

```powershell
[IIS]: PS C:\> "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

Vaya de nuevo a la dirección IP del host de contenedor, ahora debería ver la aplicación "Hello World". Nota: Es posible que tenga que cerrar las conexiones del explorador existentes o borrar la memoria caché del explorador para ver la aplicación actualizada.

![](media/HWWINServer.png)

Salga de la sesión remota de contenedor.

```powershell
[IIS]: PS C:\> exit
PS C:\>
```

### Quitar contenedor

Antes de poder quitar un contenedor, debe detenerse.

```powershell
PS C:\> Stop-Container -Name IIS
```

Cuando el contenedor se detenga, lo podrá quitar con el comando `Remove-Container`.

```powershell
PS C:\> Remove-Container -Name IIS -Force
```

Por último, puede quitar una imagen de contenedor con el comando `Remove-ContainerImage`.

```powershell
PS C:\> Remove-ContainerImage -Name WindowsServerCoreIIS -Force
```

## Contenedor de Hyper-V

Los contenedores de Hyper-V ofrecen una capa de aislamiento adicional sobre los contenedores de Windows Server. Cada contenedor de Hyper-V se crea dentro de una máquina virtual altamente optimizada. Si un contenedor de Windows Server comparte un kernel con el host de contenedor y todos los demás contenedores de Windows Server se ejecutan en ese host, un contenedor de Hyper-V está completamente aislado de los otros contenedores. Los contenedores de Hyper-V se crean y administran de forma idéntica a los contenedores de Windows Server. Para obtener más información sobre los contenedores de Hyper-V, consulte [Contenedores de Hyper-V](../management/hyperv_container.md).

> Microsoft Azure no admite contenedores de Hyper-V. Para completar los ejercicios de contenedor de Hyper-V, necesita un host de contenedor local.

### Crear contenedor

En el momento de la publicación de TP4, los contenedores de Hyper-V deben utilizar una imagen de sistema operativo Nano Server Core. Para comprobar si se ha instalado la imagen de sistema operativo de Nano Server, use el comando `Get-ContainerImage`.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

Para crear un contenedor de Hyper-V, use el comando `New-Container` y especifique un tiempo de ejecución de HyperV.

```powershell
PS C:\> New-Container -Name HYPV -ContainerImageName NanoServer -SwitchName "Virtual Switch" -RuntimeType HyperV

Name State Uptime   ParentImageName
---- ----- ------   ---------------
HYPV Off   00:00:00 NanoServer
```

Cuando se haya creado el contenedor, **no lo inicie**.

### Crear una carpeta compartida

Las carpetas compartidas exponen un directorio del host de contenedor en el contenedor. Cuando se cree una carpeta compartida, los archivos ubicados en la carpeta compartida estarán disponibles en el contenedor. En este ejemplo, se utiliza una carpeta compartida para copiar los paquetes de IIS de Nano Server en el contenedor. Estos paquetes se usarán después para instalar IIS. Para obtener más información sobre la carpeta compartida, consulte [Carpetas compartidas de contenedores](../management/manage_data.md).

Cree un directorio denominado `c:\share\en-us` en el host de contenedor.

```powershell
S C:\> New-Item -Type Directory c:\share\en-us

    Directory: C:\share

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:27 PM                en-us
```

Use el comando `Add-ContainerSharedFolder` para crear una nueva carpeta compartida en el nuevo contenedor.

> El contenedor debe estar en estado detenido cuando se crea una carpeta compartida.

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName HYPV -SourcePath c:\share -DestinationPath c:\iisinstall

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
HYPV          c:\share   c:\iisinstall   ReadWrite
```

Cuando se cree la carpeta compartida, inicie el contenedor.

```powershell
PS C:\> Start-Container -Name HYPV
```
Cree una sesión remota de PowerShell con el contenedor mediante el comando `Enter-PSSession`.

```powershell
PS C:\> Enter-PSSession -ContainerName HYPV -RunAsAdministrator
[HYPV]: PS C:\windows\system32\config\systemprofile\Documents>cd /
```
Cuando se encuentre en la sesión remota, observe que la carpeta compartida `c:\iisinstall\en-us` se ha creado, aunque está vacía.

```powershell
[HYPV]: PS C:\> ls c:\iisinstall

    Directory: C:\iisinstall

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:27 PM                en-us
```

### Crear imagen de IIS

Como el contenedor ejecuta una imagen de sistema operativo Nano Server, se necesitan los paquetes de IIS de Nano Server para instalar IIS. Puede encontrarlos en los elementos multimedia de instalación Windows Sever 2016 TP4 en el directorio `NanoServer\Packages`.

Copie `Microsoft-NanoServer-IIS-Package.cab` de `NanoServer\Packages` a `c:\share` en el host de contenedor.

Copie `NanoServer\Packages\en-us\Microsoft-NanoServer-IIS-Package.cab` en `c:\share\en-us` en el host de contenedor.

Cree un archivo en la carpeta c:\share denominado unattend.xml y copie este texto en dicho archivo.

```powershell
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <servicing>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" />
            <source location="c:\iisinstall\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="en-US" />
            <source location="c:\iisinstall\en-us\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
    </servicing>
</unattend>
```

Cuando finalice la operación, el directorio `c:\share` en el host de contenedor debería estar configurado de esta manera.

```
c:\share
|-- en-us
|    |-- Microsoft-NanoServer-IIS-Package.cab
|
|-- Microsoft-NanoServer-IIS-Package.cab
|-- unattend.xml
```

De nuevo en la sesión remota en el contenedor, tenga en cuenta que los paquetes de IIS y los archivos unattended.xml ahora son visibles en el directorio c:\iisinstall.

```powershell
[HYPV]: PS C:\> ls c:\iisinstall

    Directory: C:\iisinstall

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:32 PM                en-us
-a----       10/29/2015  11:51 PM        1922047 Microsoft-NanoServer-IIS-Package.cab
-a----       11/18/2015   5:31 PM            789 unattend.xml
```

Ejecute el siguiente comando para instalar IIS.

```powershell
[HYPV]: PS C:\> dism /online /apply-unattend:c:\iisinstall\unattend.xml

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0


[                           1.0%                           ]

[=====                      10.1%                          ]

[=====                      10.3%                          ]

[===============            26.2%                          ]
```

Cuando se haya completado la instalación de IIS, inicie manualmente IIS con el siguiente comando.

```powershell
[HYPV]: PS C:\> Net start w3svc
The World Wide Web Publishing Service service is starting.
The World Wide Web Publishing Service service was started successfully.
```

Salga de la sesión del contenedor.

```powershell
[HYPV]: PS C:\> exit
```

Detenga el contenedor.

```powershell
PS C:\> Stop-Container -Name HYPV
```

Ahora se puede capturar el estado de este contenedor en una nueva imagen de contenedor.

En este ejemplo, se crea una nueva imagen de contenedor denominada `NanoServerIIS` con un publicador `Demo` y una versión `1.0`.

```powershell
PS C:\> New-ContainerImage -ContainerName HYPV -Name NanoServerIIS -Publisher Demo -Version 1.0

Name          Publisher Version IsOSImage
----          --------- ------- ---------
NanoServerIIS CN=Demo   1.0.0.0 False
```

### Crear contenedor de IIS

Cree un nuevo contenedor de Hyper-V a partir de la imagen IIS con el comando `New-Container`.

```powershell
PS C:\> New-Container -Name IISApp -ContainerImageName NanoServerIIS -SwitchName "Virtual Switch" -RuntimeType HyperV

Name   State Uptime   ParentImageName
----   ----- ------   ---------------
IISApp Off   00:00:00 NanoServerIIS
```

Inicie el contenedor.

```powershell
PS C:\> Start-Container -Name IISApp
```

### Configurar redes

La configuración de red predeterminada de los inicios rápidos de contenedores de Windows es que los contenedores estén conectados a un conmutador virtual, configurado con traducción de direcciones de red (NAT). Debido a ello, para conectarse a una aplicación que se ejecute dentro de un contenedor, debe asignarse un puerto del host de contenedor a un puerto del contenedor.

Para este ejercicio, se hospeda un sitio web en IIS, que se ejecuta dentro de un contenedor. Para acceder al sitio web en el puerto 80, asigne el puerto 80 de la dirección IP del host de contenedor al puerto 80 de la dirección IP del contenedor.

Ejecute lo siguiente para devolver la dirección IP del contenedor.

```powershell
PS C:\> Invoke-Command -ContainerName IISApp {ipconfig}

Windows IP Configuration


Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : DNS
   Link-local IPv6 Address . . . . . : fe80::c574:5a5e:d5f5:18a0%4
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

Para crear la asignación de puertos de NAT, use el comando `Add-NetNatStaticMapping`. Los ejemplos siguientes comprueban si hay una regla de asignación de puertos y, si no hay ninguna, la crean. Tenga en cuenta que `-InternalIPAddress` debe coincidir con la dirección IP del contenedor.

```powershell
if (!(Get-NetNatStaticMapping | where {$_.ExternalPort -eq 80})) {
Add-NetNatStaticMapping -NatName "ContainerNat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
}
```
También debe abrir el puerto 80 en el host de contenedor. Tenga en cuenta que si ha creado una regla NAT para un puerto externo distinto del 80, la regla del firewall debe crearse de forma que coincida.

```powershell
if (!(Get-NetFirewallRule | where {$_.Name -eq "TCP80"})) {
    New-NetFirewallRule -Name "TCP80" -DisplayName "HTTP on TCP/80" -Protocol tcp -LocalPort 80 -Action Allow -Enabled True
}
```

### Crear aplicación

Ahora que se ha creado un contenedor a partir de la imagen de IIS y se han configurado las redes, abra un explorador y vaya a la dirección IP del host de contenedor, donde debería ver la pantalla de presentación de IIS.

![](media/iis1.png)

Una vez que se comprueba que las instancias de IIS están en ejecución, se puede crear una aplicación "Hello World" y hospedarla en la instancia de IIS. Para ello, cree una sesión de PowerShell con el contenedor.

```powershell
PS C:\> Enter-PSSession -ContainerName IISApp -RunAsAdministrator
[IISApp]: PS C:\windows\system32\config\systemprofile\Documents>
```

Ejecute el siguiente comando para quitar la pantalla de presentación de IIS.

```powershell
[IIS]: PS C:\> del C:\inetpub\wwwroot\iisstart.htm
```
Ejecute el comando siguiente para reemplazar el sitio de IIS predeterminado por un nuevo sitio estático.

```powershell
[IISApp]: PS C:\> "Hello World From a Hyper-V Container" > C:\inetpub\wwwroot\index.html
```

Vaya de nuevo a la dirección IP del host de contenedor, ahora debería ver la aplicación "Hello World". Nota: Es posible que tenga que cerrar las conexiones del explorador existentes o borrar la memoria caché del explorador para ver la aplicación actualizada.

![](media/HWWINServer.png)

Salga de la sesión remota de contenedor.

```powershell
exit
```






<!--HONumber=Feb16_HO3-->


