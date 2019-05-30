# <a name="hyper-v-backup-approaches"></a>Enfoques de copia de seguridad de Hyper-V
Hyper-V permite realizar copias de seguridad de máquinas virtuales, desde el sistema operativo host, sin necesidad de ejecutar software de copia de seguridad personalizado dentro de la máquina virtual.  Existen varios enfoques disponibles para que los desarrolladores puedan usarse en función de sus necesidades.
## <a name="hyper-v-vss-writer"></a>Escritor de VSS de Hyper-V
Hyper-V implementa un escritor de VSS en todas las versiones de Windows Server donde se admite Hyper-V.  Este VSS Writer permite a los desarrolladores usar la infraestructura de VSS existente para realizar copias de seguridad de las máquinas virtuales.  Sin embargo, está diseñado para operaciones de copia de seguridad de pequeña escala en la que se realiza una copia de seguridad de todas las máquinas virtuales de un servidor de forma simultánea.

Para comprender mejor esta arquitectura, consulte esta presentación:https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Copia de seguridad basada en WMI de Hyper-V
A partir de Windows Server 2016, Hyper-V comenzó a admitir la copia de seguridad a través de la API WMI de Hyper-V.  Este enfoque aún utiliza VSS dentro de la máquina virtual con fines de backup, pero ya no usa VSS en el sistema operativo del host.  En su lugar, se usa una combinación de puntos de referencia y seguimiento de cambios resistente (RCT) para permitir a los programadores el acceso a la información relacionada con la copia de seguridad de las máquinas virtuales de manera eficaz.  Este enfoque es más escalable que el uso de VSS en el host, pero solo está disponible en Windows Server 2016 y versiones posteriores.

Para comprender mejor esta arquitectura, consulte esta presentación:https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

También hay un ejemplo sobre cómo usar estas API disponibles aquí:https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Métodos para leer copias de seguridad de una copia de seguridad basada en WMI
Al crear copias de seguridad de máquina virtual con WMI de Hyper-V, hay tres métodos para leer los datos reales de la copia de seguridad.  Cada uno tiene ventajas y desventajas exclusivas.
### <a name="wmi-export"></a>Exportación de WMI
Los programadores pueden exportar los datos de copia de seguridad a través de las interfaces WMI de Hyper-V (como se usa en el ejemplo anterior).  Hyper-V compilará los cambios en una unidad de disco duro virtual y copiará el archivo en la ubicación solicitada.  Este método es fácil de usar, funciona en todos los escenarios y es remoto.  Sin embargo, la unidad de disco duro virtual generada a menudo crea una gran cantidad de datos para transferir a través de la red.
### <a name="win32-apis"></a>API de Win32
Los programadores pueden usar las API SetVirtualDiskInformation, GetVirtualDiskInformation y QueryChangesVirtualDisk en el conjunto de API de disco duro virtual Win32 https://docs.microsoft.com/windows/desktop/api/_vhd/ , tal y como se explica aquí: tenga en cuenta que para usar estas API, WMI debe usarse para crear referencias puntos de las máquinas virtuales asociadas.  Estas API de Win32 permiten un acceso eficaz a los datos de la máquina virtual de la que se ha realizado una copia de seguridad.  Las API de Win32 tienen varias limitaciones:
*   Solo se puede tener acceso a ellos de forma local.
*   No se admite la lectura de datos desde archivos de disco duro virtual compartido
*   Devuelven direcciones de datos que son relativas a la estructura interna del disco duro virtual

### <a name="remote-shared-virtual-disk-protocol"></a>Protocolo de disco virtual compartido remoto
Por último, si un desarrollador necesita obtener acceso eficazmente a la información de los datos de copia de seguridad de un archivo de disco duro virtual compartido, tendrá que usar el protocolo de disco virtual compartido remoto.  Este protocolo se documenta [aquí](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15).
