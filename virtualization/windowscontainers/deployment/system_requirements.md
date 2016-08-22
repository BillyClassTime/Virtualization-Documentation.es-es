---
title: Requisitos de los contenedores de Windows
description: Requisitos de los contenedores de Windows.
keywords: metadatos, contenedores
author: neilpeterson
manager: timlt
ms.date: 08/17/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: fac57150de3ffd6c7d957dd628b937d5c41c1b35
ms.openlocfilehash: f76dc45e6035c72fd7b07f25d4b4c55f2a95aafb

---

# Requisitos de los contenedores de Windows

**Esto es contenido preliminar y está sujeto a cambios.** 

En esta guía se enumeran los requisitos de un host de contenedor de Windows.

## Requisitos de sistema operativo

- La característica de contenedor de Windows solo está disponible en Windows Server 2016 (Core y con experiencia de escritorio), Nano Server y Windows 10 Professional y Enterprise (Anniversary Edition).
- Si se van a ejecutar contenedores de Hyper-V, será necesario instalar el rol de Hyper-V.
- Los hosts de contenedor de Windows Server deben tener Windows instalado en c:\\. Si solo se van a implementar contenedores de Hyper-V, esta restricción no se aplica.

## Hosts de contenedor virtualizados

Si un host de contenedor de Windows se va a ejecutar desde una máquina virtual de Hyper-V y también van a hospedar contenedores de Hyper-V, será necesario habilitar la virtualización anidada. La virtualización anidada consta de los siguientes requisitos:

- Al menos 4 GB de RAM disponibles para el host de Hyper-V virtualizado.
- Windows Server 2016 Technical Preview 5, o la compilación 10565 de Windows 10 en el sistema host, y Windows Server Technical Preview 5 (Full o Core) o Nano Server en la máquina virtual.
- Un procesador con Intel VT-x (actualmente, esta característica solo está disponible para los procesadores Intel).
- La máquina virtual del host de contenedor también necesitará al menos dos procesadores virtuales.

## Imágenes de sistemas operativos compatibles

En Windows Server Technical Preview 5 se incluyen dos imágenes de sistemas operativos de contenedor, Windows Server Core y Nano Server. Tenga en cuenta que no todas las configuraciones son compatibles con ambas imágenes del sistema operativo. En esta tabla se detallan las configuraciones compatibles.

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
<td><center>Interfaz de usuario completa de Windows Server 2016</center></td>
<td><center>Imagen de Server Core</center></td>
<td><center>Imagen de Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Imagen de Server Core</center></td>
<td><center> Imagen de Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Imagen de Nano Server</center></td>
<td><center>Imagen de Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Anniversary Edition</center></td>
<td><center>No disponible</center></td>
<td><center>Imagen de Nano Server</center></td>
</tr>
</tbody>
</table>



<!--HONumber=Aug16_HO3-->


