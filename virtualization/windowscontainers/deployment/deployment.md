## Implementación de host de contenedor

**Esto es contenido preliminar y está sujeto a cambios.**

La implementación de un host de contenedor de Windows implica pasos distintos, según el sistema operativo y el tipo de sistema host (físico o virtual). En este documento se detallarán opciones de implementación de Windows Server 2016 y Nano Server para sistemas físicos y virtuales.

Para obtener detalles sobre los requisitos del sistema, consulte [Requisitos del sistema host de contenedor de Windows](./system_requirements.md).

Hay scripts de PowerShell disponibles para automatizar la implementación de un host de contenedor de Windows.
- [Implementar un host de contenedor en una nueva máquina virtual de Hyper-V](../quick_start/container_setup.md).
- [Implementar un host de contenedor en un sistema existente](../quick_start/inplace_setup.md).
- [Implementar un host de contenedor en Azure](../quick_start/azure_setup.md).

### Host de Windows Server

Los pasos que se indican en esta tabla se pueden utilizar para implementar un host de contenedor en Windows Server 2016 TP4 y Windows Server Core 2016. Se incluyen las configuraciones necesarias para los contenedores de Hyper-V y Windows Server.

\* Solo se requiere si se implementarán contenedores de Hyper-V.  
\*\* Solo se requiere si se utilizará Docker para crear y administrar contenedores.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>Acción de implementación</strong></td>
<td width="70%"><strong>Detalles</strong></td>
</tr>
<tr>
<td>[Instalar la característica de contenedor](#role)</td>
<td>La característica de contenedor permite el uso del contenedor de Hyper-V y Windows Server.</td>
</tr>
<tr>
<td>[Habilitar la virtualización anidada *](#nest)</td>
<td>Si el propio host de contenedor es una máquina virtual de Hyper-V, debe habilitarse la virtualización anidada.</td>
</tr>
<tr>
<td>[Configurar procesadores virtuales *](#proc)</td>
<td>Si el host de contenedor es una máquina virtual de Hyper-V, se deben configurar al menos dos procesadores virtuales.</td>
</tr>
<tr>
<td>[Habilitar el rol de Hyper-V *](#hypv) </td>
<td>Hyper-V solo es necesario si se usarán los contenedores de Hyper-V.</td>
</tr>
<tr>
<td>[Crear conmutador virtual](#vswitch)</td>
<td>Los contenedores se conectan a un conmutador virtual para la conectividad de red.</td>
</tr>
<tr>
<td>[Configurar NAT](#nat)</td>
<td>Si se configura un conmutador virtual con traducción de direcciones de red, la propia NAT necesita una configuración.</td>
</tr>
<tr>
<td>[Configurar la suplantación de direcciones MAC](#mac)</td>
<td>Si el host de contenedor está virtualizado, la suplantación de direcciones MAC deberá habilitarse.</td>
</tr>
<tr>
<td>[Instalar imágenes del sistema operativo del contenedor](#img)</td>
<td>Las imágenes del sistema operativo proporcionan la base para las implementaciones de contenedores.</td>
</tr>
<tr>
<td>[Instalar Docker **](#docker)</td>
<td>Este paso es opcional, pero es necesario para crear y administrar contenedores de Windows con Docker.</td>
</tr>
</table>

### Host de Nano Server

Los pasos que se indican en esta tabla se pueden utilizar para implementar un host de contenedor en Nano Server. Se incluyen las configuraciones necesarias para los contenedores de Hyper-V y Windows Server.

\* Solo se requiere si se implementarán contenedores de Hyper-V.  
\*\* Solo se requiere si se utilizará Docker para crear y administrar contenedores.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>Acción de implementación</strong></td>
<td width="70%"><strong>Detalles</strong></td>
</tr>
<tr>
<td>[Preparar Nano Server para contenedores](#nano)</td>
<td>Preparar un VHD de Nano Server con las capacidades de Hyper-V y contenedores.</td>
</tr>
<tr>
<td>[Habilitar la virtualización anidada *](#nest)</td>
<td>Si el propio host de contenedor es una máquina virtual de Hyper-V, debe habilitarse la virtualización anidada.</td>
</tr>
<tr>
<td>[Configurar procesadores virtuales *](#proc)</td>
<td>Si el host de contenedor es una máquina virtual de Hyper-V, se deben configurar al menos dos procesadores virtuales.</td>
</tr>
<tr>
<td>[Crear conmutador virtual](#vswitch)</td>
<td>Los contenedores se conectan a un conmutador virtual para la conectividad de red.</td>
</tr>
<tr>
<td>[Configurar NAT](#nat)</td>
<td>Si se configura un conmutador virtual con traducción de direcciones de red, la propia NAT necesita una configuración.</td>
</tr>
<tr>
<td>[Configurar la suplantación de direcciones MAC](#mac)</td>
<td>Si el host de contenedor está virtualizado, la suplantación de direcciones MAC deberá habilitarse.</td>
</tr>
<tr>
<td>[Instalar imágenes del sistema operativo del contenedor](#img)</td>
<td>Las imágenes del sistema operativo proporcionan la base para las implementaciones de contenedores.</td>
</tr>
<tr>
<td>[Instalar Docker **](#docker)</td>
<td>Este paso es opcional, pero es necesario para crear y administrar contenedores de Windows con Docker. </td>
</tr>
</table>

## Pasos de implementación

### <a name=role></a>Instalar la característica de contenedor

La característica de contenedor puede instalarse en Windows Server 2016 o Windows Server 2016 Core, mediante el Administrador de Windows Server o PowerShell.

Para instalar el rol mediante PowerShell, ejecute el siguiente comando en una sesión de PowerShell con privilegios elevados.

```powershell
PS C:\> Install-WindowsFeature containers
```
El sistema debe reiniciarse cuando se complete la instalación del rol de contenedor.

```powershell
PS C:\> shutdown /r 
```

Cuando se reinicie el sistema, use el comando `Get-ContainerHost` para comprobar que el rol de contenedor se instaló correctamente:

```powershell
PS C:\> Get-ContainerHost

Name            ContainerImageRepositoryLocation
----            --------------------------------
WIN-LJGU7HD7TEP C:\ProgramData\Microsoft\Windows\Hyper-V\Container Image Store
```

### <a name=nano></a> Preparar Nano Server

La implementación de Nano Server implica la creación de un disco duro virtual preparado, que incluye el sistema operativo Nano Server y paquetes de características adicionales. En esta guía se detalla rápidamente la preparación de un disco duro virtual de Nano Server, que se puede usar para contenedores de Windows.

Para más información sobre Nano Server y para explorar las diferentes opciones de implementación de Nano Server, consulte la [Documentación de Nano Server](https://technet.microsoft.com/en-us/library/mt126167.aspx).

Cree una carpeta denominada `nano`.

```powershell
PS C:\> New-Item -ItemType Directory c:\nano
```

Busque los archivos `NanoServerImageGenerator.psm1` y `Convert-WindowsImage.ps1` de la carpeta de Nano Server en el servidor de Windows Media. Cópielos en `c:\nano`.

```powershell
#Set path to Windows Server 2016 Media
PS C:\> $WindowsMedia = "C:\Users\Administrator\Desktop\TP4 Release Media"

PS C:\> Copy-Item $WindowsMedia\NanoServer\Convert-WindowsImage.ps1 c:\nano
PS C:\> Copy-Item $WindowsMedia\NanoServer\NanoServerImageGenerator.psm1 c:\nano
```
Ejecute lo siguiente para crear un disco duro virtual de Nano Server. El parámetro `–Containers` indica que se instalará el paquete del contenedor y el parámetro `–Compute` se encarga del paquete de Hyper-V. Hyper-V solo es necesario si se crearán contenedores de Hyper-V.

```powershell
PS C:\> Import-Module C:\nano\NanoServerImageGenerator.psm1
PS C:\> New-NanoServerImage -MediaPath $WindowsMedia -BasePath c:\nano -TargetPath C:\nano\NanoContainer.vhdx -MaxSize 10GB -GuestDrivers -ReverseForwarders -Compute -Containers
```
Cuando se complete, cree una máquina virtual a partir del archivo `NanoContainer.vhdx`. Esta máquina virtual ejecutará el sistema operativo Nano Server con paquetes opcionales.

### <a name=nest></a>Configurar virtualización anidada

Si el propio host de contenedor se ejecutará en una máquina virtual de Hyper-V y también hospedará contenedores de Hyper-V, la virtualización anidada debe habilitarse. Esto se puede completar con el siguiente comando de PowerShell.

> Las máquinas virtuales deben estar desconectadas cuando se ejecute este comando.

```powershell
PS C:\> Set-VMProcessor -VMName <container host vm> -ExposeVirtualizationExtensions $true
```

### <a name=proc></a>Configurar procesadores virtuales

Si el propio host de contenedor se ejecutará en una máquina virtual de Hyper-V y también hospedará contenedores de Hyper-V, la máquina virtual requerirá al menos dos procesadores. Esto se puede configurar a través de la configuración de la máquina virtual o con el siguiente script de PowerShell.

```poweshell
PS C:\> Set-VMProcessor –VMName <VM Name> -Count 2
```

### <a name=hypv></a>Habilitar el rol de Hyper-V

Si se implementarán los contenedores de Hyper-V, el rol de Hyper-V deberá estar habilitado en el host del contenedor. Si el host del contenedor es una máquina virtual, asegúrese de que la virtualización anidada se habilitó. El rol de Hyper-V puede instalarse en Windows Server 2016 o Windows Server 2016 Core con el siguiente comando de PowerShell. Para ver la configuración de Nano Server, consulte [Preparar Nano Server](#nano).

```powershell
PS C:\> Install-WindowsFeature hyper-v
```

### <a name=vswitch></a>Crear conmutador virtual

Cada uno de los contenedores debe estar conectado a un conmutador virtual para comunicarse a través de una red. Un conmutador virtual se crea con el comando `New-VMSwitch`. Los contenedores admiten un conmutador virtual con el tipo `External` o `NAT`.

Este ejemplo crea un conmutador virtual con el nombre "Virtual Switch", un tipo NAT y una subred de NAT 172.16.0.0/12.

```powershell
PS C:\> New-VMSwitch -Name "Virtual Switch" -SwitchType NAT -NATSubnetAddress 172.16.0.0/12
```

### <a name=nat></a>Configurar NAT

Además de crear el conmutador virtual, si el tipo de conmutador es NAT, se debe crear un objeto NAT. Esto se completa con el comando `New-NetNat`. En este ejemplo se crea un objeto NAT, con el nombre `ContainerNat` y un prefijo de dirección que coincide con la subred NAT asignada al conmutador de contenedor.

```powershell
PS C:\> New-NetNat -Name ContainerNat -InternalIPInterfaceAddressPrefix "172.16.0.0/12"

Name                             : ContainerNat
ExternalIPInterfaceAddressPrefix :
InternalIPInterfaceAddressPrefix : 172.16.0.0/12
IcmpQueryTimeout                 : 30
TcpEstablishedConnectionTimeout  : 1800
TcpTransientConnectionTimeout    : 120
TcpFilteringBehavior             : AddressDependentFiltering
UdpFilteringBehavior             : AddressDependentFiltering
UdpIdleSessionTimeout            : 120
UdpInboundRefresh                : False
Store                            : Local
Active                           : True
```

<a name=mac></a>Finalmente, si el host del contenedor se ejecuta en una máquina virtual de Hyper-V, la suplantación de direcciones MAC se debe habilitar. Esto permite que cada contenedor reciba una dirección IP. Para habilitar la suplantación de direcciones MAC, ejecute el siguiente comando en el host de Hyper-V. La propiedad VMName será el nombre del host de contenedor.

```powershell
PS C:\> Get-VMNetworkAdapter -VMName <contianer host vm> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name=img></a>Instalar imágenes de sistema operativo

Una imagen de sistema operativo se utiliza como base para cualquier contenedor de Hyper-V o Windows Server. La imagen se usa para implementar un contenedor, que luego se puede modificar y capturar en una nueva imagen de contenedor. Se han creado imágenes de sistema operativo con Windows Server Core y Nano Server como el sistema operativo subyacente.

Las imágenes de sistema operativo del contenedor se pueden encontrar e instalar con el módulo de PowerShell ContainerProvider. Antes de utilizar este módulo, debe instalarse. Se pueden utilizar los siguientes comandos para instalar el módulo.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

Obtenga una lista de imágenes a partir del administrador de paquetes de PowerShell OneGet:
```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

Para descargar e instalar la imagen del sistema operativo base Nano Server, ejecute lo siguiente.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0
Downloaded in 0 hours, 0 minutes, 10 seconds.
```

Del mismo modo, este comando descarga e instala la imagen del sistema operativo base Windows Server Core.

> **Problema:** los cmdlets Save-ContainerImage e Install-ContainerImage no funcionan con una imagen de contenedor WindowsServerCore en una sesión remota de PowerShell.<br /> **Solución alternativa:** inicie sesión en el equipo con Escritorio remoto y use directamente el cmdlet Save-ContainerImage.

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0
Downloaded in 0 hours, 2 minutes, 28 seconds.
```

Compruebe que las imágenes se instalaron con el comando `Get-ContainerImage`.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```
Para más información sobre la administración de imágenes de contenedor, vea [Imágenes del contenedor de Windows](../management/manage_images.md).


### <a name=docker></a>Instalar Docker

El demonio de Docker y una interfaz de línea de comandos no se incluyen con Windows y no se instalan con la característica de contenedor de Windows. Docker no es un requisito para trabajar con contenedores de Windows. Si quiere instalar Docker, siga las instrucciones de este artículo [Docker y Windows](./docker_windows.md).




