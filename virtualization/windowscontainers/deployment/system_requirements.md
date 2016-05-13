



# Requisitos de los contenedores de Windows

**Esto es contenido preliminar y está sujeto a cambios.**

Esta guía enumera los requisitos de un host de contenedor de Windows.

## Imágenes de sistemas operativos compatibles

En Windows Server Technical Preview 4 se incluyen dos imágenes de sistemas operativos de contenedor, Windows Server Core y Nano Server. Tenga en cuenta que no todas las configuraciones son compatibles con ambas imágenes del sistema operativo. En esta tabla se detallan las configuraciones compatibles.

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
<td><center>Imagen del sistema operativo Core</center></td>
<td><center>Imagen del sistema operativo Nano</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Imagen del sistema operativo Core</center></td>
<td><center> Imagen del sistema operativo Nano</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Imagen del sistema operativo Nano</center></td>
<td><center>Imagen del sistema operativo Nano</center></td>
</tr>
</tbody>
</table>

## Requisitos de los contenedores de Hyper-V

Si un host de contenedor de Windows se ejecuta en una máquina virtual de Hyper-V y también hospeda contenedores de Hyper-V, será necesario habilitar la virtualización anidada. La virtualización anidada consta de los siguientes requisitos:

- Al menos 4 GB de RAM disponibles para el host de Hyper-V virtualizado.
- Windows Server 2016 Technical Preview 4 o la compilación 10565 de Windows 10, tanto en el host físico como en el virtualizado.
- Un procesador con Intel VT-x (actualmente, esta característica solo está disponible para los procesadores Intel).
- La máquina virtual del host de contenedor también necesitará, al menos, 2 procesadores virtuales.







<!--HONumber=Feb16_HO3-->


