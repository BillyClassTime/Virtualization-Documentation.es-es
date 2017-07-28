---
title: Dockerfile y contenedores de Windows
description: "Creación de archivos Dockerfile para contenedores de Windows."
keywords: docker, contenedores
author: PatrickLang
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
ms.openlocfilehash: 8c5e89cd3afcb109fd3eda2da7bcd1b2c7f48b88
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/21/2017
---
# Dockerfile en Windows

El motor de Docker incluye herramientas para automatizar la creación de imágenes del contenedor. Aunque se pueden crear imágenes del contenedor manualmente mediante el comando `docker commit`, la adopción de un proceso automatizado de creación de imágenes ofrece numerosas ventajas, entre otras las siguientes:

- Almacenamiento de las imágenes del contenedor como código.
- Posibilidad de volver a crear de manera rápida y precisa las imágenes del contenedor con fines de mantenimiento y actualización.
- Integración continua entre las imágenes del contenedor y el ciclo de desarrollo.

Los componentes de Docker que controlan esta automatización son el archivo Dockerfile y el comando `docker build`.

- **Dockerfile**: archivo de texto que contiene las instrucciones necesarias para crear una nueva imagen del contenedor. Estas instrucciones incluyen la identificación de una imagen existente que se usará como base, los comandos que se ejecutarán durante el proceso de creación de la imagen y un comando que se ejecutará cuando se implementen instancias nuevas de la imagen del contenedor.
- **Docker build**: comando del motor de Docker que consume un archivo Dockerfile y desencadena el proceso de creación de la imagen.

En este documento se presenta el uso de un archivo Dockerfile con contenedores de Windows, se explica la sintaxis y se describen las instrucciones Dockerfile más usadas.

A lo largo del documento se describirán los conceptos de imágenes del contenedor y capas de imagen del contenedor. Para obtener más información sobre las imágenes y la disposición en capas consulta [la guía de inicio rápido a las imágenes](../quick-start/quick-start-images.md).

Para obtener información detallada sobre los archivos Dockerfile, consulta la [referencia sobre Dockerfile en docker.com]( https://docs.docker.com/engine/reference/builder/).

## Introducción a Dockerfile

### Sintaxis básica

En su forma más básica, un archivo Dockerfile puede ser muy simple. En el ejemplo siguiente se crea una imagen, que incluye IIS, y un sitio de "hello world". Este ejemplo incluye comentarios (indicados con el símbolo `#`) que explican cada paso. En las secciones posteriores del artículo se ofrecerá más información sobre las reglas de sintaxis de Dockerfile y las instrucciones Dockerfile.

> Ten en cuenta que el Dockerfile debe crearse sin extensión. Para hacer eso en Windows, tan solo tienes que crear el archivo con el editor que desees y, a continuación, guárdalo usando el nombre "Dockerfile", incluyendo las comillas.

```none
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM microsoft/windowsservercore

# Metadata indicating an image maintainer.
MAINTAINER jshelton@contoso.com

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Para obtener más ejemplos de archivos Dockerfile para Windows, consulta el [Repositorio de Dockerfile para Windows] (https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## Instrucciones

Las instrucciones Dockerfile proporcionan al motor de Docker los pasos necesarios para crear una imagen del contenedor. Estas instrucciones se ejecutan en orden y una a una. A continuación se detallan algunas instrucciones Dockerfile básicas. Para obtener una lista completa de instrucciones Dockerfile, consulte la [referencia sobre Dockerfile en Docker.com] (https://docs.docker.com/engine/reference/builder/).

### FROM

La instrucción `FROM` establece la imagen del contenedor que se usará durante el proceso de creación de la imagen. Por ejemplo, cuando se usa la instrucción `FROM microsoft/windowsservercore`, la imagen resultante deriva de la imagen base del sistema operativo de Windows Server Core y tiene una dependencia en esta. Si la imagen especificada no está presente en el sistema donde se ejecuta el proceso de compilación de Docker, el motor de Docker intentará descargar la imagen de un Registro de imágenes público o privado.

**Formato**

La instrucción FROM tiene el formato siguiente:

```
FROM <image>
```

**Ejemplo**

```
FROM microsoft/windowsservercore
```

Para obtener información detallada sobre la instrucción FROM, consulte la [referencia sobre FROM en Docker.com]( https://docs.docker.com/engine/reference/builder/#from).

### RUN

La instrucción `RUN` especifica los comandos que se ejecutarán y se capturarán en la nueva imagen del contenedor. Estos comandos pueden incluir elementos como la instalación de software, la creación de archivos y directorios y la creación de la configuración del entorno.

**Formato**

La instrucción RUN tiene el formato siguiente:

```none
# exec form

RUN ["<executable", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

La diferencia entre exec form y shell form es la manera en que se ejecuta la instrucción `RUN`. Cuando se usa el método exec form, el programa especificado se ejecuta explícitamente.

En el ejemplo siguiente se usa el método exec form.

```none
FROM microsoft/windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

Si se examina la imagen resultante, se observa que el comando que se ejecutó es `powershell New-Item c:/test`.

```none
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

Por otro lado, en el ejemplo siguiente se ejecuta la misma operación, pero se usa shell form.

```none
FROM microsoft/windowsservercore

RUN powershell New-Item c:\test
```

Esto tiene como resultado una instrucción RUN de `cmd /S /C powershell New-Item c:\test`.

```none
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

**Consideraciones sobre Windows**

En Windows, cuando se usa la instrucción `RUN` con el formato exec, las barras diagonales inversas deben llevar un símbolo de escape.

```none
RUN ["powershell", "New-Item", "c:\\test"]
```

Si el programa de destino es Windows Installer, se necesita un paso más para poder iniciar el procedimiento de instalación (silenciosa) real: extraer la instalación a través de la marca `/x:<directory>`. Además, hay que esperar la salida del comando. De lo contrario, el proceso finalizará prematuramente, sin instalar nada. Para más información, vea el siguiente ejemplo.

**Ejemplos**

En este ejemplo se usa DISM para instalar IIS en la imagen del contenedor.
```none
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

En este ejemplo se instala el paquete redistribuible de Visual Studio. Tenga en cuenta aquí que `Start-Process` y el parámetro `-Wait` se utilizan para ejecutar el programa de instalación. Esto garantizará que la instalación se completará antes de pasar al siguiente paso en el Dockerfile.

```none
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Para obtener información detallada sobre la instrucción RUN, consulte la [referencia sobre RUN en Docker.com]( https://docs.docker.com/engine/reference/builder/#run).

### COPIAR

La instrucción `COPY` copia archivos y directorios en el sistema de archivos del contenedor. Los archivos y los directorios deben encontrarse en una ruta de acceso relacionada con el archivo Dockerfile.

**Formato**

La instrucción `COPY` tiene el formato siguiente:

```none
COPY <source> <destination>
```

Si el origen o el destino incluyen espacios en blanco, escriba la ruta de acceso entre corchetes y comillas dobles.

```none
COPY ["<source>", "<destination>"]
```

**Consideraciones sobre Windows**

En Windows, el formato de destino debe usar barras diagonales. Por ejemplo, las siguientes son instrucciones `COPY` válidas.

```none
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

Pero la siguiente no funcionará.

```none
COPY test1.txt c:\temp\
```

**Ejemplos**

En este ejemplo se agrega el contenido del directorio de origen a un directorio denominado `sqllite` en la imagen del contenedor.
```none
COPY source /sqlite/
```

En este ejemplo se agregan todos los archivos que comienzan por "config" al directorio `c:\temp` de la imagen del contenedor.
```none
COPY config* c:/temp/
```

Para obtener información detallada sobre la instrucción `COPY`, consulte la [referencia sobre COPY en Docker.com]( https://docs.docker.com/engine/reference/builder/#copy).

### AGREGAR

La instrucción ADD es muy parecida a la instrucción COPY, pero incluye funcionalidades adicionales. Además de copiar archivos del host en la imagen del contenedor, la instrucción `ADD` también puede copiar archivos de una ubicación remota con una especificación de dirección URL.

**Formato**

La instrucción `ADD` tiene el formato siguiente:

```none
ADD <source> <destination>
```

Si el origen o el destino incluyen espacios en blanco, escriba la ruta de acceso entre corchetes y comillas dobles.

```none
ADD ["<source>", "<destination>"]
```

**Consideraciones sobre Windows**

En Windows, el formato de destino debe usar barras diagonales. Por ejemplo, las siguientes son instrucciones `ADD` válidas.

```none
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

Pero la siguiente no funcionará.

```none
ADD test1.txt c:\temp\
```

Además, en Linux, la instrucción `ADD` expandirá los paquetes comprimidos al copiarlos. Esta funcionalidad no está disponible en Windows.

**Ejemplos**

En este ejemplo se agrega el contenido del directorio de origen a un directorio denominado `sqllite` en la imagen del contenedor.
```none
ADD source /sqlite/
```

En este ejemplo se agregan todos los archivos que comienzan por "config" al directorio `c:\temp` de la imagen del contenedor.
```none
ADD config* c:/temp/
```

En este ejemplo se descargará Python para Windows en el directorio `c:\temp` de la imagen del contenedor.
```none
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Para obtener información detallada sobre la instrucción `ADD`, consulte la [referencia sobre ADD en Docker.com]( https://docs.docker.com/engine/reference/builder/#add).

### WORKDIR

La instrucción `WORKDIR` establece un directorio de trabajo para otras instrucciones de Dockerfile, como `RUN` y `CMD`, y también el directorio de trabajo para ejecutar instancias de la imagen del contenedor.

**Formato**

La instrucción `WORKDIR` tiene el formato siguiente:

```none
WORKDIR <path to working directory>
```

**Consideraciones sobre Windows**

En Windows, si el directorio de trabajo incluye una barra diagonal inversa, debe llevar un símbolo de escape.

```none
WORKDIR c:\\windows
```

**Ejemplos**

```none
WORKDIR c:\\Apache24\\bin
```

Para obtener información detallada sobre la instrucción `WORKDIR`, consulte la [referencia sobre WORKDIR en Docker.com]( https://docs.docker.com/engine/reference/builder/#workdir).

### CMD

La instrucción `CMD` establece que el comando predeterminado se ejecutará al implementar una instancia de la imagen del contenedor. Por ejemplo, si el contenedor va a hospedar un servidor web NGINX, `CMD` puede incluir instrucciones para iniciar el servidor web, como `nginx.exe`. Si se especifican varias instrucciones `CMD` en un archivo Dockerfile, solo se evalúa la última.

**Formato**

La instrucción `CMD` tiene el formato siguiente:

```none
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

**Consideraciones sobre Windows**

En Windows, las rutas de acceso de archivo que se especifican en la instrucción `CMD` deben usar barras diagonales o incluir barras diagonales inversas de escape `\\`. Por ejemplo, las siguientes son instrucciones `CMD` válidas.

```none
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```
Pero la siguiente no funcionará.

```none
CMD c:\Apache24\bin\httpd.exe -w
```

Para obtener información detallada sobre la instrucción `CMD`, consulta la [referencia sobre CMD en Docker.com](https://docs.docker.com/engine/reference/builder/#cmd).

## Carácter de escape

En muchos casos, las instrucciones Dockerfile tendrán que abarcar varias líneas, para lo que se usa un carácter de escape. El carácter de escape predeterminado para Dockerfile es una barra diagonal inversa `\`. Dado que la barra diagonal inversa es un separador en las rutas de acceso de archivo en Windows, pueden surgir problemas. Para cambiar el carácter de escape predeterminado, se puede utilizar una directiva de analizador. Para más información al respecto, consulte el apartado sobre [directivas de analizador en Docker.com](https://docs.docker.com/engine/reference/builder/#parser-directives).

En el ejemplo siguiente se muestra una instrucción RUN individual que abarca varias líneas en la que se usa el carácter de escape predeterminado.

```none
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Para modificar el carácter de escape, coloque una directiva del analizador de escape en la primera línea de la instrucción Dockerfile. A continuación, puede verse un ejemplo.

> Tenga en cuenta que solo hay dos valores que puedan utilizarse como caracteres de escape, `\` y `` ` ``.

```none
# escape=`

FROM microsoft/windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Para más información sobre la directiva del analizador de escape, consulte el apartado sobre la [directiva del analizador de escape en Docker.com](https://docs.docker.com/engine/reference/builder/#escape).

## PowerShell en Dockerfile

### Comandos de PowerShell

Pueden ejecutarse comandos de PowerShell en un archivo Dockerfile mediante la operación `RUN`.

```none
FROM microsoft/windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### Llamadas de REST

PowerShell y el comando `Invoke-WebRequest` pueden resultar útiles al recopilar información o archivos de un servicio web. Por ejemplo, si estás creando una imagen que incluye Python, podrías usar el ejemplo siguiente. Considera configurar `$ProgressPreference` como `SilentlyContinue` para conseguir descargar más rápido.

```none
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> Invoke-WebRequest también funciona en el Nano Server

Otra opción para usar PowerShell para descargar archivos durante el proceso de creación de la imagen es usar la biblioteca .NET WebClient. Esto puede aumentar el rendimiento de descarga. En el ejemplo siguiente se descarga el software de Python mediante la biblioteca WebClient.

```none
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> WebClient no se admite actualmente en Nano Server.

### Secuencias de comandos de PowerShell

En algunos casos, puede ser útil copiar un script en los contenedores que se usan durante el proceso de creación de la imagen y después ejecutarlo desde dentro del contenedor. Nota: Esto limitará el almacenamiento en caché de las capas de la imagen y reducirá la legibilidad del archivo Dockerfile.

En este ejemplo se copia un script de la máquina de compilación en el contenedor mediante la instrucción `ADD`. Este script después se ejecuta con la instrucción RUN.

```
FROM microsoft/windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## Compilación de Docker

Una vez que se ha creado un archivo Dockerfile y se ha guardado en el disco, se puede ejecutar `docker build` para crear la imagen. El comando `docker build` toma varios parámetros opcionales y una ruta de acceso al archivo Dockerfile. Para obtener documentación completa sobre la compilación de Docker, incluida una lista de todas las opciones de compilación, consulte la [referencia de compilación en Docker.com](https://docs.docker.com/engine/reference/commandline/build/#build).

```none
Docker build [OPTIONS] PATH
```
Por ejemplo, el comando siguiente creará una imagen denominada "iis".

```none
docker build -t iis .
```

Cuando se inicie el proceso de compilación, la salida indicará el estado y devolverá los errores que se produzcan.

```none
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

El resultado es una nueva imagen del contenedor, en este ejemplo denominada "iis".

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## Lecturas y referencias adicionales

[Optimización de los archivos Dockerfile y la compilación de Docker para Windows] (optimize-windows-dockerfile.md)

[Referencia sobre Dockerfile en Docker.com](https://docs.docker.com/engine/reference/builder/)
