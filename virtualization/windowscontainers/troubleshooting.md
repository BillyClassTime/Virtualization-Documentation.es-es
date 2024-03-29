---
title: Solución de problemas de contenedores de Windows
description: Sugerencias de solución de problemas, scripts automatizados e información de registro para los contenedores de Windows y Docker
keywords: docker, contenedores, solución de problemas, registros
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
ms.openlocfilehash: 1de86a2492ca899dc3fb932e0d57927fa4000fd0
ms.sourcegitcommit: 15b5ab92b7b8e96c180767945fdbb2963c3f6f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/06/2019
ms.locfileid: "74911715"
---
# <a name="troubleshooting"></a>Solución de problemas

¿Tienes problemas para configurar el equipo o para ejecutar un contenedor? Hemos creado un script de PowerShell para comprobar los problemas comunes. Pruébelo primero para ver lo que encuentra y compartir los resultados.

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
Puede ver una lista de todas las pruebas que se ejecutan junto con soluciones comunes en el [archivo Léame](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md) del script.

Si eso no ayuda encontrar el origen del problema, publique la salida del script en el [Foro del contenedor](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers). Este es el mejor lugar para obtener ayuda de la comunidad, incluidos los desarrolladores e Insiders de Windows.


### <a name="finding-logs"></a>Buscar registros
Hay varios servicios que se usan para administrar contenedores de Windows. En las secciones siguientes se muestra dónde obtener los registros de cada servicio.

## <a name="docker-container-logs"></a>Registros de contenedor de Docker 
El comando `docker logs` captura los registros de un contenedor de STDOUT/STDERR, las ubicaciones de depósito de registro de aplicación estándar para las aplicaciones Linux. Normalmente, las aplicaciones de Windows no registran en STDOUT/STDERR; en su lugar, registran en ETW, registros de eventos o archivos de registro, entre otros. 

El [monitor de registro](https://github.com/microsoft/windows-container-tools/tree/master/LogMonitor), una herramienta de código abierto compatible con Microsoft, ahora está disponible en github. El monitor de registro conecta los registros de aplicación Windows a STDOUT/STDERR. El monitor de registro se configura a través de un archivo de configuración. 

### <a name="log-monitor-usage"></a>Uso del monitor de registro

LogMonitor. exe y LogMonitorConfig. JSON deben incluirse en el mismo directorio LogMonitor. 

El monitor de registro puede usarse en un patrón de uso de SHELL:

```
SHELL ["C:\\LogMonitor\\LogMonitor.exe", "cmd", "/S", "/C"]
CMD c:\windows\system32\ping.exe -n 20 localhost
```

O un patrón de uso de ENTRYPOINT:

```
ENTRYPOINT C:\LogMonitor\LogMonitor.exe c:\windows\system32\ping.exe -n 20 localhost
```

Ambos usos de ejemplo contienen la aplicación ping. exe. Otras aplicaciones (como [IIS). ServiceMonitor]( https://github.com/microsoft/IIS.ServiceMonitor)) se pueden anidar con el monitor de registro de manera similar:

```
COPY LogMonitor.exe LogMonitorConfig.json C:\LogMonitor\
WORKDIR /LogMonitor
SHELL ["C:\\LogMonitor\\LogMonitor.exe", "powershell.exe"]
 
# Start IIS Remote Management and monitor IIS
ENTRYPOINT      Start-Service WMSVC; `
                    C:\ServiceMonitor.exe w3svc;
```


El monitor de registro inicia la aplicación ajustada como proceso secundario y supervisa la salida de STDOUT de la aplicación.

Tenga en cuenta que en el patrón de uso de SHELL se debe especificar la instrucción CMD/ENTRYPOINT en el formulario de SHELL y no en el formulario exec. Cuando se usa el formulario Exec de la instrucción CMD/ENTRYPOINT, no se inicia el SHELL y la herramienta de supervisión de registros no se inicia dentro del contenedor.

Puede encontrar más información sobre el uso en el [wiki de supervisión de registros](https://github.com/microsoft/windows-container-tools/wiki). Los archivos de configuración de ejemplo para escenarios clave de contenedor de Windows (IIS, etc.) se pueden encontrar en el [repositorio de github](https://github.com/microsoft/windows-container-tools/tree/master/LogMonitor/src/LogMonitor/sample-config-files). En esta [entrada de blog](https://techcommunity.microsoft.com/t5/Containers/Windows-Containers-Log-Monitor-Opensource-Release/ba-p/973947)se puede encontrar contexto adicional.

## <a name="docker-engine"></a>Motor de Docker
El motor de Docker registra en el registro de eventos "Application" de Windows, en lugar de en un archivo. Estos registros se pueden leer, ordenar y filtrar muy fácilmente con Windows PowerShell.

Por ejemplo, esto mostrará los registros del motor de Docker de los últimos 5 minutos, empezando por los más antiguos.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

Esto también se podría canalizar fácilmente en un archivo CSV para que otra herramienta u hoja de cálculo pueda leerlo.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

### <a name="enabling-debug-logging"></a>Habilitar el registro de depuración
También puede habilitar el registro de depuración en el motor de Docker. Esto puede resultar útil para solucionar problemas si los registros normales no tienen información suficiente.

En primer lugar, abra un símbolo del sistema con privilegios elevados, ejecute `sc.exe qc docker` para obtener la línea de comandos para el servicio Docker.
Por ejemplo:
```
C:\> sc.exe qc docker
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: docker
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\Docker\dockerd.exe" --run-service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Docker Engine
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

Tome la `BINARY_PATH_NAME` actual y modifíquela:
- Agregue -D al final
- Escape las "con \
- Escriba el comando completo entre "

A continuación, ejecute `sc.exe config docker binpath=` seguido de la nueva cadena. Por ejemplo: 
```
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


Reinicie el servicio Docker
```
sc.exe stop docker
sc.exe start docker
```

Esto registrará mucha más información en el registro de eventos de aplicación, por lo que es mejor quitar la opción `-D` una vez que haya terminado de solucionar los problemas. Usa los mismos pasos anteriores sin `-D` y reinicia el servicio para deshabilitar el registro de depuración.

Una alternativa a lo anterior es ejecutar el daemon de Docker en modo de depuración desde el símbolo de PowerShell con privilegios elevados, capturando los resultados directamente en un archivo.
```PowerShell
sc.exe stop docker
<path\to\>dockerd.exe -D > daemon.log 2>&1
```

### <a name="obtaining-stack-dump"></a>Obtención del volcado de la pila

Por lo general, esto solo es útil si lo solicita explícitamente el servicio de soporte técnico de Microsoft o los desarrolladores de Docker. Se puede usar para ayudar a diagnosticar una situación en la que el Docker parece estar bloqueado. 

Descarga [docker signal.exe](https://github.com/jhowardmsft/docker-signal).

Uso:
```PowerShell
docker-signal --pid=$((Get-Process dockerd).Id)
```

El archivo de salida se ubicará en el directorio raíz de datos en el que se ejecuta Docker. El directorio predeterminado es `C:\ProgramData\Docker`. El directorio real puede confirmarse ejecutando `docker info -f "{{.DockerRootDir}}"`.

El archivo se `goroutine-stacks-<timestamp>.log`.

Tenga en cuenta que `goroutine-stacks*.log` no contiene información personal.


## <a name="host-compute-service"></a>Servicio de proceso de host
El motor de Docker depende de un servicio de contenedor de host específico de Windows. Tiene registros independientes: 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

Son visibles en el Visor de eventos y también pueden consultarse con PowerShell.

Por ejemplo:
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```

### <a name="capturing-hcs-analyticdebug-logs"></a>Capturar registros de análisis y depuración de HCS

Habilitar los registros de análisis y depuración para proceso de Hyper-V y guardarlos en `hcslog.evtx`.

```PowerShell
# Enable the analytic logs
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:true /q:true

# <reproduce your issue>

# Export to an evtx
wevtutil.exe epl Microsoft-Windows-Hyper-V-Compute-Analytic <hcslog.evtx>

# Disable
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:false /q:true
```

### <a name="capturing-hcs-verbose-tracing"></a>Capturar el rastreo detallado de HCS.

Por lo general, estos solamente son útiles si lo solicita el soporte técnico de Microsoft. 

Descarga [HcsTraceProfile.wprp](https://gist.github.com/jhowardmsft/71b37956df0b4248087c3849b97d8a71)

```PowerShell
# Enable tracing
wpr.exe -start HcsTraceProfile.wprp!HcsArgon -filemode

# <reproduce your issue>

# Capture to HcsTrace.etl
wpr.exe -stop HcsTrace.etl "some description"
```

Proporciona `HcsTrace.etl` a tu contacto de soporte técnico.
