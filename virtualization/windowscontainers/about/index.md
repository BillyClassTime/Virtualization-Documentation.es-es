---
title: About Windows Containers
description: Learn about Windows containers.
keywords: docker, containers
author: taylorb-microsoft
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 2be7a06c7b7b154e392c30981cdf954d2d1b796e
ms.sourcegitcommit: 8e193d8c274a549aef497f16dcdb00d7855e9fa7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2017
---
# Contenedores de Windows

## Qué son los contenedores

Los contenedores son una forma de encapsular una aplicación en su propio espacio aislado. Una vez que la aplicación esté en su contenedor, no tendrá conocimiento de otras aplicaciones o procesos que existan fuera de su espacio. Todo lo que necesita la aplicación para ejecutarse correctamente también se encuentra dentro de este contenedor.  Independientemente de la ubicación a la que se pueda mover el espacio, la aplicación siempre se encontrará satisfecha porque incluye todo lo que necesita para ejecutarse.

Imagina una cocina. Empaquetamos todos los electrodomésticos y mobiliario, cacerolas y sartenes, jabón para vajilla y toallas de mano. Este es nuestro contenedor

<center style="margin: 25px">![](media/box1.png)</center>

Ahora podemos tomar este contenedor y colocarlo en el apartamento que queramos, y tendremos la misma cocina. Todo lo que tenemos que hacer es conectar la electricidad y el agua y estamos listos para empezar a cocinar (porque tenemos todos los electrodomésticos que necesitamos)

<center style="margin: 25px">![](media/apartment.png)</center>

De forma muy similar, los contenedores se parecen a esta cocina. Es posible que haya diferentes tipos de habitaciones, así como un gran número de los mismos tipos de habitaciones. Lo importante es que los contenedores incluyan todo lo que necesitan.

Mira una breve introducción: [Windows-based containers: Modern app development with enterprise-grade control](https://youtu.be/Ryx3o0rD5lY) (Contenedores basados en Windows: desarrollo de aplicaciones modernas con control de nivel empresarial).

## Conceptos básicos de los contenedores

Los contenedores son un entorno de ejecución portátil, aislado y controlado por recursos que se ejecuta en una máquina host o virtual. Una aplicación o proceso que se ejecuta en un contenedor incluye todos los archivos de configuración y dependencias necesarios, por tanto, tiene la sensación de que no hay otros procesos en ejecución fuera del contenedor.

El host del contenedor aprovisiona un conjunto de recursos para el contenedor y este solo usará estos recursos. Por lo que respecta al contenedor, no existen otros recursos aparte de los que recibe y, por tanto, no puede tener acceso a recursos que un contenedor vecino puede haber aprovisionado.

Los siguientes conceptos clave te resultarán útiles cuando empieces a crear y trabajar con contenedores de Windows.

**Container Host:** Physical or Virtual computer system configured with the Windows Container feature. El host de contenedor ejecutará uno o varios contenedores de Windows.

**Imagen de contenedor:** a medida que se realicen modificaciones en un registro o un sistema de archivos de contenedores, como durante la instalación de software, se capturan en un espacio aislado. En muchos casos, querrás capturar este estado de forma que se puedan crear nuevos contenedores que heredan estos cambios. That’s what an image is – once the container has stopped you can either discard that sandbox or you can convert it into a new container image. For example, let’s imagine that you have deployed a container from the Windows Server Core OS image. You then install MySQL into this container. Creating a new image from this container would act as a deployable version of the container. This image would only contain the changes made (MySQL), however would work as a layer on top of the Container OS Image.

**Sandbox:** Once a container has been started, all write actions such as file system modifications, registry modifications or software installations are captured in this ‘sandbox’ layer.

**Container OS Image:** Containers are deployed from images. The container OS image is the first layer in potentially many image layers that make up a container. Esta imagen ofrece el entorno del sistema operativo. Una imagen de sistema operativo del contenedor es inmutable. Es decir, no se puede modificar.

**Repositorio de contenedor:** cada vez que se crea una imagen de contenedor, esta y sus dependencias se almacenan en un repositorio local. Estas imágenes se pueden reutilizar muchas veces en el host de contenedor. Las imágenes de contenedor también pueden almacenarse en un registro público o privado, como DockerHub, de forma que se puedan usar en varios host de contenedor diferentes.

<center>![](media/containerfund.png)</center>

Para alguien familiarizado con las máquinas virtuales, puede parecer que los contenedores son increíblemente similares. A container runs an operating system, has a file system and can be accessed over a network just as if it was a physical or virtual computer system. Dicho esto, la tecnología y los conceptos relacionados con los contenedores son muy diferentes de las máquinas virtuales.

Mark Russinovich, experto de MicrosoftAzure, tiene [una estupenda entrada de blog](https://azure.microsoft.com/en-us/blog/containers-docker-windows-and-trends/) en la que explica las diferencias.

## Tipos de contenedores de Windows

Windows Containers include two different container types, or runtimes.

**Windows Server Containers** – provide application isolation through process and namespace isolation technology. A Windows Server Container shares a kernel with the container host and all containers running on the host. Estos contenedores no proporcionan un límite de seguridad hostil y no deben usarse para aislar un código que no sea de confianza. Dado el espacio de kernel compartido, estos contenedores requieren la misma configuración y versión de kernel.

**Aislamiento de Hyper-V**: amplía el aislamiento que ofrecen los contenedores de WindowsServer mediante la ejecución de cada contenedor en una máquina virtual altamente optimizada. In this configuration, the kernel of the container host is not shared with other containers on the same host. Estos contenedores se han diseñado para el hospedaje multiinquilino hostil con las mismas garantías de seguridad de una máquina virtual. Dado que estos contenedores no comparten el kernel con el host u otros contenedores del equipo host, pueden ejecutar kernels con distintas versiones y configuraciones (dentro de las versiones compatibles): por ejemplo, todos los contenedores de Windows en Windows10 usan el aislamiento de Hyper-V para poder usar la versión y configuración del kernel de WindowsServer.

La ejecución de un contenedor en Windows con o sin aislamiento de Hyper-V es una decisión que ha de tomarse en el tiempo de ejecución. Puedes optar por crear inicialmente el contenedor con aislamiento de Hyper-V y más adelante, en el tiempo de ejecución, seleccionarlo para ejecutarlo en lugar de un contenedor de WindowsServer.

## ¿Qué es Docker?

A medida que lees sobre los contenedores, inevitablemente aparecerá Docker en tu lectura. Docker es el contenedor mediante el que se empaquetan y entregan las imágenes de contenedor. Este proceso automatizado genera imágenes (plantillas) que pueden ejecutarse en cualquier lugar (en entornos locales, en la nube o en un equipo personal) como contenedor.

<center>![](media/docker.png)</center>

Al igual que cualquier otro contenedor, un contenedor de WindowsServer puede administrarse con [Docker](https://www.docker.com).

## Contenedores para desarrolladores ##

From a developer’s desktop to a testing machine to a set of production machines, a Docker image can be created that will deploy identically across any environment in seconds. This story has created a massive and growing ecosystem of applications packaged in Docker containers, with DockerHub, the public containerized-application registry that Docker maintains, currently publishing more than 180,000 applications in the public community repository.

When you containerize an app, only the app and the components needed to run the app are combined into an "image". Containers are then created from this image as you need them. You can also use an image as a baseline to create another image, making image creation even faster. Multiple containers can share the same image, which means containers start very quickly and use fewer resources. For example, you can use containers to spin up light-weight and portable app components – or ‘micro-services’ – for distributed apps and quickly scale each service separately.

Because the container has everything it needs to run your application, they are very portable and can run on any machine that is running Windows Server 2016. You can create and test containers locally, then deploy that same container image to your company's private cloud, public cloud or service provider. The natural agility of Containers supports modern app development patterns in large scale, virtualized and cloud environments.

With containers, developers can build an app in any language. These apps are completely portable and can run anywhere - laptop, desktop, server, private cloud, public cloud or service provider - without any code changes.  

Containers helps developers build and ship higher-quality applications, faster.

## Containers for IT Professionals ##

IT Professionals can use containers to provide standardized environments for their development, QA, and production teams. They no longer have to worry about complex installation and configuration steps. By using containers, systems administrators abstract away differences in OS installations and underlying infrastructure.

Containers help admins create an infrastructure that is simpler to update and maintain.

## Video Overview

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## Prueba los contenedores de WindowsServer

¿Estás listo para comenzar a sacar partido de la increíble potencia de los contenedores? Consulta la siguiente documentación para comenzar a implementar tu primer contenedor: <br/>
Para los usuarios de WindowsServer, dirígete aquí: [Inicio rápido de WindowsServer](../quick-start/quick-start-windows-server.md) <br/>
Para los usuarios de Windows10, dirígete aquí: [Inicio rápido de Windows10](../quick-start/quick-start-windows-10.md)

