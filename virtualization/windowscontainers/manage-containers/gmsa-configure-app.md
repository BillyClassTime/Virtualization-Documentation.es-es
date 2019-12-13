---
title: Configuración de la aplicación para usar una cuenta de servicio administrada de grupo
description: Configuración de aplicaciones para usar cuentas de servicio administradas de grupo (GMSA) para contenedores de Windows.
keywords: Docker, contenedores, Active Directory, GMSA, aplicaciones, aplicaciones, cuenta de servicio administrada de grupo, cuentas de servicio administradas de grupo, configuración
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 6635381d5f7ddbebf7bdea4624af241b9f6a6864
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909795"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>Configuración de la aplicación para usar un gMSA

En la configuración típica, un contenedor solo tiene una cuenta de servicio administrada de grupo (gMSA) que se usa siempre que la cuenta de equipo del contenedor intenta autenticarse en los recursos de red. Esto significa que la aplicación tendrá que ejecutarse como **sistema local** o **servicio de red** si necesita usar la identidad gMSA.

## <a name="run-an-iis-app-pool-as-network-service"></a>Ejecutar un grupo de aplicaciones de IIS como servicio de red

Si va a hospedar un sitio web de IIS en el contenedor, lo único que debe hacer para aprovechar el gMSA es establecer la identidad del grupo de aplicaciones en **servicio de red**. Puede hacerlo en el Dockerfile agregando el comando siguiente:

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

Si anteriormente usó credenciales de usuario estáticas para el grupo de aplicaciones de IIS, considere la gMSA como el reemplazo de esas credenciales. Puede cambiar la gMSA entre los entornos de desarrollo, pruebas y producción, y IIS recogerá automáticamente la identidad actual sin tener que cambiar la imagen del contenedor.

## <a name="run-a-windows-service-as-network-service"></a>Ejecutar un servicio de Windows como servicio de red

Si la aplicación en contenedor se ejecuta como un servicio de Windows, puede configurar el servicio para que se ejecute como **servicio de red** en el Dockerfile:

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>Ejecución de aplicaciones de consola arbitrarias como servicio de red

En el caso de las aplicaciones de consola genéricas que no se hospedan en IIS o Service Manager, a menudo es más fácil ejecutar el contenedor como **servicio de red** , por lo que la aplicación hereda automáticamente el contexto gMSA. Esta característica está disponible a partir de la versión 1709 de Windows Server.

Agregue la siguiente línea a Dockerfile para que se ejecute como servicio de red de forma predeterminada:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

También puede conectarse a un contenedor como servicio de red de forma única con `docker exec`. Esto es especialmente útil si está solucionando problemas de conectividad en un contenedor en ejecución cuando el contenedor no se ejecuta normalmente como servicio de red.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>Pasos siguientes

Además de configurar las aplicaciones, también puede usar GMSA para:

- [Ejecutar contenedores](gmsa-run-container.md)
- [Orquestación de contenedores](gmsa-orchestrate-containers.md)

Si surgen problemas durante la instalación, consulte nuestra [Guía de solución de problemas](gmsa-troubleshooting.md) para ver las posibles soluciones.
