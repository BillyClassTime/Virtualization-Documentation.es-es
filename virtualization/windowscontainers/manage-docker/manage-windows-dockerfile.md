---
title: Dockerfile y contenedores de Windows
description: Creación de archivos Dockerfile para contenedores de Windows.
keywords: docker, contenedores
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
ms.openlocfilehash: 9fef74c029dc3efc220b1f9924d2695cdbaa61be
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909665"
---
# <a name="dockerfile-on-windows"></a>Dockerfile en Windows

El motor de Docker incluye herramientas que automatizan la creación de imágenes de contenedor. Aunque puede crear imágenes de contenedor manualmente mediante la ejecución del comando `docker commit`, la adopción de un proceso automatizado de creación de imágenes tiene muchas ventajas, entre las que se incluyen:

- Almacenamiento de las imágenes del contenedor como código.
- Posibilidad de volver a crear de manera rápida y precisa las imágenes del contenedor con fines de mantenimiento y actualización.
- Integración continua entre las imágenes del contenedor y el ciclo de desarrollo.

Los componentes de Docker que controlan esta automatización son el archivo Dockerfile y el comando `docker build`.

Dockerfile es un archivo de texto que contiene las instrucciones necesarias para crear una nueva imagen de contenedor. Estas instrucciones incluyen la identificación de una imagen existente que se usará como base, los comandos que se ejecutarán durante el proceso de creación de la imagen y un comando que se ejecutará cuando se implementen instancias nuevas de la imagen del contenedor.

Docker Build es el comando del motor de Docker que usa un Dockerfile y desencadena el proceso de creación de la imagen.

En este tema se muestra cómo usar Dockerfiles con contenedores de Windows, entender su sintaxis básica y cuáles son las instrucciones de Dockerfile más comunes.

En este documento se describe el concepto de imágenes de contenedor y las capas de imagen de contenedor. Si desea obtener más información sobre las imágenes y las capas de imágenes, consulte el artículo sobre [imágenes de contenedor](../manage-containers/container-base-images.md).

Para obtener una visión completa de Dockerfiles, consulte la [referencia de Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="basic-syntax"></a>Sintaxis básica

En su forma más básica, un archivo Dockerfile puede ser muy simple. En el ejemplo siguiente se crea una imagen, que incluye IIS, y un sitio de "hello world". Este ejemplo incluye comentarios (indicados con el símbolo `#`) que explican cada paso. En las secciones posteriores del artículo se ofrecerá más información sobre las reglas de sintaxis de Dockerfile y las instrucciones Dockerfile.

>[!NOTE]
>Un Dockerfile debe crearse sin extensión. Para hacer esto en Windows, cree el archivo con el editor que desee y guárdelo con la notación "Dockerfile" (incluidas las comillas).

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM mcr.microsoft.com/windows/servercore:ltsc2019

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Para obtener ejemplos adicionales de Dockerfiles para Windows, consulte el [repositorio de Dockerfile para Windows](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## <a name="instructions"></a>Instrucciones

Las instrucciones de Dockerfile proporcionan al motor de Docker las instrucciones necesarias para crear una imagen de contenedor. Estas instrucciones se realizan una por una y en orden. Los ejemplos siguientes son las instrucciones más usadas en Dockerfiles. Para obtener una lista completa de las instrucciones de Dockerfile, consulte la [referencia de Dockerfile](https://docs.docker.com/engine/reference/builder/).

### <a name="from"></a>FROM

La instrucción `FROM` establece la imagen del contenedor que se usará durante el proceso de creación de la imagen. Por ejemplo, cuando se usa la instrucción `FROM mcr.microsoft.com/windows/servercore`, la imagen resultante deriva de la imagen base del sistema operativo de Windows Server Core y tiene una dependencia en esta. Si la imagen especificada no está presente en el sistema donde se ejecuta el proceso de compilación de Docker, el motor de Docker intentará descargar la imagen de un Registro de imágenes público o privado.

El formato de la instrucción FROM es similar al siguiente:

```dockerfile
FROM <image>
```

Este es un ejemplo del comando FROM:

Para descargar la versión ltsc2019 de Windows Server Core de Microsoft Container Registry (MCR):
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

Para obtener información más detallada, consulte la [referencia de](https://docs.docker.com/engine/reference/builder/#from).

### <a name="run"></a>RUN

La instrucción `RUN` especifica los comandos que se ejecutarán y se capturarán en la nueva imagen del contenedor. Estos comandos pueden incluir elementos como la instalación de software, la creación de archivos y directorios y la creación de la configuración del entorno.

La instrucción de ejecución es similar a la siguiente:

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

La diferencia entre el formulario exec y Shell está en el modo en que se ejecuta la instrucción `RUN`. Cuando se usa el método exec form, el programa especificado se ejecuta explícitamente.

Este es un ejemplo del formulario Exec:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

La imagen resultante ejecuta el comando `powershell New-Item c:/test`:

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

Por el contrario, en el ejemplo siguiente se ejecuta la misma operación en el formulario de Shell:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

La imagen resultante tiene una instrucción Run de `cmd /S /C powershell New-Item c:\test`.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Consideraciones sobre el uso de ejecutar con Windows

En Windows, cuando se usa la instrucción `RUN` con el formato exec, las barras diagonales inversas deben llevar un símbolo de escape.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Cuando el programa de destino es un instalador de Windows, deberá extraer el programa de instalación a través de la marca `/x:<directory>` antes de poder iniciar el procedimiento de instalación (silencioso) real. También debe esperar a que el comando salga antes de hacer nada más. De lo contrario, el proceso finalizará prematuramente sin necesidad de instalar nada. Para más información, vea el siguiente ejemplo.

#### <a name="examples-of-using-run-with-windows"></a>Ejemplos de uso de ejecutar con Windows

En el siguiente ejemplo Dockerfile se usa DISM para instalar IIS en la imagen de contenedor:

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

En este ejemplo se instala el paquete redistribuible de Visual Studio. `Start-Process` y el parámetro `-Wait` se usan para ejecutar el instalador. Esto garantiza que la instalación se complete antes de pasar a la siguiente instrucción de Dockerfile.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Para obtener información detallada sobre la instrucción RUN, consulte la [referencia de ejecución](https://docs.docker.com/engine/reference/builder/#run).

### <a name="copy"></a>COPIAR

La instrucción `COPY` copia archivos y directorios en el sistema de archivos del contenedor. Los archivos y directorios deben estar en una ruta de acceso relativa al Dockerfile.

El formato de la instrucción `COPY` es similar al siguiente:

```dockerfile
COPY <source> <destination>
```

Si el origen o el destino incluyen un espacio en blanco, escriba la ruta de acceso entre corchetes y comillas dobles, tal como se muestra en el ejemplo siguiente:

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Consideraciones sobre el uso de la copia con Windows

En Windows, el formato de destino debe usar barras diagonales. Por ejemplo, son instrucciones de `COPY` válidas:

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

Mientras tanto, el formato siguiente con barras diagonales inversas no funcionará:

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Ejemplos de uso de COPY con Windows

En el ejemplo siguiente se agrega el contenido del directorio de origen a un directorio denominado `sqllite` en la imagen del contenedor:

```dockerfile
COPY source /sqlite/
```

En el ejemplo siguiente se agregarán todos los archivos que comienzan por config al directorio `c:\temp` de la imagen del contenedor:

```dockerfile
COPY config* c:/temp/
```

Para obtener información más detallada sobre la instrucción `COPY`, vea la [referencia de copia](https://docs.docker.com/engine/reference/builder/#copy).

### <a name="add"></a>AGREGAR

La instrucción ADD es similar a la instrucción COPY, pero con incluso más funcionalidades. Además de copiar archivos del host en la imagen del contenedor, la instrucción `ADD` también puede copiar archivos de una ubicación remota con una especificación de dirección URL.

El formato de la instrucción `ADD` es similar al siguiente:

```dockerfile
ADD <source> <destination>
```

Si el origen o el destino incluyen espacios en blanco, escriba la ruta de acceso entre corchetes y comillas dobles:

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Consideraciones para ejecutar ADD con Windows

En Windows, el formato de destino debe usar barras diagonales. Por ejemplo, son instrucciones de `ADD` válidas:

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

Mientras tanto, el formato siguiente con barras diagonales inversas no funcionará:

```dockerfile
ADD test1.txt c:\temp\
```

Además, en Linux, la instrucción `ADD` expandirá los paquetes comprimidos al copiarlos. Esta funcionalidad no está disponible en Windows.

#### <a name="examples-of-using-add-with-windows"></a>Ejemplos de uso de ADD con Windows

En el ejemplo siguiente se agrega el contenido del directorio de origen a un directorio denominado `sqllite` en la imagen del contenedor:

```dockerfile
ADD source /sqlite/
```

En el ejemplo siguiente se agregarán todos los archivos que comienzan por "config" en el directorio `c:\temp` de la imagen del contenedor.

```dockerfile
ADD config* c:/temp/
```

En el siguiente ejemplo se descargará Python para Windows en el directorio `c:\temp` de la imagen de contenedor.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Para obtener información más detallada sobre la instrucción `ADD`, vea la [referencia de Add](https://docs.docker.com/engine/reference/builder/#add).

### <a name="workdir"></a>WORKDIR

La instrucción `WORKDIR` establece un directorio de trabajo para otras instrucciones de Dockerfile, como `RUN` y `CMD`, y también el directorio de trabajo para ejecutar instancias de la imagen del contenedor.

El formato de la instrucción `WORKDIR` es similar al siguiente:

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Consideraciones sobre el uso de WORKDIR con Windows

En Windows, si el directorio de trabajo incluye una barra diagonal inversa, debe llevar un símbolo de escape.

```dockerfile
WORKDIR c:\\windows
```

**Ejemplos**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

Para obtener información detallada sobre la instrucción `WORKDIR`, consulte la [referencia de WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir).

### <a name="cmd"></a>CMD

La instrucción `CMD` establece que el comando predeterminado se ejecutará al implementar una instancia de la imagen del contenedor. Por ejemplo, si el contenedor va a hospedar un servidor Web NGINX, el `CMD` podría incluir instrucciones para iniciar el servidor Web con un comando como `nginx.exe`. Si se especifican varias instrucciones `CMD` en un archivo Dockerfile, solo se evalúa la última.

El formato de la instrucción `CMD` es similar al siguiente:

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Consideraciones sobre el uso de CMD con Windows

En Windows, las rutas de acceso de archivo que se especifican en la instrucción `CMD` deben usar barras diagonales o incluir barras diagonales inversas de escape `\\`. Las siguientes son instrucciones de `CMD` válidas:

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

Sin embargo, el siguiente formato sin las barras diagonales adecuadas no funcionará:

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

Para obtener información más detallada sobre la instrucción `CMD`, vea la [referencia de CMD](https://docs.docker.com/engine/reference/builder/#cmd).

## <a name="escape-character"></a>Carácter de escape

En muchos casos, una instrucción Dockerfile tendrá que abarcar varias líneas. Para ello, puede usar un carácter de escape. El carácter de escape predeterminado para Dockerfile es una barra diagonal inversa `\`. Sin embargo, dado que la barra diagonal inversa es también un separador de ruta de acceso de archivo en Windows, su uso para abarcar varias líneas puede causar problemas. Para evitar esto, puede usar una directiva de analizador para cambiar el carácter de escape predeterminado. Para obtener más información sobre las directivas de analizador, vea [directivas de analizador](https://docs.docker.com/engine/reference/builder/#parser-directives).

En el ejemplo siguiente se muestra una sola instrucción de ejecución que abarca varias líneas mediante el carácter de escape predeterminado:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Para modificar el carácter de escape, coloque una directiva del analizador de escape en la primera línea de la instrucción Dockerfile. Esto puede verse en el ejemplo siguiente.

>[!NOTE]
>Solo se pueden usar dos valores como caracteres de escape: `\` y `` ` ``.

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Para obtener más información sobre la Directiva de analizador de escape, vea [Directiva de analizador de escape](https://docs.docker.com/engine/reference/builder/#escape).

## <a name="powershell-in-dockerfile"></a>PowerShell en Dockerfile

### <a name="powershell-cmdlets"></a>Cmdlets de PowerShell

Los cmdlets de PowerShell se pueden ejecutar en un Dockerfile con la operación `RUN`.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>Llamadas de REST

El cmdlet de `Invoke-WebRequest` de PowerShell puede ser útil para recopilar información o archivos de un servicio Web. Por ejemplo, Si compila una imagen que incluye Python, puede establecer `$ProgressPreference` en `SilentlyContinue` para lograr descargas más rápidas, tal como se muestra en el ejemplo siguiente.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` también funciona en nano Server.

Otra opción para usar PowerShell para descargar archivos durante el proceso de creación de la imagen es usar la biblioteca .NET WebClient. Esto puede aumentar el rendimiento de descarga. En el ejemplo siguiente se descarga el software de Python mediante la biblioteca WebClient.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Nano Server no admite actualmente WebClient.

### <a name="powershell-scripts"></a>Secuencias de comandos de PowerShell

En algunos casos, puede resultar útil copiar un script en los contenedores que se usan durante el proceso de creación de la imagen y, a continuación, ejecutar el script desde dentro del contenedor.

>[!NOTE]
>Esto limitará el almacenamiento en caché del nivel de imagen y disminuirá la legibilidad del Dockerfile.

En este ejemplo se copia un script de la máquina de compilación en el contenedor mediante la instrucción `ADD`. Este script después se ejecuta con la instrucción RUN.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Compilación de Docker

Una vez que se ha creado un Dockerfile y guardado en el disco, puede ejecutar `docker build` para crear la nueva imagen. El comando `docker build` toma varios parámetros opcionales y una ruta de acceso al archivo Dockerfile. Para obtener documentación completa sobre la compilación de Docker, incluida una lista de todas las opciones de compilación, vea la [referencia de compilación](https://docs.docker.com/engine/reference/commandline/build/#build).

El formato del comando `docker build` es similar al siguiente:

```dockerfile
docker build [OPTIONS] PATH
```

Por ejemplo, el comando siguiente creará una imagen denominada "IIS".

```dockerfile
docker build -t iis .
```

Una vez iniciado el proceso de compilación, el resultado indicará el estado y devolverá los errores que se produzcan.

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM mcr.microsoft.com/windows/servercore:ltsc2019
 ---> 6801d964fda5

Step 2 : RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
 ---> Running in ae8759fb47db

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0

Enabling feature(s)
The operation completed successfully.

 ---> 4cd675d35444
Removing intermediate container ae8759fb47db

Step 3 : RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
 ---> Running in 9a26b8bcaa3a
 ---> e2aafdfbe392
Removing intermediate container 9a26b8bcaa3a

Successfully built e2aafdfbe392
```

El resultado es una nueva imagen de contenedor, que en este ejemplo se denomina "IIS".

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>Lecturas y referencias adicionales

- [Optimizar la compilación de Dockerfiles y Docker para Windows](optimize-windows-dockerfile.md)
- [Referencia de Dockerfile](https://docs.docker.com/engine/reference/builder/)
