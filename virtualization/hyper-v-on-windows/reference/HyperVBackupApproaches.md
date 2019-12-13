# <a name="hyper-v-backup-approaches"></a>Enfoques de copia de seguridad de Hyper-V
Hyper-V permite realizar copias de seguridad de máquinas virtuales, desde el sistema operativo host, sin necesidad de ejecutar software de copia de seguridad personalizado dentro de la máquina virtual.  Existen varios enfoques que los desarrolladores pueden emplear en función de sus necesidades.
## <a name="hyper-v-vss-writer"></a>VSS Writer de Hyper-V
Hyper-V implementa una VSS Writer en todas las versiones de Windows Server en las que se admite Hyper-V.  Esta VSS Writer permite a los desarrolladores emplear la infraestructura de VSS existente para realizar copias de seguridad de las máquinas virtuales.  Sin embargo, está diseñada para operaciones de copia de seguridad a pequeña escala en las que se realiza una copia de seguridad de todas las máquinas virtuales de un servidor simultáneamente.

Para comprender mejor esta arquitectura, consulte esta presentación: https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Copia de seguridad basada en WMI de Hyper-V
A partir de Windows Server 2016, Hyper-V comenzó a admitir la copia de seguridad a través de la API de WMI de Hyper-V.  Este enfoque todavía usa VSS dentro de la máquina virtual para realizar copias de seguridad, pero ya no usa VSS en el sistema operativo host.  En su lugar, se usa una combinación de puntos de referencia y el seguimiento de cambios resistentes (RCT) para permitir a los desarrolladores tener acceso a la información sobre las máquinas virtuales de las que se ha realizado una copia de seguridad de una manera eficaz.  Este enfoque es más escalable que el uso de VSS en el host, pero solo está disponible en Windows Server 2016 y versiones posteriores.

Para comprender mejor esta arquitectura, consulte esta presentación: https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

También hay un ejemplo de cómo usar estas API disponibles aquí: https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Métodos para leer copias de seguridad de la copia de seguridad basada en WMI
Al crear copias de seguridad de máquinas virtuales mediante WMI de Hyper-V, existen tres métodos para leer los datos reales de la copia de seguridad.  Cada una tiene ventajas y desventajas exclusivas.
### <a name="wmi-export"></a>Exportación WMI
Los desarrolladores pueden exportar los datos de copia de seguridad a través de las interfaces WMI de Hyper-V (como se usa en el ejemplo anterior).  Hyper-V compilará los cambios en una unidad de disco duro virtual y copiará el archivo en la ubicación solicitada.  Este método es fácil de usar, funciona en todos los escenarios y es remoto.  Sin embargo, el disco duro virtual generado a menudo crea una gran cantidad de datos para transferir a través de la red.
### <a name="win32-apis"></a>API de Win32
Los desarrolladores pueden usar las API SetVirtualDiskInformation, GetVirtualDiskInformation y QueryChangesVirtualDisk en el conjunto de API Win32 de disco duro virtual, tal como se documenta aquí: https://docs.microsoft.com/windows/desktop/api/_vhd/ tenga en cuenta que para usar estas API, es necesario usar WMI de Hyper-V para crear puntos de referencia en las máquinas virtuales asociadas.  Estas API de Win32 permiten el acceso eficaz a los datos de la máquina virtual de la que se ha realizado una copia de seguridad.  Las API de Win32 tienen varias limitaciones:
* Solo se puede tener acceso a ellos de forma local
* No admite la lectura de datos de archivos de disco duro virtual compartido
* Devuelven direcciones de datos que son relativas a la estructura interna del disco duro virtual

### <a name="remote-shared-virtual-disk-protocol"></a>Protocolo de disco virtual compartido remoto
Por último, si un desarrollador necesita acceder de forma eficaz a la información de datos de copia de seguridad desde un archivo de disco duro virtual compartido, deberá usar el protocolo de disco virtual compartido remoto.  Este protocolo se documenta [aquí](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15).
