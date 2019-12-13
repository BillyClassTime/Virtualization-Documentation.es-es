---
title: Orquestación de contenedores con gMSA
description: Orquestación de contenedores de Windows con una cuenta de servicio administrada de grupo (gMSA).
keywords: Docker, contenedores, Active Directory, GMSA, orquestación, kubernetes, cuenta de servicio administrada de grupo, cuentas de servicio administradas de grupo
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 3d102aac45a1becf1879a718bb255d753b215006
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910265"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>Orquestación de contenedores con gMSA

En entornos de producción, a menudo usará un orquestador de contenedor para implementar y administrar sus aplicaciones y servicios. Cada orquestador tiene sus propios paradigmas de administración y es responsable de aceptar las especificaciones de credenciales para proporcionar a la plataforma de contenedores de Windows.

Cuando esté orquestando contenedores con cuentas de servicio administradas de grupo (GMSA), asegúrese de que:

> [!div class="checklist"]
> * Todos los hosts de contenedor que se pueden programar para ejecutar contenedores con GMSA están Unidos a un dominio
> * Los hosts de contenedor tienen acceso para recuperar las contraseñas de todos los GMSA utilizados por los contenedores
> * Los archivos de especificación de credenciales se crean y se cargan en el orquestador o se copian en cada host de contenedor, en función de cómo el orquestador prefiera controlarlos.
> * Las redes de contenedor permiten que los contenedores se comuniquen con los controladores de Dominio de Active Directory para recuperar los vales de gMSA

## <a name="how-to-use-gmsa-with-service-fabric"></a>Cómo usar gMSA con Service Fabric

Service Fabric admite la ejecución de contenedores de Windows con un gMSA al especificar la ubicación de la especificación de credenciales en el manifiesto de aplicación. Tendrá que crear el archivo de especificación de credenciales y colocarlo en el subdirectorio **CredentialSpecs** del directorio de datos de Docker en cada host para que Service fabric pueda encontrarlo. Puede ejecutar el cmdlet **Get-CredentialSpec** , que forma parte del [módulo CredentialSpec de PowerShell](https://aka.ms/credspec), para comprobar si la especificación de credenciales se encuentra en la ubicación correcta.

Consulte [Inicio rápido: implementación de contenedores de Windows en Service fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) y [configuración de gMSA para contenedores de Windows que se ejecutan en Service fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) para obtener más información sobre cómo configurar la aplicación.

## <a name="how-to-use-gmsa-with-docker-swarm"></a>Cómo usar gMSA con Docker Swarm

Para usar un gMSA con contenedores administrados por Docker Swarm, ejecute el comando [Docker Service Create](https://docs.docker.com/engine/reference/commandline/service_create/) con el parámetro `--credential-spec`:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Vea el [ejemplo de Docker Swarm](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) para más información sobre cómo usar las especificaciones de credenciales con los servicios de Docker.

## <a name="how-to-use-gmsa-with-kubernetes"></a>Cómo usar gMSA con Kubernetes

La compatibilidad con la programación de contenedores de Windows con GMSA en Kubernetes está disponible como una característica alfa en Kubernetes 1,14. Consulte [configuración de gMSA para pods y contenedores de Windows](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) para obtener la información más reciente sobre esta característica y cómo probarla en la distribución de Kubernetes.

## <a name="next-steps"></a>Pasos siguientes

Además de la orquestación de contenedores, también puede usar GMSA para:

- [Configurar aplicaciones](gmsa-configure-app.md)
- [Ejecutar contenedores](gmsa-run-container.md)

Si surgen problemas durante la instalación, consulte nuestra [Guía de solución de problemas](gmsa-troubleshooting.md) para ver las posibles soluciones.
