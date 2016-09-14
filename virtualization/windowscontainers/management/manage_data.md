---
title: "Volúmenes de datos de contenedor"
description: "Cree y administre volúmenes de datos con contenedores de Windows."
keywords: docker, contenedores
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: f5998534-917b-453c-b873-2953e58535b1
translationtype: Human Translation
ms.sourcegitcommit: 08f893b646046d18def65602eb926bc0ea211804
ms.openlocfilehash: a175091c943cf596b2a810245b1b73b8baddb0c6

---

# Volúmenes de datos de contenedor

**Esto es contenido preliminar y está sujeto a cambios.** 

Al crear contenedores, podría tener que crear un nuevo directorio de datos o agregar un directorio existente al contenedor. Para hacerlo, puede agregar volúmenes de datos. Los volúmenes de datos son visibles para el contenedor y el host de contenedor, y se pueden compartir datos entre ellos. Los volúmenes de datos también se pueden compartir entre varios contenedores del mismo host de contenedor. En este documento se detallará el proceso de crear, inspeccionar y eliminar volúmenes de datos.

## Volúmenes de volúmenes

### Crear un volumen de datos

Cree un volumen de datos mediante el parámetro `-v` del comando `docker run`. De manera predeterminada, los volúmenes de datos nuevos se almacenan en el host en "c:\ProgramData\Docker\volumes".

En este ejemplo se crea un volumen de datos denominado "new-data-volume". Este volumen de datos será accesible en el contenedor en ejecución en "c:\new-data-volume".

```none
docker run -it -v c:\new-data-volume windowsservercore cmd
```

Para obtener más información sobre la creación de volúmenes, consulte [Manage data in containers (Administrar datos de contenedores) en docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#data-volumes).

### Montar un directorio existente

Además de crear un volumen de datos, podría interesarle pasar un directorio existente del host al contenedor. Puede hacerlo con el parámetro `-v` del comando `docker run`. Todos los archivos incluidos en el directorio de host también estarán disponibles en el contenedor. Los archivos que cree el contenedor en el volumen montado estarán disponibles en el host. Se puede montar el mismo directorio en muchos contenedores. En esta configuración, los datos pueden compartirse entre los contenedores.

En este ejemplo, el directorio de origen "c:\source" está montado en un contenedor como "c:\destination".

```none
docker run -it -v c:\source:c:\destination windowsservercore cmd
```

Para obtener más información sobre el montaje de directorios de host, consulte [Manage data in containers (Administrar datos de contenedores) en docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-directory-as-a-data-volume).

### Montar un solo archivo

No se puede montar un único archivo en un contenedor de Windows. Al ejecutar el siguiente comando no se devolverá ningún error, pero el contenedor resultante no contendrá el archivo. 

```none
docker run -it -v c:\config\config.ini microsoft/windowsservercore cmd
```

Como alternativa a eso, todos los archivos que quiera montar en un contenedor deberán montarse desde un directorio.

```none
docker run -it -v c:\config:c:\config microsoft/windowsservercore cmd
```

### Montaje de la unidad completa

Se puede montar una unidad completa con un comando similar al siguiente. Tenga en cuenta que no debe incluir ninguna barra diagonal inversa.

```none
docker run -it -v d: windowsservercore cmd
```

En este momento, el montaje de una parte de la segunda unidad no funciona. Por ejemplo, lo siguiente no es posible.

```none
docker run -it -v d:\source:d:\destination windowsservercore cmd
```

### Contenedores de volúmenes de datos

Los volúmenes de datos se pueden heredar de otros contenedores en ejecución mediante el parámetro `--volumes-from` del comando `docker run`. Mediante esta herencia, se puede crear un contenedor con el propósito explícito de hospedar volúmenes de datos para aplicaciones incluidas en contenedores. 

En este ejemplo se montan los volúmenes de datos del contenedor "cocky_bell" en un nuevo contenedor. Una vez que se inicie el nuevo contenedor, los datos que se encuentran en este volumen estarán disponibles para las aplicaciones que se ejecuten en el contenedor.  

```none
docker run -it --volumes-from cocky_bell windowsservercore cmd
```

Para obtener más información sobre los contenedores de datos, consulte [Manage data in containers (Administrar datos de contenedores) en docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-file-as-a-data-volume).

### Inspeccionar volúmenes de datos compartidos

Los volúmenes montados pueden verse con el comando `docker inspect`.

```none
docker inspect backstabbing_kowalevski
```

Esto devolverá información sobre el contenedor, incluida una sección denominada "Mounts" con datos sobre los volúmenes montados, como el directorio de origen y de destino.

```none
"Mounts": [
    {
        "Source": "c:\\container-share",
        "Destination": "c:\\data",
        "Mode": "",
        "RW": true,
        "Propagation": ""
}
```

Para obtener más información sobre la inspección de volúmenes, consulte [Manage data in containers (Administrar datos de contenedores) en docker.com](https://docs.docker.com/engine/userguide/containers/dockervolumes/#locating-a-volume).




<!--HONumber=Sep16_HO1-->


