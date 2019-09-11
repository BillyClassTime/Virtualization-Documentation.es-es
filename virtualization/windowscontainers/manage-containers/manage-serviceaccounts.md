---
title: Crear gMSAs para contenedores de Windows
description: Cómo crear cuentas de servicio administradas de grupo (gMSAs) para contenedores de Windows.
keywords: acoplador, contenedores, Active Directory, GMSA, cuenta de servicio administrado de grupo, cuentas de servicio administradas de grupo
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 9ed9029e534d56bfe1830281d0bfd3ddde0cee9e
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079669"
---
# <a name="create-gmsas-for-windows-containers"></a>Crear gMSAs para contenedores de Windows

Las redes basadas en Windows suelen usar Active Directory (AD) para facilitar la autenticación y la autorización entre usuarios, equipos y otros recursos de red. Los desarrolladores de aplicaciones empresariales suelen diseñar sus aplicaciones para que se integren en AD y se ejecuten en servidores Unidos a un dominio para aprovechar la autenticación de Windows integrada, lo que facilita que los usuarios y otros servicios inicien sesión de forma automática y transparente la aplicación con sus identidades.

Aunque los contenedores de Windows no pueden unirse a un dominio, aún pueden usar las identidades de dominio de Active Directory para admitir diversos escenarios de autenticación.

Para ello, puede configurar un contenedor de Windows para que se ejecute con una [cuenta de servicio administrada de grupo](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA), que es un tipo especial de cuenta de servicio introducido en Windows Server 2012 diseñada para permitir que varios equipos compartan una identidad sin necesidad de para conocer su contraseña.

Cuando se ejecuta un contenedor con un gMSA, el host de contenedor recupera la contraseña de gMSA de un controlador de dominio de Active Directory y la asigna a la instancia de contenedor. El contenedor usará las credenciales de gMSA cada vez que su cuenta de equipo (sistema) necesite obtener acceso a los recursos de red.

En este artículo se explica cómo empezar a usar cuentas de servicio administradas de grupo de Active Directory con contenedores de Windows.

## <a name="prerequisites"></a>Requisitos previos

Para ejecutar un contenedor de Windows con una cuenta de servicio administrada de grupo, necesitará lo siguiente:

- Un dominio de Active Directory con al menos un controlador de dominio que ejecute Windows Server 2012 o posterior. No hay requisitos de nivel funcional de bosque o dominio para usar gMSAs, pero las gMSA solo pueden ser distribuidas por controladores de dominio que ejecuten Windows Server 2012 o posterior. Para obtener más información, consulte [requisitos de Active Directory para gMSAs](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req).
- Permiso para crear una cuenta de gMSA. Para crear una cuenta de gMSA, debe ser administrador de dominio o usar una cuenta a la que se haya delegado el permiso *Create MSDS-GroupManagedServiceAccount Objects* .
- Acceso a Internet para descargar el módulo de PowerShell CredentialSpec. Si está trabajando en un entorno desconectado, puede [guardar el módulo](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1) en un equipo con acceso a Internet y copiarlo en el equipo de desarrollo o el host contenedor.

## <a name="one-time-preparation-of-active-directory"></a>Preparación única de Active Directory

Si todavía no ha creado una gMSA en su dominio, tendrá que generar la clave raíz del servicio de distribución de claves (KDS). El KDS es responsable de crear, girar y liberar la contraseña de gMSA a los hosts autorizados. Cuando un host de contenedor necesita usar el gMSA para ejecutar un contenedor, se pondrá en contacto con el KDS para recuperar la contraseña actual.

Para comprobar si ya se ha creado la clave raíz de KDS, ejecute el siguiente cmdlet de PowerShell como administrador de dominio en un controlador de dominio o miembro de dominio con las herramientas de PowerShell de AD instaladas:

```powershell
Get-KdsRootKey
```

Si el comando devuelve un identificador de clave, ya está todo listo y puede ir directamente a la sección [crear una cuenta de servicio administrada de grupo](#create-a-group-managed-service-account) . En caso contrario, continúe con la creación de la clave raíz KDS.

En un entorno de producción o un entorno de prueba con varios controladores de dominio, ejecute el siguiente cmdlet en PowerShell como administrador de dominio para crear la clave raíz KDS.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

Aunque el comando implica que la tecla se hará efectivo inmediatamente, tendrá que esperar 10 horas para que la clave raíz de KDS se replique y esté disponible para su uso en todos los controladores de dominio.

Si solo tiene un controlador de dominio en el dominio, puede acelerar el proceso configurando la clave para que sea efectiva hace 10 horas.

>[!IMPORTANT]
>No use esta técnica en un entorno de producción.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>Crear una cuenta de servicio administrada de grupo

Cada contenedor que usa la autenticación de Windows integrada necesita al menos una gMSA. El gMSA principal se usa siempre que las aplicaciones que se ejecutan como un sistema o un servicio de red tienen acceso a los recursos de la red. El nombre de la gMSA se convertirá en el nombre del contenedor en la red, independientemente del nombre de host asignado al contenedor. Los contenedores también se pueden configurar con gMSAs adicionales, en caso de que desee ejecutar un servicio o una aplicación en el contenedor como una identidad diferente de la cuenta del equipo de contenedor.

Al crear un gMSA, también puede crear una identidad compartida que se puede usar de forma simultánea en muchos equipos diferentes. El acceso a la contraseña de gMSA está protegido por una lista de control de acceso de Active Directory. Recomendamos crear un grupo de seguridad para cada cuenta de gMSA y agregar los hosts de contenedor relevantes al grupo de seguridad para limitar el acceso a la contraseña.

Por último, dado que los contenedores no registran automáticamente ningún nombre principal de servicio (SPN), tendrá que crear manualmente al menos un SPN de host para su cuenta de gMSA.

Normalmente, el SPN de host o http se registra con el mismo nombre que la cuenta de gMSA, pero es posible que tenga que usar un nombre de servicio diferente si los clientes obtienen acceso a la aplicación contenedora desde detrás de un equilibrador de carga o un nombre DNS que es diferente del nombre de gMSA.

Por ejemplo, si la cuenta gMSA se denomina "WebApp01", pero los usuarios tienen acceso al `mysite.contoso.com`sitio en, debe registrar `http/mysite.contoso.com` un SPN en la cuenta de gMSA.

Es posible que algunas aplicaciones requieran SPN adicionales para sus protocolos únicos. Por ejemplo, SQL Server requiere el `MSSQLSvc/hostname` SPN.

En la tabla siguiente se enumeran los atributos necesarios para crear un gMSA.

|propiedad gMSA | Valor obligatorio | Ejemplo |
|--------------|----------------|--------|
|Nombre | Cualquier nombre de cuenta válido. | `WebApp01` |
|ServicePrincipalName | El nombre de dominio anexado al nombre de la cuenta. | `WebApp01.contoso.com` |
|ServicePrincipalNames | Establezca como mínimo el SPN de host, agregue otros protocolos según sea necesario. | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | El grupo de seguridad que contiene los hosts del contenedor. | `WebApp01Hosts` |

Una vez que haya decidido el nombre de la gMSA, ejecute los siguientes cmdlets de PowerShell para crear el grupo de seguridad y gMSA.

> [!TIP]
> Tendrá que usar una cuenta que pertenezca al grupo de seguridad **administradores de dominio** o que se haya delegado el permiso **Create MSDS-GroupManagedServiceAccount Objects** para ejecutar los siguientes comandos.
> El cmdlet [New-ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) forma parte de las herramientas de ad PowerShell de las [herramientas de administración remota del servidor](https://aka.ms/rsat).

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -GroupScope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

Le recomendamos que cree cuentas de gMSA independientes para sus entornos de desarrollo, prueba y producción.

## <a name="prepare-your-container-host"></a>Preparar el host de contenedor

Cada host contenedor que ejecute un contenedor de Windows con un gMSA debe estar unido a un dominio y tener acceso para recuperar la contraseña de gMSA.

1. Una el equipo al dominio de Active Directory.
2. Asegúrese de que su host pertenece al grupo de seguridad que controla el acceso a la contraseña de gMSA.
3. Reinicie el equipo para que obtenga la nueva pertenencia a grupos.
4. Configurar el [escritorio de un acoplador para Windows 10](https://docs.docker.com/docker-for-windows/install/) o el [acoplador para Windows Server](https://docs.docker.com/install/windows/docker-ee/).
5. Práctica Compruebe que el host puede usar la cuenta gMSA ejecutando [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount). Si el comando devuelve **false**, siga las [instrucciones para la solución de problemas](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa).

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>Crear una especificación de credenciales

Un archivo de especificaciones de credenciales es un documento JSON que contiene metadatos sobre las cuentas de gMSA que desea que use un contenedor. Al mantener la configuración de identidad separada de la imagen del contenedor, puede cambiar el gMSA que usa el contenedor si simplemente intercambia el archivo de especificación de credenciales; no es necesario ningún cambio de código.

El archivo de especificaciones de credenciales se crea con el [módulo de PowerShell CredentialSpec](https://aka.ms/credspec) en un host de contenedor unido a un dominio.
Una vez que haya creado el archivo, puede copiarlo en otros hosts contenedores o en el Orchestrator de su contenedor.
El archivo de especificaciones de credenciales no contiene ningún secreto, como la contraseña de gMSA, ya que el host del contenedor recupera la gMSA en nombre del contenedor.

El acoplador espera encontrar el archivo de especificaciones de credenciales en el directorio **CredentialSpecs** del directorio de datos del Dock. En una instalación predeterminada, encontrarás esta carpeta en `C:\ProgramData\Docker\CredentialSpecs`.

Para crear un archivo de especificaciones de credenciales en el host contenedor:

1. Instalar las herramientas de la herramienta de PowerShell de RSAT
    - Para Windows Server, ejecute **rsat de install-WindowsFeature-ad-PowerShell**.
    - Para Windows 10, versión 1809 o posterior, ejecute **Add-WindowsCapability-online-name ' RSAT. ActiveDirectory. DS-LDS. Tools ~ ~ ~ ~ 0.0.1.0 '**.
    - Para las versiones anteriores de Windows 10, <https://aka.ms/rsat>consulte.
2. Ejecute el siguiente cmdlet para instalar la última versión del [módulo de PowerShell CredentialSpec](https://aka.ms/credspec):

    ```powershell
    Install-Module CredentialSpec
    ```

    Si no tiene acceso a Internet en el host contenedor, ejecute `Save-Module CredentialSpec` en un equipo conectado a Internet y copie la carpeta del módulo `C:\Program Files\WindowsPowerShell\Modules` o en otra ubicación `$env:PSModulePath` en el host contenedor.

3. Ejecute el siguiente cmdlet para crear el nuevo archivo de especificaciones de credenciales:

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    De forma predeterminada, el cmdlet creará una especificación CRED con el nombre de gMSA proporcionado como la cuenta de equipo para el contenedor. El archivo se guardará en el directorio CredentialSpecs del Dock usando el dominio gMSA y el nombre de cuenta para el nombre de archivo.

    Puede crear una especificación de credenciales que incluya cuentas de gMSA adicionales si está ejecutando un servicio o proceso como un gMSA secundario en el contenedor. Para ello, use el `-AdditionalAccounts` parámetro:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    Para obtener una lista completa de los parámetros admitidos, ejecute `Get-Help New-CredentialSpec`.

4. Puede mostrar una lista de todas las especificaciones de credenciales y su ruta de acceso completa con el siguiente cmdlet:

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>Pasos siguientes

Ahora que has configurado tu cuenta de gMSA, puedes usarlo para:

- [Configurar aplicaciones](gmsa-configure-app.md)
- [Ejecutar contenedores](gmsa-run-container.md)
- [Organizar contenedores](gmsa-orchestrate-containers.md)

Si tiene algún problema durante la instalación, consulte nuestra [Guía de solución de problemas](gmsa-troubleshooting.md) para ver posibles soluciones.

## <a name="additional-resources"></a>Recursos adicionales

- Para obtener más información sobre gMSAs, consulte la [información general de las cuentas de servicio administrados de grupo](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).
- Para obtener una demostración en vídeo, vea nuestra [demostración grabada](https://youtu.be/cZHPz80I-3s?t=2672) de encendido 2016.
