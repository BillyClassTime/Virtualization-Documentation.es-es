---
title: Acerca de los orquestadores de contenedor de Windows
description: Obtenga más información acerca de los Windows Container Orchestrator.
keywords: docker, contenedores
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 1ccf63b0ae55501ba32f8bdd61994e7f8006b5e6
ms.sourcegitcommit: daf1d2b5879c382404fc4d59f1c35c88650e20f7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/23/2019
ms.locfileid: "9674902"
---
# <a name="about-windows-container-orchestrators"></a>Acerca de los orquestadores de contenedor de Windows

A causa de su pequeño tamaño y la orientación de la aplicación, los contenedores son ideales para entornos de entrega ágiles y arquitecturas basadas en microservicios. Sin embargo, un entorno que usa contenedores y microservidors puede tener cientos o miles de componentes para realizar un seguimiento de ellos. Es posible que pueda administrar manualmente algunas docenas de máquinas virtuales o servidores físicos, pero no hay forma de administrar correctamente un entorno de contenedor de escala de producción sin automatización. Esta tarea debe corresponder a su Orchestrator, que es un proceso que automatiza y administra un gran número de contenedores y cómo interactúan entre sí.

Los Orchestrator realizan las siguientes tareas:

- Programación: cuando se proporciona una imagen de contenedor y una solicitud de recursos, el organizador encuentra un equipo adecuado en el que ejecutar el contenedor.
- Afinidad/antiafinidad: especifique si un conjunto de contenedores debe ejecutarse cerca de su rendimiento o estar separado por la disponibilidad.
- Supervisión de estado: se buscan errores en el contenedor y se reprograman automáticamente.
- Conmutación por error: realice un seguimiento de lo que se está ejecutando en cada equipo y reprograme contenedores de equipos con errores en nodos saludables.
- Escala: agregue o quite instancias de contenedor para que coincidan con la solicitud, de forma manual o automática.
- Red: proporciona una red de superposición que coordina contenedores para comunicarse a través de varias máquinas host.
- Detección de servicios: se habilitan contenedores para que puedan localizarse entre sí automáticamente, incluso cuando se muevan entre equipos host y cambien las direcciones IP.
- Actualizaciones de aplicación coordinadas: se administran las actualizaciones del contenedor para evitar tiempos de inactividad de la aplicación y permitir la reversión si algo va mal.

## <a name="orchestrator-types"></a>Tipos de Orchestrator

Azure ofrece dos Orchestrator de contenedor: servicio Azure Kubernetes (AKS) y Service fabric.

El [servicio Azure Kubernetes (AKS)](/azure/aks/) simplifica la creación, la configuración y la administración de un clúster de máquinas virtuales preconfiguradas para ejecutar aplicaciones de contenedor. Esto le permite usar sus habilidades existentes y hacer un dibujo en un gran y creciente cuerpo de experiencia en la comunidad para implementar y administrar aplicaciones basadas en contenedores en Microsoft Azure. Al usar AKS, puede aprovechar las características de nivel empresarial de Azure y, al mismo tiempo, mantener la portabilidad de la aplicación a través de Kubernetes y el formato de imagen del acoplador.

[Azure Service Fabric](/azure/service-fabric/) es una plataforma de sistemas distribuidos que facilita el proceso de empaquetar, implementar y administrar microservicios y contenedores escalables y de confianza. Service Fabric aborda los desafíos importantes del desarrollo y la administración de aplicaciones nativas en la nube. Los desarrolladores y los administradores pueden evitar problemas complejos de infraestructura y centrarse en la implementación de cargas de trabajo críticas y exigentes que son escalables, de confianza y fáciles de administrar. Service Fabric representa la plataforma de última generación para crear y administrar estas aplicaciones de escalado en la nube, de nivel 1 y de clase empresarial que se ejecutan en contenedores.

## <a name="getting-started"></a>Tareas iniciales

Para empezar a implementar el servicio Kubernetes de Azure, consulte la [Guía de configuración de Kubernetes](../kubernetes/getting-started-kubernetes-windows.md).

Para empezar a implementar Azure Service fabric, vea el [tutorial rápido de Service fabric](/azure/service-fabric/service-fabric-quickstart-containers.md).