---
title: Inicio rápido de implementación de contenedores (imágenes)
description: Inicio rápido de implementación de contenedores
keywords: docker, contenedores
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
ms.openlocfilehash: 41fa89dcaba38d43d39681240a1a108c9250ba78
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380439"
---
# <a name="automating-builds-and-saving-images"></a>Automatizar compilaciones y guardar imágenes

En el anterior inicio rápido de Windows Server, se ha creado un contenedor de Windows a partir de un ejemplo de .NET Core creado previamente. En este ejercicio se muestra cómo crear tu propia imagen de contenedor desde un Dockerfile y almacenar la imagen de contenedor en el registro público de Docker Hub.

Este inicio rápido es específico de los contenedores de Windows Server en Windows Server 2019 o Windows Server 2016 y usará la imagen base del contenedor de Windows Server Core. En la tabla de contenido del lado izquierdo de esta página encontrará documentación adicional de inicio rápido.

## <a name="prerequisites"></a>Requisitos previos

Asegúrese de que se cumplen los requisitos siguientes:

- Sistema de un equipo (físico o virtual) ejecutando Windows Server 2019 o Windows Server 2016.
- Configure este sistema con la característica de contenedor de Windows y Docker. Para ver un tutorial sobre estos pasos, consulta [los contenedores de Windows en Windows Server](./quick-start-windows-server.md).
- Un identificador de Docker, que se usará para insertar una imagen de contenedor en Docker Hub. Si no tiene un identificador de Docker, suscríbase para obtenerlo en [Docker Cloud](https://cloud.docker.com/).

## <a name="container-image---dockerfile"></a>Imagen de contenedor (dockerfile)

Aunque se puede crear manualmente un contenedor, modificar y luego capturarse en una imagen nueva de contenedor, Docker incluye un método para automatizar este proceso con un Dockerfile. Para este ejercicio, se requiere un identificador de Docker. Si no tiene un identificador de Docker, suscríbase para obtenerlo en [Docker Cloud]( https://cloud.docker.com/).

En el host de contenedor, cree un directorio `c:\build` y, en este directorio, cree un archivo denominado `Dockerfile`. Nota: El archivo no debe tener una extensión de archivo.

```console
powershell new-item c:\build\Dockerfile -Force
```

Abra el Dockerfile en el Bloc de notas.

```console
notepad c:\build\Dockerfile
```

Copie el texto siguiente en el Dockerfile y guarde el archivo. Estos comandos indican a Docker que cree una nueva imagen con `microsoft/iis` como base. Entonces el Dockerfile ejecuta los comandos especificados en la instrucción `RUN`, en este caso el archivo index.html se actualiza con nuevo contenido.

Para obtener más información sobre los Dockerfiles, consulte [Dockerfiles on Windows (Dockerfiles en Windows)](../manage-docker/manage-windows-dockerfile.md).

```dockerfile
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

El comando `docker build` iniciará el proceso de compilación de la imagen. El parámetro `-t` indica al proceso de compilación que asigne un nombre a la nueva imagen `iis-dockerfile`. **Reemplace "user" con el nombre de usuario de la cuenta de Docker**. Si no tiene una cuenta de Docker, suscríbase para obtenerla en [Docker Cloud](https://cloud.docker.com/).

```console
docker build -t <user>/iis-dockerfile c:\Build
```

Una vez completado, podrá comprobar que la imagen se ha creado mediante el comando `docker images`.

```console
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

Ahora, implemente un contenedor con el comando siguiente y reemplace de nuevo user con su identificador de Docker.

```console
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

Una vez creado el contenedor, vaya a la dirección IP del host de contenedor. Debería ver la aplicación hello world.

![](media/dockerfile2.png)

De vuelta en el host de contenedor, use `docker ps` para obtener el nombre del contenedor y `docker rm` para quitar el contenedor. Nota: Reemplace el nombre del contenedor de este ejemplo por el nombre real del contenedor.

Obtenga el nombre del contenedor.

```console
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

Detener el contenedor.

```console
docker stop <container name>
```

Quita el contenedor.

```console
docker rm -f <container name>
```

## <a name="docker-push"></a>Inserción de docker

Las imágenes de contenedor de Docker se pueden almacenar en un registro de contenedor. Una vez que se almacena una imagen en un registro, se puede recuperar para su uso posterior en varios hosts de contenedor. Docker proporciona un registro público para almacenar imágenes de contenedor en [Docker Hub](https://hub.docker.com/).

Para este ejercicio, se insertará la imagen personalizada hello world en su propia cuenta en Docker Hub.

En primer lugar, inicie sesión en su cuenta de Docker mediante el `docker login command`.

```console
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

Una vez iniciada la sesión, se puede insertar la imagen de contenedor en Docker Hub. Para ello, use el comando `docker push`. **Reemplace "user" con el identificador de Docker**. 

```console
docker push <user>/iis-dockerfile
```

Como Docker inserte cada capa hasta Docker Hub, docker omitirá capas que ya existen en Docker Hub o en otros registros (capas externas).  Por ejemplo, las versiones recientes de Windows Server Core que se hospedan en el registro de contenedor de Microsoft, o las capas de un registro corporativa privada, ¿se omite y no se inserta en Docker Hub.

La imagen de contenedor ya se puede descargar desde Docker Hub en cualquier host de contenedor de Windows mediante `docker pull`. Para este tutorial, se eliminará la imagen existente y después se extraerá de Docker Hub. 

```console
docker rmi <user>/iis-dockerfile
```

Al ejecutar `docker images`, mostrará que se ha quitado la imagen.

```console
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
modified-iis              latest              51f1fe8470b3        5 minutes ago       7.69 GB
microsoft/iis             latest              e4525dda8206        3 hours ago         7.61 GB
```

Por último, la extracción de Docker se puede usar para devolver la imagen al host del contenedor. Reemplace user con el nombre de usuario de la cuenta de Docker. 

```
docker pull <user>/iis-dockerfile
```

## <a name="next-steps"></a>Pasos siguientes

Si te gustaría obtener información sobre cómo empaquetar una aplicación de ejemplo ASP.NET, consulta los tutoriales de Windows10 que se vinculan a continuación.

> [!div class="nextstepaction"]
> [Contenedores en Windows 10](./quick-start-windows-10.md)
