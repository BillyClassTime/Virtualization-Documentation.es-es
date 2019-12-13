---
title: Dispositivos en contenedores en Windows
description: Compatibilidad del dispositivo con los contenedores de Windows
keywords: Docker, contenedores, dispositivos, hardware
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910605"
---
# <a name="devices-in-containers-on-windows"></a>Dispositivos en contenedores en Windows

De forma predeterminada, los contenedores de Windows tienen un acceso mínimo a los dispositivos host, al igual que los contenedores de Linux. Hay ciertas cargas de trabajo en las que es beneficioso, o incluso imperativa, tener acceso a los dispositivos de hardware del host y comunicarse con ellos. En esta guía se explica qué dispositivos se admiten en contenedores y cómo empezar.

## <a name="requirements"></a>Requisitos

Para que esta característica funcione, el entorno debe cumplir los siguientes requisitos:
- El host de contenedor debe ejecutar Windows Server 2019 o Windows 10, versión 1809 o posterior.
- La versión de la imagen base del contenedor debe ser 1809 o posterior.
- Los contenedores deben ser contenedores de Windows que se ejecuten en modo aislado de proceso.
- El host de contenedor debe ejecutar el motor de Docker 19,03 o una versión más reciente.

## <a name="run-a-container-with-a-device"></a>Ejecutar un contenedor con un dispositivo

Para iniciar un contenedor con un dispositivo, use el siguiente comando:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Debe reemplazar el `{interface class guid}` por un GUID de [clase de interfaz de dispositivo](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)adecuado, que se puede encontrar en la sección siguiente.

Para iniciar un contenedor con varios dispositivos, use el siguiente comando y cadena juntos varios `--device` argumentos:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

En Windows, todos los dispositivos declaran una lista de las clases de interfaz que implementan. Al pasar este comando a Docker, se asegurará de que todos los dispositivos que identifican como la implementación de la clase solicitada se sondearán en el contenedor.

Esto significa que **no** está asignando el dispositivo fuera del host. En su lugar, el host lo comparte con el contenedor. Del mismo modo, dado que está especificando un GUID de clase, _todos los_ dispositivos que implementan ese GUID se compartirán con el contenedor.

## <a name="what-devices-are-supported"></a>¿Qué dispositivos se admiten?

Actualmente se admiten los siguientes dispositivos (y sus GUID de clase de interfaz de dispositivo):
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Device Type</center></th>
<th><center>GUID de clase de interfaz</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>GPIO</center></td>
<td><center>916EF1CB-8426-468D-A6F7-9AE8076881B3</center></td>
</tr>
<tr valign="top">
<td><center>Bus I2C</center></td>
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
<td><center>Consulte documentos de <a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">aceleración de GPU</a></center></td>
</tr>
</tbody>
</table>

> [!IMPORTANT]
> La compatibilidad con dispositivos depende del controlador. Si intenta pasar GUID de clase no definidos en la tabla anterior, se puede producir un comportamiento indefinido.

## <a name="hyper-v-isolated-windows-container-support"></a>Compatibilidad con contenedores de Windows aislados de Hyper-V

Actualmente no se admite la asignación de dispositivos y el uso compartido de dispositivos para cargas de trabajo en contenedores de Windows aislados de Hyper-V.

## <a name="hyper-v-isolated-linux-container-support"></a>Compatibilidad con contenedores de Linux aislados de Hyper-V

Actualmente no se admite la asignación de dispositivos y el uso compartido de dispositivos para cargas de trabajo en contenedores de Linux aislados con Hyper-V.
