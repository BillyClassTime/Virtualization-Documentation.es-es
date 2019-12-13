---
title: Solución de problemas de GMSA para contenedores de Windows
description: Cómo solucionar problemas de las cuentas de servicio administradas de grupo (GMSA) para contenedores de Windows.
keywords: Docker, contenedores, Active Directory, GMSA, cuenta de servicio administrada de grupo, cuentas de servicio administradas de grupo, solución de problemas, solucionar problemas
author: rpsqrd
ms.date: 10/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 89f255e307c2a48fd743d5abd1a49bba7703aaf3
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910245"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Solución de problemas de GMSA para contenedores de Windows

## <a name="known-issues"></a>Problemas conocidos

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>El nombre de host del contenedor debe coincidir con el nombre de gMSA para Windows Server 2016 y Windows 10, versiones 1709 y 1803

Si está ejecutando Windows Server 2016, versión 1709 o 1803, el nombre de host del contenedor debe coincidir con el nombre de la cuenta de gMSA SAM.

Cuando el nombre de host no coincide con el nombre de gMSA, se producirá un error en las solicitudes de autenticación NTLM entrante y la traducción de nombre/SID (que usan muchas bibliotecas, como el proveedor de roles de pertenencia a ASP.NET). Kerberos seguirá funcionando con normalidad aunque el nombre de host y el nombre de gMSA no coincidan.

Esta limitación se corrigió en Windows Server 2019, donde el contenedor ahora usará siempre su nombre de gMSA en la red, independientemente del nombre de host asignado.

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>El uso simultáneo de un gMSA con más de un contenedor conduce a errores intermitentes en Windows Server 2016 y Windows 10, versiones 1709 y 1803

Dado que todos los contenedores deben usar el mismo nombre de host, un segundo problema afecta a las versiones de Windows anteriores a Windows Server 2019 y Windows 10, versión 1809. Cuando se asignan a varios contenedores la misma identidad y el nombre de host, se puede producir una condición de carrera cuando dos contenedores se comunican al mismo controlador de dominio simultáneamente. Cuando otro contenedor se comunica con el mismo controlador de dominio, cancelará la comunicación con los contenedores anteriores con la misma identidad. Esto puede producir errores de autenticación intermitentes y a veces se puede observar como un error de confianza al ejecutar `nltest /sc_verify:contoso.com` dentro del contenedor.

Hemos cambiado el comportamiento en Windows Server 2019 para separar la identidad del contenedor del nombre del equipo, lo que permite que varios contenedores usen el mismo gMSA simultáneamente.

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>No se puede usar GMSA con contenedores aislados de Hyper-V en las versiones 1703, 1709 y 1803 de Windows 10.

La inicialización del contenedor se bloqueará o se producirá un error al intentar usar un gMSA con un contenedor aislado de Hyper-V en las versiones 1703, 1709 y 1803 de Windows 10 y Windows Server.

Este error se corrigió en Windows Server 2019 y Windows 10, versión 1809. También puede ejecutar contenedores aislados de Hyper-V con GMSA en Windows Server 2016 y Windows 10, versión 1607.

## <a name="general-troubleshooting-guidance"></a>Guía de solución de problemas generales

Si encuentra errores al ejecutar un contenedor con un gMSA, las instrucciones siguientes pueden ayudarle a identificar la causa raíz.

### <a name="make-sure-the-host-can-use-the-gmsa"></a>Asegúrese de que el host puede usar gMSA

1. Compruebe que el host está unido a un dominio y puede tener acceso al controlador de dominio.
2. Instale las herramientas de PowerShell de AD desde RSAT y ejecute [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) para ver si el equipo tiene acceso para recuperar el gMSA. Si el cmdlet devuelve **false**, el equipo no tiene acceso a la contraseña de gMSA.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. Si **Test-ADServiceAccount** devuelve **false**, compruebe que el host pertenece a un grupo de seguridad que puede tener acceso a la contraseña de gMSA.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. Si el host pertenece a un grupo de seguridad autorizado para recuperar la contraseña de gMSA pero sigue produciendo un error en **Test-ADServiceAccount**, es posible que tenga que reiniciar el equipo para obtener un nuevo vale que refleje sus pertenencias a grupos actuales.

#### <a name="check-the-credential-spec-file"></a>Comprobar el archivo de especificación de credenciales

1. Ejecute **Get-CredentialSpec** desde el [módulo CredentialSpec de PowerShell](https://aka.ms/credspec) para buscar todas las especificaciones de credenciales en la máquina. Las especificaciones de credenciales deben almacenarse en el directorio "CredentialSpecs" en el directorio raíz de Docker. Puede encontrar el directorio raíz de Docker mediante la ejecución **de Docker info-f "{{. DockerRootDir}} "** .
2. Abra el archivo CredentialSpec y asegúrese de que los siguientes campos se rellenan correctamente:
    - **SID**: el SID de la cuenta de gMSA
    - **MachineAccountName**: el nombre de cuenta SAM de gMSA (no incluya el nombre de dominio completo o el signo de dólar)
    - **DnsTreeName**: el nombre de dominio completo de su bosque de Active Directory
    - **DnsName**: el FQDN del dominio al que pertenece el gMSA
    - **NetBiosName**: nombre de NetBIOS para el dominio al que pertenece el gMSA
    - **GroupManagedServiceAccounts/Name**: el nombre de la cuenta de gMSA Sam (no incluya el nombre de dominio completo o el signo de dólar)
    - **GroupManagedServiceAccounts/ámbito**: una entrada para el FQDN del dominio y otra para NetBIOS

    La entrada debe ser similar a la del ejemplo siguiente de una especificación de credencial completa:

    ```json
    {
        "CmsPlugins": [
            "ActiveDirectory"
        ],
        "DomainJoinConfig": {
            "Sid": "S-1-5-21-702590844-1001920913-2680819671",
            "MachineAccountName": "webapp01",
            "Guid": "56d9b66c-d746-4f87-bd26-26760cfdca2e",
            "DnsTreeName": "contoso.com",
            "DnsName": "contoso.com",
            "NetBiosName": "CONTOSO"
        },
        "ActiveDirectoryConfig": {
            "GroupManagedServiceAccounts": [
                {
                    "Name": "webapp01",
                    "Scope": "contoso.com"
                },
                {
                    "Name": "webapp01",
                    "Scope": "CONTOSO"
                }
            ]
        }
    }
    ```

3. Compruebe que la ruta de acceso al archivo de especificación de credenciales es correcta para la solución de orquestación. Si usa Docker, asegúrese de que el comando de ejecución de contenedor incluye `--security-opt="credentialspec=file://NAME.json"`, donde "NAME. JSON" se reemplaza por el nombre generado por **Get-CredentialSpec**. El nombre es un nombre de archivo plano, relativo a la carpeta CredentialSpecs en el directorio raíz de Docker.

### <a name="check-the-firewall-configuration"></a>Comprobar la configuración del firewall

Si utiliza una directiva de Firewall estricta en el contenedor o en la red del host, puede bloquear las conexiones necesarias con el controlador de Dominio de Active Directory o el servidor DNS.

| Protocolo y puerto | Propósito |
|-------------------|---------|
| TCP y UDP 53 | DNS |
| TCP y UDP 88 | Kerberos |
| TCP 139 | NetLogon |
| TCP y UDP 389 | LDAP |
| TCP 636 | SSL de LDAP |

Es posible que necesite permitir el acceso a puertos adicionales en función del tipo de tráfico que el contenedor envíe a un controlador de dominio.
Consulte [requisitos de Active Directory y Active Directory Domain Services puertos](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers) para obtener una lista completa de los puertos utilizados por Active Directory.

### <a name="check-the-container"></a>Comprobar el contenedor

1. Si está ejecutando una versión de Windows anterior a Windows Server 2019 o Windows 10, versión 1809, el nombre de host del contenedor debe coincidir con el nombre de gMSA. Asegúrese de que el parámetro `--hostname` coincida con el nombre corto gMSA (sin componente de dominio; por ejemplo, "webapp01" en lugar de "webapp01.contoso.com").

2. Compruebe la configuración de red del contenedor para comprobar que el contenedor puede resolver y tener acceso a un controlador de dominio para el dominio de gMSA. Los servidores DNS mal configurados en el contenedor son una causa común de los problemas de identidad.

3. Compruebe si el contenedor tiene una conexión válida con el dominio mediante la ejecución del siguiente cmdlet en el contenedor (con `docker exec` o un equivalente):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    La comprobación de confianza debe devolver `NERR_SUCCESS` si gMSA está disponible y la conectividad de red permite que el contenedor se comunique con el dominio. Si se produce un error, Compruebe la configuración de red del host y del contenedor. Ambos deben ser capaces de comunicarse con el controlador de dominio.

4. Compruebe si el contenedor puede obtener un vale de concesión de vales (TGT) de Kerberos válido:

    ```powershell
    klist get krbtgt
    ```

    Este comando debe devolver "se ha recuperado correctamente un vale para krbtgt" y mostrar el controlador de dominio usado para recuperar el vale. Si puede obtener un TGT pero `nltest` del paso anterior produce un error, esto puede indicar que la cuenta gMSA está mal configurada. Consulte [comprobación de la cuenta de gMSA](#check-the-gmsa-account) para obtener más información.

    Si no puede obtener un TGT dentro del contenedor, esto puede indicar problemas de conectividad de red o DNS. Asegúrese de que el contenedor puede resolver un controlador de dominio con el nombre DNS del dominio y que el controlador de dominio es enrutable desde el contenedor.

5. Asegúrese de que la aplicación está [configurada para usar gMSA](gmsa-configure-app.md). La cuenta de usuario dentro del contenedor no cambia cuando se usa un gMSA. En su lugar, la cuenta del sistema usa el gMSA cuando se comunica con otros recursos de red. Esto significa que la aplicación tendrá que ejecutarse como servicio de red o sistema local para aprovechar la identidad gMSA.

    > [!TIP]
    > Si ejecuta `whoami` o usa otra herramienta para identificar el contexto de usuario actual en el contenedor, no verá el nombre de gMSA. Esto se debe a que siempre inicia sesión en el contenedor como un usuario local en lugar de una identidad de dominio. El gMSA se usa en la cuenta de equipo cada vez que se comunica con recursos de red, por lo que la aplicación debe ejecutarse como servicio de red o sistema local.

### <a name="check-the-gmsa-account"></a>Comprobar la cuenta de gMSA

1. Si el contenedor parece estar configurado correctamente, pero los usuarios u otros servicios no pueden autenticarse automáticamente en la aplicación en contenedor, compruebe los SPN de la cuenta de gMSA. Los clientes encontrarán la cuenta gMSA con el nombre en el que llegan a la aplicación. Esto puede significar que necesitará `host` SPN adicionales para su gMSA si, por ejemplo, los clientes se conectan a la aplicación a través de un equilibrador de carga o un nombre DNS diferente.

2. Asegúrese de que el host de contenedor y gMSA pertenezcan al mismo dominio de Active Directory. El host de contenedor no podrá recuperar la contraseña de gMSA si el gMSA pertenece a un dominio diferente.

3. Asegúrese de que solo haya una cuenta en el dominio con el mismo nombre que el gMSA. los objetos gMSA tienen signos de dólar ($) anexados a sus nombres de cuenta SAM, por lo que es posible que un gMSA tenga el nombre "mi cuenta $" y que una cuenta de usuario no relacionada se denomine "mi cuenta" en el mismo dominio. Esto puede producir problemas si el controlador de dominio o la aplicación tienen que buscar el gMSA por el nombre. Puede buscar objetos con el nombre similar en AD con el siguiente comando:

    ```powershell
    # Replace "GMSANAMEHERE" with your gMSA account name (no trailing dollar sign)
    Get-ADObject -Filter 'sAMAccountName -like "GMSANAMEHERE*"'
    ```

4. Si ha habilitado la delegación sin restricciones en la cuenta gMSA, asegúrese de que el [atributo UserAccountControl](https://support.microsoft.com/en-us/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties) tenga la marca `WORKSTATION_TRUST_ACCOUNT` habilitada. Esta marca es necesaria para que NETLOGON en el contenedor se comunique con el controlador de dominio, como es el caso cuando una aplicación tiene que resolver un nombre como SID o viceversa. Puede comprobar si la marca está configurada correctamente con los siguientes comandos:

    ```powershell
    $gMSA = Get-ADServiceAccount -Identity 'yourGmsaName' -Properties UserAccountControl
    ($gMSA.UserAccountControl -band 0x1000) -eq 0x1000
    ```

    Si los comandos anteriores devuelven `False`, utilice lo siguiente para agregar la marca de `WORKSTATION_TRUST_ACCOUNT` a la propiedad UserAccountControl de la cuenta gMSA. Este comando también borrará las marcas `NORMAL_ACCOUNT`, `INTERDOMAIN_TRUST_ACCOUNT`y `SERVER_TRUST_ACCOUNT` de la propiedad UserAccountControl.

    ```powershell
    Set-ADObject -Identity $gMSA -Replace @{ userAccountControl = ($gmsa.userAccountControl -band 0x7FFFC5FF) -bor 0x1000 }
    ```
