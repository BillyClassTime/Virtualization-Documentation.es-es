---
title: Contenedor de una aplicación .NET Core
description: Aprenda a crear una aplicación básica de .NET de ejemplo con contenedores
keywords: docker, contenedores
author: cwilhit
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: cf8a14002e962242c34e9a10086120e6942d382b
ms.sourcegitcommit: 6080b2c5053720490d374f6fb0daa870d5ddd4e8
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/25/2019
ms.locfileid: "10257779"
---
# <a name="containerize-a-net-core-app"></a>Contenedor de una aplicación .NET Core

Este segmento supone que el entorno de desarrollo ya está configurado para usar contenedores. Si no tiene un entorno configurado para contenedores, visite "[configurar su entorno](./set-up-environment.md)" para obtener información sobre cómo comenzar.

Necesitará el sistema de control de código fuente git instalado en su equipo. Puedes verlo aquí: [git](https://git-scm.com/download)

## <a name="clone-the-sample-code"></a>Clonar el código de ejemplo

Todo el código fuente de ejemplo de contenedor se mantiene en el repositorio git de la [documentación](https://github.com/MicrosoftDocs/Virtualization-Documentation) en una `windows-container-samples`carpeta llamada. Clone este repositorio git en el directorio de trabajo de curent.

```Powershell
git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
```

Navegue hasta el directorio de ejemplo que `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` se encuentra en y cree un Dockerfile. Un [Dockerfile](https://docs.docker.com/engine/reference/builder/) es como un Makefile, una lista de instrucciones que indican al motor de contenedor cómo se debe crear la imagen contenedora.

```Powershell
# navigate into the sample directory
Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

# create the Dockerfile for our project
New-Item -Name Dockerfile -ItemType file
```

## <a name="write-the-dockerfile"></a>Escribir la dockerfile

Abra el dockerfile que acaba de crear (con el editor de texto que más le conviene) y agregue el siguiente contenido.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

Vamos a dividirla línea a línea y explicar qué hace cada una de las instrucciones.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

El primer grupo de líneas indica la imagen base sobre la que crearemos nuestro contenedor. Si el sistema local no tiene aún esta imagen, el docker intentará automáticamente obtenerla. El `mcr.microsoft.com/dotnet/core/sdk:2.1` viene incluido con el SDK de .net Core 2,1 instalado, de modo que depende de la tarea de crear proyectos básicos de ASP .net destinados a la versión 2,1. La siguiente instrucción cambia el directorio de trabajo de nuestro contenedor para `/app`que sea, de modo que todos los comandos que lo siguen se ejecutan en este contexto.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

A continuación, estas instrucciones copian los archivos. csproj en `build-env` el directorio `/app` del contenedor. Después de copiar este archivo, .NET lo leerá y, a continuación, obtendrá todas las dependencias y las herramientas que necesita nuestro proyecto.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

Una vez que .NET ha extraído todas las dependencias en el `build-env` contenedor, la siguiente instrucción copia todos los archivos de origen del proyecto en el contenedor. A continuación, le indicamos a .NET que publique la aplicación con la configuración release y que especifique la ruta de acceso de salida en el archivo.

La compilación debería realizarse correctamente. Ahora debemos crear la imagen final. 

> [!TIP]
> Este tutorial crea un proyecto .NET Core desde el origen. Al crear imágenes contenedoras, una buena práctica es incluir _solo_ la carga de producción y sus dependencias en la imagen del contenedor. No queremos que el SDK de .NET Core esté incluido en nuestra imagen final porque solo necesitamos el tiempo de ejecución de .NET Core, por lo que el dockerfile está escrito para usar un contenedor temporal que se `build-env` empaqueta con el SDK denominado para compilar la aplicación.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

Como nuestra aplicación es ASP.NET, especificamos una imagen con este tiempo de ejecución incluido. Después copiamos todos los archivos del directorio de salida de nuestro contenedor temporal en nuestro contenedor final. Configuramos nuestro contenedor para que se ejecute con nuestra nueva aplicación como su punto de inicio cuando se inicia el contenedor.

Hemos escrito el dockerfile para realizar una _compilación de varias fases_. Cuando se ejecuta el dockerfile, usará el contenedor temporal, `build-env`con el SDK de .net Core 2,1 para compilar la aplicación de ejemplo y, después, copiar los archivos binarios de salida en otro contenedor que contenga solo el tiempo de ejecución de .net Core 2,1, de modo que se minimice el tamaño de la contenedor final.

## <a name="run-the-app"></a>Ejecutar la aplicación

Con el dockerfile escrito, podemos apuntar al acoplador de nuestro dockerfile y decirle que construya nuestra imagen. 

>[!IMPORTANT]
>El comando ejecutado a continuación debe ejecutarse en el directorio donde se encuentra el dockerfile.

```Powershell
docker build -t my-asp-app .
```

Para ejecutar el contenedor, ejecute el siguiente comando.

```Powershell
docker run -d -p 5000:80 --name myapp my-asp-app
```

Dissect este comando:

* `-d` indica a un acoplador tun que ejecute el contenedor ' Detached ', lo que significa que no hay ninguna consola enlazada a la consola dentro del contenedor. El contenedor se ejecuta en segundo plano. 
* `-p 5000:80` indica al acoplador que asigne el puerto 5000 en el host al puerto 80 del contenedor. Cada contenedor obtiene su propia dirección IP. ASP .NET hace una lista predeterminada en el puerto 80. La asignación de puertos nos permite ir a la dirección IP del host en el puerto asignado y el acoplador reenviará todo el tráfico al puerto de destino dentro del contenedor.
* `--name myapp` indica al acoplador que proporcione a este contenedor un nombre adecuado para realizar consultas (en lugar de tener que buscar el identificador de contaienr asignado en tiempo de ejecución por Docker).
* `my-asp-app` es la imagen que desea que se ejecute el acoplador. Esta es la imagen del contenedor generada como culminación del `docker build` proceso.

Abra un explorador Web de explorador Web y desplácese `https://localhost:5000` para que la aplicación contenedora lo saluda.

>![](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>Pasos siguientes

Hemos contenedordo correctamente una aplicación Web de ASP.NET. Para ver más muestras de la aplicación y sus dockerfiles asociadas, haga clic en el botón siguiente.

> [!div class="nextstepaction"]
> [Consultar más muestras de contenedores](../samples.md)
