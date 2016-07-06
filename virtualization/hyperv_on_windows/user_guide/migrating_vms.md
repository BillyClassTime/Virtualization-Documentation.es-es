---
redirect_url: http://aka.ms/upgradevmconfig
translationtype: Human Translation
ms.sourcegitcommit: e14ede0a2b13de08cea0a955b37a21a150fb88cf
ms.openlocfilehash: 54db04f9096dc1ef9572594b321f400fbbfeada0

---

# Migrar y actualizar máquinas virtuales 

Si mueve máquinas virtuales que se crearon inicialmente con Hyper-V en Windows 8.1 o una versión anterior al host de Windows 10, no podrá usar las nuevas características de máquina virtual hasta que actualice manualmente la versión de configuración de la máquina virtual. 

Para actualizar la versión de configuración, apague la máquina virtual y después seleccione Actualizar la configuración de la máquina virtual en el Administrador de Hyper-V.  También puede abrir un símbolo del sistema con privilegios elevados de Windows PowerShell y escribir: 

 ```PowerShell
Update-VmVersion <vmname> | <vmobject>
```

## ¿Cómo se puede comprobar la versión de configuración de las máquinas virtuales que se ejecutan en Hyper-V? 

Para buscar la versión de configuración, abra un símbolo de sistemas con privilegios elevados de Windows PowerShell y ejecute el siguiente comando:

**Get-VM * | Format-Table Name, Version**

El comando de PowerShell produce el siguiente resultado de ejemplo:

```
Name        State       CPUUsage(%) MemoryAssigned(M)   Uptime              Status                  Version
    
Atlantis    Running         0       1024                00:04:20.5910000    Operating normally      5.0
    
SGC VM      Running         0       538                 00:02:44.8350000    Operating normally      6.2
```


## ¿Qué ocurre si no se actualiza la versión de configuración?

Si tiene máquinas virtuales que ha creado con una versión anterior de Hyper-V, es posible que algunas características no funcionen con esas máquinas virtuales hasta que actualice la versión de la máquina virtual.

Versión mínima de configuración de la máquina virtual para las nuevas características de Hyper-V:

| **Nombre de la característica**                       | **Versión mínima de la VM** ||
| :------------------------------------- | -----------------: || | Agregar/quitar memoria en caliente                   |                6.0 || | Agregar/quitar adaptadores de red en caliente        |                5.0 || | Arranque seguro para máquinas virtuales Linux              |                6.0 || | Puntos de control de producción                 |                6.0 || | PowerShell Direct                      |                6.2 || | Módulo de plataforma de confianza virtual (vTPM) |                6.2 || | Agrupación de máquinas virtuales               |                6.2 ||



## Versión de configuración de la máquina virtual ##

Al mover o importar una máquina virtual en un host que ejecuta Hyper-V en Windows 10 desde un host que ejecuta Windows 8.1, el archivo de configuración de la máquina virtual no se actualiza automáticamente. Esto permite que la máquina virtual se mueva de vuelta a un host que ejecute Windows 8.1. No tendrá acceso a las nuevas características de la máquina virtual hasta que actualice manualmente la versión de configuración de la máquina virtual. 

La versión de configuración de la máquina virtual representa la versión de Hyper-V con la que son compatibles la configuración de la máquina virtual, el estado guardado y los archivos de instantáneas. Las máquinas virtuales con la versión 5 de configuración son compatibles con Windows 8.1 y puede ejecutarse en Windows 8.1 y Windows 10. Las máquinas virtuales con la versión 6 de configuración son compatibles con Windows 10 y no se ejecutarán en Windows 8.1.

No es necesario actualizar todas las máquinas virtuales simultáneamente. Puede elegir actualizar máquinas virtuales concretas cuando sea necesario. Sin embargo, no tendrá acceso a las nuevas características de la máquina virtual hasta que actualice manualmente la versión de configuración de cada máquina virtual.  


----------------
**Importante **

• Después de actualizar la versión de configuración de la máquina virtual, no se podrá mover la máquina virtual a un host que ejecute Windows 8.1.

• No se puede cambiar la versión de configuración de la máquina virtual de la versión 6 a la 5.

• Debe desactivar la máquina virtual para actualizar la configuración de máquina virtual.

• Después de la actualización, la máquina virtual usará el nuevo formato de archivo de configuración. Para más información, consulte el nuevo formato de archivo de configuración de máquina virtual.

--------





## ¿Qué ocurre al actualizar la versión de una máquina virtual?
Al actualizar manualmente la versión de configuración de una máquina virtual a la versión 6.x, cambiará la estructura de archivos que se utiliza para almacenar los archivos de configuración de las máquinas virtuales y los archivos de puntos de control. 

Las máquinas virtuales actualizadas usan un nuevo formato de archivo de configuración, que está diseñado para aumentar la eficacia de lectura y escritura de datos de configuración de máquina virtual. La actualización también reduce la posibilidad de daños en los datos si se produce un error de almacenamiento. 

En la siguiente tabla se enumeran las ubicaciones de los archivos binarios y la información de extensiones de una máquina virtual actualizada.  

|**Archivos de configuración de la máquina virtual y archivos de puntos de control (versión 6.x)**|**Descripción**||
|:---------------|:----------------|| |**Configuración de máquinas virtuales** | La información de configuración se almacena en un formato de archivo binario que utiliza la extensión .vmcx. || |**Estado del tiempo de ejecución de la máquina virtual** | Los datos del estado del tiempo de ejecución se almacenan en un formato de archivo binario que utiliza la extensión .vmrs.  || |**Disco duro virtual de la máquina virtual**|Los archivos del disco duro virtual de la máquina virtual. Utilizan las extensiones de archivo .vhd o .vhdx.   || |**Archivos automáticos de disco duro virtual**| Los diferentes archivos de disco que se usan para los puntos de control de la máquina virtual. Usan extensiones de archivos .avhdx. || |**Archivos de puntos de control** |Los puntos de control se almacenan en varios archivos de puntos de control. Cada punto de control crea un archivo de configuración y un archivo de estado del tiempo de ejecución. Los archivos de puntos de control utilizan las extensiones de archivo .vmrs y .vmcx. Estos nuevos formatos de archivo también se utilizan para los puntos de control de producción y los puntos de control estándares. ||

Después de actualizar la versión de configuración de la máquina virtual a la versión 6.x, no será posible volver de la versión 6.x a la versión 5. 

La máquina virtual debe estar desactivada para actualizar su configuración.

En la siguiente tabla se enumeran las ubicaciones de archivos predeterminadas de las máquinas virtuales nuevas o actualizadas.

|   **Archivos de la máquina virtual (versión 6.x)** | **Descripción** ||
|:-----|:-----|| |**Archivo de configuración de la máquina virtual**| C:\Archivos de Programa\Microsoft\Windows\Hyper-V\Virtual Machines || |**Archivo de estado del tiempo de ejecución de la máquina virtual**| C:\Archivos de Programa\Microsoft\Windows\Hyper-V\Virtual Machines || |**Archivos de puntos de control (.vmrs, .vmcx)**| C:\Archivos de Programa\Microsoft\Windows\Snapshots || |**Archivo del disco duro virtual (.vhd/.vhdx)**| C:\Usuarios\Acceso público\Documentos públicos\Hyper-V\Virtual Hard Disks || |**Archivos automáticos de disco duro virtual (.avhdx)**| C:\Usuarios\Acceso público\Documentos públicos\Hyper-V\Virtual Hard Disks ||







<!--HONumber=Jun16_HO4-->


