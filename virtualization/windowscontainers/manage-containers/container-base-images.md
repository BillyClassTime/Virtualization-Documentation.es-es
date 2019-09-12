---
title: Historial de imágenes base de contenedor de Windows
description: Una lista de imágenes del contenedor de Windows publicada con los hashes de capa SHA256
keywords: docker, contenedores, hashes
author: patricklang
ms.date: 01/12/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: b2f2d6418fdda2ad0aa0b81c05efad6b99f74375
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107909"
---
# <a name="container-base-images"></a>Imágenes base del contenedor

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

## <a name="base-image-differences"></a>Diferencias de las imágenes básicas

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
