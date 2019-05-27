---
title: Cuentas de servicio administradas de grupo para contenedores de Windows
description: Cuentas de servicio administradas de grupo para contenedores de Windows
keywords: acoplador, contenedores, Active Directory, GMSA
author: rpsqrd
ms.date: 05/23/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 8f184e58743bd41ab208b530976772c5fcffd189
ms.sourcegitcommit: 8e7fba17c761bf8f80017ba7f9447f986a20a2a7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/24/2019
ms.locfileid: "9677324"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>Cuentas de servicio administradas de grupo para contenedores de Windows

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
# To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -Scope DomainLocal

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
5. Práctica Compruebe que el host puede usar la cuenta gMSA ejecutando [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount). Si el comando devuelve **false**, consulte la sección de [solución de problemas](#troubleshooting) para conocer los pasos de diagnóstico.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
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
    - Para Windows 10, versión 1809 o posterior, ejecute **install-WindowsCapability-online ' RSAT. ActiveDirectory. DS-LDS. Tools ~ ~ ~ ~ 0.0.1.0 '**.
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

## <a name="configure-your-application-to-use-the-gmsa"></a>Configurar la aplicación para que use el gMSA

En la configuración típica, un contenedor solo tiene una cuenta de gMSA que se usa siempre que la cuenta del equipo de contenedor intenta autenticarse en los recursos de red. Esto significa que la aplicación tendrá que ejecutarse como **sistema local** o **servicio de red** si necesita usar la identidad gMSA.

### <a name="run-an-iis-app-pool-as-network-service"></a>Ejecutar un grupo de aplicaciones de IIS como servicio de red

Si está hospedando un sitio web de IIS en su contenedor, todo lo que tiene que hacer para aprovechar el gMSA es establecer la identidad del grupo de aplicaciones en **servicio de red**. Puede hacerlo en su Dockerfile agregando el siguiente comando:

```dockerfile
RUN (Get-IISAppPool DefaultAppPool).ProcessModel.IdentityType = "NetworkService"
```

Si previamente usó credenciales de usuario estáticas para el grupo de aplicaciones de IIS, considere la gMSA como el reemplazo de esas credenciales. Puede cambiar el gMSA entre los entornos de desarrollo, prueba y producción, y IIS recogerá automáticamente la identidad actual sin necesidad de cambiar la imagen del contenedor.

### <a name="run-a-windows-service-as-network-service"></a>Ejecutar un servicio de Windows como servicio de red

Si la aplicación de contenedor se ejecuta como un servicio de Windows, puede configurar el servicio para que se ejecute como **servicio de red** en su Dockerfile:

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>Ejecutar aplicaciones de consola arbitrarias como servicio de red

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

## <a name="run-a-container-with-a-gmsa"></a>Ejecutar un contenedor con un gMSA

Para ejecutar un contenedor con un gMSA, proporcione el archivo de especificación de credenciales en el parámetro de la `--security-opt` [ejecución del acoplador](https://docs.docker.com/engine/reference/run):

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

Si el estado de la conexión DC de confianza y el estado `NERR_Success`de verificación de confianza no son, consulte la sección de [solución de problemas](#troubleshooting) para obtener sugerencias sobre cómo depurar el problema.

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

## <a name="orchestrate-containers-with-gmsa"></a>Orquestar contenedores con gMSA

En los entornos de producción, a menudo usas un Orchestrator de contenedor para implementar y administrar tus aplicaciones y servicios. Cada organizador tiene sus propios paradigmas de administración y es responsable de aceptar las especificaciones de credenciales para dar a la plataforma contenedor de Windows.

Cuando vaya a orquestar contenedores con gMSAs, asegúrese de que:

> [!div class="checklist"]
> * Todos los hosts de contenedor que se pueden programar para ejecutar contenedores con gMSAs están Unidos a un dominio
> * Los hosts contenedores tienen acceso para recuperar las contraseñas de todos los gMSAs usados por contenedores
> * Los archivos de especificación de credenciales se crean y se cargan en el Orchestrator o se copian en cada uno de ellos, en función de cómo prefiera administrar el Orchestrator.
> * Las redes de contenedor permiten que los contenedores se comuniquen con los controladores de dominio de Active Directory para recuperar vales de gMSA

### <a name="how-to-use-gmsa-with-service-fabric"></a>Cómo usar gMSA con Service fabric

Service fabric admite la ejecución de contenedores de Windows con un gMSA cuando especifica la ubicación de las especificaciones de credenciales en el manifiesto de la aplicación. Tendrá que crear el archivo de especificación de credenciales y colocarlo en el subdirectorio **CredentialSpecs** del directorio de datos de Dock en cada host para que Service fabric pueda encontrarlo. Puede ejecutar el cmdlet **Get-CredentialSpec** , que forma parte del [módulo de PowerShell CredentialSpec](https://aka.ms/credspec), para comprobar si la especificación de credenciales está en la ubicación correcta.

Vea [Inicio rápido: implementar contenedores de Windows en Service fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) y [configurar gMSA para los contenedores de Windows que se ejecutan en fabric Service](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) para obtener más información sobre cómo configurar la aplicación.

### <a name="how-to-use-gmsa-with-docker-swarm"></a>Cómo usar gMSA con Swarm de acoplamiento

Para usar un gMSA con contenedores administrados por Dock Swarm, ejecute el comando [crear del servicio](https://docs.docker.com/engine/reference/commandline/service_create/) del acoplador `--credential-spec` con el parámetro:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Para obtener más información sobre cómo usar las especificaciones de credenciales con los servicios de acoplamiento, vea el [ejemplo Swarm](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) .

### <a name="how-to-use-gmsa-with-kubernetes"></a>Cómo usar gMSA con Kubernetes

La compatibilidad con la programación de contenedores de Windows con gMSAs en Kubernetes está disponible como una característica alfa en Kubernetes 1,14. Consulte [Configure gMSA for Windows pods and containers](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) para obtener la información más reciente sobre esta característica y cómo probarla en su distribución de Kubernetes.

## <a name="example-uses"></a>Ejemplo se usa

### <a name="sql-connection-strings"></a>Cadenas de conexión de SQL

Cuando un servicio se ejecuta como sistema local o servicio de red en un contenedor, puede usar la autenticación integrada de Windows para conectarse a Microsoft SQL Server.

Este es un ejemplo de una cadena de conexión que usa la identidad del contenedor para autenticar a SQL Server:

```sql
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

En Microsoft SQL Server, cree un inicio de sesión con el nombre de dominio y gMSA, seguido por un signo $. Una vez que haya creado el inicio de sesión, puede agregarlo a un usuario en una base de datos y darle los permisos de acceso apropiados.

Ejemplo:

```sql
CREATE LOGIN "DEMO\WebApplication1$"
    FROM WINDOWS
    WITH DEFAULT_DATABASE = "MusicStore"
GO

USE MusicStore
GO
CREATE USER WebApplication1 FOR LOGIN "DEMO\WebApplication1$"
GO

EXEC sp_addrolemember 'db_datareader', 'WebApplication1'
EXEC sp_addrolemember 'db_datawriter', 'WebApplication1'
```

Para verlo en acción, consulte la [demostración grabada](https://youtu.be/cZHPz80I-3s?t=2672) disponible en el encendedor de Microsoft 2016 en la sesión "recorrer el camino a la depuración: transformar las cargas de trabajo en contenedores".

## <a name="troubleshooting"></a>Solución de problemas

### <a name="known-issues"></a>Problemas conocidos

#### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>El nombre de host del contenedor debe coincidir con el gMSA de Windows Server 2016 y Windows 10, versiones 1709 y 1803

Si está ejecutando Windows Server 2016, versión 1709 o 1803, el nombre de host del contenedor debe coincidir con el nombre de la cuenta SAM de gMSA.

Cuando el nombre de host no coincide con el nombre del gMSA, las solicitudes de autenticación NTLM de entrada y la traducción de nombres/SID (usada por muchas bibliotecas, como el proveedor de roles de pertenencia a ASP.NET) producirán un error. Kerberos seguirá funcionando normalmente incluso si el nombre de host y el nombre de gMSA no coinciden.

Esta limitación se corrigió en Windows Server 2019, donde ahora el contenedor siempre usará su nombre de gMSA en la red, independientemente del nombre de host asignado.

#### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Usar un gMSA con más de un contenedor de forma simultánea provoca errores intermitentes en Windows Server 2016 y Windows 10, versiones 1709 y 1803

Debido a que todos los contenedores deben usar el mismo nombre de host, un segundo problema afecta a las versiones de Windows anteriores a Windows Server 2019 y Windows 10, versión 1809. Cuando se asigna la misma identidad y nombre de host a varios contenedores, se puede producir una condición de carrera cuando dos contenedores hablan simultáneamente con el mismo controlador de dominio. Cuando otro contenedor se comunique con el mismo controlador de dominio, cancelará la comunicación con cualquier contenedor anterior que use la misma identidad. Esto puede provocar errores de autenticación intermitentes y, a veces, se puede observar como un error `nltest /sc_verify:contoso.com` de confianza cuando se ejecuta dentro del contenedor.

Hemos cambiado el comportamiento en Windows Server 2019 para separar la identidad del contenedor del nombre de la máquina, lo que permite que varios contenedores usen el mismo gMSA al mismo tiempo.

#### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>No puede usar gMSAs con contenedores aislados de Hyper-V en las versiones 1703, 1709 y 1803 de Windows 10

La inicialización del contenedor se bloqueará o fallará al intentar usar un gMSA con un contenedor aislado de Hyper-V en Windows 10 y Windows Server versiones 1703, 1709 y 1803.

Este error se corrigió en Windows Server 2019 y Windows 10, versión 1809. También puede ejecutar contenedores aislados de Hyper-V con gMSAs en Windows Server 2016 y Windows 10, versión 1607.

### <a name="general-troubleshooting-guidance"></a>Instrucciones para la solución de problemas generales

Si encuentra errores al ejecutar un contenedor con un gMSA, las siguientes instrucciones pueden ayudarle a identificar la causa raíz.

#### <a name="make-sure-the-host-can-use-the-gmsa"></a>Asegúrese de que el anfitrión puede usar el gMSA

1. Compruebe que el host está unido al dominio y que puede comunicarse con el controlador de dominio.
2. Instale las herramientas de AD PowerShell de RSAT y ejecute [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) para ver si el equipo tiene acceso para recuperar el gMSA. Si el cmdlet devuelve **false**, significa que el equipo no tiene acceso a la contraseña de gMSA.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
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

#### <a name="check-the-firewall-configuration"></a>Comprobar la configuración del firewall

Si está usando una directiva de Firewall estricta en el contenedor o la red host, puede bloquear las conexiones necesarias con el controlador de dominio de Active Directory o el servidor DNS.

| Protocolo y Puerto | Propósito |
|-------------------|---------|
| TCP y UDP 53 | DNS |
| TCP y UDP 88 | Kerberos |
| TCP 139 | Net |
| TCP y UDP 389 | LDAP |
| TCP 636 | SSL DE LDAP |

Es posible que tenga que permitir el acceso a puertos adicionales en función del tipo de tráfico que el contenedor envía a un controlador de dominio.
Consulte [requisitos de puertos de Active Directory y servicios de dominio de Active](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers) Directory para obtener una lista completa de los puertos usados por Active Directory.

#### <a name="check-the-container"></a>Comprobar el contenedor

1. Si está ejecutando una versión de Windows anterior a Windows Server 2019 o Windows 10, versión 1809, el nombre de host del contenedor debe coincidir con el nombre de gMSA. Asegúrese de `--hostname` que el parámetro coincide con el nombre corto gMSA (sin componente de dominio; por ejemplo, "webapp01" en lugar de "webapp01.contoso.com").

2. Compruebe la configuración de la red del contenedor para comprobar que el contenedor puede resolver y obtener acceso a un controlador de dominio para el dominio de gMSA. Los servidores DNS mal configurados en el contenedor son una causa común de problemas de identidad.

3. Compruebe si el contenedor tiene una conexión válida con el dominio ejecutando el siguiente cmdlet en el contenedor (usando `docker exec` o un equivalente):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    La verificación de confianza debe `NERR_SUCCESS` ser devuelto si el gMSA está disponible y la conectividad de red permite al contenedor hablar con el dominio. Si se produce un error, verifique la configuración de red del host y el contenedor. Ambos deben poder comunicarse con el controlador de dominio.

4. Asegúrate de que tu aplicación esté configurada [para usar el gMSA](#configure-your-application-to-use-the-gmsa). La cuenta de usuario dentro del contenedor no cambia cuando usa un gMSA. En su lugar, la cuenta del sistema usa el gMSA cuando se comunica con otros recursos de red. Esto significa que la aplicación tendrá que ejecutarse como servicio de red o sistema local para aprovechar la identidad gMSA.

    > [!TIP]
    > Si ejecuta `whoami` o usa otra herramienta para identificar el contexto de usuario actual en el contenedor, no verá el nombre del gMSA. Esto se debe a que siempre inicia sesión en el contenedor como un usuario local en lugar de una identidad de dominio. El gMSA es utilizado por la cuenta de equipo cada vez que se comunica con recursos de red, razón por la cual la aplicación necesita ejecutarse como servicio de red o sistema local.

5. Por último, si el contenedor parece estar configurado correctamente pero los usuarios u otros servicios no se pueden autenticar automáticamente en la aplicación contenedora, compruebe los SPN en su cuenta de gMSA. Los clientes encontrarán la cuenta gMSA por el nombre en el que llegan a la aplicación. Esto puede significar que necesitarás SPN adicionales `host` para tu gMSA si, por ejemplo, los clientes se conectan a la aplicación a través de un equilibrador de carga o un nombre DNS diferente.

## <a name="additional-resources"></a>Recursos adicionales

- [Introducción a las cuentas de servicio administradas de grupo](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
