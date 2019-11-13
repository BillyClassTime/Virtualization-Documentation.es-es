---
title: Contenedor de una aplicación .NET Core
description: Aprenda a crear una aplicación básica de .NET de ejemplo con contenedores
keywords: docker, contenedores
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: fab0dc46ddcc8c82a010d408032e5f3c4cea8d69
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288073"
---
# <a name="containerize-a-net-core-app"></a>Contenedor de una aplicación .NET Core

En este tema se describe cómo empaquetar una aplicación .NET de ejemplo existente para su implementación como contenedor de Windows, después de configurar su entorno según se describe en [Introducción: preparar Windows para contenedores](set-up-environment.md)y ejecutar el primer contenedor, como se describe en [ejecutar el primer contenedor de Windows](run-your-first-container.md).

También necesitará el sistema de control de código fuente git instalado en su equipo. Para instalarlo, visite [git](https://git-scm.com/download).

## <a name="clone-the-sample-code-from-github"></a>Clone el código de ejemplo de GitHub

Todo el código fuente de ejemplo de contenedor se mantiene en la [Virtualización:](https://github.com/MicrosoftDocs/Virtualization-Documentation) repositorio git de documentación (conocido informadamente como un repositorio) `windows-container-samples`en una carpeta llamada.

1. Abra una sesión de PowerShell y cambie los directorios a la carpeta en la que desea almacenar este repositorio. (Otros tipos de ventana de símbolo del sistema también funcionan, pero nuestros comandos de ejemplo usan PowerShell).
2. Clone el repositorio en el directorio de trabajo actual:

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. Navegue hasta el directorio de ejemplo que `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` se encuentra en y cree un Dockerfile, con los comandos siguientes.

   Un [Dockerfile](https://docs.docker.com/engine/reference/builder/) es como un archivo Make: es una lista de instrucciones que indican al motor de contenedor cómo crear la imagen del contenedor.

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Escribir la Dockerfile

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

Hemos escrito el dockerfile para realizar una _compilación de varias fases_. Cuando se ejecuta el dockerfile, usará el contenedor temporal, `build-env`con el SDK de .net Core 2,1 para compilar la aplicación de ejemplo y, a continuación, copiar los archivos binarios de salida en otro contenedor que contenga solo el motor de tiempo de ejecución de .net Core 2,1, de modo que se minimice el tamaño del contenedor final.

## <a name="build-and-run-the-app"></a>Compilar y ejecutar la aplicación

Con el Dockerfile escrito, podemos apuntar al acoplador de nuestro Dockerfile y decirle que cree y ejecute nuestra imagen:

1. En una ventana del símbolo del sistema, vaya al directorio donde se encuentra el dockerfile y ejecute el comando de [compilación de Dock](https://docs.docker.com/engine/reference/commandline/build/) para crear el contenedor desde el dockerfile.

   ```Powershell
   docker build -t my-asp-app .
   ```

2. Para ejecutar el contenedor recién creado, ejecute el comando [Ejecutar del acoplador](https://docs.docker.com/engine/reference/commandline/run/) .

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   Dissect este comando:

   * `-d` indica a un acoplador tun que ejecute el contenedor ' Detached ', lo que significa que no hay ninguna consola enlazada a la consola dentro del contenedor. El contenedor se ejecuta en segundo plano. 
   * `-p 5000:80` indica al acoplador que asigne el puerto 5000 en el host al puerto 80 del contenedor. Cada contenedor obtiene su propia dirección IP. ASP .NET hace una lista predeterminada en el puerto 80. La asignación de puertos nos permite ir a la dirección IP del host en el puerto asignado y el acoplador reenviará todo el tráfico al puerto de destino dentro del contenedor.
   * `--name myapp` indica al acoplador que proporcione a este contenedor un nombre adecuado para realizar consultas (en lugar de tener que buscar el identificador de contaienr asignado en tiempo de ejecución por Docker).
   * `my-asp-app` es la imagen que desea que se ejecute el acoplador. Esta es la imagen del contenedor generada como culminación del `docker build` proceso.

3. Abra un explorador Web de explorador Web y desplácese `http://localhost:5000` hasta que aparezca la aplicación contenedora, como se muestra en esta captura de pantalla:

   >![Página Web del núcleo ASP.NET, que se ejecuta desde el localhost de un contenedor](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>Pasos siguientes

1. El siguiente paso es publicar su aplicación Web de ASP.NET contenedora en un registro privado con el registro del contenedor de Azure. Esto le permite implementarlo en su organización.

   > [!div class="nextstepaction"]
   > [Crear un registro de contenedor privado](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   Cuando llegue a la sección en la que [inserta la imagen del contenedor en el registro](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry), especifique el nombre de la aplicación de ASP.net que acaba de`my-asp-app`empaquetar () junto con el registro del contenedor `contoso-container-registry`(por ejemplo:):

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   Para ver más muestras de la aplicación y sus dockerfiles asociadas, consulte [muestras de contenedores adicionales](../samples.md).

2. Una vez que haya publicado la aplicación en el registro del contenedor, el siguiente paso consistiría en implementar la aplicación en un clúster de Kubernetes que cree con el servicio de Azure Kubernetes.

   > [!div class="nextstepaction"]
   > [Crear un clúster de Kubernetes](https://docs.microsoft.com/azure/aks/windows-container-cli)
