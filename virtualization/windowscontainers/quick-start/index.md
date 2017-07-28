---
title: "Inicio rápido de contenedores de Windows"
description: "Inicio rápido de contenedores de Windows."
keywords: docker, contenedores
author: enderb-ms
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-contianers
ms.service: windows-containers
ms.assetid: 4878f5d2-014f-4f3c-9933-97f03348a147
ms.openlocfilehash: fb97f1d0f533b28acfb711e52bd021b29212f66e
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/21/2017
---
# Inicio rápido de contenedores de Windows

El inicio rápido de contenedores de Windows presenta la terminología del producto y los contenedores, guía a través de ejemplos sencillos de implementación de contenedores y además proporciona referencias de temas más avanzados. Si no está familiarizado con los contenedores o los contenedores de Windows, cada paso de este inicio rápido le proporcionará experiencias prácticas con la tecnología.

## 1. Qué son los contenedores

Son un entorno operativo portátil, aislado y controlado por recursos.

Básicamente, un contenedor es un lugar aislado donde una aplicación puede ejecutarse sin afectar al resto del sistema y sin que el sistema le afecte a ella. Los contenedores son la siguiente evolución en el campo de la virtualización.

Si estuviera dentro de un contenedor, parecería que está dentro de una máquina virtual o un equipo físico recién instalados. Y, para [Docker](https://www.docker.com/), un contenedor de Windows se puede administrar de la misma forma que cualquier otro contenedor.

## 2. Tipos de contenedores de Windows

Los contenedores de Windows incluyen dos tipos diferentes de contenedores o tiempos de ejecución.

**Contenedores de Windows Server**: Proporcionan aislamiento de aplicaciones mediante tecnología de aislamiento de procesos y espacios de nombres. Un contenedor de Windows Server comparte el kernel con el host de contenedor y con todos los contenedores que se ejecutan en el host.  Estos contenedores no proporcionan un límite de seguridad hostil y no deben usarse para aislar un código que no sea de confianza.  Estos contenedores comparten el espacio de kernel con el host y los demás contenedores en el mismo host, por lo que el kernel debe ser coherente, lo que significa una misma versión y configuración.

**Aislamiento de Hyper-V**: amplía el aislamiento que ofrecen los contenedores de Windows Server mediante la ejecución de cada contenedor en una máquina virtual altamente optimizada. En esta configuración, el kernel del host de contenedor no se comparte con otros contenedores que estén en el mismo host.  Estos contenedores se han diseñado para el hospedaje multiinquilino hostil con las mismas garantías de seguridad de una máquina virtual. Dado que estos contenedores no comparten el kernel con el host u otros contenedores del equipo host, pueden ejecutar kernels con distintas versiones y configuraciones (dentro de las versiones compatibles): por ejemplo, todos los contenedores de Windows en Windows 10 usan el aislamiento de Hyper-V para poder usar la versión y configuración del kernel de Windows Server.

## 3. Conceptos básicos de los contenedores

Al empezar a trabajar con contenedores, observará muchas similitudes entre un contenedor y una máquina virtual. Un contenedor ejecuta un sistema operativo, tiene un sistema de archivos y se puede acceder a él a través de una red, como si fuese un equipo físico o virtual. Dicho esto, la tecnología y los conceptos relacionados con los contenedores son muy diferentes de las máquinas virtuales. Los siguientes conceptos clave le resultarán útiles cuando empiece a crear y trabajar con contenedores de Windows. 

**Host de contenedor**: equipo físico o virtual configurado con la característica de contenedor de Windows.

**Imagen de sistema operativo de contenedor:** los contenedores se implementan a partir de imágenes. La imagen del sistema operativo de contenedor es la primera capa de potencialmente muchas capas de imagen que componen un contenedor. Esta imagen ofrece el entorno del sistema operativo.

**Imagen de contenedor**: una imagen de contenedor incluye el sistema operativo base, la aplicación y todas las dependencias de la aplicación necesarias para implementar rápidamente un contenedor. 

**Registro de contenedor**: las imágenes de contenedor se almacenan en un Registro de contenedor y se pueden descargar a petición. 

**Dockerfile**: los Dockerfiles se usan para automatizar la creación de imágenes de contenedor.

## Paso siguiente:

[Inicio rápido de contenedores de Windows Server](quick-start-windows-server.md)  

[Inicio rápido de contenedores de Windows 10](quick-start-windows-10.md)

