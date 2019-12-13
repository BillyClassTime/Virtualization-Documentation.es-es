---
title: Información general de almacenamiento de contenedor
description: Cómo los contenedores de Windows Server pueden usar hosts y otros tipos de almacenamiento
keywords: contenedores, volumen, almacenamiento, montaje, enlazar montajes
author: cwilhit
ms.openlocfilehash: fba08de884d59cc1b656895ec2b7078ba3975269
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910275"
---
# <a name="container-storage-overview"></a>Información general de almacenamiento de contenedor

<!-- Great diagram would be great! -->

En este tema se proporciona información general sobre las diferentes formas en que los contenedores usan el almacenamiento en Windows. Los contenedores se comportan de manera diferente que las máquinas virtuales cuando llegan al almacenamiento. Por naturaleza, los contenedores se compilan para evitar que una aplicación que se ejecuta dentro de ellos escriba el estado en todo el sistema de archivos del host. De forma predeterminada, los contenedores usan un espacio "temporal", pero Windows también proporciona un medio para conservar el almacenamiento.

## <a name="scratch-space"></a>Espacio de desecho

Los contenedores de Windows usan el almacenamiento efímero de forma predeterminada. Todas las e/s de contenedor se producen en un "espacio de desecho" y cada contenedor obtiene su propia grieta. La creación de archivos y las escrituras de archivos se capturan en el espacio de desecho y no se convierten en el host. Cuando se detiene una instancia de contenedor, se descartan todos los cambios que se produjeron en el espacio de desecho. Cuando se inicia una nueva instancia de contenedor, se proporciona un nuevo espacio de desecho para la instancia.

## <a name="layer-storage"></a>Almacenamiento en capas

Tal como se describe en la [información general](../about/index.md)de los contenedores, las imágenes de contenedor son un paquete de archivos expresado como una serie de capas. El almacenamiento en capas es todos los archivos que están integrados en el contenedor. Cada vez que `docker pull`, `docker run` dicho contenedor: son iguales.

### <a name="where-layers-are-stored-and-how-to-change-it"></a>Dónde se almacenan las capas y cómo cambiarlo

En una instalación predeterminada, las capas se almacenan en `C:\ProgramData\docker` y se distribuyen en los directorios "image" y "windowsfilter". Puedes cambiar la ubicación donde se almacenan las capas con la configuración `docker-root`, tal y como se indica en la documentación de [Docker Engine en Windows](../manage-docker/configure-docker-daemon.md).

> [!NOTE]
> NTFS solo es compatible con el almacenamiento en capas. ReFS no es compatible.

No debes modificar los archivos de los directorios de capa: se administran cuidadosamente con comandos tales como:

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
- [carga de Docker](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>Operaciones compatibles en el almacenamiento en capas

Ejecutar contenedores puede usar la mayoría de las operaciones de NTFS a excepción de las transacciones. Esto incluye la configuración de ACL y todas las ACL se comprueban dentro del contenedor. Si quieres ejecutar procesos como varios usuarios dentro de un contenedor, puedes crear usuarios en tu `Dockerfile` con `RUN net user /create ...`, establece las ACL de archivos y luego configura procesos para ejecutar con dicho usuario con la [Directiva de USUARIO de Dockerfile](https://docs.docker.com/engine/reference/builder/#user).

## <a name="persistent-storage"></a>Almacenamiento persistente

Los contenedores de Windows admiten mecanismos para proporcionar almacenamiento persistente a través de montajes y volúmenes de enlace. Para obtener más información, consulte [almacenamiento persistente en contenedores](./persistent-storage.md).

## <a name="storage-limits"></a>Límites de almacenamiento

Un patrón común para las aplicaciones Windows es consultar la cantidad de espacio de disco libre antes de instalar o crear nuevos archivos o como un desencadenador para limpiar los archivos temporales.  Con el objetivo de maximizar la compatibilidad de aplicaciones, la unidad C: de un contenedor de Windows representa un tamaño libre virtual de 20 GB.

Algunos usuarios pueden querer invalidar este valor predeterminado y configurar el espacio libre en un valor menor o mayor. Esto puede realizarse a través de la opción "size" dentro de la configuración "Storage-opt".

### <a name="examples"></a>Ejemplos

Línea de comandos: `docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

O bien, puede cambiar el archivo de configuración de Docker directamente:

```Docker Configuration File
"storage-opts": [
    "size=50GB"
  ]
```

> [!TIP]
> Este método también sirve para la compilación de Docker. Consulta el documento [configurar docker](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file) para obtener más información sobre cómo modificar el archivo de configuración docker.
