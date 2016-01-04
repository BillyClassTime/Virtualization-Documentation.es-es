# Introducción a Hyper-V en Windows 10

Tanto si es un desarrollador de software, un profesional de TI o un aficionado a la tecnología, muchas personas necesitan ejecutar varios sistemas operativos, en ocasiones en varias máquinas. No todos nosotros tenemos acceso a un conjunto completo de laboratorios para alojar todos estos equipos y es por ello que la virtualización puede representar un ahorro de espacio y tiempo.

## Usos de la virtualización

La virtualización permite que cualquier persona mantenga fácilmente varios entornos de pruebas, que están compuestos por muchos sistemas operativos, configuraciones de software y configuraciones de hardware. Hyper-V ofrece virtualización en Windows, así como un mecanismo sencillo para cambiar rápidamente entre estos entornos sin incurrir en costos de hardware adicionales.

Hyper-V puede utilizarse de muchas maneras, por ejemplo:
- Puede crearse un entorno de prueba que conste de varias máquinas virtuales en un único equipo de escritorio o portátil. Cuando se completen las pruebas, estas máquinas virtuales se pueden exportar y después importar en cualquier otro sistema de Hyper-V.

- Los desarrolladores pueden usar Hyper-V en su equipo para probar software en varios sistemas operativos. Por ejemplo, si tiene una aplicación que se debe probar en los sistemas operativos Linux, Windows 7 y Windows 8, es posible crear varias máquinas virtuales en el sistema de desarrollo, cada una de las cuales contendrá cada uno de estos sistemas operativos.

- Puede usar Hyper-V para solucionar problemas de máquinas virtuales de cualquier implementación de Hyper-V. Puede exportar una máquina virtual del entorno de producción, abrirla en el escritorio que ejecuta Hyper-V, realizar la solución de problemas necesaria y luego exportarla de nuevo al entorno de producción.

- Mediante redes virtuales, puede crear un entorno con varios equipos para pruebas, desarrollo y demostración que sea seguro y no afecte a la red de producción.

- Los entusiastas pueden usarlo para experimentar con otros sistemas operativos. Hyper-V hace que sea muy fácil crear y eliminar distintos sistemas operativos.

- Puede usar Hyper-V en un portátil para mostrar versiones anteriores de Windows o sistemas operativos que no sean Windows.


## Requisitos del sistema

Hyper-V requiere un sistema de 64 bits que tenga traducción de direcciones de segundo nivel (SLAT). SLAT es una característica presente en la generación actual de los procesadores de 64 bits de Intel y AMD. También será necesaria una versión de 64 bits de Windows 8 o posterior y al menos 4 GB de RAM. Hyper-V admite la creación de sistemas operativos de 32 bits y 64 bits en las máquinas virtuales.

La memoria dinámica de Hyper-V permite que se asigne y desasigne la memoria que necesita la máquina virtual de forma dinámica (se especifica un mínimo y máximo) y compartir la memoria no utilizada entre máquinas virtuales. Puede ejecutar 3 o 4 máquinas virtuales en un equipo que tenga 4 GB de RAM, pero necesitará más memoria RAM para 5 o más máquinas virtuales. En el otro extremo del espectro, también puede crear máquinas virtuales grandes con 32 procesadores y 512 GB de RAM, dependiendo del hardware físico.

## Sistemas operativos que puede ejecutar en una máquina virtual

El término "invitado" hace referencia a una máquina virtual y "host" se refiere al equipo que ejecuta la máquina virtual. Hyper-V en Windows admite muchos sistemas operativos invitados diferentes, entre los que se incluyen varias versiones de Linux, FreeBSD y Windows. Para obtener información sobre los sistemas operativos que se admiten como invitados en Hyper-V en Windows, consulte [Sistemas operativos invitados que se admiten en Windows](supported_guest_os.md) y [Máquinas virtuales de Linux y FreeBSD en Hyper-V](https://technet.microsoft.com/library/dn531030.aspx).


## Diferencias entre Hyper-V en Windows y Hyper-V en Windows Server

Existen algunas características que funcionan de forma diferente en Hyper-V en Windows con respecto a Hyper-V en Windows Server. A continuación se enumeran algunas:

- El modelo de administración de memoria es diferente para Hyper-V en Windows. En un servidor, la memoria de Hyper-V se administra presuponiendo que solo las máquinas virtuales se ejecutan en el servidor. En Hyper-V en Windows, la memoria se administra teniendo en cuenta que la mayoría de los equipos clientes ejecutan software además de ejecutar máquinas virtuales. Por ejemplo, un programador podría ejecutar Visual Studio y varias máquinas virtuales en el mismo equipo.

- SR-IOV en un invitado de 64 bits funciona normalmente, pero en uno de 32 bits no funciona y no se admite.


### Características de Windows Server no disponibles en Windows Hyper-V

Existen algunas características que se incluyen en Hyper-V en Server, pero no están incluidas en Hyper-V en Windows. A continuación se enumeran algunas:

- La capacidad remota de FX para virtualizar GPU

- Migración en vivo de máquinas virtuales de un host a otro

- Réplica de Hyper-V

- Canal de fibra virtual

- Redes de SR-IOV

- .VHDX compartido


> **Advertencia**: Las máquinas virtuales que se ejecutan en Hyper-V no controlan automáticamente el paso de una conexión con cable a una conexión inalámbrica. Debe cambiar la configuración del adaptador de red de las máquinas virtuales de forma manual.

## Limitaciones

El uso de la virtualización tiene limitaciones. Las características o aplicaciones que dependen de hardware concreto no funcionarán bien en una máquina virtual. Por ejemplo, los juegos o aplicaciones que requieren un procesamiento con GPU (sin ofrecer reserva de software) podrían no funcionar bien. Además, las aplicaciones que dependen de los temporizadores por debajo de 10 ms, como las aplicaciones de alta precisión sensibles a la latencia (por ejemplo, las aplicaciones de mezcla de música en directo, etc.), podrían tener problemas para ejecutarse en una máquina virtual. El sistema operativo raíz también se ejecuta encima de la capa de virtualización de Hyper-V, pero es especial porque tiene acceso directo a todo el hardware. Este es el motivo por el que las aplicaciones con requisitos de hardware especiales siguen funcionando sin problemas en el sistema operativo raíz, pero las aplicaciones de alta precisión sensibles a la latencia podrían continuar teniendo problemas al ejecutarse en el SO raíz.

Como recordatorio, debe tener una licencia válida de los sistemas operativos que use en las máquinas virtuales.

## Paso siguiente:

[Tutorial de Hyper-V en Windows 10](..\quick_start\walkthrough.md)

Descubra las [novedades](whats_new.md) de Hyper-V en Windows 10.





