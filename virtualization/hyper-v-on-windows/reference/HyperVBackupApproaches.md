# <a name="hyper-v-backup-approaches"></a>Métodos de copia de seguridad de Hyper-V
Hyper-V le permite copia de seguridad de las máquinas virtuales, desde el sistema operativo host, sin necesidad de ejecutar el software de copia de seguridad personalizado dentro de la máquina virtual.  Existen varios enfoques que están disponibles para los programadores utilicen según sus necesidades.
## <a name="hyper-v-vss-writer"></a>Escritor de VSS de Hyper-V
Hyper-V implementa un escritor VSS en todas las versiones de donde es compatible con Hyper-V de Windows Server.  El escritor VSS permite a los desarrolladores utilizar la infraestructura existente de VSS para copia de seguridad de las máquinas virtuales.  Sin embargo, se ha diseñado para operaciones de copia de seguridad de pequeña escala donde todas las máquinas virtuales en un servidor se hace una copia simultáneamente.

Para comprender mejor esta arquitectura: hacer referencia a esta presentación:https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Backup basado en Hyper-V WMI
A partir de Windows Server 2016, Hyper-V ha había comenzado a admitir la copia de seguridad a través de la API de WMI de Hyper-V.  Este enfoque aún utiliza VSS dentro de la máquina virtual para fines de copia de seguridad, pero ya no utiliza VSS en el sistema operativo host.  En su lugar, se usa una combinación de puntos de referencia y resistente (RCT) de seguimiento de cambios para permitir que los programadores tener acceso a la información acerca de cuando se realiza una copia de seguridad de las máquinas virtuales de manera eficiente.  Este enfoque es más escalable que el uso de VSS en el host, pero sólo está disponible en Windows Server 2016 y versiones posteriores.

Para comprender mejor esta arquitectura: hacer referencia a esta presentación:https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

También es un ejemplo sobre cómo usar estas API disponibles aquí:https://msconfiggallery.cloudapp.net/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Métodos para leer las copias de seguridad de copia de seguridad en función de WMI
Al crear las copias de seguridad de la máquina virtual mediante WMI de Hyper-V, hay tres métodos para leer los datos reales de la copia de seguridad.  Cada uno tiene ventajas e inconvenientes.
### <a name="wmi-export"></a>Exportación WMI
Los desarrolladores pueden exportar los datos de copia de seguridad a través de las interfaces de WMI de Hyper-V (como se utiliza en el ejemplo anterior).  Hyper-V se compile los cambios en una unidad de disco duro virtual y copie el archivo en la ubicación solicitada.  Este método es fácil de usar, funciona para todos los escenarios y utilizables de forma remota.  Sin embargo, la unidad de disco duro virtual generada a menudo crea una gran cantidad de datos que transferir a través de la red.
### <a name="win32-apis"></a>API de Win32
Los programadores pueden utilizar la SetVirtualDiskInformation, GetVirtualDiskInformation y QueryChangesVirtualDisk APIs en la API de Win32 de disco duro Virtual establecer tal como se describe aquí: https://docs.microsoft.com/en-us/windows/desktop/api/_vhd/ tenga en cuenta que para usar estas API, WMI de Hyper-V aún necesita que se usará para crear la referencia puntos en máquinas virtuales asociadas.  A continuación, permitir estas API de Win32 para un acceso eficaz a los datos de la máquina virtual de copia de seguridad.  Las API de Win32 tienen varias limitaciones:
*   Sólo puede acceder localmente
*   No no admiten la lectura de datos de los archivos de disco duro virtual compartidos
*   Devuelven las direcciones de datos que son relativas a la estructura interna del disco duro virtual

### <a name="remote-shared-virtual-disk-protocol"></a>Protocolo de disco Virtual compartido remoto
Por último, si un programador necesita acceso eficiente a información de copia de seguridad de datos desde un archivo de disco duro virtual compartido – necesitará usar el protocolo de disco Virtual compartido remoto.  Este protocolo se documenta aquí:https://msdn.microsoft.com/en-us/library/dn393384.aspx
