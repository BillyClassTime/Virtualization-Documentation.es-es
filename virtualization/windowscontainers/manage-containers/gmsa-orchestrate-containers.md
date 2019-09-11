---
title: Orquestar contenedores con un gMSA
description: Cómo organizar contenedores de Windows con una cuenta de servicio administrada de grupo (gMSA).
keywords: acoplador, contenedores, Active Directory, GMSA, orquestación, kubernetes, cuenta de servicio administrado de grupo, cuentas de servicio administrados por grupo
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b4dac775dc7a4ee6375f0d803e921527e66aae5b
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079768"
---
## <a name="orchestrate-containers-with-a-gmsa"></a>Orquestar contenedores con un gMSA

En los entornos de producción, a menudo usas un Orchestrator de contenedor para implementar y administrar tus aplicaciones y servicios. Cada organizador tiene sus propios paradigmas de administración y es responsable de aceptar las especificaciones de credenciales para dar a la plataforma contenedor de Windows.

Cuando vaya a orquestar contenedores con cuentas de servicio administradas de grupo (gMSAs), asegúrese de lo siguiente:

> [!div class="checklist"]
> * Todos los hosts de contenedor que se pueden programar para ejecutar contenedores con gMSAs están Unidos a un dominio
> * Los hosts contenedores tienen acceso para recuperar las contraseñas de todos los gMSAs usados por contenedores
> * Los archivos de especificación de credenciales se crean y se cargan en el Orchestrator o se copian en cada uno de ellos, en función de cómo prefiera administrar el Orchestrator.
> * Las redes de contenedor permiten que los contenedores se comuniquen con los controladores de dominio de Active Directory para recuperar vales de gMSA

### <a name="how-to-use-gmsa-with-service-fabric"></a>Cómo usar gMSA con Service fabric

Service fabric admite la ejecución de contenedores de Windows con un gMSA cuando especifica la ubicación de las especificaciones de credenciales en el manifiesto de la aplicación. Tendrá que crear el archivo de especificación de credenciales y colocarlo en el subdirectorio **CredentialSpecs** del directorio de datos de Dock en cada host para que Service fabric pueda encontrarlo. Puede ejecutar el cmdlet **Get-CredentialSpec** , que forma parte del [módulo de PowerShell CredentialSpec](https://aka.ms/credspec), para comprobar si la especificación de credenciales está en la ubicación correcta.

Vea [Inicio rápido: implementar contenedores de Windows en Service fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) y [configurar gMSA para los contenedores de Windows que se ejecutan en fabric Service](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) para obtener más información sobre cómo configurar la aplicación.

### <a name="how-to-use-gmsa-with-docker-swarm"></a>Cómo usar gMSA con Swarm de acoplamiento

Para usar un gMSA con contenedores administrados por Dock Swarm, ejecute el comando [crear del servicio del acoplador](https://docs.docker.com/engine/reference/commandline/service_create/) con el `--credential-spec` parámetro:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Para obtener más información sobre cómo usar las especificaciones de credenciales con los servicios de acoplamiento, vea el [ejemplo Swarm](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) .

### <a name="how-to-use-gmsa-with-kubernetes"></a>Cómo usar gMSA con Kubernetes

La compatibilidad con la programación de contenedores de Windows con gMSAs en Kubernetes está disponible como una característica alfa en Kubernetes 1,14. Consulte [Configure gMSA for Windows pods and containers](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) para obtener la información más reciente sobre esta característica y cómo probarla en su distribución de Kubernetes.

## <a name="next-steps"></a>Pasos siguientes

Además de la orquestación de contenedores, también puede usar gMSAs para:

- [Configurar aplicaciones](gmsa-configure-app.md)
- [Ejecutar contenedores](gmsa-run-container.md)

Si tiene algún problema durante la instalación, consulte nuestra [Guía de solución de problemas](gmsa-troubleshooting.md) para ver posibles soluciones.
