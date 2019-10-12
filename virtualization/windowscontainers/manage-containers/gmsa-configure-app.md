---
title: Configurar la aplicación para usar una cuenta de servicio administrada de grupo
description: Cómo configurar aplicaciones para usar cuentas de servicio administradas de grupo (gMSAs) para contenedores de Windows.
keywords: acoplador, contenedores, Active Directory, GMSA, aplicaciones, aplicaciones, cuenta de servicio administrado de grupo, cuentas de servicio administradas de grupo, configuración
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 6635381d5f7ddbebf7bdea4624af241b9f6a6864
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209875"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>Configurar la aplicación para que use un gMSA

En la configuración típica, un contenedor solo recibe una cuenta de servicio administrada de grupo (gMSA) que se usa siempre que la cuenta del equipo de contenedor intenta autenticarse en los recursos de red. Esto significa que la aplicación tendrá que ejecutarse como **sistema local** o **servicio de red** si necesita usar la identidad gMSA.

## <a name="run-an-iis-app-pool-as-network-service"></a>Ejecutar un grupo de aplicaciones de IIS como servicio de red

Si está hospedando un sitio web de IIS en su contenedor, todo lo que tiene que hacer para aprovechar el gMSA es establecer la identidad del grupo de aplicaciones en **servicio de red**. Puede hacerlo en su Dockerfile agregando el siguiente comando:

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

Si previamente usó credenciales de usuario estáticas para el grupo de aplicaciones de IIS, considere la gMSA como el reemplazo de esas credenciales. Puede cambiar el gMSA entre los entornos de desarrollo, prueba y producción, y IIS recogerá automáticamente la identidad actual sin necesidad de cambiar la imagen del contenedor.

## <a name="run-a-windows-service-as-network-service"></a>Ejecutar un servicio de Windows como servicio de red

Si la aplicación de contenedor se ejecuta como un servicio de Windows, puede configurar el servicio para que se ejecute como **servicio de red** en su Dockerfile:

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>Ejecutar aplicaciones de consola arbitrarias como servicio de red

Para las aplicaciones de consola genérica que no se hospedan en IIS o en el administrador de servicios, suele ser más fácil ejecutar el contenedor como **servicio de red** para que la aplicación herede automáticamente el contexto gMSA. Esta característica está disponible a partir de la versión 1709 de Windows Server.

Agregue la siguiente línea a la Dockerfile para que se ejecute como servicio de red de forma predeterminada:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

También puede conectarse a un contenedor como servicio de red de forma individualizada con `docker exec`. Esto es particularmente útil si está solucionando problemas de conectividad en un contenedor en ejecución cuando el contenedor normalmente no se ejecuta como servicio de red.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>Pasos siguientes

Además de configurar aplicaciones, también puede usar gMSAs para:

- [Ejecutar contenedores](gmsa-run-container.md)
- [Organizar contenedores](gmsa-orchestrate-containers.md)

Si tiene algún problema durante la instalación, consulte nuestra [Guía de solución de problemas](gmsa-troubleshooting.md) para ver posibles soluciones.
