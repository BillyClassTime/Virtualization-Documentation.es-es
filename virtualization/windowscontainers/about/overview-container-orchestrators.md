---
title: Información general sobre orquestación de contenedores de Windows
description: Más información sobre los orquestadores de contenedores de Windows.
keywords: docker, contenedores
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 99a3b47a9d80e21c246fb3b4f61d650557eb37fa
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910725"
---
# <a name="windows-container-orchestration-overview"></a>Información general sobre orquestación de contenedores de Windows

Debido a su tamaño pequeño y a la orientación de la aplicación, los contenedores son perfectos para entornos de entrega ágiles y arquitecturas basadas en microservicios. Sin embargo, un entorno que utiliza contenedores y microservidors puede tener cientos o miles de componentes para realizar un seguimiento. Es posible que pueda administrar manualmente unas docenas máquinas virtuales o servidores físicos, pero no hay ninguna manera de administrar correctamente un entorno de contenedor de escala de producción sin automatización. Esta tarea debe pertenecer al orquestador, que es un proceso que automatiza y administra un gran número de contenedores y cómo interactúan entre sí.

Los orquestadores realizan las siguientes tareas:

- Programación: cuando se proporciona una imagen de contenedor y una solicitud de recurso, el orquestador encuentra un equipo adecuado en el que se ejecuta el contenedor.
- Afinidad/antiafinidad: especifique si un conjunto de contenedores se debe ejecutar casi entre sí por el rendimiento o por separado.
- Supervisión de estado: se buscan errores en el contenedor y se reprograman automáticamente.
- Conmutación por error: realice un seguimiento de lo que se ejecuta en cada máquina y vuelva a programar los contenedores de los equipos con errores en los nodos correctos.
- Escalado: agregue o quite instancias de contenedor para que coincidan con la petición, de forma manual o automática.
- Redes: proporcione una red superpuesta que coordine los contenedores para comunicarse a través de varios equipos host.
- Detección de servicios: se habilitan contenedores para que puedan localizarse entre sí automáticamente, incluso cuando se muevan entre equipos host y cambien las direcciones IP.
- Actualizaciones de aplicación coordinadas: se administran las actualizaciones del contenedor para evitar tiempos de inactividad de la aplicación y permitir la reversión si algo va mal.

## <a name="orchestrator-types"></a>Tipos de orquestador

Azure ofrece dos orquestadores de contenedor: Azure Kubernetes Service (AKS) y Service Fabric.

[Azure Kubernetes Service (AKS)](/azure/aks/) simplifica la creación, configuración y administración de un clúster de máquinas virtuales preconfiguradas para ejecutar aplicaciones en contenedor. Esto le permite usar sus conocimientos actuales y basarse en un amplio y creciente conjunto de conocimientos de la comunidad para implementar y administrar aplicaciones basadas en contenedores en Microsoft Azure. Mediante el uso de AKS, puede aprovechar las características empresariales de Azure sin perder la portabilidad de la aplicación a través de Kubernetes y el formato de imagen de Docker.

[Azure Service Fabric](/azure/service-fabric/) es una plataforma de sistemas distribuidos que facilita el proceso de empaquetar, implementar y administrar microservicios y contenedores escalables y de confianza. Service Fabric aborda los desafíos importantes del desarrollo y la administración de aplicaciones nativas en la nube. Los desarrolladores y los administradores pueden evitar problemas complejos de infraestructura y centrarse en la implementación de cargas de trabajo críticas y exigentes que son escalables, de confianza y fáciles de administrar. Service Fabric representa la plataforma de última generación para crear y administrar estas aplicaciones de escalado en la nube, de nivel 1 y de clase empresarial que se ejecutan en contenedores.

## <a name="getting-started"></a>Introducción

Para empezar a implementar Azure Kubernetes Service, consulte la [Guía de configuración de Kubernetes](../kubernetes/getting-started-kubernetes-windows.md).

Para empezar a implementar Azure Service Fabric, consulte la guía de [Inicio rápido de Service fabric](/azure/service-fabric/service-fabric-quickstart-containers.md).