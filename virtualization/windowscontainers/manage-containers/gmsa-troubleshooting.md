---
title: Solucionar problemas de gMSAs para contenedores de Windows
description: Cómo solucionar problemas de cuentas de servicio administrados de grupo (gMSAs) para contenedores de Windows.
keywords: acoplador, contenedores, Active Directory, GMSA, cuenta de servicio administrado de grupo, cuentas de servicio administradas de grupo, solución de problemas, solución de problemas
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 00a0d9b1367da55b7669fc26a3eca303272967ab
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079769"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Solucionar problemas de gMSAs para contenedores de Windows

## <a name="known-issues"></a>Problemas conocidos

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>El nombre de host del contenedor debe coincidir con el gMSA de Windows Server 2016 y Windows 10, versiones 1709 y 1803

Si está ejecutando Windows Server 2016, versión 1709 o 1803, el nombre de host del contenedor debe coincidir con el nombre de la cuenta SAM de gMSA.

Cuando el nombre de host no coincide con el nombre del gMSA, las solicitudes de autenticación NTLM de entrada y la traducción de nombres/SID (usada por muchas bibliotecas, como el proveedor de roles de pertenencia a ASP.NET) producirán un error. Kerberos seguirá funcionando normalmente incluso si el nombre de host y el nombre de gMSA no coinciden.

Esta limitación se corrigió en Windows Server 2019, donde ahora el contenedor siempre usará su nombre de gMSA en la red, independientemente del nombre de host asignado.

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Usar un gMSA con más de un contenedor de forma simultánea provoca errores intermitentes en Windows Server 2016 y Windows 10, versiones 1709 y 1803

Debido a que todos los contenedores deben usar el mismo nombre de host, un segundo problema afecta a las versiones de Windows anteriores a Windows Server 2019 y Windows 10, versión 1809. Cuando se asigna la misma identidad y nombre de host a varios contenedores, se puede producir una condición de carrera cuando dos contenedores hablan simultáneamente con el mismo controlador de dominio. Cuando otro contenedor se comunique con el mismo controlador de dominio, cancelará la comunicación con cualquier contenedor anterior que use la misma identidad. Esto puede provocar errores de autenticación intermitentes y, a veces, se puede observar como un error `nltest /sc_verify:contoso.com` de confianza cuando se ejecuta dentro del contenedor.

Hemos cambiado el comportamiento en Windows Server 2019 para separar la identidad del contenedor del nombre de la máquina, lo que permite que varios contenedores usen el mismo gMSA al mismo tiempo.

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>No puede usar gMSAs con contenedores aislados de Hyper-V en las versiones 1703, 1709 y 1803 de Windows 10

La inicialización del contenedor se bloqueará o fallará al intentar usar un gMSA con un contenedor aislado de Hyper-V en Windows 10 y Windows Server versiones 1703, 1709 y 1803.

Este error se corrigió en Windows Server 2019 y Windows 10, versión 1809. También puede ejecutar contenedores aislados de Hyper-V con gMSAs en Windows Server 2016 y Windows 10, versión 1607.

## <a name="general-troubleshooting-guidance"></a>Instrucciones para la solución de problemas generales

Si encuentra errores al ejecutar un contenedor con un gMSA, las siguientes instrucciones pueden ayudarle a identificar la causa raíz.

### <a name="make-sure-the-host-can-use-the-gmsa"></a>Asegúrese de que el anfitrión puede usar el gMSA

1. Compruebe que el host está unido al dominio y que puede comunicarse con el controlador de dominio.
2. Instale las herramientas de AD PowerShell de RSAT y ejecute [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) para ver si el equipo tiene acceso para recuperar el gMSA. Si el cmdlet devuelve **false**, significa que el equipo no tiene acceso a la contraseña de gMSA.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. Si **prueba-ADServiceAccount** devuelve **falso**, compruebe que el host pertenece a un grupo de seguridad que puede obtener acceso a la contraseña de gMSA.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. Si su anfitrión pertenece a un grupo de seguridad autorizado para recuperar la contraseña de gMSA pero sigue produciendo errores **en la prueba de ADServiceAccount**, es posible que tenga que reiniciar el equipo para obtener un nuevo vale que refleje sus pertenencias a grupos actuales.

#### <a name="check-the-credential-spec-file"></a>Comprobar el archivo de especificaciones de credenciales

1. Ejecute **Get-CredentialSpec** desde el [módulo de PowerShell CredentialSpec](https://aka.ms/credspec) para ubicar todas las especificaciones de credenciales en el equipo. Las especificaciones de las credenciales deben almacenarse en el directorio "CredentialSpecs" debajo del directorio raíz del Dock. Puede encontrar el directorio raíz del Dock ejecutando **información de Dock-f "{{. DockerRootDir}} "**.
2. Abra el archivo CredentialSpec y asegúrese de que los siguientes campos se han rellenado correctamente:
    - **SID**: el SID de la cuenta de gMSA
    - **MachineAccountName**: el nombre de cuenta SAM de gMSA (no incluya el nombre de dominio completo o el signo de dólar)
    - **DnsTreeName**: el FQDN del bosque de Active Directory
    - **DnsName**: el FQDN del dominio al que pertenece el gMSA
    - **NetBiosName**: nombre de NetBIOS para el dominio al que pertenece el gMSA
    - **GroupManagedServiceAccounts/Name**: el nombre de cuenta SAM de gMSA (no incluye el nombre de dominio completo o el signo de dólar)
    - **GroupManagedServiceAccounts/ámbito**: una entrada para el dominio FQDN y otra para el NetBIOS

    La entrada debe ser similar al siguiente ejemplo de una especificación completa de credenciales:

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

3. Verifique que la ruta de acceso al archivo de especificaciones de credenciales sea correcta para la solución de orquestación. Si está usando el acoplador, asegúrese de que el comando ejecutar del `--security-opt="credentialspec=file://NAME.json"`contenedor incluye, donde "nombre. JSON" se reemplaza por el nombre generado por **Get-CredentialSpec**. El nombre es un nombre de archivo plano, relativo a la carpeta CredentialSpecs en el directorio raíz del Dock.

### <a name="check-the-firewall-configuration"></a>Comprobar la configuración del firewall

Si está usando una directiva de Firewall estricta en el contenedor o la red host, puede bloquear las conexiones necesarias con el controlador de dominio de Active Directory o el servidor DNS.

| Protocolo y Puerto | Propósito |
|-------------------|---------|
| TCP y UDP 53 | DNS |
| TCP y UDP 88 | Kerberos |
| TCP 139 | Net |
| TCP y UDP 389 | LDAP |
| TCP 636 | SSL DE LDAP |

Es posible que tenga que permitir el acceso a puertos adicionales en función del tipo de tráfico que el contenedor envía a un controlador de dominio.
Consulte [requisitos de puertos de Active Directory y servicios de dominio de Active](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers) Directory para obtener una lista completa de los puertos usados por Active Directory.

### <a name="check-the-container"></a>Comprobar el contenedor

1. Si está ejecutando una versión de Windows anterior a Windows Server 2019 o Windows 10, versión 1809, el nombre de host del contenedor debe coincidir con el nombre de gMSA. Asegúrese de `--hostname` que el parámetro coincide con el nombre corto gMSA (sin componente de dominio; por ejemplo, "webapp01" en lugar de "webapp01.contoso.com").

2. Compruebe la configuración de la red del contenedor para comprobar que el contenedor puede resolver y obtener acceso a un controlador de dominio para el dominio de gMSA. Los servidores DNS mal configurados en el contenedor son una causa común de problemas de identidad.

3. Compruebe si el contenedor tiene una conexión válida con el dominio ejecutando el siguiente cmdlet en el contenedor (usando `docker exec` o un equivalente):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    La verificación de confianza debe `NERR_SUCCESS` ser devuelto si el gMSA está disponible y la conectividad de red permite al contenedor hablar con el dominio. Si se produce un error, verifique la configuración de red del host y el contenedor. Ambos deben poder comunicarse con el controlador de dominio.

4. Asegúrate de que tu aplicación esté [configurada para usar el gMSA](gmsa-configure-app.md). La cuenta de usuario dentro del contenedor no cambia cuando usa un gMSA. En su lugar, la cuenta del sistema usa el gMSA cuando se comunica con otros recursos de red. Esto significa que la aplicación tendrá que ejecutarse como servicio de red o sistema local para aprovechar la identidad gMSA.

    > [!TIP]
    > Si ejecuta `whoami` o usa otra herramienta para identificar el contexto de usuario actual en el contenedor, no verá el nombre del gMSA. Esto se debe a que siempre inicia sesión en el contenedor como un usuario local en lugar de una identidad de dominio. El gMSA es utilizado por la cuenta de equipo cada vez que se comunica con recursos de red, razón por la cual la aplicación necesita ejecutarse como servicio de red o sistema local.

5. Por último, si el contenedor parece estar configurado correctamente pero los usuarios u otros servicios no se pueden autenticar automáticamente en la aplicación contenedora, compruebe los SPN en su cuenta de gMSA. Los clientes encontrarán la cuenta gMSA por el nombre en el que llegan a la aplicación. Esto puede significar que necesitarás SPN adicionales `host` para tu gMSA si, por ejemplo, los clientes se conectan a la aplicación a través de un equilibrador de carga o un nombre DNS diferente.
