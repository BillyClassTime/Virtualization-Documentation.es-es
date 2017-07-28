---
title: "Solución de problemas de contenedores de Windows"
description: "Sugerencias de solución de problemas, scripts automatizados e información de registro para los contenedores de Windows y Docker"
keywords: "docker, contenedores, solución de problemas, registros"
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
ms.openlocfilehash: 5230080386081bda8b54656d15f33b4986cfa6e3
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/21/2017
---
# Solución de problemas

¿Tienes problemas para configurar el equipo o para ejecutar un contenedor? Hemos creado un script de PowerShell para comprobar los problemas comunes. Pruébelo primero para ver lo que encuentra y compartir los resultados.

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
Puede ver una lista de todas las pruebas que se ejecutan junto con soluciones comunes en el [archivo Léame](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md) del script.

Si eso no ayuda encontrar el origen del problema, publique la salida del script en el [Foro del contenedor](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers). Este es el mejor lugar para obtener ayuda de la comunidad, incluidos los desarrolladores e Insiders de Windows.


## Buscar registros
Hay varios servicios que se usan para administrar contenedores de Windows. En las secciones siguientes se muestra dónde obtener los registros de cada servicio.

### Motor de Docker
El motor de Docker registra en el registro de eventos "Application" de Windows, en lugar de en un archivo. Estos registros se pueden leer, ordenar y filtrar muy fácilmente con Windows PowerShell.

Por ejemplo, esto mostrará los registros del motor de Docker de los últimos 5 minutos, empezando por los más antiguos.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

Esto también se podría canalizar fácilmente en un archivo CSV para que otra herramienta u hoja de cálculo pueda leerlo.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

#### Habilitar el registro de depuración
También puede habilitar el registro de depuración en el motor de Docker. Esto puede resultar útil para solucionar problemas si los registros normales no tienen información suficiente.

En primer lugar, abra un símbolo del sistema con privilegios elevados, ejecute `sc.exe qc docker` para obtener la línea de comandos para el servicio Docker.
Ejemplo:
```none
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

A continuación, ejecute `sc.exe config docker binpath= ` seguido de la nueva cadena. Por ejemplo: 
```none
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


Reinicie el servicio Docker
```none
sc.exe stop docker
sc.exe start docker
```

Esto registrará mucha más información en el registro de eventos de aplicación, por lo que es mejor quitar la opción `-D` una vez que haya terminado de solucionar los problemas. Usa los mismos pasos anteriores sin `-D` y reinicia el servicio para deshabilitar el registro de depuración.

Una alternativa a lo anterior es ejecutar el daemon de Docker en modo de depuración desde el símbolo de PowerShell con privilegios elevados, capturando los resultados directamente en un archivo.
```PowerShell
sc.exe stop docker
<path\to\>dockerd.exe -D > daemon.log 2>&1
```

#### Obtener los datos de volcado de pila y de daemon.

Por lo general, estos solamente son útiles si los solicitan explícitamente el soporte técnico de Microsoft o los desarrolladores de Docker. Pueden usarse para ayudar a diagnosticar una situación donde parezca que Docker está bloqueado. 

Descarga [docker signal.exe](https://github.com/jhowardmsft/docker-signal).

Uso:
```PowerShell
Get-Process dockerd
# Note the process ID in the `Id` column
docker-signal -pid=<id>
```

Los archivos de salida se encontrarán en el directorio raíz de datos en el que se esté ejecutando Docker. El directorio predeterminado es `C:\ProgramData\Docker`. El directorio real puede confirmarse ejecutando `docker info -f "{{.DockerRootDir}}"`.

Los archivos serán `goroutine-stacks-<timestamp>.log` y `daemon-data-<timestamp>.log`.

Ten en cuenta que `daemon-data*.log` puede contener información personal y por lo general solo debe compartirse con personal de soporte técnico de confianza. `goroutine-stacks*.log` no contiene información personal.


### Servicio de contenedor de host
El motor de Docker depende de un servicio de contenedor de host específico de Windows. Tiene registros independientes: 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

Son visibles en el Visor de eventos y también pueden consultarse con PowerShell.

Por ejemplo:
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```

#### Capturar registros de análisis y depuración de HCS

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

#### Capturar el rastreo detallado de HCS.

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