---
title: Requisitos de los contenedores de Windows
description: Requisitos de los contenedores de Windows.
keywords: metadatos, contenedores
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: ecc11468bbd5aad2638da3c4f733e4d5068f0056
ms.sourcegitcommit: 77a6195318732fa16e7d5be727bdb88f52f6db46
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/20/2017
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
Dado que los contenedores de Windows Server y el host subyacente comparten un solo kernel, la imagen base del contenedor debe coincidir con la del host.  Si las versiones son diferentes, el contenedor puede iniciarse, pero no se garantiza una funcionalidad completa. El sistema operativo Windows tiene cuatro niveles de control de versiones: Principal, Secundario, Compilación y Revisión (por ejemplo, 10.0.14393.103). El número de compilación (es decir, 14393) únicamente cambia cuando se publican nuevas versiones del sistema operativo, como por ejemplo, versión 1709, 1803, fall creators update, etc. El número de revisión (es decir, 103) se actualiza a medida que se aplican las actualizaciones de Windows.
#### <a name="build-number-new-release-of-windows"></a>Número de compilación (nueva versión de Windows)
Si el número de compilación entre el host del contenedor y la imagen del contenedor es diferente, se bloquea el inicio de los contenedores de WindowsServer, por ejemplo 10.0.14393.* (Windows Server 2016) y 10.0.16299.* (Windows Server versión 1709).  
#### <a name="revision-number-patching"></a>Número de revisión (aplicación de revisiones)
Si el número de revisión entre el host del contenedor y la imagen del contenedor es diferente, _no_ se bloquea el inicio de los contenedores de WindowsServer, por ejemplo, 10.0.14393.1914 (WindowsServer2016 con KB4051033) y 10.0.14393.1944 (WindowsServer2016 con KB4053579).  
En el caso de hosts/imágenes basadas en WindowsServer2016: la revisión de la imagen del contenedor debe coincidir con el host para estar en una configuración compatible.  A partir de WindowsServer versión 1709, esto ya no se aplica, y la imagen del contenedor y del host no necesita tener revisiones que coincidan.  Como siempre, se recomienda mantener los sistemas actualizados con las últimas revisiones y actualizaciones.
#### <a name="practical-application"></a>Aplicación práctica
Ejemplo 1: El host del contenedor ejecuta WindowsServer2016 con KB4041691.  Cualquier contenedor de WindowsServer implementado a este host debe basarse en las imágenes base del contenedor 10.0.14393.1770.  Si se aplica KB4053579 al host, las imágenes del contenedor deben actualizarse al mismo tiempo para seguir recibiendo soporte técnico.
Ejemplo 2: El host del contenedor ejecuta WindowsServer, versión 1709 con KB4043961.  Cualquier contenedor de WindowsServer implementado en este host debe basarse en una imagen base de contenedor de WindowsServer versión 1709 (10.0.16299), pero no necesita coincidir con el KB del host.  Si se aplica KB4054517 al host, las imágenes del contenedor no necesitan actualizarse, sin embargo, deben actualizarse para hacer frente a cualquier problema de seguridad.
#### <a name="querying-version"></a>Consultar versiones
Método 1: Incluido en la versión 1709, el comando ver y el símbolo del sistema cmd ahora deben devolver los detalles de revisión.
```
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125] 
```
Método 2: Consultar la siguiente clave de registro: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion. Por ejemplo:
```
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```
O bien
```
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

Para comprobar qué versión usa la imagen base, revise las etiquetas de Docker Hub o la tabla hash de la imagen proporcionada en la descripción de la imagen.  En la página [Historial de actualizaciones de Windows 10](https://support.microsoft.com/en-us/help/12387/windows-10-update-history) se muestra cuándo se lanzó cada compilación y revisión.

### <a name="hyper-v-isolation-for-containers"></a>Aislamiento de Hyper-V para contenedores
Los contenedores de Windows pueden ejecutarse con o sin aislamiento de Hyper-V.  El aislamiento de Hyper-V crea un límite seguro alrededor del contenedor con una VM optimizada.  A diferencia de los contenedores estándar de Windows, que comparten el kernel entre los contenedores y el host, cada contenedor de Hyper-V aislado tiene su propia instancia del kernel de Windows.  Por este motivo, puede tener diferentes versiones del sistema operativo en el host y la imagen del contenedor (consulta la matriz de compatibilidad que viene a continuación).  

Para ejecutar un contenedor con aislamiento de Hyper-V, solo tienes que agregar la etiqueta "--isolation=hyper-v" al comando de ejecución de docker.

### <a name="compatibility-matrix"></a>Matriz de compatibilidad
Las compilaciones de Windows Server posteriores a 2016 GA (10.0.14393.206) pueden ejecutar las imágenes de Windows Server 2016 GA, tanto de Windows Server Core como de Nano Server, en una configuración compatible, independientemente del número de revisión.
Un host de Windows Server, versión 1709 también puede ejecutar contenedores basados en WindowsServer2016, sin embargo, la inversión no es compatible.

Es importante comprender que, para tener la funcionalidad completa, la confiabilidad y las garantías de seguridad que proporcionan las actualizaciones de Windows, es necesario mantener las versiones más recientes en todos los sistemas.  
