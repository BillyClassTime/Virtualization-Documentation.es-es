# <a name="build-and-run-an-application-with-or-without-net-core-20-or-powershell-core-6"></a>Compilar y ejecutar una aplicación con o sin .NET Core 2.0 o PowerShell Core 6

La imagen de contenedor base del sistema operativo de Nano Server de esta versión ha quitado .NET Core y PowerShell, aunque .NET Core y PowerShell se admiten como un contenedor de capas de complementos en la parte superior del contenedor de base de Nano Server.  

Si el contenedor debe ejecutar código nativo o marcos abiertos como Node.js, Python, Ruby, etc., será suficiente el contenedor de base de Nano Server.  Un aspecto es que puede que no se ejecute determinado código nativo como resultado del [ahorro de superficie](https://docs.microsoft.com/en-us/windows-server/get-started/nano-in-semi-annual-channel) en esta versión, en comparación con la de Windows Server 2016. Si observas algún problema de regresión, comunícanoslo en los [foros](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers). 

Para crear el contenedor desde un Dockerfile, usa la compilación docker y, para ejecutarlo, ejecuta docker.  El comando siguiente descargará la imagen base del sistema operativo del contenedor de Nano Server, lo que puede llevar unos minutos, e imprimirá un mensaje "¡Hello World!" en la consola del host.

```
docker run microsoft/nanoserver-insider cmd /c echo Hello World!
```

Puedes crear aplicaciones más complicadas usando [Dockerfiles en Windows](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/manage-windows-dockerfile), con una sintaxis de Dockerfile como FROM, RUN, COPY, ADD, CMD, etc. Aunque no podrás ejecutar determinados comandos fuera de esta imagen base, ahora ya podrás crear imágenes de contenedor que solo contengan las cosas que necesitas para que funcione la aplicación.

Como resultado de que .NET Core y PowerShell no están disponibles en la imagen base del sistema operativo del contenedor de Nano Server, una de las dificultades es crear un contenedor con el contenido comprimido en formato zip. Con la característica [compilación multietapa](https://docs.docker.com/engine/userguide/eng-image/multistage-build/), disponible en Docker 17.05, puedes aprovechar el PowerShell de otro contenedor para descomprimir el contenido y copiarlo en el contenedor de Nano. Este enfoque puede usarse para crear un contenedor .NET Core y un contenedor de PowerShell. 

Puedes utilizar la imagen del contenedor de PowerShell usando este comando:

```
docker pull microsoft/nanoserver-insider-powershell
```

Puedes utilizar la imagen del contenedor de .NET Core usando este comando:

```
docker pull microsoft/nanoserver-insider-dotnet
```

A continuación se muestran algunos ejemplos de cómo usamos compilaciones multietapa para crear estas imágenes de contenedor.

## <a name="deploy-apps-based-on-net-core-20"></a>Implementar aplicaciones basadas en .NET Core 2.0
Puedes aprovechar la imagen del contenedor de .NET Core 2.0 de la versión de Insider para ejecutar las aplicaciones de .NET Core, en el caso de que la aplicación de .NET Core esté integrada en otro lugar y quieras ejecutarla en el contenedor.  Puedes encontrar más información sobre cómo ejecutar una aplicación de .NET Core con las imágenes de contenedor .NET Core en [GitHub de .NET Core](https://github.com/dotnet/dotnet-docker-nightly).  Si estás desarrollando una aplicación dentro del contenedor, debe usarse en su lugar el SDK de .NET Core.  Para usuarios avanzados, puedes crear tu propio contenedor de .NET Core 2.0 con la versión de .NET Core 2.0, Dockerfile, y la dirección URL especificada en [dotnet-docker-nightly](https://github.com/dotnet/dotnet-docker-nightly/tree/master/2.0). Para ello, puede usarse un contenedor de Windows Server Core para realizar la descarga y la función de descompresión.  La muestra de Dockerfile es como el [Dockerfile de tiempo de ejecución de .NET Core](https://github.com/dotnet/dotnet-docker-nightly/blob/master/2.0/runtime/nanoserver-insider/amd64/Dockerfile).


Puedes utilizar la imagen del contenedor de .NET Core usando este comando:

```
docker build -t nanoserverdnc -f Dockerfile-dotnetRuntime .
```

## <a name="run-powershell-core-6-in-a-container"></a>Ejecutar PowerShell Core 6 en un contenedor
Con el mismo método de [compilación multietapa](https://docs.docker.com/engine/userguide/eng-image/multistage-build/), se puede compilar un contenedor de PowerShell Core 6 con [este PowerShell Dockerfile](https://github.com/PowerShell/PowerShell-Docker/blob/master/release/stable/nanoserver/docker/Dockerfile).


A continuación, usa la compilación docker para crear la imagen del contenedor de PowerShell.

``` 
docker build -t nanoserverPowerShell6 -f Dockerfile-PowerShell6 .
```

Encontrarás más información en [PowerShell GitHub](https://github.com/PowerShell/PowerShell-Docker/tree/master/release).  Vale la pena mencionar que el archivo comprimido en zip de PowerShell contiene un subconjunto de .NET Core 2.0 necesario para compilar PowerShell Core 6.  Si los módulos de PowerShell dependen de .NETCore2.0, es seguro crear el contenedor de PowerShell en la parte superior del contenedor de Nano.NETCore, en lugar del contenedor de base de Nano, es decir, usando FROM microsoft/nanoserver-insider-dotnet en el Dockerfile. 

## <a name="next-steps"></a>Pasos siguientes
- Usa una de las nuevas imágenes de contenedor basadas en Nano Server, disponibles en Docker Hub, es decir, la imagen base de Nano Server, Nano con .NET Core 2.0 y Nano con PowerShell Core 6.
- Crea tu propia imagen de contenedor basada en la nueva imagen base del sistema operativo del contenedor Nano Server, usando la muestra de Dockerfile de esta guía. 
