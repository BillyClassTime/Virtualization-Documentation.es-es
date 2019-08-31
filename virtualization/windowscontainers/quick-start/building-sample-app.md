---
title: Crear una aplicación de ejemplo
description: Aprender a crear una aplicación de ejemplo mientras se saca partido de los contenedores
keywords: docker, contenedores
author: cwilhit
ms.date: 07/25/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 7ffc16e9d5b7c4b4a935a06c012b1d28b5e70f1a
ms.sourcegitcommit: 27e9cd37beaf11e444767699886e5fdea5e1a2d0
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 08/30/2019
ms.locfileid: "10058490"
---
# <a name="build-a-sample-app"></a>Crear una aplicación de ejemplo

En este ejercicio obtendrás instrucciones sobre cómo tomar una aplicación de ejemplo ASP.net y convertirla para que se ejecute en un contenedor. Si necesitas aprender cómo empezar a trabajar con los contenedores en Windows10, visita [Inicio rápido de Windows10](./quick-start-windows-10.md).

Este inicio rápido es específico de Windows10. En la tabla de contenido del lado izquierdo de esta página encontrarás documentación adicional de inicio rápido. Dado que el objetivo de este tutorial hace referencia a los contenedores, pasaremos por alto la escritura de código y nos centraremos exclusivamente en ellos. Si quieres completar el tutorial desde el principio, puedes encontrarlo en [ASP.NET Core Documentation](https://docs.microsoft.com/aspnet/core/tutorials/first-mvc-app-xplat/) (Documentación básica sobre ASP.NET)

Si no tienes el control de código fuente de Git instalado en tu equipo, puedes obtenerlo aquí: [Git](https://git-scm.com/download)

## <a name="getting-started"></a>Introducción

Este proyecto de ejemplo se ha configurado con [VSCode](https://code.visualstudio.com/). También utilizaremos PowerShell. Vamos a obtener el código de demostración de GitHub. Puedes clonar el repositorio con git o descargar el proyecto directamente en [SampleASPContainerApp](https://github.com/cwilhit/SampleASPContainerApp).

```Powershell
git clone https://github.com/cwilhit/SampleASPContainerApp.git
```

Ahora nos dirigiremos al directorio del proyecto y crearemos el Dockerfile. Un [Dockerfile](https://docs.docker.com/engine/reference/builder/) es parecido a un makefile: una lista de instrucciones que describen cómo debe crearse una imagen de contenedor.

```Powershell
#Create the dockerfile for our proj
New-Item C:/Your/Proj/Location/Dockerfile -type file
```

## <a name="writing-our-dockerfile"></a>Escribir en nuestro Dockerfile

Ahora vamos a abrir ese Dockerfile que hemos creado en la carpeta raíz del proyecto (con el editor de texto que prefieras) y le agregaremos lógica. Después lo desglosaremos línea por línea y explicaremos lo que está ocurriendo.

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

El primer grupo de líneas indica la imagen base sobre la que crearemos nuestro contenedor. Si el sistema local no tiene aún esta imagen, el docker intentará automáticamente obtenerla. Aspnetcore-build incluye dependencias para compilar nuestro proyecto. Después cambiamos el directorio de trabajo de nuestro contenedor para que sea "/app", de modo que todos los comandos que aparezcan en nuestro dockerfile se ejecutarán en dicho directorio.

>[!NOTE]
>Como debemos crear nuestro proyecto, este primer contenedor que creamos es un contenedor temporal que usaremos para hacerlo y, a continuación, descartarlo al final.

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app
```

El paso siguiente será copiar los archivos .csproj en el directorio temporal "/app" de nuestro contenedor. Esto se hace porque los archivos. csproj contienen una lista de referencias de paquete que necesitan nuestros proyectos.

Después de copiar este archivo, dotnet lo leerá y obtendrá todas las dependencias y herramientas que nuestro proyecto necesita.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

Una vez que hemos desplegado todas estas dependencias, las copiamos en el contenedor temporal. Después indicamos a dotnet que publique nuestra aplicación con una configuración de lanzamiento y que especifique la ruta de acceso de salida.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

Deberíamos haber compilado nuestro proyecto correctamente. Ahora debemos crear nuestro contenedor finalizado. Dado que nuestra aplicación es ASP.NET, especificamos una imagen con estas bibliotecas como el origen. Después copiamos todos los archivos del directorio de salida de nuestro contenedor temporal en nuestro contenedor final. Configuramos nuestro contenedor para que se ejecute con el nuevo archivo .dll que hemos compilado cuando lo iniciamos.

>[!NOTE]
>Nuestra imagen básica para este contenedor final es similar, pero diferente al ```FROM``` comando anterior, no tiene las bibliotecas capaces de _crear_ una aplicación de ASP.net, solo en ejecución.

```Dockerfile
FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

Ahora hemos realizado correctamente lo que se denomina una _compilación multietapa_. Hemos usado el contenedor temporal para compilar nuestra imagen y luego hemos movido el archivo dll publicado a otro contenedor para minimizar la huella de la aplicación del resultado final. Queremos que este contenedor tenga el mínimo absoluto necesario de dependencias para ejecutarse; si hubiéramos seguido usando nuestra primera imagen, hubiera venido incluida con otras capas (para compilar aplicaciones ASP.NET) que no eran de vital importancia y, por tanto, el tamaño de la imagen habría aumentado.

## <a name="running-the-app"></a>Ejecutar la aplicación

Ahora que se ha escrito en el dockerfile, lo único que queda por hacer es indicar al docker que compile nuestra aplicación y luego ejecute el contenedor. Especificamos el puerto en el que se publicará y después proporcionamos al contenedor una etiqueta denominada "myapp". En PowerShell, ejecuta los siguientes comandos.

>[!NOTE]
>El directorio de trabajo actual de la consola de PowerShell debe ser el directorio donde se encuentra el dockerfile creado arriba.

```Powershell
docker build -t myasp .
docker run -d -p 5000:80 --name myapp myasp
```

Para ver nuestra aplicación en ejecución, debemos visitar la dirección en la que se está ejecutando. Vamos a obtener la dirección IP mediante la ejecución de este comando.

```Powershell
 docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" myapp
```

Al ejecutar este comando, se obtendrá la dirección IP del contenedor en ejecución. Aquí te mostramos un ejemplo de lo que verás.

```Powershell
 172.19.172.12
```

Escribe esta dirección IP en el navegador web que quieras y aparecerá la aplicación ejecutándose correctamente en un contenedor.

>![](media/SampleAppScreenshot.png)

Si haces clic en "MvcMovie" en la barra de navegación, te llevará a una página web donde puedes escribir, editar y eliminar entradas de películas.

## <a name="next-steps"></a>Pasos siguientes

Hemos tomado una aplicación web ASP.NET, la hemos configurado y la hemos compilado correctamente con Docker y, además, la hemos implementado en un contenedor en ejecución. Sin embargo, hay más pasos adicionales que puedes realizar. Podrías desglosar la aplicación web en más componentes: un contenedor que ejecuta la API web, un contenedor que ejecuta el front-end y un contenedor que ejecuta SQLServer.

Ahora que ya tienes el bloqueo de los contenedores, puedes salir y crear excelentes software de contenedor.

> [!div class="nextstepaction"]
> [Consultar más muestras de contenedores](../samples.md)
