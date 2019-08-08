---
title: Requisitos de los contenedores de Windows
description: Requisitos de los contenedores de Windows.
keywords: metadatos, contenedores
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 5fc9b5c9135e87a0d3246952c35c9755e9ad209e
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998472"
---
# <a name="windows-container-requirements"></a>Requisitos de los contenedores de Windows

En esta guía se enumeran los requisitos para un host contenedor de Windows.

## <a name="os-requirements"></a>Requisitos del sistema operativo

- La característica contenedor de Windows solo está disponible en Windows Server 2016 (Core y con experiencia de escritorio), Windows 10 Professional y Enterprise (edición de aniversario) y versiones posteriores.
- El rol de Hyper-V debe instalarse antes de poder ejecutar el aislamiento de Hyper-V
- Los hosts de contenedor de Windows Server deben tener Windows instalado en c:\. Esta restricción no se aplica si solo se implementarán los contenedores aislados de Hyper-V.

## <a name="virtualized-container-hosts"></a>Hosts de contenedor virtualizados

Si se va a ejecutar un host contenedor de Windows desde una máquina virtual de Hyper-V y también va a hospedar el aislamiento Hyper-V, se debe habilitar la virtualización anidada. La virtualización anidada consta de los siguientes requisitos:

- Al menos 4 GB de RAM disponibles para el host de Hyper-V virtualizado.
- Windows Server 2019, Windows Server versión 1803, Windows Server versión 1709, Windows Server 2016 o Windows 10 en el sistema host, y Windows Server (completo, núcleo) en la máquina virtual.
- Un procesador con Intel VT-x (actualmente, esta característica solo está disponible para los procesadores Intel).
- La VM host de contenedor también necesitará al menos dos procesadores virtuales.

## <a name="supported-base-images"></a>Imágenes de base compatibles

Los contenedores de Windows se ofrecen con cuatro imágenes base de contenedor: Windows Server Core, nano Server, Windows y IoT Core. Tenga en cuenta que no todas las configuraciones son compatibles con ambas imágenes del sistema operativo. En esta tabla se detallan las configuraciones compatibles.

|Sistema operativo de host|Contenedor de Windows|Aislamiento de Hyper-V|
|---------------------|-----------------|-----------------|
|Windows Server 2016 o Windows Server 2019 (Standard o Datacenter)|Server Core, nano Server, Windows|Server Core, nano Server, Windows|
|Nano Server|Nano Server|Server Core, nano Server, Windows|
|Windows 10 Pro o Windows 10 Enterprise|No disponible|Server Core, nano Server, Windows|
|IoT Core|IoT Core|No disponible|

> [!WARNING]  
> A partir de la versión 1709 de Windows Server, nano Server ya no está disponible como host de contenedor.

### <a name="memory-requirements"></a>Requisitos de memoria

Se pueden configurar restricciones en la memoria disponible a través de los contenedores [controles de recursos](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls) o mediante la sobrecarga de un host de contenedor.  A continuación se muestra la cantidad mínima de memoria necesaria para iniciar un contenedor y ejecutar comandos básicos (ipconfig, dir, etc.).

>[!NOTE]
>Estos valores no tienen en cuenta el uso compartido de recursos entre contenedores o requisitos de la aplicación que se ejecuta en el contenedor.  Por ejemplo, un host con 512MB de memoria libre puede ejecutar varios contenedores de ServerCore en el aislamiento de Hyper-V porque estos contenedores comparten recursos.

#### <a name="windows-server-2016"></a>WindowsServer2016

| Imagen base  | Contenedor de Windows Server | Aislamiento de Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | archivo de paginación de 130 MB + 1 GB |
| ServerCore | 50 MB                     | archivo de paginación de 325 MB + 1 GB |

#### <a name="windows-server-version-1709"></a>WindowsServer, versión1709

| Imagen base  | Contenedor de Windows Server | Aislamiento de Hyper-V    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | archivo de paginación de 110 MB + 1 GB |
| ServerCore | 45 MB                     | archivo de paginación de 360 MB + 1 GB |

### <a name="base-image-differences"></a>Diferencias de las imágenes básicas

¿Cómo elige elegir la imagen base adecuada para crearla? Aunque es gratis para compilar con cualquier cosa que desee, estas son las directrices generales para cada imagen:

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): Si su aplicación necesita .NET Framework completo, esta es la mejor imagen para usar.
- [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): para las aplicaciones que solo requieren .net Core, nano Server proporcionará una imagen mucho más representativa.
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): es posible que la aplicación dependa de un componente o archivo. dll que falta en las imágenes Server Core o nano Server, como las bibliotecas GDI. Esta imagen contiene el conjunto de dependencias completo de Windows.
- [IOT Core](https://hub.docker.com/_/microsoft-windows-iotcore): esta imagen está diseñada para aplicaciones de [IOT](https://developer.microsoft.com/windows/iot). Debe usar esta imagen de contenedor cuando dirija un host de IoT Core.

Para la mayoría de los usuarios, Windows Server Core o nano Server será la imagen más adecuada para usar. A continuación se indican algunas cosas que debe tener en cuenta al crear una parte superior de nano Server:

- Se quitó la pila de servicio
- No se incluye .NETCore (aunque puedes usar la [imagen de .NETCoreNanoServer](https://hub.docker.com/r/microsoft/dotnet/))
- Se quitó PowerShell
- Se quitó WMI
- A partir de Windows Server versión 1709, las aplicaciones se ejecutan en un contexto de usuario, por lo que se producirán errores en los comandos que requieran privilegios de administrador. Puede especificar la cuenta de administrador de contenedores a través de la bandera--usuario (como la ejecución de Docker--ContainerAdministrator de usuario) sin embargo, en el futuro pretendemos quitar completamente las cuentas de administrador de nanoserver.

Estas son las principales diferencias, es decir, no es una lista exhaustiva. Hay otros componentes que no se han mencionado que tampoco se encuentran. Ten en cuenta que siempre se pueden agregar capas sobre NanoServer según estimes oportuno. Para ver un ejemplo de esto, echa un vistazo al [Dockerfile de .NETCoreNanoServer](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).