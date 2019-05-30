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
ms.openlocfilehash: c08fa4d0a89bddeddd0f0a918345c33a6e2ab893
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2019
ms.locfileid: "9680995"
---
# <a name="dockerfile-on-windows"></a>Dockerfile en Windows

El motor de acoplamiento incluye herramientas que automatizan la creación de imágenes contenedoras. Aunque puede crear imágenes de contenedor de forma manual al `docker commit` ejecutar el comando, adoptar un proceso de creación de imágenes automatizadas tiene muchos beneficios, entre los que se incluyen:

- Almacenamiento de las imágenes del contenedor como código.
- Posibilidad de volver a crear de manera rápida y precisa las imágenes del contenedor con fines de mantenimiento y actualización.
- Integración continua entre las imágenes del contenedor y el ciclo de desarrollo.

Los componentes de Docker que controlan esta automatización son el archivo Dockerfile y el comando `docker build`.

El Dockerfile es un archivo de texto que contiene las instrucciones necesarias para crear una nueva imagen de contenedor. Estas instrucciones incluyen la identificación de una imagen existente que se usará como base, los comandos que se ejecutarán durante el proceso de creación de la imagen y un comando que se ejecutará cuando se implementen instancias nuevas de la imagen del contenedor.

La compilación del acoplador es el comando del motor de acoplamiento que consume una Dockerfile y desencadena el proceso de creación de la imagen.

En este tema se explica cómo usar Dockerfiles con contenedores de Windows, comprender su sintaxis básica y cuáles son las instrucciones de Dockerfile más comunes.

En este documento se tratará el concepto de imágenes contenedoras y capas de imágenes contenedoras. Si desea obtener más información sobre las imágenes y la capa de imágenes, consulte [la guía de inicio rápido de las imágenes](../quick-start/quick-start-images.md).

Para obtener una visión completa de Dockerfiles, consulta la [referencia de Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="basic-syntax"></a>Sintaxis básica

En su forma más básica, un archivo Dockerfile puede ser muy simple. En el ejemplo siguiente se crea una imagen, que incluye IIS, y un sitio de "hello world". Este ejemplo incluye comentarios (indicados con el símbolo `#`) que explican cada paso. En las secciones posteriores del artículo se ofrecerá más información sobre las reglas de sintaxis de Dockerfile y las instrucciones Dockerfile.

>[!NOTE]
>Se debe crear una Dockerfile sin extensión. Para hacerlo en Windows, cree el archivo con el editor que desee y, a continuación, guárdelo con la notación "Dockerfile" (incluidas las comillas).

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

Para obtener más ejemplos de Dockerfiles para Windows, consulte el [Catálogo de Dockerfile para Windows](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## <a name="instructions"></a>Instrucciones

Instrucciones Dockerfile proporcionan al motor de acoplamiento las instrucciones que necesita para crear una imagen de contenedor. Estas instrucciones se realizan una por una y en orden. Los ejemplos siguientes son las instrucciones que se usan con más frecuencia en Dockerfiles. Para obtener una lista completa de instrucciones Dockerfile, consulte la [referencia de Dockerfile](https://docs.docker.com/engine/reference/builder/).

### <a name="from"></a>FROM

La instrucción `FROM` establece la imagen del contenedor que se usará durante el proceso de creación de la imagen. Por ejemplo, cuando se usa la instrucción `FROM microsoft/windowsservercore`, la imagen resultante deriva de la imagen base del sistema operativo de Windows Server Core y tiene una dependencia en esta. Si la imagen especificada no está presente en el sistema donde se ejecuta el proceso de compilación de Docker, el motor de Docker intentará descargar la imagen de un Registro de imágenes público o privado.

El formato de la instrucción FROM es similar a este:

```dockerfile
FROM <image>
```

Este es un ejemplo del comando de:

Para descargar la versión ltsc2019 de Windows Server Core desde el registro del contenedor de Microsoft (MCR):
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

Para obtener información más detallada, consulte la [referencia de](https://docs.docker.com/engine/reference/builder/#from).

### <a name="run"></a>RUN

La instrucción `RUN` especifica los comandos que se ejecutarán y se capturarán en la nueva imagen del contenedor. Estos comandos pueden incluir elementos como la instalación de software, la creación de archivos y directorios y la creación de la configuración del entorno.

La instrucción de ejecución es similar a esta:

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

La diferencia entre el formulario exec y el shell está en el `RUN` modo en que se ejecuta la instrucción. Cuando se usa el método exec form, el programa especificado se ejecuta explícitamente.

Este es un ejemplo del formulario Exec:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

La imagen resultante ejecuta el `powershell New-Item c:/test` comando:

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

Para contrastarlo, el siguiente ejemplo ejecuta la misma operación en forma de Shell:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

La imagen resultante tiene una instrucción máquina ejecutar `cmd /S /C powershell New-Item c:\test`de.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Consideraciones para el uso de ejecutar con Windows

En Windows, cuando se usa la instrucción `RUN` con el formato exec, las barras diagonales inversas deben llevar un símbolo de escape.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Cuando el programa de destino sea Windows Installer, tendrá que extraer la configuración a través de la `/x:<directory>` bandera antes de poder iniciar el procedimiento de instalación real (silencioso). También debe esperar a que se cierre el comando antes de hacer nada más. De lo contrario, el proceso finalizará prematuramente sin instalar nada. Para más información, vea el siguiente ejemplo.

#### <a name="examples-of-using-run-with-windows"></a>Ejemplos de uso de ejecutar con Windows

En el siguiente ejemplo Dockerfile se usa DISM para instalar IIS en la imagen del contenedor:

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

En este ejemplo se instala el paquete redistribuible de Visual Studio. `Start-Process` y el `-Wait` parámetro se usa para ejecutar el instalador. De esta forma, se asegurará de que la instalación se complete antes de pasar a la siguiente instrucción del Dockerfile.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Para obtener información detallada sobre las instrucciones de ejecución, consulte la [referencia de ejecutar](https://docs.docker.com/engine/reference/builder/#run).

### <a name="copy"></a>COPIAR

La `COPY` instrucción copia archivos y directorios en el sistema de archivos del contenedor. Los archivos y los directorios deben estar en una ruta de acceso relativa al Dockerfile.

El `COPY` formato de la instrucción es similar a este:

```dockerfile
COPY <source> <destination>
```

Si origen o destino incluyen espacios en blanco, escriba la ruta de acceso entre corchetes y comillas dobles, como se muestra en el siguiente ejemplo:

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Consideraciones sobre el uso de copiar con Windows

En Windows, el formato de destino debe usar barras diagonales. Por ejemplo, estas son instrucciones `COPY` válidas:

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

Mientras tanto, no funcionará el siguiente formato con barras diagonales inversas:

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Ejemplos de uso de copiar con Windows

En el ejemplo siguiente se agrega el contenido del directorio de origen a un `sqllite` directorio con el nombre de la imagen del contenedor:

```dockerfile
COPY source /sqlite/
```

En el siguiente ejemplo, se agregarán todos los archivos que comienzan `c:\temp` por config al directorio de la imagen del contenedor:

```dockerfile
COPY config* c:/temp/
```

Para obtener información más detallada sobre `COPY` la instrucción, vea la [referencia de copiar](https://docs.docker.com/engine/reference/builder/#copy).

### <a name="add"></a>ADD

La instrucción ADD es como la instrucción COPY, pero aún más capacidades. Además de copiar archivos del host en la imagen del contenedor, la instrucción `ADD` también puede copiar archivos de una ubicación remota con una especificación de dirección URL.

El `ADD` formato de la instrucción es similar a este:

```dockerfile
ADD <source> <destination>
```

Si el origen o el destino incluyen espacios en blanco, encierra la ruta entre corchetes y las comillas dobles:

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Consideraciones sobre la ejecución de ADD with Windows

En Windows, el formato de destino debe usar barras diagonales. Por ejemplo, estas son instrucciones `ADD` válidas:

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

En el ejemplo siguiente se agrega el contenido del directorio de origen a un `sqllite` directorio con el nombre de la imagen del contenedor:

```dockerfile
ADD source /sqlite/
```

En el siguiente ejemplo, se agregarán todos los archivos que comienzan por " `c:\temp` config" al directorio de la imagen del contenedor.

```dockerfile
ADD config* c:/temp/
```

En el ejemplo siguiente se descargará Python para `c:\temp` Windows en el directorio de la imagen del contenedor.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Para obtener información más detallada sobre `ADD` la instrucción, vea [Agregar referencia](https://docs.docker.com/engine/reference/builder/#add).

### <a name="workdir"></a>WORKDIR

La instrucción `WORKDIR` establece un directorio de trabajo para otras instrucciones de Dockerfile, como `RUN` y `CMD`, y también el directorio de trabajo para ejecutar instancias de la imagen del contenedor.

El `WORKDIR` formato de la instrucción es similar a este:

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

Para obtener información detallada sobre `WORKDIR` las instrucciones, consulte la [referencia de WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir).

### <a name="cmd"></a>CMD

La instrucción `CMD` establece que el comando predeterminado se ejecutará al implementar una instancia de la imagen del contenedor. Por ejemplo, si el contenedor va a hospedar un servidor Web de NGINX `CMD` , el podría incluir instrucciones para iniciar el servidor Web con un `nginx.exe`comando como. Si se especifican varias instrucciones `CMD` en un archivo Dockerfile, solo se evalúa la última.

El `CMD` formato de la instrucción es similar a este:

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Consideraciones sobre el uso de CMD con Windows

En Windows, las rutas de acceso de archivo que se especifican en la instrucción `CMD` deben usar barras diagonales o incluir barras diagonales inversas de escape `\\`. Las siguientes son instrucciones `CMD` válidas:

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

Para obtener información más detallada sobre `CMD` la instrucción, vea la [referencia de CMD](https://docs.docker.com/engine/reference/builder/#cmd).

## <a name="escape-character"></a>Carácter de escape

En muchos casos, una instrucción Dockerfile debe abarcar varias líneas. Para ello, puede usar un carácter de escape. El carácter de escape predeterminado para Dockerfile es una barra diagonal inversa `\`. Sin embargo, como la barra diagonal inversa también es un separador de ruta de acceso de archivos en Windows, su uso para abarcar varias líneas puede causar problemas. Para evitar esto, puede usar una directiva del analizador para cambiar el carácter de escape predeterminado. Para obtener más información sobre las directivas del analizador, vea [directivas del analizador](https://docs.docker.com/engine/reference/builder/#parser-directives).

En el ejemplo siguiente se muestra una única instrucción de ejecución que abarca varias líneas con el carácter de escape predeterminado:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Para modificar el carácter de escape, coloque una directiva del analizador de escape en la primera línea de la instrucción Dockerfile. Esto se puede ver en el ejemplo siguiente.

>[!NOTE]
>Solo se pueden usar dos valores como caracteres de escape `\` : `` ` ``y.

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Para obtener más información sobre la Directiva del analizador de escape, consulte la [Directiva del analizador de escape](https://docs.docker.com/engine/reference/builder/#escape).

## <a name="powershell-in-dockerfile"></a>PowerShell en Dockerfile

### <a name="powershell-cmdlets"></a>Cmdlets de PowerShell

Los cmdlets de PowerShell se pueden ejecutar en un Dockerfile `RUN` con la operación.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>Llamadas de REST

El cmdlet `Invoke-WebRequest` de PowerShell puede ser útil para recopilar información o archivos de un servicio Web. Por ejemplo, si crea una imagen que incluye Python, puede configurar `$ProgressPreference` `SilentlyContinue` para lograr descargas más rápidas, como se muestra en el siguiente ejemplo.

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
>Nano Server actualmente no es compatible con WebClient.

### <a name="powershell-scripts"></a>Secuencias de comandos de PowerShell

En algunos casos, puede ser útil copiar un script en los contenedores que usa durante el proceso de creación de la imagen y, a continuación, ejecutar la secuencia de comandos desde el contenedor.

>[!NOTE]
>Esto limitará el almacenamiento en caché de la capa de imágenes y disminuirá la legibilidad de Dockerfile.

En este ejemplo se copia un script de la máquina de compilación en el contenedor mediante la instrucción `ADD`. Este script después se ejecuta con la instrucción RUN.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Compilación de Docker

Una vez que se ha creado una Dockerfile y se ha guardado en el `docker build` disco, puede ejecutar para crear la nueva imagen. El comando `docker build` toma varios parámetros opcionales y una ruta de acceso al archivo Dockerfile. Para obtener la documentación completa de la compilación de Dock, incluida una lista de todas las opciones de compilación, vea la [referencia de compilación](https://docs.docker.com/engine/reference/commandline/build/#build).

El formato del `docker build` comando es similar a este:

```dockerfile
docker build [OPTIONS] PATH
```

Por ejemplo, el siguiente comando creará una imagen denominada "IIS".

```dockerfile
docker build -t iis .
```

Cuando se haya iniciado el proceso de compilación, la salida indicará el estado y devolverá los errores que se produzcan.

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

El resultado es una nueva imagen del contenedor, que en este ejemplo se denomina "IIS".

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>Más lecturas y referencias

- [Optimizar Dockerfiles y la compilación de Dock para Windows](optimize-windows-dockerfile.md)
- [Referencia de Dockerfile](https://docs.docker.com/engine/reference/builder/)
