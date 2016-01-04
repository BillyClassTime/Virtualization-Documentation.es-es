# Docker y Windows

**Esto es contenido preliminar y está sujeto a cambios.**

Docker es una plataforma de administración e implementación de contenedores, que funciona con contenedores de Linux y Windows. Docker se utiliza para crear, administrar y eliminar contenedores e imágenes de contenedores. Docker permite almacenar imágenes de contenedores en un registro público (Docker Hub) y registros privados (registros de confianza de Docker). Además, Docker ofrece capacidades de agrupación en clústeres de hosts de contenedor con Docker Swarm y automatización de implementaciones con Docker Compose. Para obtener más información sobre Docker y el conjunto de herramientas de Docker, visite [Docker.com](https://www.docker.com/).

> La característica de contenedores de Windows debe habilitarse antes de poder usar Docker para crear y administrar contenedores de Windows Server y Hyper-V. Para obtener instrucciones sobre cómo habilitar esta característica, consulte la [Guía de implementación de hosts de contenedor](./docker_windows.md).

## Windows Server

### Instalar Docker

El demonio de Docker y la CLI no se incluyen con Windows Server ni Windows Server Core y no se instalan con la característica de contenedor de Windows. Docker deberá instalarse por separado. Este documento le guiará a través de la instalación manual del demonio de Docker y el cliente de Docker. También se ofrecerán métodos automatizados para completar estas tareas.

El demonio de Docker y la interfaz de la línea de comandos de Docker se han desarrollado en lenguaje Go. En este momento, docker.exe no se instala como un servicio de Windows. Existen varios métodos que se pueden usar para crear un servicio de Windows; uno de los ejemplos que se muestran aquí usa `nssm.exe`.

Descargue docker.exe de `https://aka.ms/ContainerTools` y colóquelo en el directorio System32 del host del contenedor.

```powershell
PS C:\> wget https://aka.ms/ContainerTools -OutFile $env:SystemRoot\system32\docker.exe
```

Cree un directorio denominado `c:\programdata\docker`. En este directorio, cree un archivo denominado `runDockerDaemon.cmd`.

```powershell
PS C:\> New-Item -ItemType File -Path C:\ProgramData\Docker\runDockerDaemon.cmd -Force
```

Copie el texto siguiente en el archivo `runDockerDaemon.cmd`. Este archivo por lotes inicia el demonio de Docker con el comando `docker daemon –D –b “Virtual Switch”`. Nota: El nombre del conmutador virtual de este archivo deberá coincidir con el nombre de los contenedores virtuales que se utilizarán para la conectividad de red.

```powershell
@echo off
set certs=%ProgramData%\docker\certs.d

if exist %ProgramData%\docker (goto :run)
mkdir %ProgramData%\docker

:run
if exist %certs%\server-cert.pem (goto :secure)

docker daemon -D -b "Virtual Switch"
goto :eof

:secure
docker daemon -D -b "Virtual Switch" -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
```
Descargue nssm.exe de [https://nssm.cc/release/nssm-2.24.zip](https://nssm.cc/release/nssm-2.24.zip).

```powershell
PS C:\> wget https://nssm.cc/release/nssm-2.24.zip -OutFile $env:ALLUSERSPROFILE\nssm.zip
```

Extraiga los archivos y copie `nssm-2.24\win64\nssm.exe` en el directorio `c:\windows\system32`.

```powershell
PS C:\> Expand-Archive -Path $env:ALLUSERSPROFILE\nssm.zip $env:ALLUSERSPROFILE
PS C:\> Copy-Item $env:ALLUSERSPROFILE\nssm-2.24\win64\nssm.exe $env:SystemRoot\system32
```
Ejecute `nssm install` para configurar el servicio de Docker.

```powershell
PS C:\> start-process nssm install
```

Escriba los siguientes datos en los campos correspondientes del instalador del servicio NSSM.

Pestaña Aplicación:

- **Ruta de acceso:** C:\Windows\System32\cmd.exe

- **Directorio de inicio:** C:\Windows\System32

- **Argumentos:** /s /c C:\ProgramData\docker\runDockerDaemon.cmd

- **Nombre del servicio:** Docker

![](media/nssm1.png)

Pestaña Detalles:

- **Nombre para mostrar:** Docker

- **Descripción:** el demonio de Docker ofrece capacidades de administración de contenedores para los clientes de Docker.


![](media/nssm2.png)

Pestaña E/S:

- **Resultado (stdout)** C:\ProgramData\docker\daemon.log

- **Error (stderr):** C:\ProgramData\docker\daemon.log


![](media/nssm3.png)

Cuando termine, haga clic en el botón `Instalar servicio`.

Con esto completado, cuando se inicie Windows, el demonio (servicio) de Docker también se iniciará.

### Quitar Docker

Si sigue esta guía para crear un servicio de Windows desde docke.exe, el comando siguiente quitará el servicio.

```powershell
PS C:\> sc.exe delete Docker

[SC] DeleteService SUCESS
```

## Nano Server

### Instalar Docker

Descargue docker.exe de `https://aka.ms/ContainerTools` y cópielo en la carpeta `windows\system32` del host de contenedor de Nano Server.

Ejecute el comando siguiente para iniciar el demonio de Docker. Será necesario que esto se ejecute cada vez que se inicie el host del contenedor. Este comando inicia el demonio de Docker, especifica un conmutador virtual para la conectividad del contenedor y establece que el demonio escuche en el puerto 2375 las solicitudes entrantes de Docker. Con esta configuración, Docker puede administrarse desde un equipo remoto.

```powershell
PS C:\> start-process cmd "/k docker daemon -D -b <Switch Name> -H 0.0.0.0:2375”
```

### Quitar Docker

Para quitar el demonio de Docker y la CLI de Nano Server, elimine `docker.exe` del directorio Windows\system32.

```powershell
PS C:\> Remove-Item $env:SystemRoot\system32\docker.exe
```




