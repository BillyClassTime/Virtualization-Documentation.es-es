---
title: Contenedores de Windows y Linux en Windows 10
description: Inicio rápido de implementación de contenedores
keywords: acoplador, contenedores, LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 3d651a4a68acefa25f1b647b1b33618bbfb91ae9
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129387"
---
# <a name="get-started-run-your-first-container"></a>Introducción: ejecutar el primer contenedor

En el [segmento anterior](./set-up-environment.md), hemos configurado nuestro entorno para ejecutar contenedores. En este ejercicio se muestra cómo extraer una imagen de contenedor y ejecutarla.

## <a name="install-container-base-image"></a>Imagen base del contenedor de instalación

Se crean instancias de todos los `container images`contenedores de. Microsoft ofrece varias imágenes "iniciales" (llamadas `base images`) para elegir. El comando siguientes extraerá la imagen base de Nano Server.

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

> [!TIP]
> Si ve un mensaje de error que dice `no matching manifest for unknown in the manifest list entries`, asegúrese de que el acoplador no esté configurado para ejecutar los contenedores de Linux.

Una vez extraída la imagen, puede comprobar que está en el equipo consultando el repositorio local de imágenes del Dock. La ejecución del `docker images` comando devuelve una lista de las imágenes instaladas, en este caso, la imagen de nano Server.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Lea el [CLUF](../images-eula.md)de la imagen del sistema de contenedores de Windows.

## <a name="run-your-first-windows-container"></a>Ejecutar el primer contenedor de Windows

Para este sencillo ejemplo, se creará e implementará una imagen de contenedor de "Hola a todos". Para obtener la mejor experiencia, ejecute estos comandos en un shell de Windows CMD o PowerShell con privilegios elevados.

> Windows PowerShell ISE no funciona en sesiones interactivas con contenedores. Aunque el contenedor se ejecuta, parece que se cuelga.

En primer lugar, inicie un contenedor con una sesión interactiva desde la imagen `nanoserver`. Una vez que se inicie el contenedor, se le presentará un shell de comandos desde el contenedor.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

Dentro del contenedor, crearemos un simple archivo de texto "Hola a todos".

```cmd
echo "Hello World!" > Hello.txt
```   

Una vez completado, salga del contenedor.

```cmd
exit
```

Crear una nueva imagen de contenedor desde el contenedor modificado. Para ver una lista de los contenedores que se están ejecutando o han salido, ejecute lo siguiente y tome nota del identificador de contenedor.

```console
docker ps -a
```

Ejecute el siguiente comando para crear la nueva imagen de "Hola a todos". Reemplace `<containerid>` con el identificador del contenedor.

```console
docker commit <containerid> helloworld
```

Una vez finalizado, tendrá una imagen personalizada que contiene el script de hola a todos. Esto puede verse con el comando siguiente.

```console
docker images
```

Por último, ejecute el contenedor con el `docker run` comando.

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

El resultado del `docker run` comando es que se creó un contenedor a partir de la imagen ' HelloWorld ', una instancia de CMD se inició en el contenedor y ejecutó una lectura de nuestro archivo (salida demostrada en el shell) y, a continuación, se detuvo y quitó el contenedor.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre cómo entienda una aplicación de ejemplo](./building-sample-app.md)
