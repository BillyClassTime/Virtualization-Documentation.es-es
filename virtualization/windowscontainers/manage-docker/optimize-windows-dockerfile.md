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
ms.openlocfilehash: d897560061fae23fda6f88ebdad6dd804da9a8f1
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610345"
---
# <a name="optimize-windows-dockerfiles"></a>Optimizar Dockerfiles de Windows

Existen muchas formas para optimizar el proceso de compilación de Docker y las imágenes de Docker resultantes. En este artículo se explica cómo funciona el proceso de compilación de Docker y cómo crear imágenes de contenedores de Windows de forma óptima.

## <a name="image-layers-in-docker-build"></a>Capas de imagen en la compilación de Docker

Antes de que se puede optimizar la compilación de Docker, tendrás que conocer cómo funcionan las compilaciones de Docker. Durante el proceso de compilación de Docker se usa un Dockerfile y cada instrucción accionable se ejecuta, de una en una, en su propio contenedor temporal. El resultado es una nueva capa de imagen para cada instrucción accionable.

Por ejemplo, el siguiente ejemplo usa Dockerfile el `windowsservercore` imagen de sistema operativo base, instala IIS y, a continuación, crea un sitio Web sencillo.

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Cabría esperar que este Dockerfile producirá una imagen con dos capas, una para la imagen de sistema operativo de contenedor y una segunda que incluyera IIS y el sitio Web. Sin embargo, la imagen real tiene muchas capas y cada capa depende anterior.

Para hacer esto más claro, vamos a ejecutar el `docker history` comando frente a la imagen de nuestro ejemplo Dockerfile realizado.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

La salida nos muestra que esta imagen tiene cuatro niveles: la capa de base y tres capas adicionales que se asignan a cada instrucción del Dockerfile. La capa inferior (`6801d964fda5` en este ejemplo) representa la imagen base del sistema operativo. Una capa de es la instalación de IIS. La siguiente capa incluye el nuevo sitio web y así sucesivamente.

Pueden escribir Dockerfiles para minimizar las capas de imagen, optimizar el rendimiento de la compilación y optimizar la accesibilidad a través de la legibilidad. En última instancia, hay muchas formas de realizar la misma tarea de compilación de la imagen. Descripción de cómo afecta formato del Dockerfile al tiempo de compilación y la imagen crea mejora la experiencia de automatización.

## <a name="optimize-image-size"></a>Optimizar el tamaño de la imagen

Dependiendo de los requisitos de espacio, tamaño de la imagen puede ser un factor importante al crear imágenes de contenedor de Docker. Las imágenes de contenedor se mueven entre registros y host, se exportan e importan y, en última instancia, usan espacio. En esta sección te indicará cómo minimizar el tamaño de la imagen durante el proceso de compilación de Docker para contenedores de Windows.

Para obtener más información sobre los procedimientos recomendados de Dockerfile, consulte [los procedimientos recomendados para escribir Dockerfiles en Docker.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Acciones relacionadas con grupos

Dado que cada `RUN` instrucción crea una nueva capa en la imagen de contenedor, agrupación de acciones en una `RUN` instrucción puede reducir el número de capas en un archivo Dockerfile. Aunque la minimización de capas puede no afectar mucho al tamaño de la imagen, la agrupación de acciones relacionadas sí, lo que se verá en los ejemplos siguientes.

En esta sección, te enviaremos compara el ejemplo de dos Dockerfiles que hacer lo mismo. Sin embargo, un Dockerfile tiene una instrucción por cada acción, mientras que el otro tenía sus acciones relacionadas que se agrupan.

El siguiente ejemplo no agrupado Dockerfile descarga Python para Windows, lo instala y quita el archivo de instalación que has descargado una vez que se realiza la instalación. En este Dockerfile, cada acción se asigna a su propio `RUN` instrucción.

```dockerfile
FROM windowsservercore

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

El segundo ejemplo es un archivo Dockerfile que realiza la misma operación exacta. Sin embargo, todos los relacionados se han agrupado acciones en una sola `RUN` instrucción. Cada paso de la `RUN` instrucción está en una nueva línea del Dockerfile y mientras el ' \\' carácter se usa para el ajuste de línea.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

La imagen resultante tiene solo una capa adicional para la `RUN` instrucción.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Quitar archivos sobrantes

Si hay un archivo en el Dockerfile, como un instalador, que no es necesario después de se ha utilizado, puede quitarla para reducir el tamaño de la imagen. Esto debe hacerse en el mismo paso en el que se copia el archivo en la capa de imagen. Al hacerlo, impide que el archivo conserve en una capa de imagen de nivel inferior.

En el siguiente ejemplo Dockerfile, se descargado, ejecuta y quita el paquete de Python. Todo se realiza en una operación `RUN` y da lugar a una única capa de imagen.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>Optimizar la velocidad de compilación

### <a name="multiple-lines"></a>Varias líneas

Se pueden dividir operaciones en varias instrucciones individuales para optimizar la velocidad de compilación de Docker. Varios `RUN` operaciones aumentan la eficacia del almacenamiento en caché porque se crean capas individuales para cada `RUN` instrucción. Si ya se ha ejecutado una instrucción idéntica en otra operación de compilación de Docker, se reutiliza esta operación almacenada en caché (capa de imagen), lo que reduce en tiempo de ejecución de compilación de Docker.

En el siguiente ejemplo, Apache y los paquetes de redistribución de Visual Studio se descargan, instalados y limpian mediante la eliminación de archivos que ya no son necesarios. Esto se realiza con un solo `RUN` instrucción. Si alguna de estas acciones se actualiza, se vuelve a ejecutar todas las acciones.

```dockerfile
FROM windowsservercore

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

La imagen resultante tiene dos capas, uno para la imagen de sistema operativo base y otro que contiene todas las operaciones de único `RUN` instrucción.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

Por el contrario, estos son la misma división de acciones en tres `RUN` instrucciones. En este caso, cada `RUN` instrucción se almacenan en caché en una capa de imagen de contenedor y solo aquellas que necesitan modificadas volver a ejecutarse en Dockerfile posterior compilaciones.

```dockerfile
FROM windowsservercore

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

La imagen resultante se compone de cuatro capas; una capa de la imagen base del sistema operativo y cada uno de los tres `RUN` instrucciones. Dado que cada `RUN` instrucción se ha ejecutado en su propia capa, todas las ejecuciones posteriores de este Dockerfile o un conjunto idéntico de instrucciones de un Dockerfile diferente usarán capas de imagen almacenada en caché, lo que reduce el tiempo de compilación.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

¿Cómo solicitar las instrucciones es importante al trabajar con las cachés de imagen, como se verá en la siguiente sección.

### <a name="ordering-instructions"></a>Clasificación de instrucciones

Un Dockerfile se procesa de arriba a abajo y cada instrucción se compara con las capas almacenadas en caché. Cuando se encuentra una instrucción sin una capa en caché, esta instrucción y todas las siguientes se procesan en nuevas capas de la imagen del contenedor. Por este motivo, es importante el orden en que se colocan las instrucciones. Coloque las instrucciones que se van a mantener constantes hacia la parte superior del Dockerfile. Coloque las que pueden cambiar hacia la parte inferior del Dockerfile. Al hacerlo, se reduce la probabilidad de que se niegue la caché existente.

Los siguientes ejemplos muestran cómo Dockerfile puede afectar eficacia del almacenamiento en caché. En este ejemplo sencillo Dockerfile tiene cuatro carpetas numeradas.  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

La imagen resultante tiene cinco capas, una para la imagen de sistema operativo base y cada uno de los `RUN` instrucciones.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

Este Dockerfile siguiente ahora ha modificado ligeramente, con el tercer `RUN` instrucción cambia a un nuevo archivo. Cuando se ejecuta la compilación de Docker en este Dockerfile, las tres primeras instrucciones, que son idénticas a las del último ejemplo, usan las capas de la imagen almacenadas en caché. Sin embargo, dado que los cambios `RUN` instrucción no se almacenan en caché, se crea una nueva capa para la instrucción modificada y todas las instrucciones posteriores.

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Al comparar los identificadores de imagen de la nueva imagen para en el primer ejemplo de esta sección, verás que se comparten las tres primeras capas de abajo a arriba, pero la cuarta y la quinta son únicas.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>Optimización estética

### <a name="instruction-case"></a>Mayúsculas y minúsculas

Instrucciones de Dockerfile no distinguen entre mayúsculas y minúsculas, pero la convención es usar mayúsculas. Esto mejora la legibilidad al diferenciar entre la llamada y la operación de instrucción. Los dos ejemplos siguientes comparan un Dockerfile sin mayúsculas y escrita en mayúsculas.

El siguiente es un Dockerfile sin mayúsculas:

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

El siguiente es el mismo Dockerfile con letras mayúsculas:

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Ajuste de línea

Operaciones largas y complejas pueden dividirse en varias líneas mediante la barra diagonal inversa `\` caracteres. El Dockerfile siguiente instala el paquete redistribuible de Visual Studio, quita los archivos del instalador y luego crea un archivo de configuración. Estas tres operaciones se especifican en una sola línea.

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

El comando se puede dividir con barras diagonales inversas por lo tanto, que cada operación a partir del `RUN` instrucción se especifica en su propia línea.

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Otras lecturas y referencias

[Dockerfile en Windows](manage-windows-dockerfile.md)

[Procedimientos recomendados para escribir Dockerfiles en Docker.com](https://docs.docker.com/engine/reference/builder/)
