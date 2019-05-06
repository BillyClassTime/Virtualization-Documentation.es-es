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
ms.openlocfilehash: 9ff6256ab9708533f72e9b3210f8a5fd32f4048a
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610275"
---
# <a name="dockerfile-on-windows"></a>Dockerfile en Windows

El motor de Docker incluye herramientas que automatizan la creación de imágenes de contenedor. Aunque puedes crear imágenes de contenedor manualmente mediante la ejecución de la `docker commit` de comandos, la adopción de un proceso de creación de imágenes automatizada presenta muchas ventajas, incluido:

- Almacenamiento de las imágenes del contenedor como código.
- Posibilidad de volver a crear de manera rápida y precisa las imágenes del contenedor con fines de mantenimiento y actualización.
- Integración continua entre las imágenes del contenedor y el ciclo de desarrollo.

Los componentes de Docker que controlan esta automatización son el archivo Dockerfile y el comando `docker build`.

El Dockerfile es un archivo de texto que contiene las instrucciones necesarias para crear una nueva imagen de contenedor. Estas instrucciones incluyen la identificación de una imagen existente que se usará como base, los comandos que se ejecutarán durante el proceso de creación de la imagen y un comando que se ejecutará cuando se implementen instancias nuevas de la imagen del contenedor.

Compilación de docker es el comando del motor de Docker que consume un archivo Dockerfile y desencadena el proceso de creación de imagen.

Este tema muestra cómo usar los archivos Dockerfile con contenedores de Windows, comprender su sintaxis básica y cuáles son las instrucciones Dockerfile más comunes.

Este documento describe el concepto de imágenes de contenedor y capas de imagen de contenedor. Si quieres obtener más información sobre las imágenes y disposición en capas, consulta [la Guía de inicio rápido a las imágenes](../quick-start/quick-start-images.md).

Para obtener una vista completa archivos dockerfile, consulta la [referencia sobre Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="basic-syntax"></a>Sintaxis básica

En su forma más básica, un archivo Dockerfile puede ser muy simple. En el ejemplo siguiente se crea una imagen, que incluye IIS, y un sitio de "hello world". Este ejemplo incluye comentarios (indicados con el símbolo `#`) que explican cada paso. En las secciones posteriores del artículo se ofrecerá más información sobre las reglas de sintaxis de Dockerfile y las instrucciones Dockerfile.

>[!NOTE]
>Un archivo Dockerfile debe crearse sin extensión. Para hacer esto en Windows, crea el archivo con el editor y guárdelo con la notación "Dockerfile" (incluidas las comillas).

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM microsoft/windowsservercore

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Para obtener más ejemplos de archivos Dockerfile para Windows, consulta el [repositorio de Dockerfile para Windows](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## <a name="instructions"></a>Instrucciones

Las instrucciones Dockerfile proporcionan al motor de Docker las instrucciones que necesita para crear una imagen de contenedor. Estas instrucciones se ejecutan uno por uno y en orden. Los siguientes ejemplos son las más utilizadas en Dockerfiles. Para obtener una lista completa de instrucciones de Dockerfile, consulta la [referencia sobre Dockerfile](https://docs.docker.com/engine/reference/builder/).

### <a name="from"></a>FROM

La instrucción `FROM` establece la imagen del contenedor que se usará durante el proceso de creación de la imagen. Por ejemplo, cuando se usa la instrucción `FROM microsoft/windowsservercore`, la imagen resultante deriva de la imagen base del sistema operativo de Windows Server Core y tiene una dependencia en esta. Si la imagen especificada no está presente en el sistema donde se ejecuta el proceso de compilación de Docker, el motor de Docker intentará descargar la imagen de un Registro de imágenes público o privado.

Formato de la instrucción FROM pasa como este:

```dockerfile
FROM <image>
```

Este es un ejemplo del comando desde:

```dockerfile
FROM microsoft/windowsservercore
```

Para obtener más información, consulta la [referencia FROM](https://docs.docker.com/engine/reference/builder/#from).

### <a name="run"></a>RUN

La instrucción `RUN` especifica los comandos que se ejecutarán y se capturarán en la nueva imagen del contenedor. Estos comandos pueden incluir elementos como la instalación de software, la creación de archivos y directorios y la creación de la configuración del entorno.

La instrucción RUN pasa como este:

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

La diferencia entre la Exec Form y shell form es en cómo el `RUN` se ejecuta la instrucción. Cuando se usa el método exec form, el programa especificado se ejecuta explícitamente.

Este es un ejemplo del método exec form:

```dockerfile
FROM microsoft/windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

La imagen resultante se ejecuta la `powershell New-Item c:/test` comando:

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

Por otro lado, en el siguiente ejemplo se ejecuta la misma operación en forma de shell:

```dockerfile
FROM microsoft/windowsservercore

RUN powershell New-Item c:\test
```

La imagen resultante tiene una instrucción run de `cmd /S /C powershell New-Item c:\test`.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Consideraciones para el uso de ejecución con Windows

En Windows, cuando se usa la instrucción `RUN` con el formato exec, las barras diagonales inversas deben llevar un símbolo de escape.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Cuando el programa de destino es Windows installer, tendrás que extraer el programa de instalación a través de la `/x:<directory>` marca para poder iniciar el procedimiento de instalación (silenciosa) real. También debe esperar para que el comando salir antes de hacer nada más. De lo contrario, el proceso finalizará prematuramente sin instalar nada. Para más información, vea el siguiente ejemplo.

#### <a name="examples-of-using-run-with-windows"></a>Ejemplos de uso de ejecución con Windows

El siguiente ejemplo Dockerfile usa DISM para instalar IIS en la imagen de contenedor:

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

En este ejemplo se instala el paquete redistribuible de Visual Studio. `Start-Process` y el `-Wait` parámetro se usan para ejecutar el programa de instalación. Esto garantiza que la instalación se complete antes de pasar a la siguiente instrucción en el Dockerfile.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Para obtener información detallada sobre la instrucción RUN, consulte la [referencia de ejecutar](https://docs.docker.com/engine/reference/builder/#run).

### <a name="copy"></a>COPIAR

El `COPY` instrucción copia archivos y directorios al sistema de archivos del contenedor. Los archivos y directorios deben estar en una ruta de acceso relativa a Dockerfile.

El `COPY` formato de la instrucción pasa como este:

```dockerfile
COPY <source> <destination>
```

Si el origen o el destino incluye espacios en blanco, escriba la ruta de acceso corchetes y comillas dobles, como se muestra en el ejemplo siguiente:

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Consideraciones para el uso de copia con Windows

En Windows, el formato de destino debe usar barras diagonales. Por ejemplo, estos elementos son válidos `COPY` instrucciones:

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

Mientras tanto, no funcionará el siguiente formato con barras diagonales inversas:

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Ejemplos de uso de copia con Windows

El siguiente ejemplo agrega el contenido del directorio de origen a un directorio denominado `sqllite` en la imagen de contenedor:

```dockerfile
COPY source /sqlite/
```

El siguiente ejemplo agrega todos los archivos que comienzan por "config" para el `c:\temp` directorio de la imagen de contenedor:

```dockerfile
COPY config* c:/temp/
```

Para obtener más información sobre la `COPY` instrucción, consulta la [referencia sobre COPY](https://docs.docker.com/engine/reference/builder/#copy).

### <a name="add"></a>ADD

La instrucción ADD es a la instrucción COPY, pero con funcionalidades aún más. Además de copiar archivos del host en la imagen del contenedor, la instrucción `ADD` también puede copiar archivos de una ubicación remota con una especificación de dirección URL.

El `ADD` formato de la instrucción pasa como este:

```dockerfile
ADD <source> <destination>
```

Si el origen o el destino incluye espacios en blanco, escriba la ruta de acceso corchetes y comillas dobles:

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Consideraciones para ejecutar Agregar con Windows

En Windows, el formato de destino debe usar barras diagonales. Por ejemplo, estos elementos son válidos `ADD` instrucciones:

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

Mientras tanto, no funcionará el siguiente formato con barras diagonales inversas:

```dockerfile
ADD test1.txt c:\temp\
```

Además, en Linux, la instrucción `ADD` expandirá los paquetes comprimidos al copiarlos. Esta funcionalidad no está disponible en Windows.

#### <a name="examples-of-using-add-with-windows"></a>Ejemplos de uso de agregar con Windows

El siguiente ejemplo agrega el contenido del directorio de origen a un directorio denominado `sqllite` en la imagen de contenedor:

```dockerfile
ADD source /sqlite/
```

El siguiente ejemplo agrega todos los archivos que comienzan por "config" a la `c:\temp` directorio de la imagen del contenedor.

```dockerfile
ADD config* c:/temp/
```

El siguiente ejemplo descargará Python para Windows en la `c:\temp` directorio de la imagen del contenedor.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Para obtener más información sobre la `ADD` instrucción, consulta el [Agregar referencia](https://docs.docker.com/engine/reference/builder/#add).

### <a name="workdir"></a>WORKDIR

La instrucción `WORKDIR` establece un directorio de trabajo para otras instrucciones de Dockerfile, como `RUN` y `CMD`, y también el directorio de trabajo para ejecutar instancias de la imagen del contenedor.

El `WORKDIR` formato de la instrucción pasa como este:

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Consideraciones para el uso sobre WORKDIR con Windows

En Windows, si el directorio de trabajo incluye una barra diagonal inversa, debe llevar un símbolo de escape.

```dockerfile
WORKDIR c:\\windows
```

**Ejemplos**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

Para obtener información detallada sobre la `WORKDIR` instrucción, consulta la [referencia sobre WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir).

### <a name="cmd"></a>CMD

La instrucción `CMD` establece que el comando predeterminado se ejecutará al implementar una instancia de la imagen del contenedor. Por ejemplo, si el contenedor va a hospedar un servidor web NGINX, el `CMD` puede incluir instrucciones para iniciar el servidor web con un comando como `nginx.exe`. Si se especifican varias instrucciones `CMD` en un archivo Dockerfile, solo se evalúa la última.

El `CMD` formato de la instrucción pasa como este:

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Consideraciones para el uso de CMD con Windows

En Windows, las rutas de acceso de archivo que se especifican en la instrucción `CMD` deben usar barras diagonales o incluir barras diagonales inversas de escape `\\`. Las siguientes son válidas `CMD` instrucciones:

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

Para obtener más información sobre la `CMD` instrucción, consulta la [referencia sobre CMD](https://docs.docker.com/engine/reference/builder/#cmd).

## <a name="escape-character"></a>Carácter de escape

En muchos casos las instrucciones Dockerfile que abarcar varias líneas. Para ello, puedes usar un carácter de escape. El carácter de escape predeterminado para Dockerfile es una barra diagonal inversa `\`. Sin embargo, debido a la barra diagonal inversa es un separador de ruta de acceso de archivo en Windows, usando para abarcar varias líneas puede causar problemas. Para solucionar esto, puedes usar una directiva de analizador para cambiar el carácter de escape predeterminado. Para obtener más información acerca de las directivas de analizador, consulta [las directivas de analizador](https://docs.docker.com/engine/reference/builder/#parser-directives).

El siguiente ejemplo muestra una instrucción RUN individual que abarca varias líneas con el carácter de escape predeterminado:

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Para modificar el carácter de escape, coloque una directiva del analizador de escape en la primera línea de la instrucción Dockerfile. Esto puede verse en el siguiente ejemplo.

>[!NOTE]
>Solo dos valores pueden utilizarse como caracteres de escape: `\` y `` ` ``.

```dockerfile
# escape=`

FROM microsoft/windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Para obtener más información acerca de la directiva del analizador de escape, consulte la [directiva del analizador de Escape](https://docs.docker.com/engine/reference/builder/#escape).

## <a name="powershell-in-dockerfile"></a>PowerShell en Dockerfile

### <a name="powershell-cmdlets"></a>Cmdlets de PowerShell

Se pueden ejecutar los cmdlets de PowerShell en un archivo Dockerfile con el `RUN` operación.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>Llamadas de REST

De PowerShell `Invoke-WebRequest` cmdlet puede resultar útil al recopilar información o archivos desde un servicio web. Por ejemplo, si creas una imagen que incluye Python, puedes establecer `$ProgressPreference` a `SilentlyContinue` para conseguir descargar más rápido, tal como se muestra en el siguiente ejemplo.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` También funciona en Nano Server.

Otra opción para usar PowerShell para descargar archivos durante el proceso de creación de la imagen es usar la biblioteca .NET WebClient. Esto puede aumentar el rendimiento de descarga. En el ejemplo siguiente se descarga el software de Python mediante la biblioteca WebClient.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Nano Server no admite actualmente WebClient.

### <a name="powershell-scripts"></a>Secuencias de comandos de PowerShell

En algunos casos, puede resultar útil copiar un script en los contenedores que se usa durante el proceso de creación de imagen y luego ejecutar el script desde dentro del contenedor.

>[!NOTE]
>Esto limitará cualquier capa de imagen en el almacenamiento en caché y reducirá la legibilidad del Dockerfile.

En este ejemplo se copia un script de la máquina de compilación en el contenedor mediante la instrucción `ADD`. Este script después se ejecuta con la instrucción RUN.

```dockerfile
FROM microsoft/windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Compilación de docker

Una vez que se ha creado un archivo Dockerfile y guardado en el disco, puedes ejecutar `docker build` para crear la nueva imagen. El comando `docker build` toma varios parámetros opcionales y una ruta de acceso al archivo Dockerfile. Para obtener documentación completa sobre la compilación de Docker, incluida una lista de todas las opciones de compilación, consulta la [referencia de compilación](https://docs.docker.com/engine/reference/commandline/build/#build).

El formato de los `docker build` comando va como este:

```dockerfile
docker build [OPTIONS] PATH
```

Por ejemplo, el comando siguiente creará una imagen denominada "iis".

```dockerfile
docker build -t iis .
```

Cuando se inicie el proceso de compilación, la salida indican el estado y devolverá los errores que se produzcan.

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM micrsoft/windowsservercore
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

El resultado es una nueva imagen de contenedor, que en este ejemplo se denomina "iis".

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>Otras lecturas y referencias

- [Optimizar la compilación de archivos Dockerfile y Docker para Windows](optimize-windows-dockerfile.md)
- [Referencia sobre Dockerfile](https://docs.docker.com/engine/reference/builder/)
