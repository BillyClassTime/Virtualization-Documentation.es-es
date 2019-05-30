---
title: Almacenamiento de contenedores de WindowsServer
description: Cómo los contenedores de Windows Server pueden usar hosts y otros tipos de almacenamiento
keywords: contenedores, volumen, almacenamiento, montaje, enlazar montajes
author: patricklang
ms.openlocfilehash: 20179f09260b6ae5de802c2372958356f8de3aee
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2019
ms.locfileid: "9680945"
---
# <a name="overview"></a>Introducción

<!-- Great diagram would be great! -->


## <a name="layer-storage"></a>Almacenamiento en capas

Se trata de todos los archivos que están integrados en el contenedor. Cada vez que `docker pull`, `docker run` dicho contenedor: son iguales.


### <a name="where-layers-are-stored-and-how-to-change-it"></a>Dónde se almacenan las capas y cómo cambiarlo

En una instalación predeterminada, las capas se almacenan en `C:\ProgramData\docker` y se distribuyen en los directorios "image" y "windowsfilter". Puedes cambiar la ubicación donde se almacenan las capas con la configuración `docker-root`, tal y como se indica en la documentación de [Docker Engine en Windows](../manage-docker/configure-docker-daemon.md).

> [!NOTE]
> NTFS solo es compatible con el almacenamiento en capas. ReFS no es compatible.

No debes modificar los archivos de los directorios de capa: se administran cuidadosamente con comandos tales como:

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
- [docker load](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>Operaciones compatibles en el almacenamiento en capas

Ejecutar contenedores puede usar la mayoría de las operaciones de NTFS a excepción de las transacciones. Esto incluye la configuración de ACL y todas las ACL se comprueban dentro del contenedor. Si quieres ejecutar procesos como varios usuarios dentro de un contenedor, puedes crear usuarios en tu `Dockerfile` con `RUN net user /create ...`, establece las ACL de archivos y luego configura procesos para ejecutar con dicho usuario con la [Directiva de USUARIO de Dockerfile](https://docs.docker.com/engine/reference/builder/#user).


##  <a name="image-size"></a>Tamaño de la imagen
Un patrón común para las aplicaciones Windows es consultar la cantidad de espacio de disco libre antes de instalar o crear nuevos archivos o como un desencadenador para limpiar los archivos temporales.  Con el objetivo de maximizar la compatibilidad de aplicaciones, la unidad C: de un contenedor de Windows representa un tamaño de espacio libre virtual de 20GB.  Es posible que algunos usuarios quieran anular este valor predeterminado y configurar el espacio libre a un valor inferior o mayor; esto puede lograrse mediante la opción "size" de la configuración "storage-opt".

### <a name="examples"></a>Ejemplos
Línea de comandos: `docker run --storage-opt "size=50GB" microsoft/windowsservercore:1709 cmd`

Archivo de configuración de Docker
```
"storage-opts": [
    "size=50GB"
  ]
```
> Ten en cuenta que este método funciona para la compilación docker.
Consulta el documento [configurar docker](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file) para obtener más información sobre cómo modificar el archivo de configuración docker.


## <a name="persistent-volumes"></a>Volúmenes persistentes

El almacenamiento persistente se puede proporcionar a los contenedores de varias maneras:

- Enlazar montajes
- Volúmenes con nombre

Docker tiene una excelente introducción de cómo [usar volúmenes](https://docs.docker.com/engine/admin/volumes/volumes/) por lo que es mejor leerla primero. El resto de esta página se centra en las diferencias entre Linux y Windows y proporciona ejemplos en Windows.


### <a name="bind-mounts"></a>Enlazar montajes

[Enlazar montajes](https://docs.docker.com/engine/admin/volumes/bind-mounts/) permite que un contenedor comparta un directorio con el host. Esto es útil si quieres un lugar para almacenar archivos en la máquina local que están disponibles si reinicias un contenedor o quieres compartirlos con varios contenedores. Si quieres que el contenedor se ejecute en varias máquinas con acceso a los mismos archivos, debe usarse en su lugar un volumen con nombre o el montaje SMB.

#### <a name="permissions"></a>Permisos

El modelo de permiso utilizado para enlazar montajes varía según el nivel de aislamiento de tu contenedor.

Los contenedores con **aislamiento de Hyper-V**, como contenedores Linux en Windows Server, versión 1709, usa un modelo de permiso sencillo de solo escritura o lectura-escritura.
Se acceden a los archivos en el host mediante la cuenta `LocalSystem`. Si obtienes acceso denegado en el contenedor, asegúrate de que `LocalSystem` tenga acceso a ese directorio en el host.
Cuando se usa la marca de solo lectura, los cambios realizados en el volumen dentro del contenedor no se mostrarán ni permanecerán en el directorio del host.

Los contenedores de Windows Server con **aislamiento de procesos** son ligeramente diferentes, ya que usan la identidad del proceso dentro del contenedor para acceder a datos, lo que significa que se aceptan las ACL de archivos.
La identidad del proceso que se ejecuta en el contenedor ("ContainerAdministrator" en Windows Server Core y "ContainerUser" en contenedores de Nano Server, de manera predeterminada) se utilizará para tener acceso a los archivos y directorios en el volumen montado en lugar de `LocalSystem`y necesitará tener acceso para usar los datos.
Dado que estas identidades solo existen en el contexto del contenedor, no en el host donde se almacenan los archivos, debes usar un grupo de seguridad conocido, como `Authenticated Users` al configurar las ACL para tener acceso a los contenedores.

> [!WARNING]
> No unas mediante enlace directorios confidenciales, como por ejemplo, `C:\` en un contenedor que no es de confianza. Esto le permitiría cambiar archivos en el host al que normalmente no tendría acceso y podría crear una infracción de seguridad.

Ejemplo de uso: 

- `docker run -v c:\ContainerData:c:\data:RO` para acceso de solo lectura
- `docker run -v c:\ContainerData:c:\data:RW` para acceso de lectura-escritura
- `docker run -v c:\ContainerData:c:\data` para acceso de lectura-escritura (opción predeterminada)

#### <a name="symlinks"></a>Symlinks

Symlinks se resuelven en el contenedor. Si unes mediante enlace una ruta de acceso de host a un contenedor que es un symlink o incluye symlinks: el contenedor no podrá tener acceso a ellos.

#### <a name="smb-mounts"></a>Montajes de SMB

En Windows Server, versión 1709, una nueva característica denominada "Asignación global de SMB" hace posible montar un recurso compartido de SMB en el host y luego pasar los directorios del recurso compartido a un contenedor. El contenedor no debe configurarse con un servidor, recurso compartido, nombre de usuario o contraseña en concreto: todo se administra en el host en su lugar. El contenedor funcionará de la misma forma que si tuviera el almacenamiento local.

##### <a name="configuration-steps"></a>Pasos de configuración

1. En el host contenedor, asigne globalmente el recurso compartido SMB remoto:
    ```
    $creds = Get-Credential
    New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
    ```
    Este comando usará las credenciales para autenticar con el servidor SMB remoto. Después asigna la ruta de acceso de recurso compartido remoto a la letra de unidad G: (puede ser cualquier otra letra de unidad disponible). Los contenedores creados en este host del contenedor ahora pueden tener sus volúmenes de datos asignados a una ruta de acceso en la unidad G:.

    > Nota: Al usar la asignación global de SMB para contenedores, todos los usuarios en el host del contenedor pueden acceder al recurso compartido remoto. Cualquier aplicación que se ejecuta en el host del contenedor también tendrá acceso al recurso compartido remoto asignado.

2. Crea contenedores con volúmenes de datos asignados al docker de recurso compartido de SMB globalmente montado, ejecuta: name demo - v g:\ContainerData:G:\AppData1 Microsoft/windowsservercore:1709 cmd.exe

    Dentro del contenedor, G:\AppData1 se asignará al directorio de "ContainerData" del recurso compartido remoto. Todos los datos almacenados en el recurso compartido remoto asignado globalmente estarán disponibles para aplicaciones dentro del contenedor. Varios contenedores pueden obtener acceso de lectura y escritura a estos datos compartidos con el mismo comando.

Esta compatibilidad de asignación global de SMB es una característica de cliente de SMB que puede funcionar en la parte superior de cualquier servidor SMB compatible, entre los que se incluyen:

- Servidor de archivos de escalabilidad horizontal en la parte superior de Storage Spaces Direct (S2D) o una SAN tradicional
- Azure Files (recurso compartido de SMB)
- Servidor de archivos tradicional
- Implementación de terceros 3 del protocolo SMB (ejemplo: dispositivos NAS)

> [!NOTE]
> La asignación global de SMB no es compatible con los recursos compartidos de DFS, DFSN y DFSR en Windows Server versión 1709.

### <a name="named-volumes"></a>Volúmenes con nombre

Los volúmenes con nombre te permiten crear un volumen según el nombre, asignarle a un contenedor y reutilizarlo más adelante con el mismo nombre. No tienes que realizar un seguimiento de la ruta de acceso real de dónde se creó, solo el nombre. Docker Engine en Windows tiene un complemento de volumen con nombre integrado que puede crear volúmenes en la máquina local. Un complemento adicional es necesario si quieres usar volúmenes con nombre en varias máquinas.

Pasos de ejemplo:

1. `docker volume create unwound` -Crear un volumen denominado "unwound"
2. `docker run -v unwound:c:\data microsoft/windowsservercore` -Iniciar un contenedor con el volumen asignado a c:\data
3. Escribir algunos archivos en c:\data en el contenedor y luego detener el contenedor
4. `docker run -v unwound:c:\data microsoft/windowsservercore` - Iniciar un nuevo contenedor
5. Ejecutar `dir c:\data` en el contenedor nuevo: los archivos siguen estando ahí

> [!NOTE]
> Windows Server convertirá los nombres de ruta de destino (la ruta dentro del contenedor) a minúsculas. `-v unwound:c:\MyData`i. e `-v unwound:/app/MyData` o en los contenedores de Linux, dará como resultado un directorio dentro del contenedor de `c:\mydata`, o `/app/mydata` en los contenedores de Linux, que se asignará (y se creará, si no existe).
