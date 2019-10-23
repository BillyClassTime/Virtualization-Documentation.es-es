---
title: Imágenes base del contenedor de Windows
description: Información general sobre las imágenes base del contenedor de Windows y cuándo usarlas.
keywords: docker, contenedores, hashes
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 2a69fbace51589cce08476bd68fdb5c34a7907e6
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254275"
---
# <a name="container-base-images"></a>Imágenes base del contenedor

Windows ofrece cuatro imágenes base de contenedor de las que los usuarios pueden compilar. Cada imagen base es una forma diferente del sistema operativo Windows, tiene un espacio en disco diferente y lleva una cantidad diferente del conjunto de API de Windows.

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-servercore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows Server Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Es compatible con las aplicaciones de .NET Framework tradicionales.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-nanoserver" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Nano Server</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Creado para las aplicaciones básicas de .NET.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Proporciona el conjunto completo de API de Windows.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-iotcore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows IoT Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Diseñado especialmente para aplicaciones de IoT.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>Detección de imágenes

Todas las imágenes base del contenedor de Windows se pueden detectar mediante el [concentrador de acoplamiento](https://hub.docker.com/_/microsoft-windows-base-os-images). Las imágenes básicas del contenedor de Windows se atienden desde [MCR.Microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/), el registro del contenedor de Microsoft (MCR). Este es el motivo por el que los comandos de extracción de las imágenes base del contenedor de Windows tienen el siguiente aspecto:

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

La MCR no tiene su propia experiencia de catálogo y está pensada para admitir catálogos existentes, como el concentrador de acoplamiento. Gracias al tamaño global de Azure y junto con la CDN de Azure, el MCR ofrece una experiencia de extracción de imagen que es coherente y rápida. Los clientes de Azure que ejecutan sus cargas de trabajo en Azure, aprovechan las mejoras de rendimiento en la red, así como una estrecha integración con el MCR (el origen de las imágenes de contenedor de Microsoft), Azure Marketplace y el creciente número de servicios en Azure que ofrecen contenedores como formato de paquete de implementación.

## <a name="choosing-a-base-image"></a>Elegir una imagen base

¿Cómo se elige la imagen base adecuada para la creación? Para la mayoría de `Windows Server Core` los `Nanoserver` usuarios, y será la imagen más adecuada para usar.

### <a name="guidelines"></a>Instrucciones

 A medida que sea libre de dirigir la imagen que desee, estas son algunas pautas para ayudarle a elegir:

- **¿La aplicación requiere .NET Framework completo?** Si la respuesta a esta pregunta es sí, debe dirigirse `Windows Server Core`a.
- **¿Estás creando una aplicación de Windows basada en .NET Core?** Si la respuesta a esta pregunta es sí, debe dirigirse `Nanoserver`a.
- **¿Estás creando una aplicación de IoT?** Si la respuesta a esta pregunta es sí, debe dirigirse `IoT Core`a.
- **¿Falta una dependencia de la aplicación en el contenedor de Windows Server Core?** Si la respuesta a esta pregunta es afirmativa, debe intentar destinar `Windows`. Esta imagen es mucho más grande que las otras imágenes base, pero incluye muchas de las bibliotecas básicas de Windows (como la biblioteca GDI).
- **¿Eres Windows Insider?** En caso afirmativo, te recomendamos que uses la versión de Insider de las imágenes. Consulte "imágenes básicas para participantes de Windows Insider" a continuación.

> [!TIP]
> Muchos usuarios de Windows desean descontar las aplicaciones que tienen una dependencia en .NET. Además de las cuatro imágenes básicas que se describen aquí, Microsoft publica varias imágenes de contenedores de Windows que vienen preconfiguradas con los conocidos marcos de Microsoft, como la imagen de [.NET Framework](https://hub.docker.com/_/microsoft-dotnet-framework) y la imagen de [ASP .net](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/) .

### <a name="base-images-for-windows-insiders"></a>Imágenes básicas para participantes de Windows Insider

Microsoft proporciona las versiones "Insider" de cada imagen base del contenedor. Estas imágenes de contenedor de Insider llevan el mejor desarrollo de características de nuestras imágenes de contenedor. Si está ejecutando un host que es una versión Insider de Windows (ya sea Windows Insider o Windows Server Insider), es preferible usar estas imágenes. Las imágenes de Insider están disponibles en el hub del Dock:

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

Lectura [use contenedores con el programa Windows Insider](../deploy-containers/insider-overview.md) para obtener más información.

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core vs nanoserver

`Windows Server Core` y `Nanoserver` son las imágenes base más comunes para el destino. La diferencia clave entre estas imágenes es que nanoserver tiene una superficie de API significativamente más pequeña. PowerShell, WMI y la pila de mantenimiento de Windows no están presentes en la imagen de nanoservidor.

Nanoserver se creó para proporcionar la suficiente superficie de API para ejecutar aplicaciones que tengan una dependencia en .NET Core u otros marcos de trabajo de código abierto modernos. Para compensar la superficie de APi más pequeña, la imagen de nanoservidor tiene un espacio en disco significativamente más pequeño que el resto de las imágenes de la base de Windows. Ten en cuenta que siempre se pueden agregar capas sobre NanoServer según estimes oportuno. Para ver un ejemplo de esto, echa un vistazo al [Dockerfile de .NETCoreNanoServer](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).
