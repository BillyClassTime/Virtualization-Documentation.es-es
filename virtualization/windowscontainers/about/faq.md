---
title: "Preguntas más frecuentes sobre los contenedores de Windows"
description: "Preguntas más frecuentes sobre los contenedores de Windows"
keywords: docker, containers
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
translationtype: Human Translation
ms.sourcegitcommit: cfa3c14e932f8b86edf6667200ac028ea0a16b67
ms.openlocfilehash: c3a7decaf087741c82419a8a541867ae01f0f4da

---

# Preguntas más frecuentes

## Acerca de los contenedores de Windows

**¿Qué es un contenedor de Windows Server?**

Los contenedores de Windows Server son un método ligero de virtualización del sistema operativo utilizado para separar las aplicaciones o los servicios de otros servicios que se ejecutan en el mismo host de contenedor. Para habilitar esto, cada contenedor tiene su propia vista del sistema operativo, los procesos, el sistema de archivos, el registro y las direcciones IP.  

**¿Qué es un contenedor de Hyper-V?**

Puede pensar en un contenedor de Hyper-V como un contenedor de Windows Server que se ejecuta dentro de una partición de Hyper-V.

Los contenedores de Hyper-V ofrecen una opción de implementación adicional entre el contenedor de Windows Server de gran eficiencia y alta densidad y la máquina virtual aislada de Hyper-V virtualizada por hardware. En entornos donde haya aplicaciones de diferentes límites de confianza en el mismo host, puede requerirse aislamiento adicional. Los contenedores de Hyper-V ofrecerán mayor aislamiento mediante una virtualización optimizada y el sistema operativo Windows Server que separa los contenedores entre sí y del sistema operativo host. Las dos opciones de implementación de contenedor usan las mismas API de administración, herramientas y formatos de imagen, de forma que los clientes pueden simplemente elegir en tiempo de implementación el modo de implementación que mejor se adapte sus requisitos.

**¿Cuál es la diferencia entre los contenedores de Windows Server y Linux?**

Los contenedores de Windows Server y Linux son similares, los dos implementan tecnologías similares dentro de su kernel y su sistema operativo base. La diferencia reside en la plataforma y las cargas de trabajo que se ejecutan dentro de los contenedores.  
Cuando un cliente utiliza contenedores de Windows Server, puede integrarlos con las tecnologías de Windows existentes, como. NET, ASP.NET, PowerShell y muchas más.

**Como desarrollador, ¿tengo que volver a escribir la aplicación para cada tipo de contenedor?**

No, las imágenes de contenedores de Windows son comunes para todos los contenedores de Windows Server y de Hyper-V. La elección del tipo de contenedor se realiza cuando se inicia el contenedor. Desde la perspectiva del desarrollador, los contenedores de Windows Server y Hyper-V son dos versiones de lo mismo.  Ofrecen la misma experiencia de desarrollo, programación y administración, son abiertos y extensibles e incluirán el mismo nivel de integración y compatibilidad a través de Docker.

Un desarrollador puede crear una imagen de contenedor con un contenedor de Windows Server e implementarla como un contenedor de Hyper-V o viceversa, sin ningún cambio aparte de especificar la marca de tiempo de ejecución adecuada.

Los contenedores de Windows Server ofrecerán mayor densidad y rendimiento (por ejemplo, un menor tiempo de spin up, un rendimiento en tiempo de ejecución más rápido en comparación con las configuraciones anidadas) cuando la velocidad es clave. Los contenedores de Hyper-V ofrecen mayor aislamiento, garantizando que el código que se ejecuta en un contenedor no se puede poner en peligro ni afectar negativamente al sistema operativo host u otros contenedores que se ejecutan en el mismo host. Esto es útil en los escenarios de varios inquilinos (con requisitos para el hospedaje de código no seguro), entre los que se incluyen las aplicaciones de SaaS y el hospedaje de procesos.

**¿Son un complemento los contenedores de Hyper-V o Windows Server, o bien se integrarán dentro de Windows Server?**

Las capacidades de contenedor se integrarán en Windows Server 2016. Permanezca atento para más información cuando se acerque la fecha de disponibilidad general.  

**¿Cuál es la relación entre los contenedores de Windows Server y Drawbridge?**

Drawbridge fue uno de muchos proyectos de investigación que nos ayudaron a obtener información valiosa sobre los contenedores.  Gran parte de la tecnología de contenedores de Windows Server 2016 surgió de nuestra experiencia con Drawbridge y nos complace incluir excelentes tecnologías de contenedor para nuestros clientes en Windows Server 2016.

**¿Cuáles son los requisitos previos para los contenedores de Windows Server y Hyper-V?**

Tanto los contenedores de Windows Server como los de Hyper-V requieren Windows Server 2016. Estas tecnologías no funcionarán con versiones anteriores de Windows.


## Administración de contenedores de Windows

**¿Los contenedores de Hyper-V también estarán disponibles para el ecosistema de Docker?**

Sí, los contenedores de Hyper-V ofrecerán el mismo nivel de integración y administración con Docker que con los contenedores de Windows Server.  El objetivo es tener una experiencia multiplataforma, coherente y abierta.  
La plataforma Docker también simplificará enormemente y mejorará la experiencia de trabajo a través de nuestras opciones de contenedor. Una aplicación desarrollada con contenedores de Windows Server puede implementarse como un contenedor de Hyper-V sin cambios.


**¿Puedo ejecutar contenedores de Windows en ESXi u otro hipervisor que no sea de Hyper-V?**

Sí, el contenedor de Windows se ejecuta en cualquier instalación de Server Core TP3.  Siga las instrucciones para [habilitar la característica de contenedores en contexto](../quick_start/inplace_setup.md).

## Ecosistema abierto de Microsoft

**¿Microsoft participa en la iniciativa de contenedores abiertos (OCI, Open Container Initiative)?**

Para garantizar que el formato de empaquetado permanece universal, Docker organizó recientemente la Open Container Initiative (OCI), con el objetivo de garantizar que el empaquetado de contenedores permanece como un formato abierto y guiado por una fundación, con Microsoft como uno de los miembros fundadores.

**¿Está Microsoft realmente asociado con Docker?**

Sí.  
Nuestra asociación con Docker permite a los desarrolladores crear, administrar e implementar contenedores de Windows Server y Linux con el mismo conjunto de herramientas de Docker. Los desarrolladores que tienen como destino Windows Server ya no tienen que elegir entre usar la amplia gama de tecnologías de Windows Server y la creación de aplicaciones en contenedores.  

Docker es dos cosas, el grupo de código abierto de proyectos y la empresa Docker. Consideramos que esta asociación incluye las dos. Parte del éxito de Docker es debido al vibrante ecosistema que se ha creado en torno a la tecnología de contenedores de Docker. Microsoft está contribuyendo al proyecto Docker, habilitando la compatibilidad con los contenedores de Windows Server y Hyper-V.  

Para obtener más información, consulte la entrada de blog [New Windows Server containers and Azure support for Docker](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/?WT.mc_id=Blog_ServerCloud_Announce_TTD) (Nuevos contenedores de Windows Server y compatibilidad de Azure con Docker).



<!--HONumber=Jun16_HO4-->


