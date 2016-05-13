




# Implementación de host de contenedor - Nano Server

**Esto es contenido preliminar y está sujeto a cambios.**

La implementación de un host de contenedor de Windows implica pasos distintos, según el sistema operativo y el tipo de sistema host (físico o virtual). Los pasos que se describen en este documento se usan para implementar un host de contenedor de Windows para Nano Server, en un sistema físico o virtual. Para instalar un host de contenedor de Windows en Windows Server, consulte [Implementación de host de contenedor - Windows Server](./deployment.md).

Para obtener detalles sobre los requisitos del sistema, consulte [Requisitos del sistema host de contenedor de Windows](./system_requirements.md).

Hay scripts de PowerShell disponibles para automatizar la implementación de un host de contenedor de Windows.
- [Implementación de un host de contenedor en una nueva máquina virtual de Hyper-V](../quick_start/container_setup.md).
- [Implementación de un host de contenedor en un sistema existente](../quick_start/inplace_setup.md).


# Host de Nano Server

Los pasos que se indican en esta tabla se pueden utilizar para implementar un host de contenedor en Nano Server. Se incluyen las configuraciones necesarias para los contenedores de Hyper-V y Windows Server.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>Acción de implementación</strong></td>
<td width="70%"><strong>Detalles</strong></td>
</tr>
<tr>
<td>[Preparación de Nano Server para contenedores](#nano)</td>
<td>Preparar un VHD de Nano Server con las capacidades de Hyper-V y contenedores.</td>
</tr>
<tr>
<td>[Creación de un conmutador virtual](#vswitch)</td>
<td>Los contenedores se conectan a un conmutador virtual para la conectividad de red.</td>
</tr>
<tr>
<td>[Configuración de NAT](#nat)</td>
<td>Si se configura un conmutador virtual con traducción de direcciones de red, la propia NAT necesita una configuración.</td>
</tr>
<tr>
<td>[Instalación de imágenes del sistema operativo del contenedor](#img)</td>
<td>Las imágenes del sistema operativo proporcionan la base para las implementaciones de contenedores.</td>
</tr>
<tr>
<td>[Instalación de Docker](#docker)</td>
<td>Este paso es opcional, pero es necesario para crear y administrar contenedores de Windows con Docker. </td>
</tr>
</table>

Estos pasos deben realizarse si se van a usar contenedores de Hyper-V. Tenga en cuenta que los pasos marcados con * son necesarios si el host del contenedor es una máquina virtual de Hyper-V.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>Acción de implementación</strong></td>
<td width="70%"><strong>Detalles</strong></td>
</tr>
<tr>
<td>[Habilitación del rol de Hyper-V](#hypv) </td>
<td>Hyper-V solo es necesario si se usarán los contenedores de Hyper-V.</td>
</tr>
<tr>
<td>[Habilitación de la virtualización anidada*](#nest)</td>
<td>Si el propio host de contenedor es una máquina virtual de Hyper-V, debe habilitarse la virtualización anidada.</td>
</tr>
<tr>
<td>[Configuración de procesadores virtuales*](#proc)</td>
<td>Si el host de contenedor es una máquina virtual de Hyper-V, se deben configurar al menos dos procesadores virtuales.</td>
</tr>
<tr>
<td>[Deshabilitación de la memoria dinámica*](#dyn)</td>
<td>Si el host del contenedor es una máquina virtual de Hyper-V, la memoria dinámica debe estar deshabilitada.</td>
</tr>
<tr>
<td>[Configuración de la suplantación de direcciones MAC*](#mac)</td>
<td>Si el host de contenedor está virtualizado, la suplantación de direcciones MAC deberá habilitarse.</td>
</tr>
</table>

## Pasos de implementación

### <a name=nano></a> Preparación de Nano Server

La implementación de Nano Server implica la creación de un disco duro virtual preparado, que incluye el sistema operativo Nano Server y paquetes de características adicionales. En esta guía se detalla rápidamente la preparación de un disco duro virtual de Nano Server, que se puede usar para contenedores de Windows. Para más información sobre Nano Server y para explorar las diferentes opciones de implementación de Nano Server, consulte la [Documentación de Nano Server](https://technet.microsoft.com/en-us/library/mt126167.aspx).

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
Ejecute lo siguiente para crear un disco duro virtual de Nano Server. El parámetro `-Containers` indica que el paquete del contenedor está instalado y el parámetro `-Compute` se encarga del paquete de Hyper-V. Hyper-V solo es necesario si se usan contenedores de Hyper-V.

```powershell
PS C:\> Import-Module C:\nano\NanoServerImageGenerator.psm1

PS C:\> New-NanoServerImage -MediaPath $WindowsMedia -BasePath c:\nano -TargetPath C:\nano\NanoContainer.vhdx -MaxSize 10GB -GuestDrivers -ReverseForwarders -Compute -Containers
```
Cuando se complete la operación, cree una máquina virtual a partir del archivo `NanoContainer.vhdx`. Esta máquina virtual ejecutará el sistema operativo Nano Server y paquetes opcionales.

### <a name=vswitch></a>Creación de un conmutador virtual

Cada uno de los contenedores debe estar conectado a un conmutador virtual para comunicarse a través de una red. Un conmutador virtual se crea con el comando `New-VMSwitch`. Los contenedores admiten un conmutador virtual con el tipo `External` o `NAT`. Para obtener más información sobre las redes de contenedor de Windows, consulte [Red de contenedores](../management/container_networking.md).

Este ejemplo crea un conmutador virtual con el nombre "Virtual Switch", un tipo NAT y una subred de NAT 172.16.0.0/12.

```powershell
PS C:\> New-VMSwitch -Name "Virtual Switch" -SwitchType NAT -NATSubnetAddress "172.16.0.0/12"
```

### <a name=nat></a>Configuración de NAT

Además de crear el conmutador virtual, si el tipo de conmutador es NAT, se debe crear un objeto NAT. Esta operación se completa con el comando `New-NetNat`. En este ejemplo se crea un objeto NAT, con el nombre `ContainerNat` y un prefijo de dirección que coincida con la subred NAT asignada al conmutador de contenedor.

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

### <a name=img></a>Instalación de imágenes de sistema operativo

Una imagen de sistema operativo se utiliza como base para cualquier contenedor de Hyper-V o Windows Server. La imagen se usa para implementar un contenedor, que luego se puede modificar y capturar en una nueva imagen de contenedor. Se han creado imágenes de sistema operativo con Windows Server Core y Nano Server como el sistema operativo subyacente.

Las imágenes de sistema operativo del contenedor se pueden encontrar e instalar con el módulo de PowerShell ContainerProvider. Antes de utilizar este módulo, debe instalarse. Se pueden utilizar los siguientes comandos para instalar el módulo.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

Use `Find-ContainerImage` para obtener una lista de imágenes del administrador de paquetes de PowerShell OneGet.

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```
**Nota**: En este momento, solo la imagen del sistema operativo Nano Server es compatible con un host de contenedor de Nano Server. Para descargar e instalar la imagen del sistema operativo base Nano Server, ejecute lo siguiente.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

Compruebe que la imagen se instaló con el comando `Get-ContainerImage`.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
```
Para más información sobre la administración de imágenes de contenedor, consulte [Imágenes del contenedor de Windows](../management/manage_images.md).


### <a name=docker></a>Instalación de Docker

El demonio de Docker y una interfaz de línea de comandos no se incluyen con Windows y no se instalan con la característica de contenedor de Windows. Docker no es un requisito para trabajar con contenedores de Windows. Si quiere instalar Docker, siga las instrucciones de este artículo [Docker y Windows](./docker_windows.md).


## Host de contenedor de Hyper-V

### <a name=hypv></a>Habilitación del rol de Hyper-V

En Nano Server, esta acción se puede llevar a cabo al crear la imagen de Nano Server. Consulte las instrucciones en [Preparación de Nano Server para contenedores](#nano).

### <a name=nest></a>Virtualización anidada

Si el propio host de contenedor se ejecutará en una máquina virtual de Hyper-V y también hospedará contenedores de Hyper-V, la virtualización anidada debe habilitarse. Esto se puede completar con el siguiente comando de PowerShell.

**Nota**: Las máquinas virtuales deben estar desconectadas cuando se ejecute este comando.

```powershell
PS C:\> Set-VMProcessor -VMName <VM Name> -ExposeVirtualizationExtensions $true
```

### <a name=proc></a>Configuración de procesadores virtuales

Si el propio host de contenedor se ejecutará en una máquina virtual de Hyper-V y también hospedará contenedores de Hyper-V, la máquina virtual requerirá al menos dos procesadores. Esto se puede configurar a través de la configuración de la máquina virtual o con el siguiente comando.

**Nota**: Las máquinas virtuales deben estar desconectadas cuando se ejecute este comando.

```poweshell
PS C:\> Set-VMProcessor -VMName <VM Name> -Count 2
```

### <a name=dyn></a>Deshabilitación de la memoria dinámica

Si el host de contenedor es una máquina virtual de Hyper-V, la memoria dinámica debe estar deshabilitada en la máquina virtual del host de contenedor. Esto se puede configurar a través de la configuración de la máquina virtual o con el siguiente comando.

**Nota**: Las máquinas virtuales deben estar desconectadas cuando se ejecute este comando.

```poweshell
PS C:\> Set-VMMemory <VM Name> -DynamicMemoryEnabled $false
```

### <a name=mac></a>Suplantación de direcciones MAC

Finalmente, si el host de contenedor se ejecuta en una máquina virtual de Hyper-V, la suplantación de direcciones MAC se debe habilitar. Esto permite que cada contenedor reciba una dirección IP. Para habilitar la suplantación de direcciones MAC, ejecute el siguiente comando en el host de Hyper-V. La propiedad VMName será el nombre del host de contenedor.

```powershell
PS C:\> Get-VMNetworkAdapter -VMName <VM Name> | Set-VMNetworkAdapter -MacAddressSpoofing On
```






<!--HONumber=Feb16_HO4-->


