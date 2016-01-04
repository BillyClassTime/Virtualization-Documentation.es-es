# Imágenes del contenedor

**Esto es contenido preliminar y está sujeto a cambios.**

Las imágenes de contenedor se utilizan para implementar contenedores. Estas imágenes pueden incluir un sistema operativo, aplicaciones y todas las dependencias de aplicaciones. Por ejemplo, puede desarrollar una imagen de contenedor que se haya configurado previamente con Nano Server, IIS y una aplicación que se ejecuta en IIS. Esta imagen de contenedor se puede almacenar después en un registro de contenedor para su uso posterior, se puede implementar en cualquier host de contenedor de Windows (local, en la nube o incluso en un servicio de contenedor) y también se puede utilizar como base para una nueva imagen de contenedor.

Hay dos tipos de imágenes de contenedor:

- Imágenes del sistema operativo base: las ofrece Microsoft e incluyen los componentes principales del sistema operativo.
- Imágenes del contenedor: una imagen de contenedor que se ha creado a partir de una imagen del sistema operativo base.

## PowerShell

### Enumerar imágenes

Ejecute `get-containerImage` para devolver una lista de imágenes en el host del contenedor. El tipo de imagen de contenedor se diferencia cuando incluye la propiedad `IsOSImage`.

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### Instalación de imágenes de sistema operativo base

Las imágenes de sistema operativo del contenedor se pueden encontrar e instalar con el módulo de PowerShell ContainerProvider. Antes de utilizar este módulo, deberá instalarse. Se pueden utilizar los siguientes comandos para instalar el módulo.

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

Del mismo modo, este comando descargará e instalará la imagen del sistema operativo base de Windows Server Core.

> **Problema:** los cmdlets Save-ContainerImage e Install-ContainerImage no funcionan con una imagen de contenedor WindowsServerCore en una sesión de comunicación remota de PowerShell. **Solución alternativa:** inicie sesión en el equipo con Escritorio remoto y use directamente el cmdlet Save-ContainerImage.

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
Para obtener más información sobre la administración de imágenes de contenedor, vea [Imágenes del contenedor de Windows](../management/manage_images.md).

### Crear nueva imagen

```powershell
PS C:\> New-ContainerImage -Container $container -Publisher Demo -Name DemoImage -Version 1.0
```

### Quitar imagen

Las imágenes del contenedor no se pueden quitar si cualquier contenedor, incluso en un estado detenido, tiene una dependencia de la imagen.

Quite una sola imagen con PowerShell.

```powershell
PS C:\> Get-ContainerImage -Name newimage | Remove-ContainerImage -Force
```

### Dependencia de imagen

Cuando se crea una nueva imagen, se vuelve dependiente de la imagen a partir de la cual se creó. Esta dependencia se puede ver con el comando `get-containerimage`. Si no se muestra una imagen primaria, esto indica que la imagen es una imagen de sistema operativo.

```powershell
PS C:\> Get-ContainerImage | select Name, ParentImage

Name              ParentImage
----              -----------
NanoServerIIS     ContainerImage (Name = 'NanoServer') [Publisher = 'CN=Microsoft', Version = '10.0.10586.0']
NanoServer
WindowsServerCore
```

## Docker

### Enumerar imágenes

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### Crear nueva imagen

```powershell
C:\> docker commit 475059caef8f windowsservercoreiis

ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### Quitar imagen

Las imágenes del contenedor no se pueden quitar si cualquier contenedor, incluso en un estado detenido, tiene una dependencia de la imagen.

Cuando se quita una imagen con Docker, se puede hacer referencia a las imágenes por nombre de la imagen o identificador.

```powershell
C:\> docker rmi windowsservercoreiis

Untagged: windowsservercoreiis:latest
Deleted: ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### Docker Hub

El registro de Docker Hub contiene imágenes pregeneradas que se pueden descargar en un host de contenedor. Cuando estas imágenes se hayan descargado, pueden utilizarse como base para aplicaciones de contenedor de Windows.

Para ver una lista de imágenes disponibles en Docker Hub, use el comando `docker search`. Nota: La imagen del sistema operativo base Windows Serve Core deberá instalarse antes de extraer imágenes dependientes de Windows Server Core de Docker Hub.

```powershell
C:\> docker search *

NAME                    DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/aspnet        ASP.NET 5 framework installed in a Windows...   1         [OK]       [OK]
microsoft/django        Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35      .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/golang        Go Programming Language installed in a Win...   1                    [OK]
microsoft/httpd         Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis           Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/mongodb       MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/mysql         MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/nginx         Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/node          Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/php           PHP running on Internet Information Servic...   1                    [OK]
microsoft/python        Python installed in a Windows Server Core ...   1                    [OK]
microsoft/rails         Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/redis         Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/ruby          Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sqlite        SQLite installed in a Windows Server Core ...   1                    [OK]
```

Para descargar una imagen de Docker Hub, use `docker pull`.

```powershell
C:\> docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

La imagen ahora estará visible cuando se ejecute `docker images`.

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```

### Dependencia de imagen

Para ver las dependencias de la imagen con Docker, se puede usar el comando `docker history`

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```



