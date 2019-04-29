---
title: Cuentas de servicio administradas de grupo para contenedores de Windows
description: Cuentas de servicio administradas de grupo para contenedores de Windows
keywords: docker, contenedores, active directory, gmsa
author: rpsqrd
ms.date: 03/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 20daa81a571fde23b91e24e9713e37d225870ec0
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9574966"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>Cuentas de servicio administradas de grupo para contenedores de Windows

En una red basada en Windows, es habitual usar Active Directory (AD) para facilitar la autenticación y autorización entre los usuarios, equipos y otros recursos de red. Los desarrolladores de aplicaciones de empresa diseñan suelen ser integrado en AD y se ejecuta en servidores unido al dominio para sacar partido de la autenticación integrada de Windows, lo que facilita el proceso para los usuarios y otros servicios de forma automática y transparente, iniciar sesión en sus aplicaciones la aplicación con su identidad.

Aunque los contenedores de Windows no pueden estar unidos a un dominio, pueden seguir usando las identidades de dominio de Active Directory para admitir los distintos escenarios de autenticación.
Para lograr esto, puedes configurar un contenedor de Windows para ejecutar con una [cuenta de servicio administrada por grupo](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA), un tipo especial de cuenta de servicio que se introdujo en Windows Server 2012 que está diseñado para permitir que varios equipos compartir una identidad sin necesidad de Para conocer su contraseña.
Cuando se ejecuta un contenedor con una gMSA, el host de contenedor recupera la contraseña de la gMSA de un controlador de dominio de Active Directory y le da a la instancia de contenedor.
El contenedor usará las credenciales de gMSA siempre que su cuenta de equipo (SYSTEM) necesita tener acceso a los recursos de red.

En este artículo se explica cómo empezar a usar cuentas de servicio administrada de grupo de Active Directory con los contenedores de Windows.

## <a name="prerequisites"></a>Requisitos previos

Para ejecutar un contenedor de Windows con una cuenta de servicio administrada de grupo, necesitarás lo siguiente:

**Un dominio de Active Directory con al menos un controlador de dominio que ejecute Windows Server 2012 o posterior.**
No hay dominio o bosque funcional nivel requisitos para usar gmsa las, pero solo pueden ser las contraseñas gMSA distribuidas por los controladores de dominio que ejecute Windows Server 2012 o posterior.
Para obtener más información, consulta [los requisitos de Active Directory para gmsa las](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req).



**Permiso para crear una cuenta de gMSA.**
Para crear una cuenta de gMSA, tendrás que ser un administrador de dominio o usar una cuenta que se ha delegado en el permiso de *crear objetos de msDS-GroupManagedServiceAccount* .



**Acceso a internet para descargar el módulo de CredentialSpec PowerShell.**
Si estás trabajando en un entorno desconectado, puedes [Guardar el módulo](https://docs.microsoft.com/en-us/powershell/module/powershellget/save-module?view=powershell-5.1) en un equipo con internet acceder y copiarla en el host de máquina o contenedor de desarrollo.

## <a name="one-time-preparation-of-active-directory"></a>Preparación de un solo uso de Active Directory

Si aún no ha creado una gMSA en tu dominio, es probable que necesitas generar la clave de raíz del servicio de distribución de claves.
El KDS es responsable de crear, girar y liberar la contraseña de gMSA a los hosts autorizados.
Cuando un host de contenedor debe utilizar la gMSA para ejecutar un contenedor, se pondrá en contacto el KDS para recuperar la contraseña actual.

Para comprobar si el KDS raíz clave ya se ha creado, ejecute el siguiente comando de PowerShell como *Administrador de dominio* en un controlador de dominio o un miembro de dominio tenga instaladas las herramientas de AD PowerShell:

```powershell
Get-KdsRootKey
```

Si el comando devuelve una clave de Id., estás establecen y puede avanzar a la sección [crear una cuenta de servicio administrada de grupo](#create-a-group-managed-service-account) .
De lo contrario, seguir para crear la clave de raíz KDS.

En un entorno de producción o el entorno de prueba con varios controladores de dominio, ejecuta el siguiente comando en PowerShell como *Administrador de dominio* para crear la clave de raíz KDS.
Aunque el comando implica que la clave será eficaz inmediatamente, tendrás que esperar 10 horas antes de que la clave de raíz KDS se replica y disponibles para su uso en todos los controladores de dominio.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

Si solo tiene un controlador de dominio de tu dominio, puede acelerar el proceso configurando la clave para que sea eficaz hace 10 horas.
No se usa esta técnica en un entorno de producción.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>Crear una cuenta de servicio administrada de grupo

Cada contenedor que usará la autenticación integrada de Windows necesita al menos una gMSA.
La gMSA principal se usa siempre que las aplicaciones que se ejecuta como *sistema* o *Servicio de red* acceso a recursos de la red.
El nombre de la gMSA se convertirá en nombre del contenedor de la red, independientemente del nombre de host asignado al contenedor.
Los contenedores también pueden configurarse con gmsa adicional las, en caso de que desea ejecutar una aplicación o servicio en el contenedor como una identidad de la cuenta de equipo de contenedor diferentes.

Al crear una gMSA, vas a crear una identidad compartida que se puede usar simultáneamente entre varias máquinas.
Acceso a la contraseña de gMSA está protegido por una lista de Control de acceso de Active Directory.
Te recomendamos crear un grupo de seguridad para cada cuenta de gMSA y agregar los hosts de contenedor relevante para el grupo de seguridad para limitar el acceso a la contraseña.

Por último, dado que los contenedores no se registra automáticamente cualquier nombres principales de servicio (SPN), tendrás que crear manualmente al menos un "host" SPN para tu cuenta de gMSA.
Por lo general, el host o el SPN de http está registrado con el mismo nombre que la cuenta gMSA, pero tienes que usar un nombre de servicio diferente si los clientes acceso a la aplicación en contenedores desde detrás de un equilibrador de carga o el nombre DNS que es diferente del nombre del gMSA.
Por ejemplo, si la cuenta gMSA es *WebApp01* , pero los usuarios obtener acceso al sitio en *mysite.contoso.com* debes registrar un `http/mysite.contoso.com` SPN en la cuenta gMSA.
Algunas aplicaciones pueden requerir SPN adicionales para sus protocolos únicos: por ejemplo, SQL Server requiere el SPN "MSSQLSvc o del host".

La siguiente tabla enumera los atributos requeridos al crear una gMSA.

Propiedad gMSA | Valor requerido | Ejemplo
--------------|----------------|--------
Nombre | Cualquier nombre de cuenta válido. | `WebApp01`
DnsHostName | El nombre de dominio que se anexa al nombre de cuenta. | `WebApp01.contoso.com`
ServicePrincipalNames | Establecer al menos la host SPN, agregar otros protocolos según sea necesario. | `'host/WebApp01', 'host/WebApp01.contoso.com'`
PrincipalsAllowedToRetrieveManagedPassword | El grupo de seguridad que contiene los hosts de contenedor. | `WebApp01Hosts`

Cuando sepas qué vas a llamar a la gMSA, ejecute los siguientes comandos en PowerShell para crear el grupo de seguridad y gMSA.

> [!TIP]
> Tendrás que usar una cuenta que pertenece al grupo de seguridad de **Administradores de dominio** o se ha delegado el permiso de **crear objetos de msDS-GroupManagedServiceAccount** para ejecutar los siguientes comandos.
> El cmdlet [New-ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) es parte de las herramientas de PowerShell de anuncios de [Herramientas de administración remota del servidor](https://aka.ms/rsat).

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

Se recomienda que crees cuentas de gMSA independientes para los entornos de desarrollo, prueba y producción.

## <a name="prepare-your-container-host"></a>Preparar el host de contenedor

Cada host de contenedor que se ejecutará un contenedor de Windows con una gMSA debe ser dominio unido y tiene acceso para recuperar la contraseña de gMSA.

1.  Unir el equipo al dominio de Active Directory.
2.  Asegúrate de que el host que pertenece al grupo de seguridad controlar el acceso a la contraseña de gMSA.
3.  Por lo que obtiene su pertenencia a grupos nuevos, reinicie el equipo.
4.  Configurar [Docker escritorio para Windows 10](https://docs.docker.com/docker-for-windows/install/) o [Docker para Windows Server](https://docs.docker.com/install/windows/docker-ee/).
5.  (Recomendado) Comprueba que el host puede utilizar la cuenta gMSA mediante la ejecución de [Prueba ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/activedirectory/test-adserviceaccount). Si el comando devuelve **False**, consulte la sección de [solución de problemas](#troubleshooting) de los pasos de diagnóstico.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>Crear una especificación de credenciales

Un archivo de especificación de credenciales es un documento JSON que contiene metadatos sobre las cuentas de gMSA que desee que usa un contenedor.
Al mantener la configuración de identidad independiente de la imagen del contenedor, puede cambiar fácilmente la gMSA utilizada por el contenedor con simplemente de intercambio en el archivo de especificación de credenciales: sin cambios de código necesarios.

El archivo de especificación de credenciales se crea mediante el [módulo de CredentialSpec PowerShell](https://aka.ms/credspec) en un host de contenedor unido al dominio.
Cuando hayas creado el archivo, se puede copiar a otros hosts de contenedor o su orquestador de contenedores.
El archivo de especificación de credenciales no contiene ninguna información confidencial, como la contraseña de gMSA, dado que el host de contenedor recupera la gMSA en nombre del contenedor.

Docker espera encontrar el archivo de especificación de credenciales en el directorio **CredentialSpecs** en el directorio de datos de Docker.
En una instalación predeterminada, encontrarás esta carpeta en `C:\ProgramData\Docker\CredentialSpecs`.

Realiza los siguientes pasos para crear un archivo de especificación de credenciales en el host de contenedor:
1.  Instalar las herramientas RSAT AD PowerShell
    -   Para Windows Server, ejecute `Install-WindowsFeature RSAT-AD-PowerShell`
    -   Para Windows 10, versión 1809 o posterior, ejecute `Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'`
    -   Para las versiones anteriores de Windows 10, consultahttps://aka.ms/rsat
2.  Instalar la versión más reciente del [módulo de CredentialSpec PowerShell](https://aka.ms/credspec):

    ```powershell
    Install-Module CredentialSpec
    ```

    Si no tienes acceso a internet en el host de contenedor, ejecutar `Save-Module CredentialSpec` en un equipo conectado a internet y copiar la carpeta del módulo para `C:\Program Files\WindowsPowerShell\Modules` u otra ubicación en `$env:PSModulePath` en el host de contenedor.

3.  Crear el nuevo archivo de especificación de credenciales:

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    De manera predeterminada, el cmdlet creará una especificación de la aplicación con el nombre de gMSA proporcionado como la cuenta de equipo para el contenedor.
    El archivo se guardará en el directorio de Docker CredentialSpecs utilizando el nombre de dominio y la cuenta de gMSA para el nombre de archivo.

    Puedes crear una especificación de credenciales que incluye las cuentas de gMSA adicionales si estás ejecutando un servicio o proceso como una gMSA secundaria en el contenedor.
    Para ello, usa el `-AdditionalAccounts` parámetro:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    Para obtener una lista completa de parámetros admitidos, ejecutar `Get-Help New-CredentialSpec`.

4.  Puedes mostrar una lista de todas las especificaciones de credenciales y su ruta de acceso completa con el siguiente comando:

    ```powershell
    Get-CredentialSpec
    ```

## <a name="configuring-your-application-to-use-the-gmsa"></a>Configurar la aplicación para utilizar la gMSA

En la configuración típica, un contenedor solo dispone de una cuenta de gMSA que se usa siempre que la cuenta de equipo de contenedor intenta autenticar a los recursos de red.
Esto significa que la aplicación para ejecutarse como **Sistema Local** o **Servicio de red** si debe usar la identidad de gMSA.

### <a name="run-an-iis-app-pool-as-network-service"></a>Ejecutar un grupo de aplicación IIS como servicio de red

Si estás hospedando un sitio Web IIS en el contenedor, todo lo que necesitas hacer para aprovechar la gMSA se establece la identidad del grupo de aplicaciones al **Servicio de red**.
Puedes hacerlo en el Dockerfile agregando el siguiente comando:

```dockerfile
RUN (Get-IISAppPool DefaultAppPool).ProcessModel.IdentityType = "NetworkService"
```

Si usaste anteriormente las credenciales de usuario estática para el grupo de aplicaciones de IIS, considera la posibilidad de la gMSA como el reemplazo de esas credenciales.
Puedes cambiar la gMSA entre los entornos de desarrollo, prueba y producción y IIS se seleccionarán automáticamente la identidad actual sin tener que cambiar la imagen del contenedor.

### <a name="run-a-windows-service-as-network-service"></a>Ejecutar un servicio de Windows como servicio de red

Si la aplicación en contenedores se ejecuta como un servicio de Windows, puedes establecer el servicio para ejecutarse como **Servicio de red** en el Dockerfile:

```dockerfile
RUN cmd /c 'sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""'
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>Ejecutar aplicaciones de consola arbitraria como servicio de red

Para las aplicaciones de consola genérico que no se hospedan en IIS o el Administrador de servicios, suele ser más fácil ejecutar el contenedor como **Servicio de red** para que la aplicación hereda automáticamente el contexto de gMSA.
Esta característica está disponible a partir de Windows Server versión 1709.

Agrega la siguiente línea en el Dockerfile hacer que se ejecute como servicio de red de manera predeterminada:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

También puedes conectarte a un contenedor como servicio de red de forma excepcional con `docker exec`.
Esto es especialmente útil si estás solucionar problemas de conectividad en un contenedor en ejecución cuando el contenedor no se ejecuta normalmente como servicio de red.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="run-a-container-with-a-gmsa"></a>Ejecutar un contenedor con una gMSA

Para ejecutar un contenedor con una gMSA, proporcionar el archivo de especificación de credenciales para el `--security-opt` parámetro de [run de docker](https://docs.docker.com/engine/reference/run):

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

En Windows Server 2016, 1709 y 1803, el nombre de host del contenedor *debe coincidir con* la gMSA nombre corto.
En el ejemplo anterior, el nombre de cuenta SAM gMSA es "webapp01" para que el nombre de host de contenedor se establece en el mismo.
En Windows Server 2019 y versiones posteriores, el campo de nombre de host no es necesario, pero el contenedor se sigue identificarse por el nombre de gMSA en lugar del nombre de host, incluso si proporcionas explícitamente uno diferente.

Para comprobar si funciona correctamente la gMSA, ejecute el siguiente comando en el contenedor:

```
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

Si el *Estado de conexión de controlador de dominio de confianza* y *Confiar en el estado de verificación* no están *NERR_Success*, echa un vistazo a la sección de [solución de problemas](#troubleshooting) para obtener sugerencias sobre cómo depurar el problema.

Puedes comprobar la identidad de gMSA desde dentro del contenedor ejecuta el siguiente comando y comprobando el nombre del cliente:

```
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

Para abrir PowerShell o a otra aplicación de consola como la cuenta de gMSA, puede solicitar el contenedor se ejecute en la cuenta de la cuenta en lugar de ContainerAdministrator normal (o ContainerUser para NanoServer) de servicio de red:

```powershell
# NOTE: you can only run as SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Cuando ejecutas como servicio de red, puedes probar la autenticación de red como la gMSA al intentar conectarse a SYSVOL en un controlador de dominio:

```
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="orchestrating-containers-with-gmsa"></a>Orquestación de contenedores con gMSA

En entornos de producción, a menudo se usará un orquestador de contenedores para implementar y administrar tus aplicaciones y servicios.
Cada orchestrator tiene sus propio paradigmas de administración y es responsable de aceptar las especificaciones de credenciales para dar a la plataforma de contenedor de Windows.

Cuando estás orquestar contenedores con gmsa las, comprueba que:
> [!div class="checklist"]
> * Todos los hosts de contenedor que se pueden programar para ejecutar contenedores con gmsa las están unidos a un dominio
> * Los hosts de contenedor tienen acceso para recuperar las contraseñas para todas las gmsa las usados por los contenedores
> * Los archivos de especificación de credenciales se crean y cargados en el orquestador o copia en cada host de contenedor, dependiendo de cómo se prefiere el orchestrator controlarlos.
> * Redes de contenedor permiten a los contenedores para comunicarse con los controladores de dominio de Active Directory para recuperar los vales de gMSA

### <a name="using-gmsa-with-service-fabric"></a>Uso de gMSA con Service Fabric

Service Fabric admite la ejecución de contenedores de Windows con una gMSA cuando se especifica la ubicación de especificación de credenciales en el manifiesto de la aplicación.
Tendrás que crear el archivo de especificación de credenciales y colocar en el subdirectorio **CredentialSpecs** del directorio de datos de Docker en cada host para que pueda localizarla Service Fabric.
El `Get-CredentialSpec` cmdlet, parte del [módulo de CredentialSpec PowerShell](https://aka.ms/credspec), puede usarse para comprobar si la especificación de credenciales se encuentra en la ubicación correcta.

Consulta [Inicio rápido: implementar contenedores de Windows en Service Fabric](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-quickstart-containers) y [Configurar gMSA para contenedores de Windows que se ejecutan en Service Fabric](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) para obtener más información sobre cómo configurar la aplicación.

### <a name="using-gmsa-with-docker-swarm"></a>Uso de gMSA con Docker Swarm

Para utilizar una gMSA con contenedores administrados por el enjambre de Docker, usa el comando de [servicio de docker crear](https://docs.docker.com/engine/reference/commandline/service_create/) con la `--credential-spec` parámetro:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Vea el [ejemplo enjambre de Docker](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) para obtener más información sobre cómo usar las especificaciones de credenciales con servicios de Docker.

### <a name="using-gmsa-with-kubernetes"></a>Uso de gMSA con Kubernetes

Soporte técnico para la programación de los contenedores de Windows con gmsa las en Kubernetes está disponible como una característica de alfa en 1.14 de Kubernetes.
Ver [gMSA de configurar para pods de Windows y los contenedores](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) de la información más reciente acerca de esta característica y la información sobre cómo probarlo en la distribución de Kubernetes.

## <a name="example-uses"></a>Usos de ejemplo

### <a name="sql-connection-strings"></a>Cadenas de conexión de SQL
Cuando un servicio se ejecuta como sistema local o servicio de red en un contenedor, puede usar la autenticación integrada de Windows para conectarse a Microsoft SQL Server.

Este es un ejemplo de una cadena de conexión que utiliza la identidad de contenedor para autenticarse en SQL Server:

```
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

En Microsoft SQL Server, cree un inicio de sesión con el nombre de dominio y gMSA, seguido por un signo $.
Una vez creado el inicio de sesión, puede agregarse a un usuario en una base de datos y concederle los permisos de acceso adecuados.

Por ejemplo: 

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

Para verlo en funcionamiento, echa un vistazo a la [demo grabada](https://youtu.be/cZHPz80I-3s?t=2672) disponible desde Microsoft Ignite 2016 en la sesión "Walk the Path to Containerization - transforming workloads into containers" (Recorre el camino para la creación de contenedores: transformar las cargas de trabajo en contenedores).

## <a name="troubleshooting"></a>Solución de problemas

### <a name="known-issues"></a>Problemas conocidos

**Nombre de host de contenedor debe coincidir con el nombre de la gMSA de WS2016, 1709 y 1803.**

Si estás ejecutando Windows Server 2016, 1709 y 1803, el nombre de host del contenedor debe coincidir con el nombre de cuenta SAM gMSA.
Cuando el nombre de host no coincide con el nombre de gMSA, entrante las solicitudes de autenticación NTLM y traducción SID/nombre (utilizada por muchas de las bibliotecas, como el proveedor de funciones de pertenencia a grupos ASP.NET) se producirá un error.
Kerberos seguirán funcionando con normalidad incluso si no coincide con el nombre de host.

Esta limitación se corrigió en Windows Server 2019, donde el contenedor ahora siempre usará su nombre de gMSA en la red, independientemente del nombre de host asignado.

**Utilizar una gMSA con más de un contenedor simultáneamente conduce a errores intermitentes en WS2016, 1709 y 1803.**

Como resultado de la limitación anterior que requieren todos los contenedores para usar el mismo nombre de host, un segundo problema afecta a las versiones de Windows anteriores a Windows Server 2019 y Windows 10, versión 1809.
Cuando varios contenedores se asignan la misma identidad y el nombre de host, puede producirse una condición de carrera cuando dos contenedores hablar con el mismo controlador de dominio al mismo tiempo.
Cuando otro contenedor se comunica con el mismo controlador de dominio, cancelará la comunicación con los contenedores anteriores con la misma identidad.
Esto puede provocar errores de autenticación intermitente y a veces se puede apreciar como un error de confianza al ejecutar `nltest /sc_verify:contoso.com` dentro del contenedor.

Hemos cambiado el comportamiento de Windows Server 2019 para separar la identidad de contenedor del nombre del equipo, lo que permite a varios contenedores usar la misma gMSA al mismo tiempo.

**No puedes usar gmsa las con contenedores de Hyper-V aislado en Windows, versiones 1703, 1709 y 1803.**

Inicialización del contenedor se bloquea o se producirá un error al intentar utilizar una gMSA con un contenedor aislado de Hyper-V en Windows 10 y Windows Server, versiones 1703, 1709 y 1803.

Este error se corrigió en Windows Server 2019 y Windows 10, versión 1809. También puedes ejecutar contenedores de Hyper-V aislado con gmsa las en Windows Server 2016 y Windows 10, versión 1607.

### <a name="general-troubleshooting-guidance"></a>Instrucciones generales sobre la solución de problemas

Si estás encontrar errores al ejecutar un contenedor con una gMSA, los siguientes pasos pueden ayudarte a identificar la causa.

**Asegúrese de que el host puede utilizar la gMSA**

1.  Comprueba que la host es dominio unido y puede contactar con el controlador de dominio.
2.  Instalar las herramientas de PowerShell de anuncios desde RSAT y ejecutar [ADServiceAccount de prueba](https://docs.microsoft.com/en-us/powershell/module/activedirectory/test-adserviceaccount) para ver si el equipo tiene acceso para recuperar la gMSA. Si el cmdlet devolverá **False**, el equipo no tiene acceso a la contraseña de gMSA.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```
3.  Si la prueba ADServiceAccount devuelve **False**, comprueba que la host pertenece a un grupo de seguridad que tiene acceso para recuperar la contraseña de gMSA.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4.  Si el host pertenece a un grupo de seguridad autorizado para recuperar la contraseña de gMSA, pero aún se produce un error de prueba ADServiceAccount, puede que tengas que reiniciar el equipo para que pueda obtener un nuevo vale que refleja su pertenencia al grupo actual.

**Comprobar el archivo de especificación de credenciales**

1.  Ejecutar `Get-CredentialSpec` desde el [Módulo de PowerShell CredentialSpec](https://aka.ms/credspec) para buscar todas las especificaciones de la máquina de credenciales. Las especificaciones de credenciales deben almacenarse en el directorio "CredentialSpecs" en el directorio raíz de Docker. Puedes encontrar el Docker directorio raíz mediante la ejecución de `docker info -f "{{.DockerRootDir}}"`.
2.  Abre el archivo CredentialSpec y asegúrese de que los campos siguientes se rellenan correctamente:
    -   **SID**: el SID de tu cuenta de gMSA
    -   **MachineAccountName**: el nombre de cuenta SAM gMSA (no incluya el nombre de dominio completo o una señal de estimada)
    -   **DnsTreeName**: el FQDN del bosque de AD
    -   **DnsName**: el FQDN del dominio al que pertenece la gMSA
    -   **NombreNetBIOS**: nombre NETBIOS para el dominio al que pertenece la gMSA
    -   **GroupManagedServiceAccounts o el nombre**: el nombre de cuenta SAM gMSA (no incluya el nombre de dominio completo o una señal de estimada)
    -   **GroupManagedServiceAccounts y ámbito**: una entrada para el FQDN del dominio y otro para el NETBIOS

    Consulta el ejemplo completo siguiente para una especificación de credencial completa:

    ```json
    {
    "CmsPlugins":[
        "ActiveDirectory"
    ],
    "DomainJoinConfig":{
        "Sid":"S-1-5-21-702590844-1001920913-2680819671",
        "MachineAccountName":"webapp01",
        "Guid":"56d9b66c-d746-4f87-bd26-26760cfdca2e",
        "DnsTreeName":"contoso.com",
        "DnsName":"contoso.com",
        "NetBiosName":"CONTOSO"
    },
    "ActiveDirectoryConfig":{
        "GroupManagedServiceAccounts":[
        {
            "Name":"webapp01",
            "Scope":"contoso.com"
        },
        {
            "Name":"webapp01",
            "Scope":"CONTOSO"
        }
        ]
    }
    }
    ```

3.  Comprobar que la ruta de acceso del archivo de especificación de credenciales es correcto para la solución de orquestación. Si estás usando Docker, asegúrese de que incluye el contenedor ejecuta el comando `--security-opt="credentialspec=file://NAME.json"`, donde "NAME.json" se reemplaza con el nombre los resultados por **Get-CredentialSpec**. El nombre es un nombre de archivo plana con respecto a la carpeta CredentialSpecs en el directorio raíz de Docker.

**Comprueba el contenedor**

1.  Si estás ejecutando una versión de Windows anteriores a Windows Server 2019 o Windows 10, versión 1809, el nombre de host de contenedor debe coincidir con el nombre de la gMSA. Garantizar el `--hostname` parámetro coincide con el nombre corto de gMSA (no hay ningún componente de dominio, por ejemplo, "webapp01" no "webapp01.contoso.com").

2.  Comprobar la configuración de red de contenedor para comprobar el contenedor puede resolver y tener acceso a un controlador de dominio de la gMSA. Los servidores DNS mal configurados en el contenedor son una causa común de problemas de identidad.

3.  Comprueba si el contenedor tiene una conexión válida en el dominio ejecutando el siguiente comando en el contenedor (con `docker exec` o un equivalente):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    La comprobación de confianza debería devolver NERR_SUCCESS si está disponible la gMSA y conectividad de red permite que el contenedor para comunicarse con el dominio.
    Si se produce un error, comprueba la configuración de red del host y el contenedor: ambos deberá ser capaces de comunicarse con el controlador de dominio.

4.  Asegúrate de que la aplicación está [configurado para utilizar la gMSA](#configuring-your-application-to-use-the-gmsa). La cuenta de usuario dentro del contenedor no cambia al utilizar una gMSA--en su lugar, usa la gMSA en la cuenta del sistema cuando comunica con otros recursos de red. Como tal, la aplicación que se ejecutan como servicio de red o del sistema Local para aprovechar la identidad de gMSA.

    > [!TIP]
    > Si ejecutas `whoami` o usar otra herramienta para probar e identificar el contexto del usuario actual en el contenedor, nunca se verá el nombre de la gMSA.
    > Esto es porque siempre se inicia en el contenedor como un usuario local, no una identidad de dominio.
    > La cuenta de equipo usa la gMSA cada vez que se comunica con los recursos de red, es la razón por la aplicación necesita para ejecutarse como sistema Local o servicio de red.

5.  Por último, si el contenedor se parece para configurarse correctamente, pero los usuarios u otros servicios son no se puede autenticar automáticamente a la aplicación en contenedores, comprueba el SPN en tu cuenta de gMSA. Los clientes buscará la cuenta gMSA por el nombre en la que lleguen a la aplicación. Esto puede significar que tendrás que adicionales `host` SPN para la gMSA si, por ejemplo, en que los clientes se conectan a la aplicación a través de un equilibrador de carga u otro nombre DNS.

## <a name="additional-resources"></a>Recursos adicionales
-   [Introducción a las cuentas de servicio administradas de grupo](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
