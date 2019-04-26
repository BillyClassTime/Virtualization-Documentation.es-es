---
title: Dispositivos en contenedores en Windows
description: ¿Qué soporte técnico de dispositivo existe para contenedores en Windows
keywords: docker, contenedores, los dispositivos, hardware
author: cwilhit
ms.openlocfilehash: 18ae4ab229a677c63c3e17d684a3c3193df49c5e
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576686"
---
# <a name="devices-in-containers-on-windows"></a>Dispositivos en contenedores en Windows

De manera predeterminada, los contenedores de Windows se otorga acceso mínimo para dispositivos de host, al igual que los contenedores de Linux. Hay determinadas cargas de trabajo que resulta útil, o incluso imperativo--para acceder y comunicarse con dispositivos de hardware de host. Esta guía trata los dispositivos que son compatibles con los contenedores y cómo empezar a trabajar.

> [!IMPORTANT]
> Esta característica requiere una versión de Docker que admita la `--device` opción de línea de comandos para contenedores de Windows. Se ha programado el soporte de Docker formal para la próxima versión de 19.03 motor de Docker EE. Hasta entonces, el [origen en dirección ascendente](https://master.dockerproject.org/) de Docker contiene los bits necesarios.

## <a name="requirements"></a>Requisitos

Para que funcione esta característica, el entorno debe cumplir los siguientes requisitos:
- El host de contenedor debe ejecutar Windows Server 2019 o Windows 10, versión 1809 o posterior.
- La versión de la imagen base de contenedor debe ser 1809 o posterior.
- Los contenedores deben ser contenedores de Windows que se ejecutan en el modo de aislamiento de proceso.
- El host de contenedor debe ejecutar motor de Docker 19.03 o posterior.

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
<tr valign="top">
<td><center>Aceleración de GPU de DirectX</center></td>
<td><center>Consulta a los documentos dedicados</center></td>
</tr>
</tbody>
</table>

> [!TIP]
> Los dispositivos mostrados anteriormente son los dispositivos _solo_ admitidos actualmente en los contenedores de Windows. Intenta pasar cualquier otro GUID de clase dará como resultado el contenedor no pueda iniciarse.

## <a name="hyper-v-isolated-windows-container-support"></a>Soporte técnico de contenedor de Hyper-V-aislada Windows

Asignación del dispositivo y el dispositivo de uso compartido de cargas de trabajo en contenedores de Windows aislado Hyper-V no se admite actualmente.

## <a name="hyper-v-isolated-linux-container-support"></a>Soporte de contenedor de Hyper-V-aislada Linux

Asignación del dispositivo y el dispositivo de uso compartido de cargas de trabajo en contenedores de Linux aislada Hyper-V no se admite actualmente.
