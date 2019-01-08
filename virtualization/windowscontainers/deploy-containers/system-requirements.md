---
title: Requisitos de los contenedores de Windows
description: Requisitos de los contenedores de Windows.
keywords: metadatos, contenedores
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 478305ff2298a0392935f9857febc445c1199b83
ms.sourcegitcommit: 5300274fd7b88c6cf5e37b2f4c02779efaa3a613
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 01/08/2019
ms.locfileid: "8996049"
---
# <a name="windows-container-requirements"></a>Requisitos de los contenedores de Windows

En esta guía se enumeran los requisitos de un host de contenedor de Windows.

## <a name="os-requirements"></a>Requisitos de sistema operativo

- La característica de contenedor de Windows solo está disponible en Windows Server 2016 (Core y con experiencia de escritorio), Windows 10 Professional y Enterprise (Anniversary Edition) y versiones posteriores.
- El rol de Hyper-V debe instalarse antes de ejecutar los contenedores de Hyper-V.
- Los hosts de contenedor de Windows Server deben tener Windows instalado en c:\. Esta restricción no se aplica si solo se van a implementar contenedores de Hyper-V.

## <a name="virtualized-container-hosts"></a>Hosts de contenedor virtualizados

Si un host de contenedor de Windows se va a ejecutar desde una máquina virtual de Hyper-V y también van a hospedar contenedores de Hyper-V, será necesario habilitar la virtualización anidada. La virtualización anidada consta de los siguientes requisitos:

- Al menos 4 GB de RAM disponibles para el host de Hyper-V virtualizado.
- Windows Server 2019, versión 1803, Windows Server versión 1709, Windows Server 2016 o Windows 10 en el sistema host y Windows Server (Full Core) en la máquina virtual de Windows Server.
- Un procesador con Intel VT-x (actualmente, esta característica solo está disponible para los procesadores Intel).
- La máquina virtual del host de contenedor también necesitará al menos dos procesadores virtuales.

## <a name="supported-base-images"></a>Imágenes base compatibles

Los contenedores de Windows se ofrecen con cuatro imágenes base de contenedor: Windows Server Core, Nano Server, Windows y IoT Core. Tenga en cuenta que no todas las configuraciones son compatibles con ambas imágenes del sistema operativo. En esta tabla se detallan las configuraciones compatibles.

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
<td><center>Windows Server 2016 / 2019 (Standard o Datacenter)</center></td>
<td><center>Server Core, Nano Server, Windows</center></td>
<td><center>Server Core, Nano Server, Windows</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core, Nano Server, Windows</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro / Enterprise</center></td>
<td><center>No disponible</center></td>
<td><center>Server Core, Nano Server, Windows</center></td>
</tr>
<tr valign="top">
<td><center>IoT Core</center></td>
<td><center>IoT Core</center></td>
<td><center>No disponible</center></td>
</tr>
</tbody>
</table>

> [!Warning]  
> <span id="warn-1">A partir de WindowsServer, versión1709, NanoServer ya no está disponible como host de contenedor.</span>


### <a name="memory-requirements"></a>Requisitos de memoria
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


### <a name="base-image-differences"></a>Diferencias de la imagen base

¿Cómo se elige la imagen base de derecha a desarrollarla? Aunque tienes la libertad de crear lo que quieras, estas son las directrices generales de cada imagen:

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): si la aplicación necesita completo de .NET framework, se trata de la imagen mejor usar.
- [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): para las aplicaciones que requieren solo .NET Core, Nano Server proporcionará una mucho más finos imagen.
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): es posible depende de la aplicación en un componente o .dll que falta en Server Core o imágenes de Nano Server, como las bibliotecas de GDI. Esta imagen conlleva el conjunto completo de dependencia de Windows.
- [IoT Core](https://hub.docker.com/_/microsoft-windows-iotcore): esta imagen está diseñado específicamente para [aplicaciones de IoT](https://developer.microsoft.com/en-us/windows/iot). Debes usar esta imagen de contenedor cuando el destino es un host de IoT Core.

Para la mayoría de los usuarios, Windows Server Core o Nano Server será la imagen más adecuada para usar. A continuación son algunas cosas a tener en cuenta si tienes pensado crear la aplicación sobre Nano Server:

- Se quitó la pila de servicio
- No se incluye .NETCore (aunque puedes usar la [imagen de .NETCoreNanoServer](https://hub.docker.com/r/microsoft/dotnet/))
- Se quitó PowerShell
- Se quitó WMI
- A partir de Windows Server versión 1709, las aplicaciones se ejecutan en un contexto de usuario, por lo que se producirán errores en los comandos que requieran privilegios de administrador. Puedes especificar la cuenta de administrador de contenedor de la cuenta mediante la marca --user (es decir, docker run --user ContainerAdministrator). Sin embargo, en el futuro nuestra intención es quitar por completo las cuentas de administrador de NanoServer.

Estas son las principales diferencias, es decir, no es una lista exhaustiva. Hay otros componentes que no se han mencionado que tampoco se encuentran. Ten en cuenta que siempre se pueden agregar capas sobre NanoServer según estimes oportuno. Para ver un ejemplo de esto, echa un vistazo al [Dockerfile de .NETCoreNanoServer](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).

