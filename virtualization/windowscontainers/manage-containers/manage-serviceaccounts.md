---
title: Crear GMSA para contenedores de Windows
description: Cómo crear cuentas de servicio administradas de grupo (GMSA) para contenedores de Windows.
keywords: Docker, contenedores, Active Directory, GMSA, cuenta de servicio administrada de grupo, cuentas de servicio administradas de grupo
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 9ed9029e534d56bfe1830281d0bfd3ddde0cee9e
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910255"
---
# <a name="create-gmsas-for-windows-containers"></a>Crear GMSA para contenedores de Windows

Las redes basadas en Windows suelen usar Active Directory (AD) para facilitar la autenticación y autorización entre usuarios, equipos y otros recursos de red. A menudo, los desarrolladores de aplicaciones empresariales diseñan sus aplicaciones para que estén integradas en AD y se ejecuten en servidores Unidos a un dominio para aprovechar la autenticación integrada de Windows, lo que facilita que los usuarios y otros servicios inicien sesión de forma transparente y automática en la aplicación con sus identidades.

Aunque los contenedores de Windows no pueden estar Unidos a un dominio, pueden seguir usando Active Directory identidades de dominio para admitir varios escenarios de autenticación.

Para ello, puede configurar un contenedor de Windows para que se ejecute con una [cuenta de servicio administrada de grupo](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA), que es un tipo especial de cuenta de servicio que se presentó en Windows Server 2012 diseñado para permitir que varios equipos compartan una identidad sin necesidad de conocer su contraseña.

Al ejecutar un contenedor con un gMSA, el host de contenedor recupera la contraseña de gMSA de un Active Directory controlador de dominio y la asigna a la instancia del contenedor. El contenedor usará las credenciales de gMSA cada vez que su cuenta de equipo (sistema) necesite tener acceso a los recursos de red.

En este artículo se explica cómo empezar a usar cuentas de servicio administradas de grupo de Active Directory con contenedores de Windows.

## <a name="prerequisites"></a>Requisitos previos

Para ejecutar un contenedor de Windows con una cuenta de servicio administrada de grupo, necesitará lo siguiente:

- Un dominio de Active Directory con al menos un controlador de dominio que ejecute Windows Server 2012 o posterior. No hay ningún requisito de nivel funcional de bosque o dominio para usar GMSA, pero las contraseñas de gMSA solo pueden distribuirse mediante controladores de dominio que ejecuten Windows Server 2012 o posterior. Para obtener más información, consulte [requisitos de Active Directory para GMSA](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req).
- Permiso para crear una cuenta de gMSA. Para crear una cuenta de gMSA, debe ser un administrador de dominio o usar una cuenta que tenga el permiso *Create MSDS-GroupManagedServiceAccount Objects* .
- Acceso a Internet para descargar el módulo de PowerShell CredentialSpec. Si está trabajando en un entorno desconectado, puede [guardar el módulo](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1) en un equipo con acceso a Internet y copiarlo en el equipo de desarrollo o en el host de contenedor.

## <a name="one-time-preparation-of-active-directory"></a>Preparación única de Active Directory

Si aún no ha creado un gMSA en el dominio, deberá generar la clave raíz del servicio de distribución de claves (KDS). KDS es responsable de crear, girar y liberar la contraseña de gMSA en los hosts autorizados. Cuando un host de contenedor necesita usar gMSA para ejecutar un contenedor, se pondrá en contacto con KDS para recuperar la contraseña actual.

Para comprobar si ya se ha creado la clave raíz de KDS, ejecute el siguiente cmdlet de PowerShell como administrador de dominio en un controlador de dominio o miembro de dominio con las herramientas de PowerShell de AD instaladas:

```powershell
Get-KdsRootKey
```

Si el comando devuelve un identificador de clave, se establece todo y puede ir directamente a la sección [creación de una cuenta de servicio administrada de grupo](#create-a-group-managed-service-account) . De lo contrario, continúe con para crear la clave raíz KDS.

En un entorno de producción o un entorno de prueba con varios controladores de dominio, ejecute el siguiente cmdlet en PowerShell como administrador de dominio para crear la clave raíz KDS.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

Aunque el comando implica que la clave entrará en vigor de inmediato, tendrá que esperar 10 horas para que la clave raíz KDS se replique y esté disponible para su uso en todos los controladores de dominio.

Si solo tiene un controlador de dominio en el dominio, puede acelerar el proceso estableciendo la clave para que sea efectiva hace 10 horas.

>[!IMPORTANT]
>No utilice esta técnica en un entorno de producción.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>Crear una cuenta de servicio administrada de grupo

Cada contenedor que utiliza la autenticación integrada de Windows necesita al menos un gMSA. El gMSA principal se usa siempre que las aplicaciones que se ejecutan como un sistema o un servicio de red acceden a los recursos de la red. El nombre del gMSA se convertirá en el nombre del contenedor en la red, independientemente del nombre de host que se haya asignado al contenedor. Los contenedores también se pueden configurar con GMSA adicionales, en caso de que desee ejecutar un servicio o una aplicación en el contenedor como una identidad diferente de la cuenta de equipo del contenedor.

Cuando se crea una gMSA, también se crea una identidad compartida que se puede usar simultáneamente en muchas máquinas diferentes. El acceso a la contraseña de gMSA está protegido por una lista de Access Control de Active Directory. Se recomienda crear un grupo de seguridad para cada cuenta de gMSA y agregar los hosts de contenedor correspondientes al grupo de seguridad para limitar el acceso a la contraseña.

Por último, dado que los contenedores no registran automáticamente ningún nombre de entidad de seguridad de servicio (SPN), tendrá que crear manualmente al menos un SPN de host para la cuenta de gMSA.

Normalmente, el host o el SPN http se registran con el mismo nombre que la cuenta gMSA, pero puede que tenga que usar un nombre de servicio diferente si los clientes tienen acceso a la aplicación en contenedor desde detrás de un equilibrador de carga o un nombre DNS diferente del nombre de gMSA.

Por ejemplo, si la cuenta gMSA se denomina "WebApp01", pero los usuarios acceden al sitio en `mysite.contoso.com`, debe registrar un SPN `http/mysite.contoso.com` en la cuenta gMSA.

Algunas aplicaciones pueden requerir SPN adicionales para sus protocolos únicos. Por ejemplo, SQL Server requiere el SPN `MSSQLSvc/hostname`.

En la tabla siguiente se enumeran los atributos necesarios para crear un gMSA.

|propiedad gMSA | Valor obligatorio | Ejemplo |
|--------------|----------------|--------|
|Nombre | Cualquier nombre de cuenta válido. | `WebApp01` |
|ServicePrincipalName | El nombre de dominio anexado al nombre de la cuenta. | `WebApp01.contoso.com` |
|ServicePrincipalNames | Establezca al menos el SPN del host, agregue otros protocolos según sea necesario. | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | El grupo de seguridad que contiene los hosts del contenedor. | `WebApp01Hosts` |

Una vez que haya decidido el nombre del gMSA, ejecute los siguientes cmdlets en PowerShell para crear el grupo de seguridad y gMSA.

> [!TIP]
> Tendrá que usar una cuenta que pertenezca al grupo de seguridad **Admins** . del dominio o se le haya delegado el permiso **Create MSDS-GroupManagedServiceAccount Objects** para ejecutar los siguientes comandos.
> El cmdlet [New-ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) forma parte de las herramientas de POWERSHELL de AD de [herramientas de administración remota del servidor](https://aka.ms/rsat).

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

Se recomienda crear cuentas de gMSA independientes para los entornos de desarrollo, prueba y producción.

## <a name="prepare-your-container-host"></a>Preparación del host de contenedor

Cada host de contenedor que ejecutará un contenedor de Windows con un gMSA debe estar unido a un dominio y tener acceso para recuperar la contraseña de gMSA.

1. Una el equipo al dominio de Active Directory.
2. Asegúrese de que el host pertenece al grupo de seguridad que controla el acceso a la contraseña de gMSA.
3. Reinicie el equipo para que obtenga su nueva pertenencia al grupo.
4. Configure [Docker Desktop para Windows 10](https://docs.docker.com/docker-for-windows/install/) o [Docker para Windows Server](https://docs.docker.com/install/windows/docker-ee/).
5. Recomendar Compruebe que el host puede usar la cuenta de gMSA ejecutando [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount). Si el comando devuelve **false**, siga las [instrucciones de solución de problemas](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa).

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>Crear una especificación de credenciales

Un archivo de especificación de credenciales es un documento JSON que contiene metadatos sobre las cuentas de gMSA que desea que use un contenedor. Al mantener la configuración de identidad separada de la imagen de contenedor, puede cambiar la gMSA que usa el contenedor simplemente intercambiando el archivo de especificación de credenciales, sin que sea necesario realizar cambios en el código.

El archivo de especificación de credenciales se crea mediante el [módulo de PowerShell CredentialSpec](https://aka.ms/credspec) en un host de contenedor unido a un dominio.
Una vez que haya creado el archivo, puede copiarlo en otros hosts de contenedor o en el orquestador de contenedores.
El archivo de especificación de credenciales no contiene ningún secreto, como la contraseña de gMSA, ya que el host de contenedor recupera el gMSA en nombre del contenedor.

Docker espera encontrar el archivo de especificación de credenciales en el directorio **CredentialSpecs** del directorio de datos de Docker. En una instalación predeterminada, encontrará esta carpeta en `C:\ProgramData\Docker\CredentialSpecs`.

Para crear un archivo de especificación de credenciales en el host de contenedor:

1. Instalación de las herramientas de PowerShell para AD de RSAT
    - En Windows Server, ejecute **install-WINDOWSFEATURE RSAT-ad-PowerShell**.
    - En Windows 10, versión 1809 o posterior, ejecute **Add-WindowsCapability-online-name ' RSAT. ActiveDirectory. DS-LDS. Tools ~ ~ ~ ~ 0.0.1.0 '** .
    - Para versiones anteriores de Windows 10, consulte <https://aka.ms/rsat>.
2. Ejecute el siguiente cmdlet para instalar la versión más reciente del [módulo CredentialSpec de PowerShell](https://aka.ms/credspec):

    ```powershell
    Install-Module CredentialSpec
    ```

    Si no tiene acceso a Internet en el host de contenedor, ejecute `Save-Module CredentialSpec` en un equipo conectado a Internet y copie la carpeta del módulo en `C:\Program Files\WindowsPowerShell\Modules` u otra ubicación en `$env:PSModulePath` en el host del contenedor.

3. Ejecute el siguiente cmdlet para crear el nuevo archivo de especificación de credenciales:

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    De forma predeterminada, el cmdlet creará una especificación CRED usando el nombre gMSA proporcionado como cuenta de equipo para el contenedor. El archivo se guardará en el directorio Docker CredentialSpecs con el dominio y el nombre de cuenta de gMSA para el nombre de archivo.

    Puede crear una especificación de credenciales que incluya cuentas de gMSA adicionales si está ejecutando un servicio o proceso como un gMSA secundario en el contenedor. Para ello, use el parámetro `-AdditionalAccounts`:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    Para obtener una lista completa de los parámetros admitidos, ejecute `Get-Help New-CredentialSpec`.

4. Puede mostrar una lista de todas las especificaciones de credenciales y su ruta de acceso completa con el siguiente cmdlet:

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha configurado la cuenta de gMSA, puede usarla para:

- [Configurar aplicaciones](gmsa-configure-app.md)
- [Ejecutar contenedores](gmsa-run-container.md)
- [Orquestación de contenedores](gmsa-orchestrate-containers.md)

Si surgen problemas durante la instalación, consulte nuestra [Guía de solución de problemas](gmsa-troubleshooting.md) para ver las posibles soluciones.

## <a name="additional-resources"></a>Recursos adicionales

- Para obtener más información sobre GMSA, consulte la [información general sobre las cuentas de servicio administradas de grupo](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).
- Para ver una demostración en vídeo, vea nuestra [demostración grabada](https://youtu.be/cZHPz80I-3s?t=2672) de encendido 2016.
