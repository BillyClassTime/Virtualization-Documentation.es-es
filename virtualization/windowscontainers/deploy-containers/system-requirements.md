---
title: Requisitos de los contenedores de Windows
description: Requisitos de los contenedores de Windows.
keywords: metadatos, contenedores
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 88d094202c49cf725e9d608a0810e7d9f8a1e271
ms.sourcegitcommit: 7fc79235cbee052e07366b8a6aa7e035a5e3434f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/13/2018
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
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
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

> [!Warning]  
> <span id="warn-1">A partir de WindowsServer, versión1709, NanoServer ya no está disponible como host de contenedor.</span>


### <a name="memory-requirments"></a>Requisitos de memoria
Se pueden configurar restricciones en la memoria disponible a través de los contenedores [controles de recursos](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls) o mediante la sobrecarga de un host de contenedor.  La cantidad mínima de la memoria necesaria para iniciar un contenedor y ejecutar comandos básicos (ipconfig, dir, etc.) se enumera a continuación.  __Ten en cuenta que estos valores no tienen en cuenta el uso compartido de recursos entre los contenedores o los requisitos de la aplicación que se ejecuta en el contenedor.  Por ejemplo, un host con 512MB de memoria libre puede ejecutar varios contenedores de ServerCore en el aislamiento de Hyper-V porque estos contenedores comparten recursos.__

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

Estas son las principales diferencias, es decir, no es una lista exhaustiva. Hay otros componentes que no se han mencionado que tampoco se encuentran. Ten en cuenta que siempre se pueden agregar capas sobre NanoServer según estimes oportuno. Para ver un ejemplo de esto, echa un vistazo al [Dockerfile de .NETCoreNanoServer](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile).

