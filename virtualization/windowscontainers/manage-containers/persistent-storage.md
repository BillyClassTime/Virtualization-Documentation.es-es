---
title: Almacenamiento persistente en contenedores
description: Cómo pueden el almacenamiento persistente los contenedores de Windows
keywords: contenedores, volumen, almacenamiento, montaje, enlazar montajes
author: cwilhit
ms.openlocfilehash: 945a78d4ecb9c96da4de8f7246f84b6b444dd5b5
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909675"
---
# <a name="persistent-storage-in-containers"></a>Almacenamiento persistente en contenedores

<!-- Great diagram would be great! -->

Puede haber casos en los que sea importante que una aplicación pueda conservar los datos en un contenedor o mostrar los archivos en un contenedor que no se incluyeron en el momento de la compilación del contenedor. Se puede proporcionar almacenamiento persistente a los contenedores de dos maneras:

- Enlazar montajes
- Volúmenes con nombre

Docker tiene una excelente introducción de cómo [usar volúmenes](https://docs.docker.com/engine/admin/volumes/volumes/) por lo que es mejor leerla primero. El resto de esta página se centra en las diferencias entre Linux y Windows y proporciona ejemplos en Windows.

## <a name="bind-mounts"></a>Enlazar montajes

[Enlazar montajes](https://docs.docker.com/engine/admin/volumes/bind-mounts/) permite que un contenedor comparta un directorio con el host. Esto es útil si quieres un lugar para almacenar archivos en la máquina local que están disponibles si reinicias un contenedor o quieres compartirlos con varios contenedores. Si quieres que el contenedor se ejecute en varias máquinas con acceso a los mismos archivos, debe usarse en su lugar un volumen con nombre o el montaje SMB.

### <a name="permissions"></a>Permisos

El modelo de permiso utilizado para enlazar montajes varía según el nivel de aislamiento de tu contenedor.

Los contenedores que usan el **aislamiento de Hyper-V** usan un modelo de permisos simple de solo lectura o de lectura y escritura. Se acceden a los archivos en el host mediante la cuenta `LocalSystem`. Si obtienes acceso denegado en el contenedor, asegúrate de que `LocalSystem` tenga acceso a ese directorio en el host. Cuando se usa la marca de solo lectura, los cambios realizados en el volumen dentro del contenedor no se mostrarán ni permanecerán en el directorio del host.

Los contenedores de Windows que usan el **aislamiento de proceso** son ligeramente diferentes porque usan la identidad de proceso dentro del contenedor para obtener acceso a los datos, lo que significa que se respetan las ACL de archivos. La identidad del proceso que se ejecuta en el contenedor ("ContainerAdministrator" en Windows Server Core y "ContainerUser" en contenedores de Nano Server, de manera predeterminada) se utilizará para tener acceso a los archivos y directorios en el volumen montado en lugar de `LocalSystem`y necesitará tener acceso para usar los datos.

Puesto que estas identidades solo existen en el contexto del contenedor, no en el host donde se almacenan los archivos, debe usar un grupo de seguridad conocido como `Authenticated Users` al configurar las ACL para conceder acceso a los contenedores.

> [!WARNING]
> No unas mediante enlace directorios confidenciales, como por ejemplo, `C:\` en un contenedor que no es de confianza. Esto le permitiría cambiar archivos en el host al que normalmente no tendría acceso y podría crear una infracción de seguridad.

Ejemplo de uso:

- `docker run -v c:\ContainerData:c:\data:RO` para el acceso de solo lectura
- `docker run -v c:\ContainerData:c:\data:RW` para el acceso de lectura y escritura
- `docker run -v c:\ContainerData:c:\data` para el acceso de lectura y escritura (valor predeterminado)

### <a name="symlinks"></a>Symlinks

Symlinks se resuelven en el contenedor. Si unes mediante enlace una ruta de acceso de host a un contenedor que es un symlink o incluye symlinks: el contenedor no podrá tener acceso a ellos.

### <a name="smb-mounts"></a>Montajes de SMB

En la versión 1709 y posteriores de Windows Server, la característica denominada "asignación global SMB" permite montar un recurso compartido de SMB en el host y, a continuación, pasar los directorios de ese recurso compartido a un contenedor. El contenedor no debe configurarse con un servidor, recurso compartido, nombre de usuario o contraseña en concreto: todo se administra en el host en su lugar. El contenedor funcionará de la misma forma que si tuviera el almacenamiento local.

#### <a name="configuration-steps"></a>Pasos de configuración

1. En el host de contenedor, asigne globalmente el recurso compartido SMB remoto:
    ```
    $creds = Get-Credential
    New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
    ```
    Este comando usará las credenciales para autenticarse con el servidor de SMB remoto. Después asigna la ruta de acceso de recurso compartido remoto a la letra de unidad G: (puede ser cualquier otra letra de unidad disponible). Los contenedores creados en este host del contenedor ahora pueden tener sus volúmenes de datos asignados a una ruta de acceso en la unidad G:.

    > [!NOTE]
    > Cuando se usa la asignación global SMB para contenedores, todos los usuarios del host de contenedor pueden tener acceso al recurso compartido remoto. Cualquier aplicación que se ejecuta en el host del contenedor también tendrá acceso al recurso compartido remoto asignado.

2. Crea contenedores con volúmenes de datos asignados al docker de recurso compartido de SMB globalmente montado, ejecuta: name demo - v g:\ContainerData:G:\AppData1 Microsoft/windowsservercore:1709 cmd.exe

    Dentro del contenedor, G:\AppData1 se asignará al directorio de "ContainerData" del recurso compartido remoto. Todos los datos almacenados en el recurso compartido remoto asignado globalmente estarán disponibles para aplicaciones dentro del contenedor. Varios contenedores pueden obtener acceso de lectura y escritura a estos datos compartidos con el mismo comando.

Esta compatibilidad de asignación global de SMB es una característica de cliente de SMB que puede funcionar en la parte superior de cualquier servidor SMB compatible, entre los que se incluyen:

- Servidor de archivos de escalabilidad horizontal en la parte superior de Storage Spaces Direct (S2D) o una SAN tradicional
- Azure Files (recurso compartido de SMB)
- Servidor de archivos tradicional
- Implementación de terceros 3 del protocolo SMB (ejemplo: dispositivos NAS)

> [!NOTE]
> La asignación global de SMB no es compatible con los recursos compartidos de DFS, DFSN y DFSR en Windows Server versión 1709.

## <a name="named-volumes"></a>Volúmenes con nombre

Los volúmenes con nombre te permiten crear un volumen según el nombre, asignarle a un contenedor y reutilizarlo más adelante con el mismo nombre. No tienes que realizar un seguimiento de la ruta de acceso real de dónde se creó, solo el nombre. Docker Engine en Windows tiene un complemento de volumen con nombre integrado que puede crear volúmenes en la máquina local. Un complemento adicional es necesario si quieres usar volúmenes con nombre en varias máquinas.

Pasos de ejemplo:

1. `docker volume create unwound`: creación de un volumen denominado "desenredado"
2. `docker run -v unwound:c:\data microsoft/windowsservercore`: Inicio de un contenedor con el volumen asignado a c:\Datos
3. Escribir algunos archivos en c:\data en el contenedor y luego detener el contenedor
4. `docker run -v unwound:c:\data microsoft/windowsservercore`: Inicio de un nuevo contenedor
5. Ejecutar `dir c:\data` en el contenedor nuevo: los archivos siguen estando ahí

> [!NOTE]
> Windows Server convertirá las rutas de acceso de destino (la ruta de acceso dentro del contenedor) en minúsculas; i. e. `-v unwound:c:\MyData`, o `-v unwound:/app/MyData` en contenedores de Linux, dará como resultado un directorio dentro del contenedor de `c:\mydata`o `/app/mydata` en contenedores de Linux, que se asigna (y se crea, si no existe).
