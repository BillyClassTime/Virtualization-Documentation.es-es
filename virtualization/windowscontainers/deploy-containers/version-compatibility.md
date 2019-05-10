---
title: Compatibilidad con versiones de contenedores de Windows
description: Cómo puede Windows ejecutar compilaciones y contenedores en varias versiones de Windows
keywords: metadatos, contenedores, versión
author: taylorb-microsoft
ms.openlocfilehash: 9e20a28f62ec4bb9456307acb7768e1123a57188
ms.sourcegitcommit: 03e9203e9769997d8be3f66dc7935a3e5c0a83e1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621633"
---
# <a name="windows-container-version-compatibility"></a>Compatibilidad con versiones de contenedor de Windows

Windows Server 2016 y actualización de aniversario de Windows 10 (versión 14393 en ambos) fueron los primeros lanzamientos de Windows que podían compilar y ejecutar contenedores de Windows Server. Los contenedores compilados con estas versiones se pueden ejecutar en las versiones más recientes, como Windows Server versión 1709, pero hay algunos aspectos que debes saber antes de empezar.

Dado que hemos mejorado las características de los contenedores de Windows, tuvimos que hacer algunos cambios que pueden afectar a la compatibilidad. Los contenedores antiguos también podrán ejecutarse en los hosts más recientes con [aislamiento de Hyper-V](../manage-containers/hyperv-container.md)y usarán la misma versión de kernel (antigua). Sin embargo, si quieres ejecutar un contenedor basado en una compilación más reciente de Windows, se puede ejecutar solo en la compilación más reciente del host.

|Versión del SO de contenedor|Versión del SO de host|Compatibilidad|
|---|---|---|
|WindowsServer2016<br>Compilaciones: 14393.* |WindowsServer2016<br>Compilaciones: 14393.* |Admite `process` o `hyperv` aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |WindowsServer, versión1709<br>Compilaciones: 16299.* |Solo es compatible con `hyperv` aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |Windows10FallCreatorsUpdate<br>Compilaciones: 16299.* |Solo es compatible con `hyperv` aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |Windows Server versión 1803<br>Compilaciones 17134.* |Solo es compatible con `hyperv` aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |Windows 10, versión 1803<br>Compilaciones 17134.* |Solo es compatible con `hyperv` aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |Windows Server 2019<br>Compilaciones 17763.* |Solo es compatible con `hyperv` aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |Windows 10, versión 1809<br>Compilaciones 17763.* |Solo es compatible con `hyperv` aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |WindowsServer2016<br>Compilaciones: 14393.* |No se admite|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows Server, versión 1709<br>Compilaciones: 16299.* |Admite `process` o `hyperv` aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows10FallCreatorsUpdate<br>Compilaciones: 16299.* |Solo es compatible con `hyperv` aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows Server, versión 1803<br>Compilaciones 17134.* |Solo es compatible con `hyperv` aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows 10, versión 1803<br>Compilaciones 17134.* |Solo es compatible con `hyperv` aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows Server 2019<br>Compilaciones 17763.* |Solo es compatible con `hyperv` aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows 10, versión 1809<br>Compilaciones 17763.* |Solo es compatible con `hyperv` aislamiento|
|Windows Server, versión 1803<br>Compilaciones 17134.* |WindowsServer2016<br>Compilaciones: 14393.* |No se admite|
|Windows Server, versión 1803<br>Compilaciones 17134.* |Windows Server, versión 1709<br>Compilaciones: 16299.* |No se admite|
|Windows Server, versión 1803<br>Compilaciones 17134.* |Windows10FallCreatorsUpdate<br>Compilaciones: 16299.* |No se admite|
|Windows Server, versión 1803<br>Compilaciones 17134.* |Windows Server, versión 1803<br>Compilaciones 17134.* |Admite `process` o `hyperv` aislamiento|
|Windows Server, versión 1803<br>Compilaciones 17134.* |Windows 10, versión 1803<br>Compilaciones 17134.* |Solo es compatible con `hyperv` aislamiento|
|Windows Server, versión 1803<br>Compilaciones 17134.* |Windows Server 2019<br>Compilaciones 17763.* |Solo es compatible con `hyperv` aislamiento|
|Windows Server, versión 1803<br>Compilaciones 17134.* |Windows 10, versión 1809<br>Compilaciones 17763.* |Solo es compatible con `hyperv` aislamiento|
|Windows Server 2019<br>Compilaciones 17763.* |WindowsServer2016<br>Compilaciones: 14393.* |No se admite|
|Windows Server 2019<br>Compilaciones 17763.* |Windows Server, versión 1709<br>Compilaciones: 16299.* |No se admite
|Windows Server 2019<br>Compilaciones 17763.* |Windows10FallCreatorsUpdate<br>Compilaciones: 16299.* |No se admite|
|Windows Server 2019<br>Compilaciones 17763.* |Windows Server, versión 1803<br>Compilaciones 17134.* |No se admite|
|Windows Server 2019<br>Compilaciones 17763.* |Windows 10, versión 1803<br>Compilaciones 17134.* |No se admite|
|Windows Server 2019<br>Compilaciones 17763.* |Windows Server 2019<br>Compilaciones 17763.* |Admite `process` o `hyperv` aislamiento|
|Windows Server 2019<br>Compilaciones 17763.* |Windows 10, versión 1809<br>Compilaciones 17763.* |Solo es compatible con `hyperv` aislamiento|

## <a name="matching-container-host-version-with-container-image-versions"></a>Versión coincidente de host de contenedor con las versiones de imagen de contenedor

### <a name="windows-server-containers"></a>Contenedores de Windows Server

Debido a los contenedores de Windows Server y la host subyacente comparten un solo kernel, imagen base del contenedor debe coincidir con del host. Si las versiones son diferentes, el contenedor puede iniciarse, pero no se garantiza una funcionalidad completa. El sistema operativo Windows tiene cuatro niveles de control de versiones: principal, secundario, compilación y revisión. Por ejemplo, 10.0.14393.103 versión tendría una versión principal de 10, una versión secundaria de 0, un número de compilación de 14393 y un número de revisión de 103. El número de compilación solo cambia cuando las nuevas versiones del sistema operativo se publiquen, por ejemplo, versión 1709, 1803, Update de Fall Creators y así sucesivamente. El número de revisión se actualiza cuando se aplican las actualizaciones de Windows.

#### <a name="build-number-new-release-of-windows"></a>Número de compilación (nueva versión de Windows)

Contenedores de Windows Server se bloquean el inicio cuando el número de compilación entre el host de contenedor y la imagen del contenedor es diferente. Por ejemplo, cuando el host de contenedor es versión 10.0.14393 (Windows Server 2016) y la imagen de contenedor es la versión 10.0.16299 (Windows Server versión 1709), no se inicia el contenedor.  

#### <a name="revision-number-patching"></a>Número de revisión (aplicación de revisiones)

Contenedores de Windows Server no se bloquea el inicio de los números de revisión de la host de contenedor y la imagen del contenedor es diferentes. Por ejemplo, si el host de contenedor es la versión, 10.0.14393.1914 (Windows Server 2016 con KB4051033) y la imagen del contenedor es la versión 10.0.14393.1944 (Windows Server 2016 con KB4053579), a continuación, la imagen inicie aunque su revisión números son diferentes.

Hosts basados en Windows Server 2016 o imágenes, revisión de la imagen de contenedor debe coincidir con el host para estar en una configuración compatible. Sin embargo, para hosts o imágenes que usan Windows Server versión 1709 y posterior, no se aplica esta regla y la imagen del contenedor y el host no necesita tener revisiones que coincidan. Te recomendamos que mantener los sistemas actualizados con las últimas revisiones y actualizaciones.

#### <a name="practical-application"></a>Aplicación práctica

Ejemplo 1: El host del contenedor ejecuta Windows Server 2016 con KB4041691. Cualquier contenedor de Windows Server implementado a este host debe basarse en las imágenes base del contenedor 10.0.14393.1770 de versión. Si se aplica KB4053579 a la host de contenedor, también debes actualizar las imágenes para asegurarse de que el contenedor del host es compatible con ellos.

Ejemplo 2: La host del contenedor ejecuta Windows Server versión 1709 con KB4043961. Cualquier contenedor de Windows Server implementado a este host debe basarse en una imagen base de contenedor de Windows Server versión 1709 (10.0.16299), pero no es necesario para que coincida con el KB del host. Si se aplica KB4054517 al host, todavía se admitirá las imágenes del contenedor, pero te recomendamos que actualices para solucionar posibles problemas de seguridad.

#### <a name="querying-version"></a>Consultar versiones

Método 1: Incluido en la versión 1709, el comando del símbolo del sistema y **ver** cmd ahora devolver los detalles de revisión.

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

Método 2: Consultar la siguiente clave del registro: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion

Por ejemplo:

```batch
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```

```batch
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

Para comprobar qué versión usa la imagen base, revise las etiquetas de Docker hub o la tabla de hash de la imagen proporcionada en la descripción de la imagen. La página [historial de actualizaciones de Windows 10](https://support.microsoft.com/help/12387/windows-10-update-history) muestra cuándo se lanzó cada compilación y revisión.

### <a name="hyper-v-isolation-for-containers"></a>Aislamiento de Hyper-V para contenedores

Puedes ejecutar contenedores de Windows con o sin aislamiento de Hyper-V. El aislamiento de Hyper-V crea un límite seguro alrededor del contenedor con una VM optimizada. A diferencia de los contenedores de Windows estándares que comparten el kernel entre los contenedores y el host, cada contenedor de Hyper-V aislado tiene su propia instancia del kernel de Windows. Esto significa que puede tener diferentes versiones del sistema operativo en la imagen y el host de contenedor (para obtener más información, consulta la siguiente matriz de compatibilidad).  

Para ejecutar un contenedor con aislamiento de Hyper-V, solo tienes que agregar la etiqueta `--isolation=hyperv` al comando de ejecución de docker.

## <a name="errors-from-mismatched-versions"></a>Errores de las versiones que no coinciden

Si intentas ejecutar una combinación no admitida, obtendrás el error siguiente:

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

Hay tres maneras puede resolver el error:

- Volver a compilar el contenedor en función de la versión correcta de `microsoft/nanoserver` o `microsoft/windowsservercore`
- Si el host es más recientes, ejecutar **docker run--aislamiento = hyperv …**
- Prueba a ejecutar el contenedor en otro host con la misma versión de Windows

## <a name="choose-which-container-os-version-to-use"></a>Elige la versión del sistema operativo de contenedor

>[!NOTE]
>A partir de 16 de abril de 2019, la etiqueta "más reciente" ya no se publica o se mantiene para las [imágenes de contenedor del sistema operativo base de Windows](https://hub.docker.com/_/microsoft-windows-base-os-images). Por favor, declara una etiqueta específica cuando extraer o haga referencia a imágenes desde estos repositorios.

Debe saber qué versión debe utilizar para el contenedor. Por ejemplo, si deseas Windows Server, versión 1809 como el contenedor del sistema operativo y quieres tener las últimas revisiones de él, debes usar la etiqueta `1809` al especificar qué versión de las imágenes de contenedor base del sistema operativo quieres, de este modo:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809
...
```

Sin embargo, si quieres una revisión específica de Windows Server, versión 1809, puedes especificar el número de KB en la etiqueta. Por ejemplo, para obtener un servidor Nano imagen de contenedor de base del sistema operativo de Windows Server, versión 1809 con el KB4493509 aplicado, se especificaría este modo:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809-KB4493509
...
```

También puedes especificar las revisiones exactas que necesitas con el esquema que hemos usado anteriormente, especificando la versión del sistema operativo en la etiqueta:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:10.0.17763.437
...
```

Las imágenes base de Server Core en función de Windows Server 2019 y Windows Server 2016 son que versiones de [Canal de mantenimiento a largo plazo (LTSC)](https://docs.microsoft.com/en-us/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc) . Si por ejemplo desea Windows Server 2019 como contenedor de la imagen de Server Core del sistema operativo y quieres tener las últimas revisiones de él, puedes especificar LTSC las versiones de este modo:

``` dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019
...
```

## <a name="matching-versions-using-docker-swarm"></a>Coincidencia de versiones mediante Docker Swarm

Enjambre de docker actualmente no tiene un modo integrado para que coincida con la versión de Windows que usa un contenedor en un host con la misma versión. Si actualizas el servicio para que use un contenedor más reciente, se ejecutará correctamente.

Si necesitas ejecutar varias versiones de Windows durante un largo período de tiempo, hay dos enfoques que puedes realizar: o bien configurar los hosts de Windows para siempre usa el aislamiento de Hyper-V o usar restricciones de etiqueta.

### <a name="finding-a-service-that-wont-start"></a>Encontrar un servicio que no se inicia

Si no se inicia un servicio, verás que la `MODE` es `replicated` pero `REPLICAS` se notarás 0. Para ver si la versión del sistema operativo es el problema, ejecute los siguientes comandos:

Ejecute **ls de servicio de docker** para buscar el nombre del servicio:

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

Ejecuta el **servicio de docker ps (nombre del servicio)** para obtener el estado y los intentos más recientes:

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

Si ves `starting container failed: ...`, puedes ver el error completo con el **servicio docker ps, n-truncar (nombre del contenedor)**:

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

Este es el mismo error como `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`.

### <a name="fix---update-the-service-to-use-a-matching-version"></a>Corrección: actualizar el servicio para que use una versión coincidente

Hay dos consideraciones para tener en cuenta con Docker Swarm. En el caso donde tienes un archivo de compose que tiene un servicio que usa una imagen que no se creó, querrás actualizar la referencia según corresponda. Por ejemplo:

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

La otra consideración es si la imagen que apuntan a es aquella que has creado tú mismo (por ejemplo, contoso/myimage):

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

En este caso, debes usar el método descrito en [errores de las versiones que no coincidentes](#errors-from-mismatched-versions) para modificar ese dockerfile en lugar de la línea de docker-compose.

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>Mitigación: usar el aislamiento de Hyper-V con Docker Swarm

Hay una propuesta para admitir el uso de aislamiento de Hyper-V por contenedor, pero todavía no está terminado el código. Puedes seguir el progreso en [GitHub](https://github.com/moby/moby/issues/31616). Hasta que esté listo, podría ser necesario configurar los hosts para que siempre se ejecuten con aislamiento de Hyper-V.

Para ello, se debe cambiar la configuración del servicio Docker y luego reiniciar el motor de Docker.

1. Edita `C:\ProgramData\docker\config\daemon.json`
2. Agrega una línea con `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >No existe el archivo daemon.json de manera predeterminada. Si observas que es así cuando echas un vistazo en el directorio, debes crear el archivo. A continuación, querrás copiar en el siguiente ejemplo:

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. Cierra y guarda el archivo y luego reiniciar el motor de docker mediante la ejecución de los siguientes cmdlets de PowerShell:

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. Después de que se ha reiniciado el servicio, inicia los contenedores. Una vez que estén en ejecución, puedes comprobar el nivel de aislamiento de un contenedor examinando el contenedor con el cmdlet siguiente:

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

Se mostrarán los valores "process" o "hyperv". Si modificaste y estableciste tu archivo daemon.json como se describe más arriba, debería mostrarse "hyperv".

### <a name="mitigation---use-labels-and-constraints"></a>Mitigación: usar etiquetas y restricciones

Aquí te mostramos cómo usar etiquetas y restricciones para que coincida con las versiones:

1. Agregar etiquetas a cada nodo.

    En cada nodo, agrega dos etiquetas: `OS` y `OsVersion`. En este procedimiento se supone que estás ejecutando de forma local, pero podría modificarse para establecerlas en un host remoto.

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    Comprobarlas ejecutando el comando **inspeccionar el nodo de docker** , que debe mostrar las etiquetas recién agregadas:

    ```yaml
           "Spec": {
                "Labels": {
                   "OS": "windows",
                   "OsVersion": "10.0.16296"
               },
                "Role": "manager",
                "Availability": "active"
            }
    ```

2. Agregar una restricción de servicio.

    Ahora que has etiquetado cada nodo, puedes actualizar las restricciones que determinan la ubicación de los servicios. En el ejemplo siguiente, sustituye "contoso_service" con el nombre de tu servicio real:

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    Esto aplica y limita los lugares en los que se puede ejecutar un nodo.

Para obtener más información sobre cómo usar las restricciones de servicios, echa un vistazo a que el [servicio de crear la referencia](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint).

## <a name="matching-versions-using-kubernetes"></a>Coincidencia de versiones mediante Kubernetes

El mismo problema que se describe en [las versiones de coincidencia con el enjambre de Docker](#matching-versions-using-docker-swarm) puede suceder cuando se programan pods en Kubernetes. Este problema puede evitarse con estrategias similares:

- Volver a compilar el contenedor en función de la misma versión de sistema operativo de desarrollo y producción. Para obtener información sobre cómo hacerlo, consulta [elegir qué versión del sistema operativo del contenedor](#choose-which-container-os-version-to-use).
- Usa etiquetas de nodo y nodeSelectors para asegurarte de que los pods se programen en nodos compatibles si hay nodos de 1709 de versión de Windows Server 2016 y Windows Server en el mismo clúster
- Usar clústeres independientes según la versión de SO

### <a name="finding-pods-failed-on-os-mismatch"></a>Buscar pods con errores debido a la falta de coincidencia del SO

En este caso, una implementación incluía un pod programado en un nodo con una versión de sistema operativo no coincidente y sin aislamiento de Hyper-V habilitado.

El mismo error se muestra en los eventos que se muestran con `kubectl describe pod <podname>`. Después de varios intentos, el estado del pod probablemente será `CrashLoopBackOff`.

```
$ kubectl -n plang describe pod fabrikamfiber.web-789699744-rqv6p

Name:           fabrikamfiber.web-789699744-rqv6p
Namespace:      plang
Node:           38519acs9011/10.240.0.6
Start Time:     Mon, 09 Oct 2017 19:40:30 +0000
Labels:         io.kompose.service=fabrikamfiber.web
                pod-template-hash=789699744
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-789699744","uid":"b5062a08-ad29-11e7-b16e-000d3a...
Status:         Running
IP:             10.244.3.169
Created By:     ReplicaSet/fabrikamfiber.web-789699744
Controlled By:  ReplicaSet/fabrikamfiber.web-789699744
Containers:
  fabrikamfiberweb:
    Container ID:       docker://eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a
    Image:              patricklang/fabrikamfiber.web:latest
    Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
    Port:               80/TCP
    State:              Waiting
      Reason:           CrashLoopBackOff
    Last State:         Terminated
      Reason:           ContainerCannotRun
      Message:          container eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{037b6606-bc9c-461f-ae02-829c28410798}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Layers":[{"ID":"f8bc427f-7aa3-59c6-b271-7331713e9451","Path":"C:\\ProgramData\\docker\\windowsfilter\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881"},{"ID":"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47","Path":"C:\\ProgramData\\docker\\windowsfilter\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5"},{"ID":"4f624ca7-2c6d-5c42-b73f-be6e6baf2530","Path":"C:\\ProgramData\\docker\\windowsfilter\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67"},{"ID":"88e360ff-32af-521d-9a3f-3760c12b35e2","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e"},{"ID":"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a","Path":"C:\\ProgramData\\docker\\windowsfilter\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461"},{"ID":"c2b3d728-4879-5343-a92a-b735752a4724","Path":"C:\\ProgramData\\docker\\windowsfilter\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3"},{"ID":"2973e760-dc59-5800-a3de-ab9d93be81e5","Path":"C:\\ProgramData\\docker\\windowsfilter\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75"},{"ID":"454a7d36-038c-5364-8a25-fa84091869d6","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0"},{"ID":"9b748c8c-69eb-55fb-a1c1-5688cac4efd8","Path":"C:\\ProgramData\\docker\\windowsfilter\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec"},{"ID":"bfde5c26-b33f-5424-9405-9d69c2fea4d0","Path":"C:\\ProgramData\\docker\\windowsfilter\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2"},{"ID":"bdabfbf5-80d1-57f1-86f3-448ce97e2d05","Path":"C:\\ProgramData\\docker\\windowsfilter\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf"},{"ID":"ad9b34f2-dcee-59ea-8962-b30704ae6331","Path":"C:\\ProgramData\\docker\\windowsfilter\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8"}],"HostName":"fabrikamfiber.web-789699744-rqv6p","MappedDirectories":[{"HostPath":"c:\\var\\lib\\kubelet\\pods\\b50f0027-ad29-11e7-b16e-000d3afd2878\\volumes\\kubernetes.io~secret\\default-token-rw9dn","ContainerPath":"c:\\var\\run\\secrets\\kubernetes.io\\serviceaccount","ReadOnly":true,"BandwidthMaximum":0,"IOPSMaximum":0}],"HvPartition":false,"EndpointList":null,"NetworkSharedContainerName":"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13","Servicing":false,"AllowUnqualifiedDNSQuery":false}
      Exit Code:        128
      Started:          Mon, 09 Oct 2017 20:27:08 +0000
      Finished:         Mon, 09 Oct 2017 20:27:08 +0000
    Ready:              False
    Restart Count:      10
    Environment:        <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
Conditions:
  Type          Status
  Initialized   True
  Ready         False
  PodScheduled  True
Volumes:
  default-token-rw9dn:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-rw9dn
    Optional:   false
QoS Class:      BestEffort
Node-Selectors: beta.kubernetes.io/os=windows
Tolerations:    <none>
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
  ---------     --------        -----   ----                    -------------                           --------        ------                  -------
  49m           49m             1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-789699744-rqv6p to 38519acs9011
  49m           49m             1       kubelet, 38519acs9011                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
  29m           29m             1       kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Failed to pull image "patricklang/fabrikamfiber.web:latest": rpc error: code = 2 desc = Error response from daemon: {"message":"Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io: no such host"}
  49m           3m              12      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
  33m           2m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Error: failed to start container "fabrikamfiberweb": Error response from daemon: {"message":"container fabrikamfiberweb encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {\"SystemType\":\"Container\",\"Name\":\"fabrikamfiberweb\",\"Owner\":\"docker\",\"IsDummy\":false,\"VolumePath\":\"\\\\\\\\?\\\\Volume{037b6606-bc9c-461f-ae02-829c28410798}\",\"IgnoreFlushesDuringBoot\":true,\"LayerFolderPath\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\fabrikamfiberweb\",\"Layers\":[{\"ID\":\"f8bc427f-7aa3-59c6-b271-7331713e9451\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881\"},{\"ID\":\"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5\"},{\"ID\":\"4f624ca7-2c6d-5c42-b73f-be6e6baf2530\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67\"},{\"ID\":\"88e360ff-32af-521d-9a3f-3760c12b35e2\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e\"},{\"ID\":\"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461\"},{\"ID\":\"c2b3d728-4879-5343-a92a-b735752a4724\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3\"},{\"ID\":\"2973e760-dc59-5800-a3de-ab9d93be81e5\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75\"},{\"ID\":\"454a7d36-038c-5364-8a25-fa84091869d6\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0\"},{\"ID\":\"9b748c8c-69eb-55fb-a1c1-5688cac4efd8\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec\"},{\"ID\":\"bfde5c26-b33f-5424-9405-9d69c2fea4d0\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2\"},{\"ID\":\"bdabfbf5-80d1-57f1-86f3-448ce97e2d05\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf\"},{\"ID\":\"ad9b34f2-dcee-59ea-8962-b30704ae6331\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8\"}],\"HostName\":\"fabrikamfiber.web-789699744-rqv6p\",\"MappedDirectories\":[{\"HostPath\":\"c:\\\\var\\\\lib\\\\kubelet\\\\pods\\\\b50f0027-ad29-11e7-b16e-000d3afd2878\\\\volumes\\\\kubernetes.io~secret\\\\default-token-rw9dn\",\"ContainerPath\":\"c:\\\\var\\\\run\\\\secrets\\\\kubernetes.io\\\\serviceaccount\",\"ReadOnly\":true,\"BandwidthMaximum\":0,\"IOPSMaximum\":0}],\"HvPartition\":false,\"EndpointList\":null,\"NetworkSharedContainerName\":\"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13\",\"Servicing\":false,\"AllowUnqualifiedDNSQuery\":false}"}
  33m           11s             151     kubelet, 38519acs9011                                           Warning         FailedSync              Error syncing pod
  32m           11s             139     kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         BackOff                 Back-off restarting failed container
```

### <a name="mitigation---using-node-labels-and-nodeselector"></a>Mitigación: usar etiquetas de nodo y de nodeSelector

Ejecutar **kubectl obtener nodo** para obtener una lista de todos los nodos. Después de eso, puedes ejecutar **kubectl describir nodo (nombre de nodo)** para obtener más detalles.

En el ejemplo siguiente, dos nodos de Windows se ejecutan versiones diferentes:

```
$ kubectl get node

NAME                        STATUS    AGE       VERSION
38519acs9010                Ready     21h       v1.7.7-7+e79c96c8ff2d8e
38519acs9011                Ready     4h        v1.7.7-25+bc3094f1d650a2
k8s-linuxpool1-38519084-0   Ready     21h       v1.7.7
k8s-master-38519084-0       Ready     21h       v1.7.7

$ kubectl describe node 38519acs9010

Name:                   38519acs9010
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_D2_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9010
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 01:41:02 +0000

...
  
System Info:
 Machine ID:                    38519acs9010
 System UUID:
 Boot ID:
 Kernel Version:                10.0 14393 (14393.1715.amd64fre.rs1_release_inmarket.170906-1810)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
 ...
 
$ kubectl describe node 38519acs9011
Name:                   38519acs9011
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_DS1_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9011
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 18:13:25 +0000
Conditions:
...

System Info:
 Machine ID:                    38519acs9011
 System UUID:
 Boot ID:
 Kernel Version:                10.0 16299 (16299.0.amd64fre.rs3_release.170922-1354)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
...

```

Vamos a usar en este ejemplo para mostrar cómo hacer coincidir las versiones:

1. Anota el nombre de cada nodo y `Kernel Version` desde la información del sistema.

    En nuestro ejemplo, la información tendrá este aspecto:

    Nombre         | Versión
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. Agrega una etiqueta para cada nodo denominada `beta.kubernetes.io/osbuild`. Windows Server 2016 admitirse sin aislamiento de Hyper-V debe versiones principales y secundarias (14393.1715 en este ejemplo). Windows Server versión 1709 solo necesita la versión principal (16299 en este ejemplo) para que coincida con.

    En este ejemplo, el comando para agregar las etiquetas tiene este aspecto:

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. Comprueba que las etiquetas estén presentes y ejecutando **kubectl obtener nodos: Mostrar etiquetas**.

    En este ejemplo, el resultado tendrá este aspecto:

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. Agregar selectores de nodo para las implementaciones. En este caso de ejemplo, agregaremos un `nodeSelector` a la especificación de contenedor con `beta.kubernetes.io/os` = windows y `beta.kubernetes.io/osbuild` = 14393.* o 16299 para que coincida con el sistema operativo base utilizado por el contenedor.

    A continuación se muestra un ejemplo completo para ejecutar un contenedor creado para Windows Server 2016:

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      annotations:
        kompose.cmd: kompose convert -f docker-compose-combined.yml
        kompose.version: 1.2.0 (99f88ef)
      creationTimestamp: null
      labels:
        io.kompose.service: fabrikamfiber.web
      name: fabrikamfiber.web
    spec:
      replicas: 1
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            io.kompose.service: fabrikamfiber.web
        spec:
          containers:
          - image: patricklang/fabrikamfiber.web:latest
            name: fabrikamfiberweb
            ports:
            - containerPort: 80
            resources: {}
          restartPolicy: Always
          nodeSelector:
            "beta.kubernetes.io/os": windows
            "beta.kubernetes.io/osbuild": "14393.1715"
    status: {}
    ```

    El pod ya puede iniciarse con la implementación actualizada. Los selectores de nodo también se muestran en `kubectl describe pod <podname>`, por lo que puedes ejecutar este comando para comprobar que se agregaron.

    La salida en nuestro ejemplo es el siguiente:

    ```
    $ kubectl -n plang describe po fa

    Name:           fabrikamfiber.web-1780117715-5c8vw
    Namespace:      plang
    Node:           38519acs9010/10.240.0.4
    Start Time:     Tue, 10 Oct 2017 01:43:28 +0000
    Labels:         io.kompose.service=fabrikamfiber.web
                    pod-template-hash=1780117715
    Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-1780117715","uid":"6a07aaf3-ad5c-11e7-b16e-000d3...
    Status:         Running
    IP:             10.244.1.84
    Created By:     ReplicaSet/fabrikamfiber.web-1780117715
    Controlled By:  ReplicaSet/fabrikamfiber.web-1780117715
    Containers:
      fabrikamfiberweb:
        Container ID:       docker://c94594fb53161f3821cf050d9af7546991aaafbeab41d333d9f64291327fae13
        Image:              patricklang/fabrikamfiber.web:latest
        Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
        Port:               80/TCP
        State:              Running
          Started:          Tue, 10 Oct 2017 01:43:42 +0000
        Ready:              True
        Restart Count:      0
        Environment:        <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
    Conditions:
      Type          Status
      Initialized   True
      Ready         True
      PodScheduled  True
    Volumes:
      default-token-rw9dn:
        Type:       Secret (a volume populated by a Secret)
        SecretName: default-token-rw9dn
        Optional:   false
    QoS Class:      BestEffort
    Node-Selectors: beta.kubernetes.io/os=windows
                    beta.kubernetes.io/osbuild=14393.1715
    Tolerations:    <none>
    Events:
      FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
      ---------     --------        -----   ----                    -------------                           --------        ------                  -------
      5m            5m              1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-1780117715-5c8vw to 38519acs9010
      5m            5m              1       kubelet, 38519acs9010                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Started                 Started container
    ```
