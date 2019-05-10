# <a name="hyper-v-backup-approaches"></a>Métodos de copia de seguridad de Hyper-V
Hyper-V permite a las máquinas virtuales copia de seguridad, desde el sistema operativo de host, sin necesidad de ejecutar software de copia de seguridad personalizado en la máquina virtual.  Hay varios enfoques que están disponibles para los desarrolladores usar según sus necesidades.
## <a name="hyper-v-vss-writer"></a>Escritor VSS de Hyper-V
Hyper-V implementa un escritor de VSS en todas las versiones de Windows Server que sean compatibles con Hyper-V.  Este sistema de escritura VSS permite a los desarrolladores usar la infraestructura existente de VSS para máquinas virtuales de copia de seguridad.  Sin embargo, se ha diseñado para las operaciones de copia de seguridad de pequeña escala donde todas las máquinas virtuales en un servidor se realiza una copia al mismo tiempo.

Para comprender mejor esta arquitectura: hacer referencia a esta presentación:https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>WMI de Hyper-V en función de copia de seguridad
A partir de Windows Server 2016, Hyper-V ha había comenzado a admitir la copia de seguridad a través de la API de WMI de Hyper-V.  Este enfoque aún utiliza VSS dentro de la máquina virtual para fines de copia de seguridad, pero ya no usa VSS en el sistema operativo host.  En su lugar, se usa una combinación de puntos de referencia y resistente (RCT) de seguimiento de cambios para permitir a los desarrolladores acceso a la información sobre cuando se realiza una copia de seguridad de máquinas virtuales de manera eficiente.  Este enfoque es más escalable que el uso de VSS en el host, sin embargo, solo está disponible en Windows Server 2016 y versiones posteriores.

Para comprender mejor esta arquitectura: hacer referencia a esta presentación:https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

También es un ejemplo de cómo usar estas API disponibles aquí:https://msconfiggallery.cloudapp.net/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Métodos para leer las copias de seguridad de copia de seguridad en función de WMI
Al crear las copias de seguridad de la máquina virtual con WMI de Hyper-V, hay tres métodos para leer los datos reales de la copia de seguridad.  Cada uno tiene ventajas e inconvenientes.
### <a name="wmi-export"></a>Exportación WMI
Los desarrolladores pueden exportar los datos de copia de seguridad a través de las interfaces de WMI de Hyper-V (tal y como se utiliza en el ejemplo anterior).  Hyper-V se compile los cambios en una unidad de disco duro virtual y copia el archivo a la ubicación solicitada.  Este método es fácil de usar, funciona para todos los escenarios y utilizables de forma remota.  Sin embargo, la unidad de disco duro virtual generada a menudo crea una gran cantidad de datos para transferirlos en la red.
### <a name="win32-apis"></a>API de Win32
Los desarrolladores pueden usar la SetVirtualDiskInformation, GetVirtualDiskInformation y QueryChangesVirtualDisk APIs en la API de Win32 de disco duro Virtual establecido como se indica aquí: https://docs.microsoft.com/windows/desktop/api/_vhd/ Ten en cuenta que para usar estas API, WMI de Hyper-V aún debe usarse para crear la referencia apunta en máquinas virtuales asociadas.  Estas API de Win32, a continuación, permitir el acceso eficaz a los datos de la máquina virtual de copia de seguridad.  Las API de Win32 tienen varias limitaciones:
*   Solo pueden acceder localmente
*   No no admiten la lectura de datos de los archivos de disco duro virtual compartidos
*   Devuelven las direcciones de datos que son relativas a la estructura interna de disco duro virtual

### <a name="remote-shared-virtual-disk-protocol"></a>Protocolo de disco Virtual compartido remoto
Por último, si un desarrollador necesita acceso eficaz a la información de copia de seguridad de datos desde un archivo de disco duro virtual compartido: necesitan usar el protocolo de disco Virtual compartido remoto.  Este protocolo se documenta [aquí](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15).
