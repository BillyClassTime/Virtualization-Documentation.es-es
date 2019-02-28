---
title: Dispositivos en contenedores en Windows
description: ¿Qué soporte técnico de dispositivo existe para contenedores en Windows
keywords: docker, contenedores, los dispositivos, hardware
author: cwilhit
ms.openlocfilehash: da9785b051826efa4bb2c64542a7c75a12ddd2b4
ms.sourcegitcommit: 4490d384ade48699e7f56dc265185dac75bf9d77
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 02/05/2019
ms.locfileid: "9058995"
---
**Actualmente, este es el material de vista previa. Consulta el artículo 4 en la sección "Requisitos" a continuación para obtener más información.**

# <a name="devices-in-containers-on-windows"></a>Dispositivos en contenedores en Windows

De manera predeterminada, los contenedores de Windows se otorga acceso mínimo para dispositivos de host, al igual que los contenedores de Linux. Hay determinadas cargas de trabajo que resulta útil, o incluso imperativo--para acceder y comunicarse con dispositivos de hardware de host. Esta guía trata los dispositivos que son compatibles con los contenedores y cómo empezar a trabajar.

## <a name="requirements"></a>Requisitos

- Debes estar ejecutando Windows Server 2019 o posterior o Windows 10 Pro/Enterprise con la de octubre de 2018 Update
- La versión de la imagen de contenedor debe ser 1809 o posterior.
- Los contenedores deben ser contenedores de Windows que se ejecutan en el modo de aislamiento de proceso.
- Aunque la funcionalidad de dispositivos de Windows existe en el demonio de Docker, no existe aún en el cliente de Docker (consulta esta [solicitud de extracción](https://github.com/docker/cli/pull/1606) para realizar un seguimiento). Debes esperar a que una versión futura de Docker para Windows y Docker EE con este código para aprovechar las ventajas de esta característica. Este documento se actualizará cuando cambia el estado.

## <a name="run-a-container-with-a-device"></a>Ejecutar un contenedor con un dispositivo

Para iniciar un contenedor con un dispositivo, utilice el siguiente comando:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Debes reemplazar el `{interface class guid}` con una apropiada [GUID de clase de interfaz de dispositivo](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/overview-of-device-interface-classes), que se puede encontrar en la sección siguiente.

Para iniciar un contenedor con varios dispositivos, usa el siguiente comando y encadenar múltiples `--device` argumentos:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

En Windows, todos los dispositivos declaran una lista de las clases de interfaz que implementan. Mediante la aprobación de este comando para Docker, garantiza que todos los dispositivos que se identifican como la implementación de la clase solicitada se se asocia en el contenedor.

Esto significa que **no** va a asignar el dispositivo de host. En su lugar, el host es compartir con el contenedor. De igual modo, puesto que especifica un GUID de clase, _todos los_ dispositivos que implementan dicho GUID se compartirán con el contenedor.

## <a name="what-devices-are-supported"></a>¿Qué son compatibles con dispositivos

Los siguientes dispositivos (y el GUID de clase de la interfaz de dispositivo) son compatibles actualmente:
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Tipo de dispositivo</center></th>
<th><center>GUID de clase de interfaz</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>GPIO</center></td>
<td><center>916EF1CB-8426-468D-A6F7-9AE8076881B3</center></td>
</tr>
<tr valign="top">
<td><center>Bus i2c</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>Puerto COM</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>Bus SPI</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
</tbody>
</table>

> [!TIP]
> Los dispositivos mostrados anteriormente son los dispositivos _solo_ admitidos actualmente en los contenedores de Windows. Intenta pasar cualquier otro GUID de clase dará como resultado el contenedor no pueda iniciarse.

## <a name="hyper-v-container-device-support"></a>Compatibilidad con dispositivos de contenedor de Hyper-V

Asignación del dispositivo y el uso compartido no se admiten en contenedores de Hyper-V aislado hoy en día.

## <a name="linux-containers-on-windows-lcow-device-support"></a>Contenedores de Linux en la compatibilidad con dispositivos de Windows (LCOW)

Asignación del dispositivo y el uso compartido no se admiten en LCOW hoy en día.
