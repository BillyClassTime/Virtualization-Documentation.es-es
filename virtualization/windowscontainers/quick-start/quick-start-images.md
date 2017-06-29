---
title: "Inicio rápido de implementación de contenedores (imágenes)"
description: "Inicio rápido de implementación de contenedores"
keywords: docker, contenedores
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
ms.openlocfilehash: 355daae1b673f0b05f08d0706664967a825de6f7
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: es-ES
---
# <a name="container-images-on-windows-server"></a>Imágenes de contenedores en Windows Server

En el anterior inicio rápido de Windows Server, se ha creado un contenedor de Windows a partir de un ejemplo de .NET Core creado previamente. En este ejercicio, se detallará la creación de imágenes de contenedor personalizadas de forma manual, se automatizará la creación de imágenes de contenedor mediante un Dockerfile y se almacenarán imágenes de contenedor en el registro público de Docker Hub.

Este inicio rápido es específico de los contenedores de Windows Server en Windows Server 2016 y usará la imagen base del contenedor de Windows Server Core. En la tabla de contenido del lado izquierdo de esta página encontrará documentación adicional de inicio rápido.

**Requisitos previos:**

- Un equipo (físico o virtual) con Windows Server 2016.
- Configure este sistema con la característica de contenedor de Windows y Docker. Para ver un tutorial sobre estos pasos, consulte [Windows Containers on Windows Server (Contenedores de Windows en Windows Server)](./quick-start-windows-server.md).
- Un identificador de Docker, que se usará para insertar una imagen de contenedor en Docker Hub. Si no tiene un identificador de Docker, suscríbase para obtenerlo en [Docker Cloud](https://cloud.docker.com/).

## <a name="1-container-image---manual"></a>1. Imagen de contenedor (manual)

Para una mejor experiencia, realice este ejercicio desde un shell de comandos (cmd.exe) de Windows.

El primer paso para la creación manual de una imagen de contenedor es implementar un contenedor. En este ejemplo, implemente un contenedor de IIS a partir de la imagen de IIS creada previamente. Una vez implementado el contenedor, trabajará en una sesión de shell desde dentro del contenedor. La sesión interactiva se inicia con la marca `-it`. Para obtener información más detallada sobre los comandos Run de Docker, consulte [Docker Run Reference (Referencia de Run de Docker)](https://docs.docker.com/engine/reference/run/) en Docker.com. 

> Este paso puede tardar debido al tamaño de la imagen base de Windows Server Core.

```none
docker run -d --name myIIS -p 80:80 microsoft/iis
```

Ahora, el contenedor se ejecutará en segundo plano. El comando predeterminado incluido en el contenedor, `ServiceMonitor.exe`, supervisa el progreso de IIS y detiene automáticamente el contenedor si el IIS se detiene. Para obtener más información sobre cómo se creó esta imagen, consulta [Microsoft/docker-iis](https://github.com/Microsoft/iis-docker) en GitHub.

A continuación, inicia un cmd interactivo en el contenedor. Esto te permitirá ejecutar comandos en el contenedor en ejecución sin detener IIS o ServiceMonitor.

```none
docker exec -i myIIS cmd 
```

A continuación, puedes hacer un cambio en el contenedor en ejecución. Ejecuta el siguiente comando para quitar la pantalla de presentación de IIS.

```none
del C:\inetpub\wwwroot\iisstart.htm
```

Y el siguiente para reemplazar el sitio de IIS predeterminado por un nuevo sitio estático.

```none
echo "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

Desde otro sistema, vaya a la dirección IP del host de contenedor. Ahora debería ver la aplicación "Hello World".

**Nota:** Si está trabajando en Azure, debe existir una regla de grupo de seguridad de red que permita el tráfico en el puerto 80. Para más información, consulte [Cómo administrar grupos de seguridad de red con el Portal de Azure](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg).

![](media/hello.png)

De vuelta en el contenedor, salga de la sesión interactiva del contenedor.

```none
exit
```

Ahora se puede capturar el contenedor modificado en una nueva imagen de contenedor. Para ello, necesitará el nombre del contenedor. Se puede encontrar mediante el comando `docker ps -a`.

```none
docker ps -a

CONTAINER ID     IMAGE                             COMMAND   CREATED             STATUS   PORTS   NAMES
489b0b447949     microsoft/iis   "cmd"     About an hour ago   Exited           pedantic_lichterman
```

Para crear una nueva imagen de contenedor, use el comando `docker commit`. La confirmación de Docker adopta el formato "confirmación de docker nombre de contenedor nombre de nueva imagen". Nota: Reemplace el nombre del contenedor de este ejemplo por el nombre real del contenedor.

```none
docker commit pedantic_lichterman modified-iis
```

Para comprobar que se ha creado la nueva imagen, use el comando `docker images`.  

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
modified-iis        latest              3e4fdb6ed3bc        About a minute ago   10.17 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago          9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago          9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago          9.344 GB
```

Ahora se puede implementar esta imagen. El contenedor resultante incluirá todas las modificaciones capturadas.

## <a name="2-container-image---dockerfile"></a>2. Imagen de contenedor (Dockerfile)

En el ejercicio anterior, se creó un contenedor de forma manual, se modificó y después se capturó en una nueva imagen de contenedor. Docker incluye un método para automatizar este proceso mediante un Dockerfile. Este ejercicio tendrá resultados prácticamente idénticos al anterior; solo que esta vez el proceso estará automatizado. Para este ejercicio, se requiere un identificador de Docker. Si no tiene un identificador de Docker, suscríbase para obtenerlo en [Docker Cloud]( https://cloud.docker.com/).

En el host de contenedor, cree un directorio `c:\build` y, en este directorio, cree un archivo denominado `Dockerfile`. Nota: El archivo no debe tener una extensión de archivo.

```none
powershell new-item c:\build\Dockerfile -Force
```

Abra el Dockerfile en el Bloc de notas.

```none
notepad c:\build\Dockerfile
```

Copie el texto siguiente en el Dockerfile y guarde el archivo. Estos comandos indican a Docker que cree una nueva imagen con `microsoft/iis` como base. Entonces el Dockerfile ejecuta los comandos especificados en la instrucción `RUN`, en este caso el archivo index.html se actualiza con nuevo contenido. 

Para obtener más información sobre los Dockerfiles, consulte [Dockerfiles on Windows (Dockerfiles en Windows)](../manage-docker/manage-windows-dockerfile.md).

```none
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

El comando `docker build` iniciará el proceso de compilación de la imagen. El parámetro `-t` indica al proceso de compilación que asigne un nombre a la nueva imagen `iis-dockerfile`. **Reemplace "user" con el nombre de usuario de la cuenta de Docker**. Si no tiene una cuenta de Docker, suscríbase para obtenerla en [Docker Cloud](https://cloud.docker.com/).

```none
docker build -t <user>/iis-dockerfile c:\Build
```

Una vez completado, podrá comprobar que la imagen se ha creado mediante el comando `docker images`.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

Ahora, implemente un contenedor con el comando siguiente y reemplace de nuevo user con su identificador de Docker.

```none
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

Una vez creado el contenedor, vaya a la dirección IP del host de contenedor. Debería ver la aplicación hello world.

![](media/dockerfile2.png)

De vuelta en el host de contenedor, use `docker ps` para obtener el nombre del contenedor y `docker rm` para quitar el contenedor. Nota: Reemplace el nombre del contenedor de este ejemplo por el nombre real del contenedor.

Obtenga el nombre del contenedor.

```none
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

Quite el contenedor.

```none
docker rm -f <container name>
```

## <a name="3-docker-push"></a>3. Inserción de Docker

Las imágenes de contenedor de Docker se pueden almacenar en un registro de contenedor. Una vez que se almacena una imagen en un registro, se puede recuperar para su uso posterior en varios hosts de contenedor. Docker proporciona un registro público para almacenar imágenes de contenedor en [Docker Hub](https://hub.docker.com/).

Para este ejercicio, se insertará la imagen personalizada hello world en su propia cuenta en Docker Hub.

En primer lugar, inicie sesión en su cuenta de Docker mediante el `docker login command`.

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

Una vez iniciada la sesión, se puede insertar la imagen de contenedor en Docker Hub. Para ello, use el comando `docker push`. **Reemplace "user" con el identificador de Docker**. 

```none
docker push <user>/iis-dockerfile
```

La imagen de contenedor ya se puede descargar desde Docker Hub en cualquier host de contenedor de Windows mediante `docker pull`. Para este tutorial, se eliminará la imagen existente y después se extraerá de Docker Hub. 

```none
docker rmi <user>/iis-dockerfile
```

Al ejecutar `docker images`, mostrará que se ha quitado la imagen.

```none
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
modified-iis              latest              51f1fe8470b3        5 minutes ago       7.69 GB
microsoft/iis             latest              e4525dda8206        3 hours ago         7.61 GB
```

Por último, la extracción de Docker se puede usar para devolver la imagen al host del contenedor. Reemplace user con el nombre de usuario de la cuenta de Docker. 

```none
docker pull <user>/iis-dockerfile
```

## <a name="next-steps"></a>Pasos siguientes

[Contenedores de Windows en Windows 10](./quick-start-windows-10.md)
