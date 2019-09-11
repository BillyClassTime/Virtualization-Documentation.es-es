---
title: Ejecutar un contenedor con un gMSA
description: Cómo ejecutar un contenedor de Windows con una cuenta de servicio administrada de grupo (gMSA).
keywords: acoplador, contenedores, Active Directory, GMSA, cuenta de servicio administrado de grupo, cuentas de servicio administradas de grupo
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b9c0406b5fe9527d88365dabf0cfd10114c34c74
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079764"
---
# <a name="run-a-container-with-a-gmsa"></a>Ejecutar un contenedor con un gMSA

Para ejecutar un contenedor con una cuenta de servicio administrada de grupo (gMSA), proporcione el archivo de `--security-opt` especificación de credenciales en el parámetro de la [ejecución del acoplador](https://docs.docker.com/engine/reference/run):

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>En Windows Server 2016 versiones 1709 y 1803, el nombre de host del contenedor debe coincidir con el nombre corto gMSA.

En el ejemplo anterior, el nombre de cuenta SAM de gMSA es "webapp01", por lo que el nombre de host del contenedor también se denomina "webapp01".

En Windows Server 2019 y versiones posteriores, el campo hostname no es necesario, pero el contenedor seguirá identificándose por el nombre de gMSA en lugar del nombre de host, incluso si proporciona explícitamente una diferente.

Para comprobar si el gMSA está funcionando correctamente, ejecute el siguiente cmdlet en el contenedor:

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

Si el estado de la conexión DC de confianza y el estado `NERR_Success`de verificación de confianza no son así, siga las instrucciones para la [solución de problemas](gmsa-troubleshooting.md#check-the-container) para depurar el problema.

Para comprobar la identidad de gMSA desde dentro del contenedor, ejecute el siguiente comando y compruebe el nombre de cliente:

```powershell
PS C:\> klist get krbtgt

Current LogonId is 0:0xaa79ef8
A ticket to krbtgt has been retrieved successfully.

Cached Tickets: (2)

#0>     Client: webapp01$ @ CONTOSO.COM
        Server: krbtgt/webapp01 @ CONTOSO.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
        Start Time: 3/21/2019 4:17:53 (local)
        End Time:   3/21/2019 14:17:53 (local)
        Renew Time: 3/28/2019 4:17:42 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called: dc01.contoso.com

[...]
```

Para abrir PowerShell u otra aplicación de consola como la cuenta gMSA, puede pedir al contenedor que se ejecute en la cuenta servicio de red en lugar de la cuenta normal ContainerAdministrator (o ContainerUser para nanoserver):

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Si está ejecutando como servicio de red, puede probar la autenticación de red como la gMSA intentando conectarse a SYSVOL en un controlador de dominio:

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="next-steps"></a>Pasos siguientes

Además de la ejecución de contenedores, también puede usar gMSAs para:

- [Configurar aplicaciones](gmsa-configure-app.md)
- [Organizar contenedores](gmsa-orchestrate-containers.md)

Si tiene algún problema durante la instalación, consulte nuestra [Guía de solución de problemas](gmsa-troubleshooting.md) para ver posibles soluciones.
