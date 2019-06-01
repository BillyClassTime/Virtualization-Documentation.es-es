---
title: Compatibilidad con versiones de contenedores de Windows
description: Cómo puede Windows ejecutar compilaciones y contenedores en varias versiones de Windows
keywords: metadatos, contenedores, versión
author: taylorb-microsoft
ms.openlocfilehash: 4d01fb1d11ee9e8a5fa4271699a5a7c59c27409d
ms.sourcegitcommit: 71e46750813a996cecc445181974a79b95affc8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/31/2019
ms.locfileid: "9685350"
---
# <a name="windows-container-version-compatibility"></a>Compatibilidad de versiones de contenedor de Windows

La actualización de aniversario de Windows Server 2016 y Windows 10 (ambas versiones 14393) fueron las primeras versiones de Windows que podían compilar y ejecutar contenedores de Windows Server. Los contenedores compilados con estas versiones se pueden ejecutar en las versiones más recientes, como Windows Server versión 1709, pero hay algunos aspectos que debes saber antes de empezar.

Dado que hemos mejorado las características de los contenedores de Windows, tuvimos que hacer algunos cambios que pueden afectar a la compatibilidad. Los contenedores más antiguos se ejecutarán de la misma en hosts más recientes con el [aislamiento de Hyper-V](../manage-containers/hyperv-container.md)y usarán la misma versión de kernel (la más antigua). Sin embargo, si desea ejecutar un contenedor basado en una compilación de Windows más reciente, solo se puede ejecutar en la compilación de host más reciente.

|Versión del SO de contenedor|Versión del SO de host|Compatibilidad|
|---|---|---|
|Windows Server 2019, versión 1903<br>Compilaciones 18362. * |Windows Server, versión 1903<br>Compilaciones 18362. * |Compatibilidad `process` o `hyperv` aislamiento|
|Windows Server 2019, versión 1903<br>Compilaciones 18362. * |Windows 10, versión 1903<br>Compilaciones 18362. * |Solo admite `hyperv` el aislamiento|
|Windows Server 2019, versión 1903<br>Compilaciones 18362. * |Windows 10, versión 1809<br>Compilaciones 17763. * |No se admite|
|Windows Server 2019, versión 1903<br>Compilaciones 18362. * |Windows Server 2019<br>Compilaciones 17763. * |No se admite|
|Windows Server 2019, versión 1903<br>Compilaciones 18362. * |Windows 10, versión 1803<br>Compilaciones 17134. * |No se admite|
|Windows Server 2019, versión 1903<br>Compilaciones 18362. * |Windows Server, versión 1803<br>Compilaciones 17134. * |No se admite|
|Windows Server 2019, versión 1903<br>Compilaciones 18362. * |Windows10FallCreatorsUpdate<br>Compilaciones: 16299.* |No se admite|
|Windows Server 2019, versión 1903<br>Compilaciones 18362. * |Windows Server, versión 1709<br>Compilaciones: 16299.* |No se admite|
|Windows Server 2019, versión 1903<br>Compilaciones 18362. * |WindowsServer2016<br>Compilaciones: 14393.* |No se admite|
|Windows Server 2019<br>Compilaciones 17763. * |Windows Server, versión 1903<br>Compilaciones 18362. * |Solo admite `hyperv` el aislamiento|
|Windows Server 2019<br>Compilaciones 17763. * |Windows 10, versión 1903<br>Compilaciones 18362. * |Solo admite `hyperv` el aislamiento|
|Windows Server 2019<br>Compilaciones 17763. * |Windows 10, versión 1809<br>Compilaciones 17763. * |Solo admite `hyperv` el aislamiento|
|Windows Server 2019<br>Compilaciones 17763. * |Windows Server 2019<br>Compilaciones 17763. * |Compatibilidad `process` o `hyperv` aislamiento|
|Windows Server 2019<br>Compilaciones 17763. * |Windows 10, versión 1803<br>Compilaciones 17134. * |No se admite|
|Windows Server 2019<br>Compilaciones 17763. * |Windows Server, versión 1803<br>Compilaciones 17134. * |No se admite|
|Windows Server 2019<br>Compilaciones 17763. * |Windows10FallCreatorsUpdate<br>Compilaciones: 16299.* |No se admite|
|Windows Server 2019<br>Compilaciones 17763. * |Windows Server, versión 1709<br>Compilaciones: 16299.* |No se admite|
|Windows Server 2019<br>Compilaciones 17763. * |WindowsServer2016<br>Compilaciones: 14393.* |No se admite|
|Windows Server, versión 1803<br>Compilaciones 17134. * |Windows Server, versión 1903<br>Compilaciones 18362. * |Solo admite `hyperv` el aislamiento|
|Windows Server, versión 1803<br>Compilaciones 17134. * |Windows 10, versión 1903<br>Compilaciones 18362. * |Solo admite `hyperv` el aislamiento|
|Windows Server, versión 1803<br>Compilaciones 17134. * |Windows 10, versión 1809<br>Compilaciones 17763. * |Solo admite `hyperv` el aislamiento|
|Windows Server, versión 1803<br>Compilaciones 17134. * |Windows Server 2019<br>Compilaciones 17763. * |Solo admite `hyperv` el aislamiento|
|Windows Server, versión 1803<br>Compilaciones 17134. * |Windows 10, versión 1803<br>Compilaciones 17134. * |Solo admite `hyperv` el aislamiento|
|Windows Server, versión 1803<br>Compilaciones 17134. * |Windows Server, versión 1803<br>Compilaciones 17134. * |Compatibilidad `process` o `hyperv` aislamiento|
|Windows Server, versión 1803<br>Compilaciones 17134. * |Windows10FallCreatorsUpdate<br>Compilaciones: 16299.* |No se admite|
|Windows Server, versión 1803<br>Compilaciones 17134. * |Windows Server, versión 1709<br>Compilaciones: 16299.* |No se admite|
|Windows Server, versión 1803<br>Compilaciones 17134. * |WindowsServer2016<br>Compilaciones: 14393.* |No se admite|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows Server, versión 1903<br>Compilaciones 18362. * |Solo admite `hyperv` el aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows 10, versión 1903<br>Compilaciones 18362. * |Solo admite `hyperv` el aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows 10, versión 1809<br>Compilaciones 17763. * |Solo admite `hyperv` el aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows Server 2019<br>Compilaciones 17763. * |Solo admite `hyperv` el aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows 10, versión 1803<br>Compilaciones 17134. * |Solo admite `hyperv` el aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows Server, versión 1803<br>Compilaciones 17134. * |Solo admite `hyperv` el aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows10FallCreatorsUpdate<br>Compilaciones: 16299.* |Solo admite `hyperv` el aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |Windows Server, versión 1709<br>Compilaciones: 16299.* |Compatibilidad `process` o `hyperv` aislamiento|
|Windows Server, versión 1709<br>Compilaciones: 16299.* |WindowsServer2016<br>Compilaciones: 14393.* |No se admite|
|WindowsServer2016<br>Compilaciones: 14393.* |Windows Server, versión 1903<br>Compilaciones 18362. * |Solo admite `hyperv` el aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |Windows 10, versión 1903<br>Compilaciones 18362. * |Solo admite `hyperv` el aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |Windows 10, versión 1809<br>Compilaciones 17763. * |Solo admite `hyperv` el aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |Windows Server 2019<br>Compilaciones 17763. * |Solo admite `hyperv` el aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |Windows10FallCreatorsUpdate<br>Compilaciones: 16299.* |Solo admite `hyperv` el aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |Windows Server versión 1803<br>Compilaciones 17134. * |Solo admite `hyperv` el aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |Windows 10, versión 1803<br>Compilaciones 17134. * |Solo admite `hyperv` el aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |WindowsServer, versión1709<br>Compilaciones: 16299.* |Solo admite `hyperv` el aislamiento|
|WindowsServer2016<br>Compilaciones: 14393.* |WindowsServer2016<br>Compilaciones: 14393.* |Compatibilidad `process` o `hyperv` aislamiento|

## <a name="matching-container-host-version-with-container-image-versions"></a>Versión de host de contenedor coincidente con versiones de imagen de contenedor

### <a name="windows-server-containers"></a>Contenedores de Windows Server

Dado que los contenedores de Windows Server y el host subyacente comparten un único núcleo, la imagen base del contenedor debe coincidir con la del host. Si las versiones son distintas, el contenedor puede iniciarse, pero no se garantiza la funcionalidad completa. El sistema operativo Windows tiene cuatro niveles de versiones: principal, secundario, compilación y revisión. Por ejemplo, la versión 10.0.14393.103 tendría una versión principal de 10, una versión secundaria de 0, un número de compilación de 14393 y un número de revisión de 103. El número de compilación solo cambia cuando se publican nuevas versiones del sistema operativo, como la versión 1709, 1803, la actualización del otoño de creadores y así sucesivamente. El número de revisión se actualiza cuando se aplican las actualizaciones de Windows.

#### <a name="build-number-new-release-of-windows"></a>Número de compilación (nueva versión de Windows)

Los contenedores de Windows Server están bloqueados para iniciarse cuando el número de compilación entre el host del contenedor y la imagen del contenedor es diferente. Por ejemplo, cuando el host del contenedor es la versión 10.0.14393. * (Windows Server 2016) y la imagen del contenedor es la versión 10.0.16299. * (Windows Server versión 1709), el contenedor no se inicia.  

#### <a name="revision-number-patching"></a>Número de revisión (revisión)

Los contenedores de Windows Server no están bloqueados para iniciarse cuando los números de revisión del host contenedor y la imagen de contenedor son diferentes. Por ejemplo, si el host del contenedor es la versión 10.0.14393.1914 (Windows Server 2016 con KB4051033 aplicado) y la imagen del contenedor es la versión 10.0.14393.1944 (Windows Server 2016 con KB4053579 aplicado), la imagen se iniciará aunque la revisión los números son diferentes.

En el caso de hosts o imágenes basados en Windows Server 2016, la revisión de la imagen del contenedor debe coincidir con el host para que esté en una configuración admitida. Sin embargo, para los hosts o las imágenes que usan Windows Server versión 1709 y versiones posteriores, esta regla no se aplica y la imagen del host y del contenedor no debe tener revisiones coincidentes. Le recomendamos mantener sus sistemas actualizados con las últimas revisiones y actualizaciones.

#### <a name="practical-application"></a>Aplicación práctica

Ejemplo 1: el host de contenedor ejecuta Windows Server 2016 con KB4041691 aplicado. Cualquier contenedor de Windows Server distribuido en este host debe basarse en las imágenes base del contenedor de la versión 10.0.14393.1770. Si aplica KB4053579 al contenedor host, también debe actualizar las imágenes para asegurarse de que el contenedor host las admite.

Ejemplo 2: el host de contenedor ejecuta la versión 1709 de Windows Server con KB4043961 aplicado. Cualquier contenedor de Windows Server distribuido en este host debe estar basado en una imagen base del contenedor de Windows Server versión 1709 (10.0.16299), pero no es necesario que coincida con el host KB. Si se aplica KB4054517 al host, las imágenes contenedoras seguirán siendo compatibles, pero le recomendamos que las actualice para resolver cualquier posible problema de seguridad.

#### <a name="querying-version"></a>Consultar versiones

Método 1: presentado en la versión 1709, el símbolo del sistema cmd y el comando **Ver** ahora devuelven los detalles de la revisión.

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

Método 2: consultar la siguiente clave del registro: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion

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

Para comprobar qué versión usa la imagen base, revise las etiquetas en el concentrador de acoplamiento o la tabla hash de imagen proporcionada en la descripción de la imagen. La página del [historial de actualizaciones de Windows 10](https://support.microsoft.com/help/12387/windows-10-update-history) muestra Cuándo se liberó cada compilación y revisión.

### <a name="hyper-v-isolation-for-containers"></a>Aislamiento de Hyper-V para contenedores

Puede ejecutar contenedores de Windows con o sin el aislamiento Hyper-V. El aislamiento de Hyper-V crea un límite seguro alrededor del contenedor con una VM optimizada. A diferencia de los contenedores de Windows estándar que comparten el kernel entre contenedores y el host, cada contenedor aislado de Hyper-V tiene su propia instancia del núcleo de Windows. Esto significa que puede tener diferentes versiones del sistema operativo en la imagen y el host del contenedor (para obtener más información, consulte la siguiente matriz de compatibilidad).  

Para ejecutar un contenedor con aislamiento de Hyper-V, solo tienes que agregar la etiqueta `--isolation=hyperv` al comando de ejecución de docker.

## <a name="errors-from-mismatched-versions"></a>Errores de las versiones que no coinciden

Si intenta ejecutar una combinación no admitida, recibirá el siguiente error:

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

Hay tres formas de resolver este error:

- Vuelva a crear el contenedor basándose en la `mcr.microsoft.com/windows/nanoserver` versión correcta de o `mcr.microsoft.com/windows/servercore`
- Si el host es más reciente, ejecute el **acoplador y el aislamiento de HyperV...**
- Intente ejecutar el contenedor en otro host con la misma versión de Windows.

## <a name="choose-which-container-os-version-to-use"></a>Elegir la versión del sistema operativo de contenedor que se va a usar

>[!NOTE]
>A partir del 16 de abril de 2019, la etiqueta "última" ya no se publica ni se mantiene para las [imágenes del contenedor del sistema operativo Windows base](https://hub.docker.com/_/microsoft-windows-base-os-images). Declara una etiqueta específica al extraer o hacer referencia a las imágenes de estas Repos.

Debe saber qué versión necesita usar para su contenedor. Por ejemplo, si quiere que la versión 1809 de Windows Server sea su sistema operativo de contenedor y desee tener las revisiones más recientes para él, debe `1809` usar la etiqueta al especificar la versión de las imágenes del contenedor de sistema operativo que desea, por ejemplo:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809
...
```

Sin embargo, si desea una revisión específica de la versión 1809 de Windows Server, puede especificar el número de KB en la etiqueta. Por ejemplo, para obtener una imagen del contenedor del sistema operativo de nano Server base de la versión 1809 de Windows Server con el KB4493509 aplicado, debe especificarse así:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:1809-KB4493509
...
```

También puede especificar las revisiones exactas que necesita con el esquema que hemos usado anteriormente, especificando la versión del sistema operativo en la etiqueta:

``` dockerfile
FROM mcr.microsoft.com/windows/nanoserver:10.0.17763.437
...
```

Las imágenes básicas de Server Core basadas en Windows Server 2019 y Windows Server 2016 son versiones de [canal de mantenimiento a largo plazo (LTSC)](https://docs.microsoft.com/en-us/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc) . Si, por ejemplo, desea que Windows Server 2019 como el sistema operativo de contenedor de la imagen de Server Core y desee tener las revisiones más recientes para él, puede especificar las versiones de LTSC como:

``` dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019
...
```

## <a name="matching-versions-using-docker-swarm"></a>Coincidencia de versiones mediante Docker Swarm

El acoplador Swarm actualmente no tiene una forma integrada para hacer coincidir la versión de Windows que un contenedor usa con un host con la misma versión. Si actualiza el servicio para usar un contenedor más reciente, se ejecutará correctamente.

Si necesita ejecutar varias versiones de Windows durante un período de tiempo prolongado, hay dos enfoques que puede realizar: Configure los hosts de Windows para usar siempre el aislamiento de Hyper-V o las restricciones de etiqueta.

### <a name="finding-a-service-that-wont-start"></a>Encontrar un servicio que no se inicia

Si un servicio no se inicia, verá que el `MODE` es `replicated` , pero `REPLICAS` se detendrá en 0. Para ver si el problema se debe a la versión del sistema operativo, ejecute los siguientes comandos:

Ejecute el **servicio de dockr LS** para buscar el nombre del servicio:

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

Ejecute el **servicio de dockr PS (nombre de servicio)** para obtener el estado y los intentos más recientes:

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

Si ve `starting container failed: ...`, puede ver el error completo con el **servicio de acoplamiento PS--no-trunc (nombre de contenedor)**:

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

Este es el mismo error que `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`.

### <a name="fix---update-the-service-to-use-a-matching-version"></a>Corrección: actualizar el servicio para que use una versión coincidente

Hay dos consideraciones para tener en cuenta con Docker Swarm. En caso de que tenga un archivo de redacción que tiene un servicio que usa una imagen que no ha creado, es recomendable que actualice la referencia según corresponda. Por ejemplo:

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

La otra consideración es si la imagen a la que apunta es una que ha creado usted mismo (por ejemplo, contoso/My Image):

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

En este caso, debes usar el método descrito en [errores de versiones](#errors-from-mismatched-versions) no coincidentes para modificar ese dockerfile en lugar de la línea de redacción del acoplador.

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>Mitigación: usar el aislamiento de Hyper-V con Docker Swarm

Hay una propuesta de compatibilidad con el aislamiento de Hyper-V por contenedor, pero aún no se ha realizado el código. Puedes seguir el progreso en [GitHub](https://github.com/moby/moby/issues/31616). Hasta que esté listo, podría ser necesario configurar los hosts para que siempre se ejecuten con aislamiento de Hyper-V.

Para ello, se debe cambiar la configuración del servicio Docker y luego reiniciar el motor de Docker.

1. Edita `C:\ProgramData\docker\config\daemon.json`
2. Agrega una línea con `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >El archivo daemon. JSON no existe de forma predeterminada. Si observas que es así cuando echas un vistazo en el directorio, debes crear el archivo. Después, querrás copiar lo siguiente:

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. Cierre y guarde el archivo y, a continuación, reinicie el motor del Dock ejecutando los siguientes cmdlets de PowerShell:

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. Una vez que haya reiniciado el servicio, inicie los contenedores. Una vez que se estén ejecutando, puede comprobar el nivel de aislamiento de un contenedor inspeccionando el contenedor con el siguiente cmdlet:

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

Se mostrarán los valores "process" o "hyperv". Si modificaste y estableciste tu archivo daemon.json como se describe más arriba, debería mostrarse "hyperv".

### <a name="mitigation---use-labels-and-constraints"></a>Mitigación: usar etiquetas y restricciones

A continuación se explica cómo usar etiquetas y restricciones para hacer coincidir las versiones:

1. Agregar etiquetas a cada nodo.

    En cada nodo, agregue dos etiquetas: `OS` y `OsVersion`. En este procedimiento se supone que estás ejecutando de forma local, pero podría modificarse para establecerlas en un host remoto.

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    Después, puede comprobar estas ejecutando el comando **inspeccionar nodo de acoplamiento** , que debería mostrar las etiquetas recién agregadas:

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

2. Agregue una restricción de servicio.

    Ahora que ha etiquetado cada nodo, puede actualizar las restricciones que determinan la ubicación de los servicios. En el siguiente ejemplo, reemplace "contoso_service" por el nombre de su servicio real:

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    Esto aplica y limita los lugares en los que se puede ejecutar un nodo.

Para obtener más información sobre cómo usar las restricciones de servicio, consulte la [referencia de crear servicio](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint).

## <a name="matching-versions-using-kubernetes"></a>Coincidencia de versiones mediante Kubernetes

El mismo problema descrito en [las versiones coincidentes que usan Swarm de acoplamiento](#matching-versions-using-docker-swarm) puede producirse cuando se programan los pods en Kubernetes. Este problema se puede evitar con estrategias similares:

- Vuelva a crear el contenedor basándose en la misma versión del sistema operativo en desarrollo y producción. Para obtener más información, vea [elegir la versión del sistema operativo de contenedor que se va a usar](#choose-which-container-os-version-to-use).
- Use etiquetas de nodo y nodeSelectors para asegurarse de que los pods se programan en nodos compatibles si los nodos Windows Server 2016 y Windows Server versión 1709 están en el mismo clúster
- Usar clústeres independientes según la versión de SO

### <a name="finding-pods-failed-on-os-mismatch"></a>Buscar pods con errores debido a la falta de coincidencia del SO

En este caso, una implementación incluía un pod programado en un nodo con una versión del sistema operativo que no coincide y sin aislamiento de Hyper-V habilitado.

El mismo error se muestra en los eventos que se muestran con `kubectl describe pod <podname>`. Después de varios intentos, el estado de la Pod `CrashLoopBackOff`probablemente será.

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

### <a name="mitigation---using-node-labels-and-nodeselector"></a>Mitigación: uso de etiquetas de nodo y nodeSelector

Ejecute el **nodo get de kubectl** para obtener una lista de todos los nodos. Después de ese, puede ejecutar **kubectl describir nodo (nombre del nodo)** para obtener más detalles.

En el ejemplo siguiente, se ejecutan versiones diferentes de dos nodos de Windows:

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

Vamos a usar este ejemplo para mostrar cómo hacer coincidir las versiones:

1. Tome nota de cada nombre de nodo `Kernel Version` y de la información del sistema.

    En nuestro ejemplo, la información tendrá un aspecto similar a este:

    Nombre         | Versión
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. Agrega una etiqueta para cada nodo denominada `beta.kubernetes.io/osbuild`. Windows Server 2016 necesita que las versiones principales y secundarias (14393,1715 en este ejemplo) se admitan sin el aislamiento de Hyper-V. Windows Server versión 1709 solo necesita la versión principal (16299 en este ejemplo) para que coincidan.

    En este ejemplo, el comando para agregar las etiquetas tiene el siguiente aspecto:

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. Verifique que las etiquetas estén allí ejecutando **kubectl Get Nodes--show-Labels**.

    En este ejemplo, el resultado tendrá el siguiente aspecto:

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. Agregue selectores de nodo a las implementaciones. En este caso de ejemplo, agregaremos `nodeSelector` a a la especificación de contenedor `beta.kubernetes.io/os` con = Windows `beta.kubernetes.io/osbuild` y = 14393. * o 16299 para coincidir con el sistema operativo base usado por el contenedor.

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

    El pod ya puede iniciarse con la implementación actualizada. Los selectores de nodo también se `kubectl describe pod <podname>`muestran en, por lo que puede ejecutar ese comando para verificar que se hayan agregado.

    El resultado de nuestro ejemplo es el siguiente:

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
