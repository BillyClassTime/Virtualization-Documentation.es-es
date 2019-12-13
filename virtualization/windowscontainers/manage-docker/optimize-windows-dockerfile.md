---
title: Optimizar Dockerfiles de Windows
description: Optimizar Dockerfiles para contenedores de Windows.
keywords: docker, contenedores
author: cwilhit
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
ms.openlocfilehash: ae633c7ba5d9672335addcc582988fc47c13ed79
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910155"
---
# <a name="optimize-windows-dockerfiles"></a>Optimizar Dockerfiles de Windows

Hay muchas maneras de optimizar el proceso de compilación de Docker y las imágenes de Docker resultantes. En este artículo se explica cómo funciona el proceso de compilación de Docker y cómo crear imágenes óptimamente para contenedores de Windows.

## <a name="image-layers-in-docker-build"></a>Capas de imagen en la compilación de Docker

Antes de poder optimizar la compilación de Docker, debe saber cómo funciona la compilación de Docker. Durante el proceso de compilación de Docker se usa un Dockerfile y cada instrucción accionable se ejecuta, de una en una, en su propio contenedor temporal. El resultado es una nueva capa de imagen para cada instrucción accionable.

Por ejemplo, en el siguiente ejemplo de Dockerfile se usa la imagen de sistema operativo base `mcr.microsoft.com/windows/servercore:ltsc2019`, se instala IIS y, a continuación, se crea un sitio web sencillo.

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Podría esperar que este Dockerfile genere una imagen con dos capas, una para la imagen del sistema operativo del contenedor y otra que incluye IIS y el sitio Web. Sin embargo, la imagen real tiene muchas capas y cada capa depende de la anterior.

Para que esto sea más claro, vamos a ejecutar el comando `docker history` en la imagen que se ha realizado en el ejemplo Dockerfile.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

La salida muestra que esta imagen tiene cuatro capas: la capa base y tres capas adicionales que se asignan a cada instrucción del Dockerfile. La capa inferior (`6801d964fda5` en este ejemplo) representa la imagen base del sistema operativo. Una capa arriba es la instalación de IIS. La siguiente capa incluye el nuevo sitio web y así sucesivamente.

Dockerfiles se puede escribir para minimizar las capas de imagen, optimizar el rendimiento de la compilación y optimizar la accesibilidad a través de la legibilidad. En última instancia, hay muchas formas de realizar la misma tarea de compilación de la imagen. Entender cómo el formato de Dockerfile afecta al tiempo de compilación y la imagen que crea mejora la experiencia de automatización.

## <a name="optimize-image-size"></a>Optimizar el tamaño de la imagen

En función de los requisitos de espacio, el tamaño de la imagen puede ser un factor importante a la hora de crear imágenes de contenedor de Docker. Las imágenes de contenedor se mueven entre registros y host, se exportan e importan y, en última instancia, usan espacio. En esta sección se explica cómo minimizar el tamaño de la imagen durante el proceso de compilación de Docker para contenedores de Windows.

Para obtener información adicional sobre los procedimientos recomendados de Dockerfile, consulte [procedimientos recomendados para escribir Dockerfiles en Docker.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Acciones relacionadas con grupos

Dado que cada `RUN` instrucción crea una nueva capa en la imagen de contenedor, la agrupación de acciones en una `RUN` instrucción puede reducir el número de capas en un Dockerfile. Aunque la minimización de capas puede no afectar mucho al tamaño de la imagen, la agrupación de acciones relacionadas sí, lo que se verá en los ejemplos siguientes.

En esta sección, se compararán dos Dockerfiles de ejemplo que hacen lo mismo. Sin embargo, un Dockerfile tiene una instrucción por acción, mientras que la otra tenía sus acciones relacionadas agrupadas.

En el siguiente ejemplo de Dockerfile desagrupado se descarga Python para Windows, se instala y se quita el archivo de instalación descargado una vez finalizada la instalación. En este Dockerfile, cada acción recibe su propia instrucción `RUN`.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

La imagen resultante se compone de tres capas adicionales, una para cada instrucción `RUN`.

```dockerfile
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

El segundo ejemplo es un Dockerfile que realiza la misma operación exacta. Sin embargo, todas las acciones relacionadas se han agrupado en una sola instrucción `RUN`. Cada paso de la instrucción `RUN` está en una nueva línea de Dockerfile, mientras que el carácter '\\' se usa para el ajuste de línea.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

La imagen resultante tiene solo una capa adicional para la instrucción `RUN`.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Quitar archivos sobrantes

Si hay un archivo en el Dockerfile, como un instalador, que no necesita después de que se haya usado, puede quitarlo para reducir el tamaño de la imagen. Esto debe hacerse en el mismo paso en el que se copia el archivo en la capa de imagen. Al hacerlo, se impide que el archivo se conserve en una capa de imagen de nivel inferior.

En el siguiente ejemplo de Dockerfile, el paquete de Python se descarga, ejecuta y se quita. Todo se realiza en una operación `RUN` y da lugar a una única capa de imagen.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>Optimizar la velocidad de compilación

### <a name="multiple-lines"></a>Varias líneas

Puede dividir las operaciones en varias instrucciones individuales para optimizar la velocidad de compilación de Docker. Varias operaciones `RUN` aumentan la eficacia del almacenamiento en caché porque se crean capas individuales para cada instrucción `RUN`. Si ya se ha ejecutado una instrucción idéntica en una operación de compilación de Docker diferente, se reutiliza esta operación almacenada en caché (capa de imagen), lo que da lugar a una disminución del tiempo de ejecución de compilación de Docker.

En el ejemplo siguiente, se descargan los paquetes de redistribución de Apache y Visual Studio, se instalan y, después, se quitan los archivos que ya no son necesarios. Todo esto se realiza con una sola instrucción `RUN`. Si se actualiza alguna de estas acciones, se volverán a ejecutar todas las acciones.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \

  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \

  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force; \
  Remove-Item c:\php.zip
```

La imagen resultante tiene dos capas, una para la imagen base del sistema operativo y otra que contiene todas las operaciones de la instrucción `RUN` única.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

En comparación, estas acciones se dividen en tres instrucciones `RUN`. En este caso, cada instrucción `RUN` se almacena en caché en una capa de imagen de contenedor y solo se deben volver a ejecutar las que han cambiado en las compilaciones posteriores de Dockerfile.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
    Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
    Remove-Item c:\apache.zip -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
    start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist.exe -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
    Remove-Item c:\php.zip -Force
```

La imagen resultante se compone de cuatro capas; una capa para la imagen base del sistema operativo y cada una de las tres instrucciones `RUN`. Dado que cada `RUN` instrucción se ejecuta en su propia capa, las ejecuciones posteriores de este Dockerfile o un conjunto idéntico de instrucciones en un Dockerfile diferente utilizarán capas de imágenes almacenadas en caché, lo que reduce el tiempo de compilación.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

La forma de ordenar las instrucciones es importante cuando se trabaja con memorias caché de imágenes, como se verá en la sección siguiente.

### <a name="ordering-instructions"></a>Instrucciones de ordenación

Un Dockerfile se procesa de arriba a abajo y cada instrucción se compara con las capas almacenadas en caché. Cuando se encuentra una instrucción sin una capa en caché, esta instrucción y todas las siguientes se procesan en nuevas capas de la imagen del contenedor. Por este motivo, es importante el orden en que se colocan las instrucciones. Coloque las instrucciones que se van a mantener constantes hacia la parte superior del Dockerfile. Coloque las que pueden cambiar hacia la parte inferior del Dockerfile. Al hacerlo, se reduce la probabilidad de que se niegue la caché existente.

En los siguientes ejemplos se muestra cómo la ordenación de instrucciones Dockerfile puede afectar a la eficacia del almacenamiento en caché. Este Dockerfile de ejemplo sencillo tiene cuatro carpetas numeradas.  

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

La imagen resultante tiene cinco capas, una para la imagen base del sistema operativo y cada una de las instrucciones `RUN`.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

Ahora se ha modificado ligeramente el siguiente Dockerfile, con la tercera `RUN` instrucción cambiada a un archivo nuevo. Cuando se ejecuta la compilación de Docker en este Dockerfile, las tres primeras instrucciones, que son idénticas a las del último ejemplo, usan las capas de la imagen almacenadas en caché. Sin embargo, dado que la instrucción de `RUN` modificada no se almacena en caché, se crea una nueva capa para la instrucción modificada y todas las instrucciones posteriores.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Cuando se comparan los identificadores de imagen de la nueva imagen en el primer ejemplo de esta sección, observará que las tres primeras capas de la parte inferior a la superior están compartidas, pero la cuarta y la quinta son únicas.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>Optimización cosmética

### <a name="instruction-case"></a>Caso de instrucción

Las instrucciones de Dockerfile no distinguen mayúsculas de minúsculas, pero la Convención es usar mayúsculas. Esto mejora la legibilidad al diferenciar entre la llamada de instrucción y la operación de instrucción. En los dos ejemplos siguientes se compara un Dockerfile en mayúsculas y en mayúsculas.

A continuación se encuentra un Dockerfile inversado:

```dockerfile
# Sample Dockerfile

from mcr.microsoft.com/windows/servercore:ltsc2019
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

Lo siguiente es el mismo Dockerfile con el uso de mayúsculas:

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Ajuste de línea

Las operaciones largas y complejas se pueden dividir en varias líneas mediante la barra diagonal inversa `\` carácter. El Dockerfile siguiente instala el paquete redistribuible de Visual Studio, quita los archivos del instalador y luego crea un archivo de configuración. Estas tres operaciones se especifican en una sola línea.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

El comando se puede dividir con barras diagonales inversas para que cada operación de una `RUN` instrucción se especifique en su propia línea.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Lecturas y referencias adicionales

[Dockerfile en Windows](manage-windows-dockerfile.md)

[Prácticas recomendadas para escribir Dockerfiles en Docker.com](https://docs.docker.com/engine/reference/builder/)
