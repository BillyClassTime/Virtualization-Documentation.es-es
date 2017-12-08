---
title: Requisitos de los contenedores de Windows
description: Requisitos de los contenedores de Windows.
keywords: metadatos, contenedores
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 6ae690ff6592198bc16cbaf60489d3ed5aceeeb0
ms.sourcegitcommit: 64f5f8d838f72ea8e0e66a72eeb4ab78d982b715
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/22/2017
---
# <a name="windows-container-requirements"></a>Requisitos de los contenedores de Windows

En esta guía se enumeran los requisitos de un host de contenedor de Windows.

## <a name="os-requirements"></a>Requisitos de sistema operativo

- La característica de contenedores de Windows solo está disponible en la compilación 1709 de Windows Server 2016 (Core y con experiencia de escritorio) y en Windows 10 Professional y Enterprise (Anniversary Edition).
- El rol de Hyper-V debe instalarse antes de ejecutar los contenedores de Hyper-V.
- Los hosts de contenedor de Windows Server deben tener Windows instalado en c:\. Esta restricción no se aplica si solo se van a implementar contenedores de Hyper-V.

## <a name="virtualized-container-hosts"></a>Hosts de contenedor virtualizados

Si un host de contenedor de Windows se va a ejecutar desde una máquina virtual de Hyper-V y también van a hospedar contenedores de Hyper-V, será necesario habilitar la virtualización anidada. La virtualización anidada consta de los siguientes requisitos:

- Al menos 4 GB de RAM disponibles para el host de Hyper-V virtualizado.
- Windows Server 2016 (compilación 1709), Windows Server 2016 o Windows 10 en el sistema host, y Windows Server (Full o Core) en la máquina virtual.
- Un procesador con Intel VT-x (actualmente, esta característica solo está disponible para los procesadores Intel).
- La máquina virtual del host de contenedor también necesitará al menos dos procesadores virtuales.

## <a name="supported-base-images"></a>Imágenes base compatibles

Los contenedores de Windows se ofrecen con dos imágenes base de contenedor, Windows Server Core y Nano Server. Tenga en cuenta que no todas las configuraciones son compatibles con ambas imágenes del sistema operativo. En esta tabla se detallan las configuraciones compatibles.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Sistema operativo host</center></th>
<th><center>Contenedor de Windows Server</center></th>
<th><center>Contenedor de Hyper-V</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016 (Standard o Datacenter)</center></td>
<td><center>Server Core/Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>NanoServer*</center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro / Enterprise</center></td>
<td><center>No disponible</center></td>
<td><center>ServerCore/NanoServer</center></td>
</tr>
</tbody>
</table>
* A partir de WindowsServer, versión1709, NanoServer ya no está disponible como host de contenedor.

### <a name="memory-requirments"></a>Requisitos de memoria
Se pueden configurar restricciones en la memoria disponible a través de los contenedores [controles de recursos](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls) o mediante la sobrecarga de un host de contenedor.  La cantidad mínima de la memoria necesaria para iniciar un contenedor y ejecutar comandos básicos (ipconfig, dir, etc.) se enumera a continuación.  Ten en cuenta que estos valores no tienen en cuenta el uso compartido de recursos entre los contenedores o los requisitos de la aplicación que se ejecuta en el contenedor.

#### <a name="windows-server-2016"></a>WindowsServer2016
| Imagen base  | Contenedor de WindowsServer | Aislamiento de Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| NanoServer | 40MB                     | Archivo de paginación 130MB + 1GB |
| ServerCore | 50MB                     | Archivo de paginación 325MB + 1GB |

#### <a name="windows-server-version-1709"></a>WindowsServer, versión1709
| Imagen base  | Contenedor de WindowsServer | Aislamiento de Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| NanoServer | 30MB                     | Archivo de paginación 110MB + 1GB |
| ServerCore | 45MB                     | Archivo de paginación 360MB + 1GB |


### <a name="nano-server-vs-windows-server-core"></a>Comparación de NanoServer y WindowsServerCore

¿Cómo se elige entre WindowsServerCore y NanoServer? Aunque tienes la libertad de crear lo que quieras, si crees que tu aplicación necesita compatibilidad completa con .NETFramework, debes usar [WindowsServerCore](https://hub.docker.com/r/microsoft/windowsservercore/). Por otra parte, si la aplicación se crea para la nube y usa .NETCore, debes usar [NanoServer](https://hub.docker.com/r/microsoft/nanoserver/). Esto se debe a que NanoServer se creó con la intención de tener una huella del menor tamaño posible, por lo tanto, se quitaron varias bibliotecas que no eran esenciales. Es importante tener esto en cuenta si tienes pensado crear la aplicación sobre NanoServer:

- Se quitó la pila de servicio
- No se incluye .NETCore (aunque puedes usar la [imagen de .NETCoreNanoServer](https://hub.docker.com/r/microsoft/dotnet/))
- Se quitó PowerShell
- Se quitó WMI

Estas son las principales diferencias, es decir, no es una lista exhaustiva. Hay otros componentes que no se han mencionado que tampoco se encuentran. Ten en cuenta que siempre se pueden agregar capas sobre NanoServer según estimes oportuno. Para ver un ejemplo de esto, echa un vistazo al [Dockerfile de .NET Core Nano Server](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile).

## <a name="matching-container-host-version-with-container-image-versions"></a>Coincidencia de la versión del host de contenedor con las versiones de la imagen de contenedor
### <a name="windows-server-containers"></a>Contenedores de Windows Server
Dado que los contenedores de Windows Server y el host subyacente comparten un solo kernel, la imagen base del contenedor debe coincidir con la del host.  Si las versiones son diferentes, el contenedor puede iniciarse, pero no se garantiza una funcionalidad completa. Por lo tanto, no se admiten las versiones que no coinciden.  El sistema operativo Windows tiene cuatro niveles de control de versiones: Principal, Secundario, Compilación y Revisión (por ejemplo, 10.0.14393.0). El número de compilación solo cambia cuando se publican nuevas versiones del sistema operativo. El número de revisión se actualiza cuando se aplican las actualizaciones de Windows. Si el número de compilación es diferente, se bloquea el inicio de los contenedores de Windows Server. Por ejemplo, 10.0.14300.1030 (Technical Preview 5) y 10.0.14393 (Windows Server 2016 RTM). Si el número de compilación coincide pero el número de revisión es diferente, no se bloquea el inicio: por ejemplo 10.0.14393 (Windows Server 2016 RTM) y 10.0.14393.206 (Windows Server 2016 GA). Aunque técnicamente no están bloqueados, esta configuración podría no funcionar correctamente en todas las circunstancias y, por lo tanto, no se admite en entornos de producción. 

Para comprobar qué versión tiene instalada un host de Windows, consulte HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion.  Para comprobar qué versión usa la imagen base, revise las etiquetas de Docker Hub o la tabla hash de la imagen proporcionada en la descripción de la imagen.  En la página [Historial de actualizaciones de Windows 10](https://support.microsoft.com/en-us/help/12387/windows-10-update-history) se muestra cuándo se lanzó cada compilación y revisión.

En este ejemplo, 14393 es el número de compilación principal y 321 es la revisión.
```
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

### <a name="hyper-v-isolation-for-containers"></a>Aislamiento de Hyper-V para contenedores
Los contenedores de Windows pueden ejecutarse con o sin aislamiento de Hyper-V.  El aislamiento de Hyper-V crea un límite seguro alrededor del contenedor con una VM optimizada.  A diferencia de los contenedores estándar de Windows, que comparten el kernel entre los contenedores y el host, cada contenedor de Hyper-V aislado tiene su propia instancia del kernel de Windows.  Por este motivo, puede tener diferentes versiones del sistema operativo en el host y la imagen del contenedor (consulta la matriz de compatibilidad que viene a continuación).  

Para ejecutar un contenedor con aislamiento de Hyper-V, solo tienes que agregar la etiqueta "--isolation=hyper-v" al comando de ejecución de docker.

### <a name="compatibility-matrix"></a>Matriz de compatibilidad
Las compilaciones de Windows Server posteriores a 2016 GA (10.0.14393.206) pueden ejecutar las imágenes de Windows Server 2016 GA, tanto de Windows Server Core como de Nano Server, en una configuración compatible, independientemente del número de revisión.    

Es importante comprender que, para tener la funcionalidad completa, la confiabilidad y las garantías de seguridad que proporcionan las actualizaciones de Windows, es necesario mantener las versiones más recientes en todos los sistemas.  
