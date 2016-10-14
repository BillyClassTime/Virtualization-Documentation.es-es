---
title: Requisitos de los contenedores de Windows
description: Requisitos de los contenedores de Windows.
keywords: metadatos, contenedores
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: 08355d7a7da50d0f244bd750508fd42084818d7f
ms.openlocfilehash: ac48bc7ee7b70483d8a368749aea0862c52f049c

---

# Requisitos de los contenedores de Windows

En esta guía se enumeran los requisitos de un host de contenedor de Windows.

## Requisitos de sistema operativo

- La característica de contenedor de Windows solo está disponible en Windows Server 2016 (Core y con experiencia de escritorio), Nano Server y Windows 10 Professional y Enterprise (Anniversary Edition).
- Si se van a ejecutar contenedores de Hyper-V, será necesario instalar el rol Hyper-V.
- Los hosts de contenedor de Windows Server deben tener Windows instalado en c:\.. Si solo se implementarán los contenedores de Hyper-V, esta restricción no se aplica.

## Hosts de contenedor virtualizados

Si un host de contenedor de Windows se va a ejecutar desde una máquina virtual de Hyper-V y también van a hospedar contenedores de Hyper-V, será necesario habilitar la virtualización anidada. La virtualización anidada consta de los siguientes requisitos:

- Al menos 4 GB de RAM disponibles para el host de Hyper-V virtualizado.
- Windows Server 2016, o Windows 10 en el sistema host, y Windows Server (Full o Core) o Nano Server en la máquina virtual.
- Un procesador con Intel VT-x (actualmente, esta característica solo está disponible para los procesadores Intel).
- La máquina virtual del host de contenedor también necesitará al menos dos procesadores virtuales.

## Imágenes base compatibles

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
<td><center>Windows Server 2016 con Experiencia de escritorio</center></td>
<td><center>Server Core / Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Server Core / Nano Server</center></td>
<td><center>Server Core / Nano Server</center></td>
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


<!--HONumber=Oct16_HO2-->


