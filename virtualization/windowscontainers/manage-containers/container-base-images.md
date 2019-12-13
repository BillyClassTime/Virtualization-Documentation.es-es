---
title: Imágenes base del contenedor de Windows
description: Información general de las imágenes base del contenedor de Windows y cuándo utilizarlas.
keywords: docker, contenedores, hashes
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 2a69fbace51589cce08476bd68fdb5c34a7907e6
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909785"
---
# <a name="container-base-images"></a>Imágenes base de contenedor

Windows ofrece cuatro imágenes base de contenedor desde las que los usuarios pueden compilar. Cada imagen base es un tipo diferente del sistema operativo Windows, tiene una superficie de disco diferente y lleva una cantidad diferente de la API de Windows establecida.

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
                        <p>Admite aplicaciones tradicionales de .NET Framework.</p>
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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">nano Server</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Creado para aplicaciones de .NET Core.</p>
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
                        <p>Diseñado específicamente para las aplicaciones de IoT.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>Detección de imágenes

Todas las imágenes base del contenedor de Windows se pueden detectar a través de [Docker Hub](https://hub.docker.com/_/microsoft-windows-base-os-images). Las imágenes base del contenedor de Windows se sirven desde [MCR.Microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/), Microsoft Container Registry (MCR). Este es el motivo por el que los comandos de extracción para las imágenes base del contenedor de Windows tienen el aspecto siguiente:

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

MCR no tiene su propia experiencia de catálogo y está pensada para admitir catálogos existentes como Docker Hub. Gracias a la superficie global de Azure y junto con Azure CDN, el MCR ofrece una experiencia de extracción de imágenes que es coherente y rápida. Los clientes de Azure que ejecutan sus cargas de trabajo en Azure, se benefician de las mejoras de rendimiento en la red, así como una estrecha integración con MCR (el origen de las imágenes de contenedor de Microsoft), Azure Marketplace y el creciente número de servicios de Azure que ofrecen contenedores como formato de paquete de implementación.

## <a name="choosing-a-base-image"></a>Elección de una imagen base

¿Cómo elegir la imagen base adecuada en la que se basará? Para la mayoría de los usuarios, `Windows Server Core` y `Nanoserver` serán la imagen más adecuada para su uso.

### <a name="guidelines"></a>Instrucciones

 Si bien tiene la posibilidad de dirigirse a la imagen que quiera, estas son algunas directrices que le ayudarán a dirigir su elección:

- **¿La aplicación requiere la versión completa de .NET Framework?** Si la respuesta a esta pregunta es sí, debe tener como destino `Windows Server Core`.
- **¿Está compilando una aplicación de Windows basada en .NET Core?** Si la respuesta a esta pregunta es sí, debe tener como destino `Nanoserver`.
- **¿Va a crear una aplicación de IoT?** Si la respuesta a esta pregunta es sí, debe tener como destino `IoT Core`.
- **¿Falta la imagen de contenedor de Windows Server Core una dependencia que su aplicación necesita?** Si la respuesta a esta pregunta es afirmativa, debe intentar tener como destino `Windows`. Esta imagen es mucho más grande que las otras imágenes base, pero incluye muchas de las bibliotecas de Windows principales (como la biblioteca GDI).
- **¿Es un usuario Insider de Windows?** En caso afirmativo, considere la posibilidad de usar la versión Insider de las imágenes. Vea "imágenes básicas para Windows Insider" a continuación.

> [!TIP]
> Muchos usuarios de Windows quieren incluir en un contenedor las aplicaciones que tienen una dependencia en .NET. Además de las cuatro imágenes base descritas aquí, Microsoft publica varias imágenes de contenedor de Windows que están preconfiguradas con los marcos de Microsoft más populares, como la imagen de [.NET Framework](https://hub.docker.com/_/microsoft-dotnet-framework) y la imagen de [ASP .net](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/) .

### <a name="base-images-for-windows-insiders"></a>Imágenes base para Windows Insider

Microsoft proporciona versiones "Insider" de cada imagen base del contenedor. Estas imágenes de contenedor de Insider llevan el desarrollo de características más reciente y mejor en nuestras imágenes de contenedor. Cuando se ejecuta un host que es una versión de Windows Insider (ya sea Windows Insider o Windows Server Insider), es preferible usar estas imágenes. Las imágenes de Insider están disponibles en Docker Hub:

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

Lea [usar contenedores con el programa Windows Insider](../deploy-containers/insider-overview.md) para obtener más información.

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core frente a nanoservidor

`Windows Server Core` y `Nanoserver` son las imágenes base más comunes de destino. La diferencia clave entre estas imágenes es que nanoserver tiene una superficie de API significativamente menor. PowerShell, WMI y la pila de servicio de Windows no están presentes en la imagen de nanoserver.

Se compiló nanoserver para proporcionar una superficie de API suficiente para ejecutar aplicaciones que tienen una dependencia en .NET Core u otros marcos de código abierto modernos. Como contrapartida a la superficie de APi más pequeña, la imagen de nanoserver tiene una superficie de disco considerablemente menor que el resto de las imágenes base de Windows. Ten en cuenta que siempre se pueden agregar capas sobre Nano Server según estimes oportuno. Para ver un ejemplo de esto, echa un vistazo al [Dockerfile de .NET Core Nano Server](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).
