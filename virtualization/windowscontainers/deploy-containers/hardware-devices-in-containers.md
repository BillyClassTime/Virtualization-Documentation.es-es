---
title: Dispositivos en contenedores de Windows
description: ¿Qué compatibilidad con dispositivos hay en los contenedores de Windows?
keywords: acoplador, contenedores, dispositivos, hardware
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 2f8fd4b2e7113fbb7c323d89f3c72df5e1a4437e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2019
ms.locfileid: "10045035"
---
# <a name="devices-in-containers-on-windows"></a>Dispositivos en contenedores de Windows

De forma predeterminada, los contenedores de Windows reciben un acceso mínimo a los dispositivos de hospedaje, como los contenedores de Linux. Hay ciertas cargas de trabajo en las que es beneficioso, o incluso imperativo, tener acceso a dispositivos de hardware de hospedaje y comunicarse con ellos. Esta guía cubre qué dispositivos son compatibles con los contenedores y cómo comenzar.

## <a name="requirements"></a>Requisitos

Para que esta característica funcione, su entorno debe cumplir con los siguientes requisitos:
- El host contenedor debe ejecutar Windows Server 2019 o Windows 10, versión 1809 o posterior.
- La versión de la imagen base del contenedor debe ser 1809 o posterior.
- Los contenedores deben ser contenedores de Windows que se ejecuten en modo de aislamiento de procesos.
- El host contenedor debe ejecutar el motor de acoplamiento 19,03 o una versión posterior.

## <a name="run-a-container-with-a-device"></a>Ejecutar un contenedor con un dispositivo

Para iniciar un contenedor con un dispositivo, use el siguiente comando:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Debe reemplazar el `{interface class guid}` por un GUID de [clase de interfaz de dispositivo](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)adecuado, que se encuentra en la sección siguiente.

Para iniciar un contenedor con varios dispositivos, use el comando siguiente y la cadena juntos `--device` con varios argumentos:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

En Windows, todos los dispositivos declaran una lista de clases de interfaz que implementan. Al pasar este comando a Docker, se asegurará de que todos los dispositivos que identifiquen como implementación de la clase solicitada se sondearán en el contenedor.

Esto significa que **no** va a asignar el dispositivo fuera del host. En su lugar, el host la está compartiendo con el contenedor. Del mismo modo, dado que está especificando un GUID de clase, _todos los_ dispositivos que implementan ese GUID se compartirán con el contenedor.

## <a name="what-devices-are-supported"></a>¿Qué dispositivos son compatibles?

Hoy se admiten los siguientes dispositivos (y sus GUID de clase de interfaz de dispositivo):
  
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
<td><center>Bus I2C</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>Puerto COM</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>SPI bus</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>Aceleración de la GPU DirectX</center></td>
<td><center>Ver documentos de <a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">aceleración por GPU</a></center></td>
</tr>
</tbody>
</table>

> [!IMPORTANT]
> La compatibilidad del dispositivo depende del controlador. Intentar pasar GUID de clase no definidos en la tabla anterior puede dar lugar a un comportamiento indefinido.

## <a name="hyper-v-isolated-windows-container-support"></a>Compatibilidad con Hyper-V de contenedor de Windows aislado

La asignación de dispositivos y el uso compartido de dispositivos para cargas de trabajo en contenedores de Windows aislados no se admite actualmente.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-V-compatibilidad con contenedores de Linux aislados

No se admite la asignación de dispositivos ni el uso compartido de dispositivos para cargas de trabajo en contenedores Linux aislados con Hyper-V.
