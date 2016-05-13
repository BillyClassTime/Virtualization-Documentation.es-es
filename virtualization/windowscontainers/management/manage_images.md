---
author: neilpeterson
---

# Imágenes del contenedor

**Esto es contenido preliminar y está sujeto a cambios.**

Las imágenes de contenedor se utilizan para implementar contenedores. Estas imágenes pueden incluir un sistema operativo, aplicaciones y todas las dependencias de aplicaciones. Por ejemplo, puede desarrollar una imagen de contenedor que se haya configurado previamente con Nano Server, IIS y una aplicación que se ejecuta en IIS. Esta imagen de contenedor se puede almacenar después en un registro de contenedor para su uso posterior, se puede implementar en cualquier host de contenedor de Windows (local, en la nube o incluso en un servicio de contenedor) y también se puede utilizar como base para una nueva imagen de contenedor.

Hay dos tipos de imágenes de contenedor:

- **Imágenes del sistema operativo base**: las ofrece Microsoft e incluyen los componentes principales del sistema operativo.
- **Imágenes de contenedor**: son imágenes de contenedor personalizadas que se derivan de una imagen del sistema operativo base.

## Imágenes del sistema operativo base

### Instalar la imagen

Las imágenes del sistema operativo del contenedor se pueden encontrar e instalar mediante el módulo PowerShell ContainerProvider, para así administrar tanto PowerShell como Docker. Antes de utilizar este módulo, deberá instalarse. Se puede usar el siguiente comando para instalar el módulo.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

Una vez instalado, se puede devolver una lista de imágenes del sistema operativo usando `Find-ContainerImage`.

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

Para descargar e instalar la imagen del sistema operativo base Nano Server, ejecute lo siguiente. El parámetro `-version` es opcional. Sin una versión de imagen de SO base especificada, se instalará la versión más reciente.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

Del mismo modo, este comando descargará e instalará la imagen del sistema operativo base de Windows Server Core. El parámetro `-version` es opcional. Sin una versión de imagen de SO base especificada, se instalará la versión más reciente.

> **Problema:** es posible que los cmdlets Save-ContainerImage e Install-ContainerImage no funcionen con una imagen de contenedor WindowsServerCore en una sesión de comunicación remota de PowerShell. **Solución alternativa:** inicie sesión en la máquina con Escritorio remoto y use directamente el cmdlet Save-ContainerImage.

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

> **Install-ContainerImage** instala una imagen del sistema operativo base que se usará tanto en los contenedores administrados de PowerShell como en los de Docker. Si se descarga la imagen del sistema operativo base, pero no aparece cuando se ejecuta `docker images`, reinicie el servicio de Docker mediante el applet del panel de control de servicios o el comando “sc docker stop” y, a continuación, use “sc docker start”

### Instalación sin conexión

Las imágenes del sistema operativo base también pueden instalarse sin conexión a Internet. Para ello, las imágenes se descargarán en un equipo que tenga conexión a Internet, se copiarán en el sistema de destino y, a continuación, se importarán mediante el comando `Install-ContainerOSImages`.

Antes de descargar la imagen del sistema operativo base, ejecute el siguiente comando para preparar el sistema con el proveedor de imágenes de contenedores.

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

Para descargar una imagen, use el comando `Save-ContainerImage`.

```powershell
PS C:\> Save-ContainerImage -Name NanoServer -Destination c:\container-image\NanoServer.wim
```

Una vez hecho esto, podrá copiar la imagen de contenedor descargada en un host de contenedores diferente e instalarla mediante el comando `Install-ContainerOSImage`.

```powershell
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### Etiquetar imágenes

Cuando se hace referencia a una imagen de contenedor mediante el nombre, el motor de Docker buscará la versión más reciente de la imagen. Si no se puede determinar la versión más reciente, se producirá el siguiente error.

```powershell
PS C:\> docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

Una vez instaladas las imágenes del sistema operativo base de Windows Server Core o Nano Server, deberá etiquetarlas con una versión que indique “reciente”. Para ello, use el comando `docker tag`.

Para obtener más información sobre `docker tag`, consulte [Tag, push, and pull your images on docker.com (Etiquetar, insertar y extraer etiquetas en docker.com)](https://docs.docker.com/mac/step_six/).

```powershell
PS C:\> docker tag <image id> windowsservercore:latest
```

Cuando estén etiquetadas, la salida `docker images` mostrará dos versiones de la misma imagen: una con una etiqueta de la versión de la imagen y una segunda con una etiqueta que indica que la versión es “la más reciente”. Ahora puede hacer referencia a la imagen por su nombre.

```powershell
PS C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14289.1000     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14289.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### Desinstalar una imagen del sistema operativo

Las imágenes del sistema operativo base se pueden desinstalar mediante el comando `Uninstall-ContainerOSImage`. En el ejemplo siguiente, se desinstalará la imagen del sistema operativo base NanoServer.

```powershell
Get-ContainerImage -Name NanoServer | Uninstall-ContainerOSImage
```

## Imágenes de contenedor en PowerShell

### Enumerar imágenes

Ejecute `Get-ContainerImage` para que devuelva una lista de imágenes en el host de contenedor. El tipo de imagen de contenedor se diferencia cuando incluye la propiedad `IsOSImage`.

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### Crear nuevas imágenes

Puede crear una nueva imagen de contenedor a partir de un contenedor ya existente. Para ello, use el comando `New-ContainerImage`.

```powershell
PS C:\> New-ContainerImage -Container $container -Publisher Demo -Name DemoImage -Version 1.0
```

### Quitar la imagen

Las imágenes del contenedor no se pueden quitar si cualquier contenedor, incluso en un estado detenido, tiene una dependencia de la imagen.

Quite una sola imagen con PowerShell.

```powershell
PS C:\> Get-ContainerImage -Name newimage | Remove-ContainerImage -Force
```

### Dependencia de imagen

Cuando se crea una nueva imagen, se vuelve dependiente de la imagen a partir de la cual se creó. Esta dependencia se puede ver con el comando `Get-ContainerImage`. Si no se muestra una imagen primaria, esto indica que la imagen es una imagen de sistema operativo.

```powershell
PS C:\> Get-ContainerImage | select Name, ParentImage

Name              ParentImage
----              -----------
NanoServerIIS     ContainerImage (Name = 'NanoServer') [Publisher = 'CN=Microsoft', Version = '10.0.10586.0']
NanoServer
WindowsServerCore
```

### Mover el repositorio de imágenes

Cuando se crea una nueva imagen de contenedor mediante el comando `New-ContainerImage`, esta se guarda en la ubicación predeterminada “C:\ProgramData\Microsoft\Windows\Hyper-V\Almacén de imágenes de contenedor”. Asimismo, puede mover el repositorio mediante el comando `Move-ContainerImageRepository`. El siguiente ejemplo crearía un repositorio de imágenes de contenedor nuevo en la ubicación “c:\container-images”.

```powershell
Move-ContainerImageRepository -Path c:\container-images
```
> Tenga en cuenta que la ruta de acceso que se usa con el comando `Move-ContainerImageRepository` no debe existir cuando ejecute ese comando.

## Imágenes de contenedor en Docker

### Enumerar imágenes

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### Crear nuevas imágenes

Puede crear una nueva imagen de contenedor a partir de un contenedor ya existente. Para ello, use el comando `docker commit`. En el siguiente ejemplo se crea una nueva imagen de contenedor con el nombre “windowsservercoreiis”.

```powershell
C:\> docker commit 475059caef8f windowsservercoreiis

ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### Quitar la imagen

Las imágenes del contenedor no se pueden quitar si cualquier contenedor, incluso en un estado detenido, tiene una dependencia de la imagen.

Cuando se quita una imagen con Docker, se puede hacer referencia a las imágenes por nombre de la imagen o identificador.

```powershell
C:\> docker rmi windowsservercoreiis

Untagged: windowsservercoreiis:latest
Deleted: ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### Dependencia de imagen

Para ver las dependencias de la imagen con Docker, se puede usar el comando `docker history`

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker Hub

El registro de Docker Hub contiene imágenes pregeneradas que se pueden descargar en un host de contenedor. Cuando estas imágenes se hayan descargado, pueden utilizarse como base para aplicaciones de contenedor de Windows.

Para ver una lista de imágenes disponibles en Docker Hub, use el comando `docker search`. Nota: tendrá que instalar las imágenes del sistema operativo base Windows Serve Core o Nano Server antes de extraer esas imágenes de contenedor dependientes de Docker Hub.

> Las imágenes que comienzan con "nano-" dependen de la imagen del sistema operativo base Nano Server.

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
microsoft/nano-golang   Go Programming Language installed in a Nan...   1                    [OK]
microsoft/nano-httpd    Apache httpd installed in a Nano Server ba...   1                    [OK]
microsoft/nano-iis      Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/nano-mysql    MySQL installed in a Nano Server based con...   1                    [OK]
microsoft/nano-nginx    Nginx installed in a Nano Server based con...   1                    [OK]
microsoft/nano-node     Node installed in a Nano Server based cont...   1                    [OK]
microsoft/nano-python   Python installed in a Nano Server based co...   1                    [OK]
microsoft/nano-rails    Ruby on Rails installed in a Nano Server b...   1                    [OK]
microsoft/nano-redis    Redis installed in a Nano Server based con...   1                    [OK]
microsoft/nano-ruby     Ruby installed in a Nano Server based cont...   1                    [OK]
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






<!--HONumber=Mar16_HO3-->


