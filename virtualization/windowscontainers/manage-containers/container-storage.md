---
title: Descripción general del almacenamiento de contenedores
description: Cómo los contenedores de Windows Server pueden usar hosts y otros tipos de almacenamiento
keywords: contenedores, volumen, almacenamiento, montaje, enlazar montajes
author: cwilhit
ms.openlocfilehash: fba08de884d59cc1b656895ec2b7078ba3975269
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209755"
---
# <a name="container-storage-overview"></a>Descripción general del almacenamiento de contenedores

<!-- Great diagram would be great! -->

En este tema se ofrece información general sobre las diferentes maneras en que los contenedores usan el almacenamiento en Windows. Los contenedores se comportan de manera diferente que las máquinas virtuales cuando se trata de almacenamiento. Por naturaleza, los contenedores se crean para evitar que una aplicación que se ejecuta dentro de ellos escriba el estado de todo el sistema de archivos del host. Los contenedores usan un espacio "de grietas" de forma predeterminada, pero Windows también proporciona un medio para conservar el almacenamiento.

## <a name="scratch-space"></a>Espacio de desecho

Los contenedores de Windows usan almacenamiento efímero de forma predeterminada. Todas las e/s de contenedor se producen en un "espacio de desecho" y cada contenedor obtiene su propia grieta. La creación de archivos y las escrituras de archivos se capturan en el espacio de desecho y no en el host. Cuando se detiene una instancia de contenedor, se descartan todos los cambios que se produjeron en el espacio de desecho. Cuando se inicia una nueva instancia de contenedor, se proporciona un nuevo espacio de tachado para la instancia.

## <a name="layer-storage"></a>Almacenamiento en capas

Como se describe en la [Descripción general](../about/index.md)de los contenedores, las imágenes de contenedor son un conjunto de archivos expresados como una serie de capas. Almacenamiento de capas es todos los archivos que están integrados en el contenedor. Cada vez que `docker pull`, `docker run` dicho contenedor: son iguales.

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

## <a name="persistent-storage"></a>Almacenamiento persistente

Los contenedores de Windows admiten mecanismos para proporcionar almacenamiento permanente a través de montajes y volúmenes enlazados. Para obtener más información, vea [almacenamiento persistente en contenedores](./persistent-storage.md).

## <a name="storage-limits"></a>Límites de almacenamiento

Un patrón común para las aplicaciones Windows es consultar la cantidad de espacio de disco libre antes de instalar o crear nuevos archivos o como un desencadenador para limpiar los archivos temporales.  Con el objetivo de maximizar la compatibilidad de aplicaciones, la unidad C: en un contenedor de Windows representa un tamaño libre virtual de 20 GB.

Es posible que algunos usuarios deseen invalidar este valor predeterminado y configurar el espacio libre en un valor más pequeño o más grande. Esto se puede llevar a cabo con la opción "tamaño" dentro de la configuración "almacenamiento-opt".

### <a name="examples"></a>Ejemplos

Línea de comandos: `docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

También puede cambiar el archivo de configuración del acoplador directamente:

```Docker Configuration File
"storage-opts": [
    "size=50GB"
  ]
```

> [!TIP]
> Este método también funciona para la compilación del Dock. Consulta el documento [configurar docker](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file) para obtener más información sobre cómo modificar el archivo de configuración docker.
