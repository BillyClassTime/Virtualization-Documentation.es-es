---
title: Optimizar Dockerfiles de Windows
description: Optimizar Dockerfiles para contenedores de Windows.
keywords: docker, contenedores
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
translationtype: Human Translation
ms.sourcegitcommit: 7ebd83d5d3a098fc8760f5dfba7e350c3f167232
ms.openlocfilehash: 19a363aa013b51e0c80d56572de77e94f27e546f

---
# Optimizar Dockerfiles de Windows

**Esto es contenido preliminar y está sujeto a cambios.** 

Pueden usarse varios métodos para optimizar el proceso de compilación de Docker y las imágenes de Docker resultantes. En este documento se explica cómo funciona el proceso de compilación de Docker y se muestran varias tácticas que se pueden usar para una creación de imagen óptima con contenedores de Windows.

## Compilación de Docker

### Capas de imagen

Antes de examinar la optimización de las compilaciones de Docker, es importante comprender cómo funcionan las compilaciones de Docker. Durante el proceso de compilación de Docker se usa un Dockerfile y cada instrucción accionable se ejecuta, de una en una, en su propio contenedor temporal. El resultado es una nueva capa de imagen para cada instrucción accionable. 

Observe el Dockerfile siguiente. En este ejemplo se usa la imagen base del sistema operativo `windowsservercore`, IIS está instalado y luego se crea un sitio web sencillo.

```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

A partir de este Dockerfile, se podría esperar que la imagen resultante estuviera compuesta por dos capas, una para la imagen del sistema operativo del contenedor y una segunda que incluyera IIS y el sitio web, pero no es así. La nueva imagen se compone de varias capas, cada una de ellas dependiente de la anterior. Para visualizar esto, se puede ejecutar el comando `docker history` en la nueva imagen. Al hacerlo, se muestra que la imagen se compone de cuatro capas, la base y luego tres capas adicionales, una para cada instrucción del Dockerfile.

```none
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

Cada una de estas capas se puede asignar a una instrucción del Dockerfile. La capa inferior (`6801d964fda5` en este ejemplo) representa la imagen base del sistema operativo. Una capa más arriba se puede ver la instalación de IIS. La siguiente capa incluye el nuevo sitio web y así sucesivamente.

Se pueden escribir Dockerfiles para minimizar las capas de imagen, optimizar el rendimiento de la compilación y también optimizar aspectos estéticos como la legibilidad. En última instancia, hay muchas formas de realizar la misma tarea de compilación de la imagen. La comprensión de cómo afecta el formato de un Dockerfile al tiempo de compilación y a la imagen resultante mejora la experiencia de automatización. 

## Optimizar el tamaño de la imagen

Al compilar imágenes de contenedor de Docker, el tamaño de la imagen puede ser un factor importante. Las imágenes de contenedor se mueven entre registros y host, se exportan e importan y, en última instancia, usan espacio. Se pueden usar varias tácticas durante el proceso de compilación de Docker para minimizar el tamaño de la imagen. En esta sección se detallan algunas de estas tácticas específicas de los contenedores de Windows. 

Para obtener información adicional sobre los procedimientos recomendados de Dockerfile, consulte [Best practices for writing Dockerfiles (Procedimientos recomendados para escribir Dockerfiles)]( https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/) en Docker.com.

### Acciones relacionadas con grupos

Dado que cada instrucción `RUN` crea una nueva capa en la imagen del contenedor, la agrupación de acciones en una instrucción `RUN` puede reducir el número de capas. Aunque la minimización de capas puede no afectar mucho al tamaño de la imagen, la agrupación de acciones relacionadas sí, lo que se verá en los ejemplos siguientes.

Los dos ejemplos siguientes muestran la misma operación, que produce imágenes de contenedor de capacidad idéntica; sin embargo, los dos Dockerfiles se construyen de forma diferente. También se comparan las imágenes resultantes.  

En este primer ejemplo se descarga Python para Windows, lo instala y limpia quitando el archivo de instalación descargado. Cada una de estas acciones se ejecuta en su propia instrucción `RUN`.

```none
FROM windowsservercore

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

La imagen resultante se compone de tres capas adicionales, una para cada instrucción `RUN`.

```none
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

Para comparar, aquí está la misma operación, aunque todos los pasos se ejecutan con la misma instrucción `RUN`. Observe que cada paso de la instrucción `RUN` está en una nueva línea del Dockerfile y se usa el carácter '\' para el ajuste de línea. 

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

Esta imagen resultante se compone de una capa adicional para la instrucción `RUN`.

```none
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB                
```

### Quitar archivos sobrantes

Si un archivo, como un instalador, no es necesario después de haberse usado, quítelo para reducir el tamaño de la imagen. Esto debe hacerse en el mismo paso en el que se copia el archivo en la capa de imagen. Esto evita que el archivo se conserve en una capa de la imagen de nivel inferior.

En este ejemplo se descarga el paquete de Python, se ejecuta y luego se quita el archivo ejecutable. Todo se realiza en una operación `RUN` y da lugar a una única capa de imagen.

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## Optimizar la velocidad de compilación

### Varias líneas

Al optimizar la velocidad de compilación de Docker, puede ser conveniente separar las operaciones en varias instrucciones individuales. Con varias operaciones `RUN` aumenta la eficacia del almacenamiento en caché. Dado que se crean capas individuales para cada instrucción `RUN`, si ya se ha ejecutado un paso idéntico en otra operación de compilación de Docker, se vuelve a usar esta operación almacenada en caché (capa de imagen). El resultado es que el tiempo de ejecución de la compilación de Docker se reduce.

En el ejemplo siguiente se descargan los paquetes de redistribución de Apache y Visual Studio, se instalan y luego se limpian los archivos no necesarios. Todo se hace con una instrucción `RUN`. Si alguna de estas acciones se actualiza, se vuelven a ejecutar todas.

```none
FROM windowsservercore

RUN powershell -Command \
    
  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    
  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredistexe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force
```

La imagen resultante consta de dos capas, una para la imagen base del sistema operativo y la segunda que contiene todas las operaciones a partir de la instrucción única `RUN`.

```none
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

Para comparar, aquí están las mismas acciones divididas en tres instrucciones `RUN`. En este caso, cada instrucción `RUN` se almacena en caché en una capa de imagen de contenedor y solo es necesario volver a ejecutar aquellas que han cambiado en posteriores compilaciones del Dockerfile.

```none
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

La imagen resultante se compone de cuatro capas, una para la imagen base del sistema operativo y luego una para cada instrucción `RUN`. Dado que cada instrucción `RUN` se ha ejecutado en su propia capa, todas las ejecuciones posteriores de este Dockerfile o de un conjunto idéntico de instrucciones de un Dockerfile diferente usarán la capa de imagen almacenada en caché, lo que disminuirá el tiempo de compilación. La clasificación de las instrucciones es importante al trabajar con la caché de imágenes. Para obtener más detalles, consulte la sección siguiente de este documento.

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

### Clasificación de instrucciones

Un Dockerfile se procesa de arriba a abajo y cada instrucción se compara con las capas almacenadas en caché. Cuando se encuentra una instrucción sin una capa en caché, esta instrucción y todas las siguientes se procesan en nuevas capas de la imagen del contenedor. Por este motivo, es importante el orden en que se colocan las instrucciones. Coloque las instrucciones que se van a mantener constantes hacia la parte superior del Dockerfile. Coloque las que pueden cambiar hacia la parte inferior del Dockerfile. Al hacerlo, se reduce la probabilidad de que se niegue la caché existente.

La intención de este ejemplo es demostrar cómo puede afectar la clasificación de las instrucciones del Dockerfile a la eficacia del almacenamiento en caché. En este sencillo Dockerfile, se crean cuatro carpetas numeradas.  

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```
La imagen resultante tiene cinco capas, una para la imagen base del sistema operativo y una para cada instrucción `RUN`.

```none
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B    
```

Ahora el Dockerfile se ha modificado ligeramente. Observe que la tercera instrucción `RUN` ha cambiado. Cuando se ejecuta la compilación de Docker en este Dockerfile, las tres primeras instrucciones, que son idénticas a las del último ejemplo, usan las capas de la imagen almacenadas en caché. Pero, dado que la instrucción `RUN` modificada no se ha almacenado en caché, se crea una nueva capa para esta y para todas las instrucciones posteriores.

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Si compara los identificadores de imagen de la nueva imagen con los del último ejemplo, verá que las tres primeras capas (de abajo a arriba) son compartidas, pero que la cuarta y la quinta son únicas.

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## Optimización estética

### Mayúsculas y minúsculas de las instrucciones

La instrucciones de Dockerfile no distinguen mayúsculas de minúsculas, aunque la convención es usar mayúsculas. Esto mejora la legibilidad al diferenciar entre la llamada y la operación de la instrucción. Los dos ejemplos siguientes muestran este concepto. 

Minúsculas:
```none
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```
Mayúsculas: 
```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### Ajuste de línea

Las operaciones largas y complejas pueden dividirse en varias líneas con el carácter de barra diagonal inversa `\`. El Dockerfile siguiente instala el paquete redistribuible de Visual Studio, quita los archivos del instalador y luego crea un archivo de configuración. Estas tres operaciones se especifican en una sola línea.

```none
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```
El comando se puede volver a escribir para que cada operación a partir de la instrucción `RUN` se especifique en su propia línea. 

```none
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## Lecturas y referencias adicionales

[Dockerfile en Windows] (./manage_windows_dockerfile.md)

[Best practices for writing Dockerfiles (Procedimientos recomendados para escribir Dockerfiles) en Docker.com](https://docs.docker.com/engine/reference/builder/)



<!--HONumber=Aug16_HO5-->


