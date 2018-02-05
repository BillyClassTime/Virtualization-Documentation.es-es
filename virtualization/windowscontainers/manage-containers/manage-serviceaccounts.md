---
title: Cuentas de servicio de Active Directory para contenedores de Windows
description: Cuentas de servicio de Active Directory para contenedores de Windows
keywords: docker, contenedores, active directory
author: PatrickLang
ms.date: 11/04/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: df9ca8a4bcd6bf959e221593ea69d5ed624cdae1
ms.sourcegitcommit: 6beac5753c9f65bb6352df8c829c2e62e24bd2e2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/17/2018
---
# <a name="active-directory-service-accounts-for-windows-containers"></a>Cuentas de servicio de Active Directory para contenedores de Windows

Puede que los usuarios y otros servicios necesiten realizar conexiones autenticadas a sus aplicaciones y servicios para poder mantener los datos seguros y prevenir el uso sin autorización. Los dominios de Windows Active Directory (AD) admiten la autenticación de contraseña y certificado de forma nativa. Al compilar la aplicación o servicio en un host unido a un dominio de Windows, utiliza la identidad del host de forma predeterminada si se ejecuta como sistema local o servicio de red. De lo contrario, puede configurar otra cuenta de AD para la autenticación en su lugar.

Aunque los contenedores de Windows no pueden estar unidos al dominio, puede aprovechar las identidades de dominio de Active Directory del mismo modo que cuando un dispositivo está unido a un dominio Kerberos. Con los controladores de dominio de Windows Server 2012 R2, se publicó una nueva cuenta de dominio que se denomina cuenta de servicio administrada de grupo (gMSA), diseñada para que la compartiesen los servicios. Mediante el uso de cuentas de servicio administradas de grupo (gMSA), los propios contenedores de Windows y los servicios que hospedan pueden configurarse para utilizar una gMSA específica como su identidad de dominio. Los servicios que se ejecuten como sistema local o servicio de red utilizarán la identidad de contenedor de Windows del mismo modo que usan la identidad del host unido al dominio hoy en día. No hay ninguna contraseña ni clave privada de certificado almacenada en la imagen de contenedor que se pueda exponer sin darse cuenta y el contenedor puede volver a implementarse en entornos de desarrollo, prueba y producción sin volver a generarse para cambiar las contraseñas almacenadas o certificados. 


# <a name="glossary--references"></a>Glosario y referencias
- [Active Directory](http://social.technet.microsoft.com/wiki/contents/articles/1026.active-directory-services-overview.aspx) es un servicio utilizado para la detección, búsqueda y replicación de la información de cuenta de servicio, equipo y usuario de Windows. 
  - [Los servicios de dominio de Active Directory](https://technet.microsoft.com/en-us/library/dd448614.aspx) proporcionan un dominio de Windows Active Directory usado para autenticar equipos y usuarios. 
  - Los dispositivos están _unidos al dominio_ cuando son miembros del dominio de Active Directory. “Unido a dominio” es un estado de dispositivo que no solo proporciona una identidad de equipo de dominio al dispositivo, sino que activa varios servicios de unión al dominio.
  - Las [cuentas de servicio administradas de grupo](https://technet.microsoft.com/en-us/library/jj128431(v=ws.11).aspx), abreviado normalmente como gMSA, son un tipo de cuenta de Active Directory que facilita servicios seguros mediante Active Directory sin compartir una contraseña. Varias máquinas o contenedores comparten la misma gMSA según sea necesario para autenticar las conexiones entre los servicios.
- Módulo de PowerShell _CredentialSpec_: este módulo se utiliza para configurar cuentas de servicio administradas de grupo para su uso con contenedores. Los pasos de ejemplo y el módulo de script están disponibles en [windows-server-container-tools](https://github.com/Microsoft/Virtualization-Documentation/tree/live/windows-server-container-tools) vea ServiceAccount.

# <a name="how-it-works"></a>Cómo funciona

Actualmente, las cuentas de servicio administradas de grupo se usan para asegurar conexiones entre un equipo o servicio y otro. Estos son los pasos generales a seguir:

1. Crear una gMSA
2. Configurar el servicio para ejecutarse como la gMSA
3. Dar acceso a los secretos de gMSA en Active Directory al host unido al dominio que ejecuta el servicio
4. Permitir el acceso a gMSA en el otro servicio, como recursos compartidos de un archivo o base de datos

Cuando se inicia el servicio, el host unido al dominio obtiene automáticamente los secretos de la gMSA de Active Directory y ejecuta el servicio usando esa cuenta. Dado que ese servicio se ejecuta como la gMSA, puede acceder a los recursos a los que tiene permitido acceder la gMSA.

Los contenedores de Windows siguen un proceso similar:

1. Crear una gMSA. De forma predeterminada, un administrador de dominio o un operador de la cuenta debe hacerlo. De lo contrario, puede delegar privilegios para crear y administrar gMSA a los administradores de los servicios que las utilizan. Consulte [Introducción a gMSA](https://technet.microsoft.com/en-us/library/jj128431(v=ws.11).aspx)
2. Dar acceso a la gMSA al contenedor unido al dominio
3. Permitir el acceso a gMSA en el otro servicio, como recursos compartidos de un archivo o base de datos
4. Usar el módulo de PowerShell CredentialSpec desde [windows-server-container-tools](https://github.com/Microsoft/Virtualization-Documentation/tree/live/windows-server-container-tools) para almacenar la configuración necesaria para utilizar la gMSA
5. Iniciar el contenedor con una opción adicional `--security-opt "credentialspec=..."`

[!NOTE]
Es posible que tengas que permitir la traducción SID/nombre anónima en el host del contenedor tal y como se describe [aquí](https://docs.microsoft.com/en-us/windows/device-security/security-policy-settings/network-access-allow-anonymous-sidname-translation) ya que, de lo contrario, se podría producir un error en las cuentas que no pueden traducirse a SID.

Cuando se inicia el contenedor, aparecerán los servicios instalados que se ejecutan como sistema local o servicio de red ejecutándose como la gMSA. Esto es similar a cómo funcionan esas cuentas en hosts unidos a dominios, excepto que se utiliza una gMSA en lugar de una cuenta de equipo. 

![Diagrama: cuentas de servicio](media/serviceaccount_diagram.png)


# <a name="example-uses"></a>Usos de ejemplo


## <a name="sql-connection-strings"></a>Cadenas de conexión de SQL
Cuando un servicio se ejecuta como sistema local o servicio de red en un contenedor, puede usar la autenticación integrada de Windows para conectarse a Microsoft SQL Server.

Ejemplo:

```
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

En Microsoft SQL Server, cree un inicio de sesión con el nombre de dominio y gMSA, seguido por un signo $. Una vez creado el inicio de sesión, puede agregarse a un usuario en una base de datos y concederle los permisos de acceso adecuados.

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
