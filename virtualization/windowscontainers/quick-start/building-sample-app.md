---
title: Inclusión de una aplicación .NET Core en un contenedor
description: Aprenda a crear una aplicación .NET Core de ejemplo con contenedores
keywords: docker, contenedores
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: fab0dc46ddcc8c82a010d408032e5f3c4cea8d69
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910145"
---
# <a name="containerize-a-net-core-app"></a>Inclusión de una aplicación .NET Core en un contenedor

En este tema se describe cómo empaquetar una aplicación .NET de ejemplo existente para su implementación como un contenedor de Windows, después de configurar el entorno, tal como se describe en [Introducción: preparar Windows para contenedores](set-up-environment.md)y ejecutar el primer contenedor, tal como se describe en [ejecutar el primer contenedor de Windows](run-your-first-container.md).

También necesitará el sistema de control de código fuente de Git instalado en el equipo. Para instalarlo, visite [git](https://git-scm.com/download).

## <a name="clone-the-sample-code-from-github"></a>Clonación del código de ejemplo de GitHub

Todo el código fuente de ejemplo de contenedor se mantiene en el repositorio de Git [de la documentación de virtualización](https://github.com/MicrosoftDocs/Virtualization-Documentation) (conocido como un repositorio) en una carpeta denominada `windows-container-samples`.

1. Abra una sesión de PowerShell y cambie los directorios a la carpeta en la que desea almacenar este repositorio. (También funcionan los tipos de ventana del símbolo del sistema, pero los comandos de ejemplo usan PowerShell).
2. Clone el repositorio en el directorio de trabajo actual:

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. Navegue hasta el directorio de ejemplo que se encuentra en `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` y cree un Dockerfile con los siguientes comandos.

   Un [Dockerfile](https://docs.docker.com/engine/reference/builder/) es como un archivo make, es una lista de instrucciones que indican al motor de contenedor cómo compilar la imagen del contenedor.

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Escritura de Dockerfile

Abra el Dockerfile que acaba de crear con el editor de texto que desee y, a continuación, agregue el siguiente contenido:

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

Vamos a dividirlo línea a línea y explicaremos lo que hace cada instrucción.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

El primer grupo de líneas indica la imagen base sobre la que crearemos nuestro contenedor. Si el sistema local no tiene aún esta imagen, el docker intentará automáticamente obtenerla. Los `mcr.microsoft.com/dotnet/core/sdk:2.1` vienen empaquetados con el SDK de .NET Core 2,1 instalado, por lo que es la tarea de compilar proyectos de ASP .NET Core que tienen como destino la versión 2,1. La siguiente instrucción cambia el directorio de trabajo en el contenedor para que se `/app`, por lo que todos los comandos que siguen a este se ejecutan en este contexto.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

A continuación, estas instrucciones copian los archivos. csproj en el directorio de `/app` del contenedor de `build-env`. Después de copiar este archivo, .NET leerá de él y, a continuación, desplazará y recuperará todas las dependencias y herramientas que necesita nuestro proyecto.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

Una vez que .NET ha extraído todas las dependencias en el contenedor de `build-env`, la siguiente instrucción copia todos los archivos de origen del proyecto en el contenedor. A continuación, indicamos a .NET que publique la aplicación con una configuración de versión y especifique la ruta de acceso de salida en.

La compilación debe realizarse correctamente. Ahora debemos compilar la imagen final. 

> [!TIP]
> En esta guía de inicio rápido se crea un proyecto de .NET Core desde el origen. Al compilar imágenes de contenedor, se recomienda incluir _solo_ la carga de producción y sus dependencias en la imagen de contenedor. No queremos que el SDK de .NET Core esté incluido en la imagen final porque solo necesitamos el entorno de tiempo de ejecución de .NET Core, por lo que el dockerfile se escribe para usar un contenedor temporal que está empaquetado con el SDK llamado `build-env` para compilar la aplicación.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

Dado que nuestra aplicación es ASP.NET, se especifica una imagen con este tiempo de ejecución incluido. Después copiamos todos los archivos del directorio de salida de nuestro contenedor temporal en nuestro contenedor final. Configuramos el contenedor para que se ejecute con nuestra nueva aplicación como punto de entrada cuando se inicia el contenedor.

Hemos escrito la dockerfile para realizar una _compilación en varias fases_. Cuando se ejecuta dockerfile, usará el contenedor temporal, `build-env`, con el SDK de .NET Core 2,1 para compilar la aplicación de ejemplo y, a continuación, copiar los archivos binarios de salida en otro contenedor que solo contenga el tiempo de ejecución de .NET Core 2,1 para minimizar el tamaño del contenedor final.

## <a name="build-and-run-the-app"></a>Compilar y ejecutar la aplicación

Con el Dockerfile escrito, podemos apuntar a Docker en nuestro Dockerfile y decirle que compile y ejecute la imagen:

1. En una ventana del símbolo del sistema, navegue hasta el directorio donde reside el dockerfile y, a continuación, ejecute el comando de [compilación de Docker](https://docs.docker.com/engine/reference/commandline/build/) para compilar el contenedor desde dockerfile.

   ```Powershell
   docker build -t my-asp-app .
   ```

2. Para ejecutar el contenedor recién creado, ejecute el comando [Docker Run](https://docs.docker.com/engine/reference/commandline/run/) .

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   Vamos a analizar minuciosamente este comando:

   * `-d` indica a Docker tun que ejecute el contenedor ' Detached ', lo que significa que no hay ninguna consola enlazada a la consola dentro del contenedor. El contenedor se ejecuta en segundo plano. 
   * `-p 5000:80` indica a Docker que asigne el puerto 5000 en el host al puerto 80 del contenedor. Cada contenedor obtiene su propia dirección IP. ASP .NET escucha de forma predeterminada en el puerto 80. La asignación de puertos nos permite ir a la dirección IP del host en el puerto asignado y Docker reenviará todo el tráfico al puerto de destino dentro del contenedor.
   * `--name myapp` indica a Docker que proporcione a este contenedor un nombre adecuado para consultar (en lugar de tener que buscar el identificador de contaienr asignado en tiempo de ejecución por Docker).
   * `my-asp-app` es la imagen que queremos que se ejecute Docker. Esta es la imagen de contenedor que se genera como culminación del proceso de `docker build`.

3. Abra un explorador Web del explorador Web y navegue a `http://localhost:5000` para ver la aplicación en contenedor, como se muestra en esta captura de pantalla:

   >![ASP.NET Core Página Web, que se ejecuta desde el host local en un contenedor](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>Pasos siguientes

1. El siguiente paso consiste en publicar la aplicación Web de ASP.NET en contenedor en un registro privado mediante Azure Container Registry. Esto le permite implementarlo en su organización.

   > [!div class="nextstepaction"]
   > [Crear un registro de contenedor privado](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   Cuando llegue a la sección donde Inserte la [imagen de contenedor en el registro](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry), especifique el nombre de la aplicación ASP.net que acaba de empaquetar (`my-asp-app`) junto con el registro de contenedor (por ejemplo: `contoso-container-registry`):

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   Para ver más ejemplos de aplicaciones y sus dockerfiles asociadas, consulte [ejemplos de contenedor adicionales](../samples.md).

2. Una vez que haya publicado la aplicación en el registro de contenedor, el paso siguiente sería implementar la aplicación en un clúster de Kubernetes que cree con Azure Kubernetes Service.

   > [!div class="nextstepaction"]
   > [Creación de un clúster de Kubernetes](https://docs.microsoft.com/azure/aks/windows-container-cli)
