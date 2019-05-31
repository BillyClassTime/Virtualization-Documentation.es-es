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
ms.openlocfilehash: 871884c04b4165da4a5ab8af65bcda252672efbc
ms.sourcegitcommit: bea2c90f31a38fc7fda356619f0dd812f79d008f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/31/2019
ms.locfileid: "9685282"
---
# <a name="optimize-windows-dockerfiles"></a>Optimizar Dockerfiles de Windows

Hay muchas maneras de optimizar el proceso de compilación del acoplador y las imágenes del Dock resultante. En este artículo se explica cómo funciona el proceso de compilación del acoplamiento y cómo crear imágenes de contenedores de Windows de manera óptima.

## <a name="image-layers-in-docker-build"></a>Capas de imagen en la compilación del acoplador

Antes de poder optimizar la compilación de Dock, tendrá que saber cómo funciona la compilación de Docker. Durante el proceso de compilación de Docker se usa un Dockerfile y cada instrucción accionable se ejecuta, de una en una, en su propio contenedor temporal. El resultado es una nueva capa de imagen para cada instrucción accionable.

Por ejemplo, el siguiente ejemplo de Dockerfile usa `windowsservercore` la imagen de sistema operativo base, instala IIS y, a continuación, crea un sitio web simple.

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Es posible que este Dockerfile le cree una imagen con dos capas, una para la imagen del sistema operativo del contenedor y otra que incluye IIS y el sitio Web. Sin embargo, la imagen real tiene muchas capas, y cada capa depende de la anterior.

Para que esto sea más claro, ejecutamos el `docker history` comando en la imagen que realizaste nuestro ejemplo de Dockerfile.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

La salida nos muestra que esta imagen tiene cuatro capas: la capa base y tres capas adicionales asignadas a cada instrucción de la Dockerfile. La capa inferior (`6801d964fda5` en este ejemplo) representa la imagen base del sistema operativo. Un nivel superior es la instalación de IIS. La siguiente capa incluye el nuevo sitio web y así sucesivamente.

Dockerfiles se puede escribir para minimizar las capas de la imagen, optimizar el rendimiento de la compilación y optimizar la accesibilidad a través de la legibilidad. En última instancia, hay muchas formas de realizar la misma tarea de compilación de la imagen. Comprender cómo el formato de Dockerfile afecta al tiempo de compilación y la imagen que crea mejora la experiencia de automatización.

## <a name="optimize-image-size"></a>Optimizar tamaño de imagen

En función de los requisitos de espacio, el tamaño de la imagen puede ser un factor importante al crear imágenes de contenedor de Dock. Las imágenes de contenedor se mueven entre registros y host, se exportan e importan y, en última instancia, usan espacio. En esta sección se explica cómo minimizar el tamaño de la imagen durante el proceso de compilación del acoplador para contenedores de Windows.

Para obtener más información sobre los procedimientos recomendados de Dockerfile, vea [procedimientos recomendados para escribir Dockerfiles en Docker.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Acciones relacionadas con grupos

Como cada `RUN` instrucción crea una nueva capa en la imagen del contenedor, las acciones de agrupación en una `RUN` instrucción pueden reducir el número de capas en un Dockerfile. Aunque la minimización de capas puede no afectar mucho al tamaño de la imagen, la agrupación de acciones relacionadas sí, lo que se verá en los ejemplos siguientes.

En esta sección, compararemos dos ejemplos de Dockerfiles que hacen las mismas cosas. Sin embargo, una Dockerfile tiene una instrucción por acción, mientras que la otra tiene las acciones relacionadas agrupadas.

El siguiente ejemplo Dockerfile desagrupado descarga Python para Windows, lo instala y quita el archivo de instalación descargado una vez que se haya instalado la instalación. En este Dockerfile, cada acción tiene su propia `RUN` instrucción.

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

El segundo ejemplo es un Dockerfile que realiza exactamente la misma operación. Sin embargo, todas las acciones relacionadas se han agrupado en `RUN` una única instrucción. Cada paso de la `RUN` instrucción está en una nueva línea de la Dockerfile, mientras que el carácter ' \ \ ' se usa para el ajuste de línea.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

La imagen resultante solo tiene una capa adicional para la `RUN` instrucción.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Quitar archivos sobrantes

Si hay un archivo en su Dockerfile, como un instalador, que no necesita después de haberlo usado, puede quitarlo para reducir el tamaño de la imagen. Esto debe hacerse en el mismo paso en el que se copia el archivo en la capa de imagen. De este modo, se evitará que el archivo se mantenga en una capa de imagen de nivel inferior.

En el siguiente ejemplo Dockerfile, el paquete python se descarga, se ejecuta y después se quita. Todo se realiza en una operación `RUN` y da lugar a una única capa de imagen.

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

Puede dividir las operaciones en varias instrucciones individuales para optimizar la velocidad de compilación del acoplador. Varias `RUN` operaciones aumentan la eficacia del almacenamiento en caché porque se crean `RUN` capas individuales para cada instrucción. Si ya se ha ejecutado una instrucción idéntica en una operación de compilación de acoplamiento diferente, esta operación almacenada en caché (capa de imagen) se vuelve a utilizar, lo que reduce el tiempo de ejecución de compilación del Dock.

En el siguiente ejemplo, tanto Apache como Visual Studio redistribuir paquetes se descargan, instalan y, a continuación, se limpian quitando los archivos que ya no se necesitan. Esto se hace todo con una única `RUN` instrucción. Si se actualiza cualquiera de estas acciones, todas las acciones se volverán a ejecutar.

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

La imagen resultante tiene dos capas, una para la imagen del sistema operativo base y otra que contiene todas las operaciones de `RUN` la instrucción única.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

En comparación, estas acciones se dividen en tres `RUN` instrucciones. En este caso, cada `RUN` instrucción se almacena en caché en una capa de imagen de contenedor y solo se debe volver a ejecutar las instrucciones que hayan cambiado en las compilaciones Dockerfile subsiguientes.

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

La imagen resultante consta de cuatro capas; una capa para la imagen del sistema operativo base y cada una `RUN` de las tres instrucciones. Como cada `RUN` instrucción se ejecutaba en su propia capa, cualquier ejecución posterior de esta Dockerfile o un conjunto idéntico de instrucciones en un Dockerfile diferente usarán capas de imagen almacenadas en caché, lo que reduce el tiempo de compilación.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

El orden de las instrucciones es importante al trabajar con cachés de imágenes, como verá en la siguiente sección.

### <a name="ordering-instructions"></a>Instrucciones de ordenación

Un Dockerfile se procesa de arriba a abajo y cada instrucción se compara con las capas almacenadas en caché. Cuando se encuentra una instrucción sin una capa en caché, esta instrucción y todas las siguientes se procesan en nuevas capas de la imagen del contenedor. Por este motivo, es importante el orden en que se colocan las instrucciones. Coloque las instrucciones que se van a mantener constantes hacia la parte superior del Dockerfile. Coloque las que pueden cambiar hacia la parte inferior del Dockerfile. Al hacerlo, se reduce la probabilidad de que se niegue la caché existente.

En los siguientes ejemplos se muestra cómo la ordenación de instrucciones Dockerfile puede afectar a la eficacia del almacenamiento en caché. Este sencillo Dockerfile de ejemplo tiene cuatro carpetas numeradas.  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

La imagen resultante tiene cinco capas, una para la imagen del sistema operativo base y cada `RUN` una de las instrucciones.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

El siguiente Dockerfile se ha modificado ligeramente, con la tercera `RUN` instrucción cambiada a un archivo nuevo. Cuando se ejecuta la compilación de Docker en este Dockerfile, las tres primeras instrucciones, que son idénticas a las del último ejemplo, usan las capas de la imagen almacenadas en caché. Sin embargo, dado que `RUN` la instrucción modificada no se almacena en caché, se crea una nueva capa para la instrucción modificada y todas las instrucciones posteriores.

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Cuando compare los identificadores de imagen de la nueva imagen con los del primer ejemplo de esta sección, observará que las tres primeras capas de la parte inferior a la superior están compartidas, pero la cuarta y la quinta son únicas.

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

### <a name="instruction-case"></a>Caso de instrucciones

Las instrucciones de Dockerfile no distinguen entre mayúsculas y minúsculas, pero la Convención es de uso en mayúsculas. Esto mejora la legibilidad al diferenciar entre la llamada de instrucción y la operación de instrucción. En los dos ejemplos siguientes se compara un Dockerfile con mayúsculas y minúsculas.

A continuación se reDockerfile:

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

A continuación se encuentra el mismo Dockerfile con mayúsculas y minúsculas:

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Ajuste de línea

Las operaciones largas y complejas pueden dividirse en varias líneas con el carácter `\` de barra diagonal inversa. El Dockerfile siguiente instala el paquete redistribuible de Visual Studio, quita los archivos del instalador y luego crea un archivo de configuración. Estas tres operaciones se especifican en una sola línea.

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

El comando se puede dividir con barras diagonales inversas para que todas las operaciones de `RUN` la instrucción se especifiquen en su propia línea.

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Más lecturas y referencias

[Dockerfile en Windows](manage-windows-dockerfile.md)

[Procedimientos recomendados para escribir Dockerfiles en Docker.com](https://docs.docker.com/engine/reference/builder/)
