---
title: Requisitos de los contenedores de Windows
description: Requisitos de los contenedores de Windows.
keywords: metadatos, contenedores
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: c6f8dd68c6c9f346e26d29b0072a0e3d8c18759f
ms.sourcegitcommit: 8f096e9d557c60985c8512b43e4e4058cb307ed2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2017
---
# <a name="windows-container-requirements"></a>Requisitos de los contenedores de Windows

En esta guía se enumeran los requisitos de un host de contenedor de Windows.

## <a name="os-requirements"></a>Requisitos de sistema operativo

- La característica de contenedor de Windows solo está disponible en Windows Server 2016 (Core y con experiencia de escritorio), Nano Server y Windows 10 Professional y Enterprise (Anniversary Edition).
- El rol de Hyper-V debe instalarse antes de ejecutar los contenedores de Hyper-V.
- Los hosts de contenedor de Windows Server deben tener Windows instalado en c:\. Esta restricción no se aplica si solo se van a implementar contenedores de Hyper-V.

## <a name="virtualized-container-hosts"></a>Hosts de contenedor virtualizados

Si un host de contenedor de Windows se va a ejecutar desde una máquina virtual de Hyper-V y también van a hospedar contenedores de Hyper-V, será necesario habilitar la virtualización anidada. La virtualización anidada consta de los siguientes requisitos:

- Al menos 4 GB de RAM disponibles para el host de Hyper-V virtualizado.
- Windows Server 2016, o Windows 10 en el sistema host, y Windows Server (Full o Core) o Nano Server en la máquina virtual.
- Un procesador con Intel VT-x (actualmente, esta característica solo está disponible para los procesadores Intel).
- La máquina virtual del host de contenedor también necesitará al menos dos procesadores virtuales.

## <a name="supported-base-images"></a>Imágenes base compatibles

Los contenedores de Windows se ofrecen con dos imágenes base de contenedor, Windows Server Core y Nano Server. Tenga en cuenta que no todas las configuraciones son compatibles con ambas imágenes del sistema operativo. En esta tabla se detallan las configuraciones compatibles.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Sistema operativo host</center></th>
<th><center>Contenedor de Windows Server</center></th>
<th><center>Contenedor de Hyper-V</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>Windows Server 2016 (Standard o Datacenter)</center></td>
<td><center>Server Core/Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server</center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro / Enterprise</center></td>
<td><center>No disponible</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
</tbody>
</table>

## <a name="matching-container-host-version-with-container-image-versions"></a>Coincidencia de la versión del host de contenedor con las versiones de la imagen de contenedor
### <a name="windows-server-containers"></a>Contenedores de Windows Server
Dado que los contenedores de Windows Server y el host subyacente comparten un solo kernel, la imagen base del contenedor debe coincidir con la del host.  Si las versiones son diferentes, el contenedor puede iniciarse, pero no se garantiza una funcionalidad completa. Por lo tanto, no se admiten las versiones que no coinciden.  El sistema operativo Windows tiene cuatro niveles de control de versiones: Principal, Secundario, Compilación y Revisión (por ejemplo, 10.0.14393.0). El número de compilación solo cambia cuando se publican nuevas versiones del sistema operativo. El número de revisión se actualiza cuando se aplican las actualizaciones de Windows. Si el número de compilación es diferente, se bloquea el inicio de los contenedores de Windows Server. Por ejemplo, 10.0.14300.1030 (Technical Preview 5) y 10.0.14393 (Windows Server 2016 RTM). Si el número de compilación coincide pero el número de revisión es diferente, no se bloquea el inicio: por ejemplo 10.0.14393 (Windows Server 2016 RTM) y 10.0.14393.206 (Windows Server 2016 GA). Aunque técnicamente no están bloqueados, esta configuración podría no funcionar correctamente en todas las circunstancias y, por lo tanto, no se admite en entornos de producción. 

Para comprobar qué versión tiene instalada un host de Windows, consulte HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion.  Para comprobar qué versión usa la imagen base, revise las etiquetas de Docker Hub o la tabla hash de la imagen proporcionada en la descripción de la imagen.  En la página [Historial de actualizaciones de Windows 10](https://support.microsoft.com/en-us/help/12387/windows-10-update-history) se muestra cuándo se lanzó cada compilación y revisión.

En este ejemplo, 14393 es el número de compilación principal y 321 es la revisión.
```none
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

### <a name="hyper-v-containers"></a>Contenedores de Hyper-V
A diferencia de los contenedores de Windows Server, en cuyo caso los contenedores y el host comparten el kernel, cada contenedor de Hyper-V usa su propia instancia del kernel de Windows.  Por este motivo, es posible que las versiones del host de contenedor y de la imagen de contenedor no coincidan.  En este momento, las compilaciones con un número de compilación igual o mayor que Windows Server 2016 GA (10.0.14393.206) pueden ejecutar las imágenes de Windows Server 2016 GA de Windows Server Core y de Nano Server en una configuración compatible, independientemente del número de revisión.  En respuesta a los comentarios de los clientes, en el futuro proporcionaremos instrucciones específicas sobre la distancia que puede separar a los números de compilación.  Es importante comprender que, para tener la funcionalidad completa, la confiabilidad y las garantías de seguridad que proporcionan las actualizaciones de Windows, es necesario mantener las versiones más recientes en todos los sistemas.  
