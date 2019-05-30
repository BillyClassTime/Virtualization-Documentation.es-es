---
title: Configuración de máquinas virtuales anidadas para comunicarse directamente con recursos en una red virtual de Azure
description: Virtualización anidada
keywords: Windows 10, Hyper-v, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: 2f1c6a124ba4f2f9d199d3cc5bb38c9082f72b3d
ms.sourcegitcommit: a7f9ab96be359afb37783bbff873713770b93758
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2019
ms.locfileid: "9681145"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Configurar máquinas virtuales anidadas para comunicarse con recursos en una red virtual de Azure

La orientación original sobre la implementación y la configuración de máquinas virtuales anidadas dentro de Azure requieren que usted acceda a estas máquinas virtuales a través de un conmutador NAT. Esto presenta varias limitaciones:

1. Las máquinas virtuales anidadas no pueden tener acceso a recursos locales o dentro de una red virtual de Azure.
2. Los recursos locales o los recursos de Azure solo pueden acceder a las máquinas virtuales anidadas a través de un NAT, lo que significa que varios invitados no pueden compartir el mismo puerto.

Este documento le guiará a través de una implementación en la que usamos RRAS, rutas definidas por el usuario, una subred dedicada al NAT saliente para permitir el acceso de Internet invitado y un espacio de direcciones "flotante" que permita que las VM anidadas se comporten y se comuniquen como cualquier otra máquina virtual se implementa directamente en una VNet dentro de Azure.

Antes de comenzar esta guía, haga lo siguiente:

1. Lea las [instrucciones proporcionadas aquí](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization) en la virtualización anidada.
2. Lea este artículo completo antes de la implementación.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>Información general de lo que estamos haciendo y por qué.
* Crearemos una VM con capacidad de anidamiento que tenga dos NICs. 
* Se usará una NIC para proporcionar nuestras máquinas virtuales anidadas con acceso a Internet a través de NAT y la otra NIC se usará para enrutar el tráfico de nuestro conmutador interno a los recursos externos al hipervisor. Cada NIC tendrá que estar en un dominio de enrutamiento diferente, lo que significa una subred diferente.
* Esto significa que necesitaremos una red virtual con al menos tres subredes. Una para NAT, una para enrutamiento LAN y otra que no se usan, pero que está "reservada" para nuestras máquinas virtuales anidadas. Los nombres que usamos para las subredes de este documento son "NAT", "Hyper-V-LAN" y "fantasma".
* El tamaño de estas subredes está a su discreción, pero hay algunas consideraciones. El tamaño de las subredes "fantasma" determina el número de direcciones IP que tiene para sus VM anidadas. Además, el tamaño de las subredes "NAT" y "Hyper-V-LAN" determina el número de direcciones IP que tiene para los hipervisores. Por lo tanto, podrías hacer que las subredes sean realmente pequeñas aquí si solo planeabas tener uno o dos hipervisores.
* Contexto: las máquinas virtuales anidadas no recibirán DHCP de la VNet a la que su host está conectado aunque configure un conmutador interno o externo. 
  * Esto significa que el host de Hyper-V debe proporcionar DHCP.
* El host de Hyper-V no reconoce las concesiones actualmente asignadas en la VNet, por lo que, para evitar una situación en la que el host asigna una IP ya existente, debemos asignar un bloque de IPs para que lo use solo el host de Hyper-V. Esto nos permitirá evitar un escenario de IP duplicado.
  * El bloque de IPs que elijas corresponderá a una subred dentro de la misma VNet en la que se encuentra su Hyper-V.
  * El motivo por el que deseamos que corresponda a una subred existente es realizar la administración de los anuncios BGP a través de ExpressRoute. Si acabamos de crear un intervalo IP para que el host Hyper-V lo usara, deberíamos crear una serie de rutas estáticas para permitir que los clientes locales se comuniquen con las máquinas virtuales anidadas. Esto significa que esto no es un requisito difícil, ya que puede hacer un intervalo IP para las máquinas virtuales anidadas y, a continuación, crear todas las rutas necesarias para dirigir a los clientes al host de Hyper-V para ese intervalo.
* Crearemos un conmutador interno dentro de Hyper-V y, a continuación, asignaremos a la interfaz recién creada una dirección IP dentro de un rango que describimos para DHCP. Esta dirección IP se convertirá en la puerta de enlace predeterminada de nuestras máquinas virtuales anidadas y se usará para la ruta entre el conmutador interno y la NIC del host que está conectado a nuestra red virtual.
* Instalaremos el rol de enrutamiento y acceso remoto en el host, lo que hará que nuestro host se convierta en un enrutador.  Esto es necesario para permitir la comunicación entre recursos externos al host y a nuestras máquinas virtuales anidadas.
* Le indicaremos a otros recursos cómo obtener acceso a estas máquinas virtuales anidadas. Esto obliga a crear una tabla de rutas definida por el usuario que contenga una ruta estática para el intervalo IP en el que residen las máquinas virtuales anidadas. Esta ruta estática apuntará a la dirección IP de Hyper-V.
* A continuación, colocarás este UDR en la subred de la puerta de enlace para que los clientes provenientes de instalaciones locales sepan llegar a nuestras máquinas virtuales anidadas.
* También colocarás esta UDR en cualquier otra subred dentro de Azure que requiera conectividad a las máquinas virtuales anidadas.
* Para varios hosts de Hyper-V, crearía subredes "flotantes" adicionales y agregaría una ruta estática adicional a la UDR.
* Al retirar un host de Hyper-V, eliminará o volverá a utilizar nuestra subred "flotante" y quitará la ruta estática de nuestro UDR, o si este es el último host de Hyper-V, quite la UDR por completo.

## <a name="creating-the-host"></a>Crear el host

Mostraré los valores de configuración que tengan una preferencia personal, como el nombre de la VM, el grupo de recursos, etc.

1. Vaya a portal.azure.com
2. Haga clic en "crear un recurso" en la esquina superior izquierda
3. Seleccione "Windows Server 2016 VM" de la columna popular
4. En la pestaña "conceptos básicos" Asegúrate de seleccionar un tamaño de VM capaz de virtualizar la virtualización.
5. Ir a la pestaña "funciones de red"
6. Crear una nueva red virtual con la siguiente configuración
    * Espacio de direcciones de red virtual: 10.0.0.0/22
    * Subred 1
        * Nombre: NAT
        * Espacio de direcciones: 10.0.0.0/24
    * Subred 2
        * Nombre: Hyper-V-LAN
        * Espacio de direcciones: 10.0.1.0/24
    * Subred 3
        * Nombre: fantasma
        * Espacio de direcciones: 10.0.2.0/24
    * Subred 4
        * Nombre: Azure-VMs
        * Espacio de direcciones: 10.0.3.0/24
7. Asegúrese de haber seleccionado la subred NAT para la VM
8. Ve a "revisar + crear" y selecciona "crear".

## <a name="create-the-second-network-interface"></a>Crear la segunda interfaz de red
1. Una vez que la VM haya terminado de provisionar, búsquelo en el portal de Azure
2. Detener la VM
3. Una vez que se ha detenido, ve a "redes" en configuración
4. "Adjuntar interfaz de red"
5. "Crear interfaz de red"
6. Dele un nombre (no importa lo que te asignes, pero asegúrate de recordarlo).
7. Seleccione "Hyper-V-LAN" para la subred
8. Asegúrese de seleccionar el mismo grupo de recursos en el que reside su host
9. Crée
10. Esto le llevará a la pantalla anterior, asegúrese de seleccionar la interfaz de red que acaba de crear y seleccionar "Aceptar".
11. Vuelva al panel "Descripción general" y vuelva a iniciar la VM una vez completada la acción anterior
12. Vaya a la segunda NIC que acabamos de crear, puede encontrarla en el grupo de recursos que seleccionó anteriormente
13. Vaya a "configuraciones de IP" y alterne "reenvío de IP" a "habilitado" y, después, guarde el cambio.

## <a name="setting-up-hyper-v"></a>Configuración de Hyper-V
1. Remoto en su host
2. Abrir un símbolo del sistema de PowerShell con privilegios elevados
3. Ejecute el siguiente comando: `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`
4. Esto reiniciará el host
5. Vuelva a conectar con el host para continuar con el resto de la configuración

## <a name="creating-our-virtual-switch"></a>Creación de nuestro Switch virtual

1. Abra PowerShell en el modo administrativo.
2. Crear un modificador interno: `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. Asignar la interfaz recién creada a IP: `New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>Instalar y configurar DHCP

*Muchas personas pierden este componente cuando intentan obtener una virtualización anidada en primer lugar. A diferencia de las instalaciones locales en las que las VM de invitado recibirán DHCP de la red en la que se encuentra su host, se debe proporcionar la VM anidada en Azure a través del host en el que se ejecutan. O bien, debe asignar estáticamente una dirección IP a cada VM anidada, que no es escalable.*

1. Instale el rol DHCP: `Install-WindowsFeature DHCP -IncludeManagementTools`
2. Crear el ámbito DHCP: `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`
3. Configure las opciones de la puerta de enlace DNS y predeterminada para el ámbito: `Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`
    * Asegúrese de introducir un servidor DNS válido si quiere que la resolución de nombres funcione. En este caso, uso el [DNS recursivo de Azure](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances).

## <a name="installing-remote-access"></a>Instalación de acceso remoto

1. Abra el administrador del servidor y seleccione Agregar roles y características.
2. Seleccione "siguiente" hasta que llegue a "roles de servidor".
3. Active "acceso remoto" y haga clic en "siguiente" hasta que llegue a "servicios de rol".
4. Active "Routing", seleccione "Add features" y, a continuación, seleccione "Next" (instalar). Complete el asistente y espere a que se complete la instalación.

## <a name="configuring-remote-access"></a>Configurar el acceso remoto

1. Abra el administrador del servidor y seleccione "herramientas" y, a continuación, seleccione "enrutamiento y acceso remoto".
2. En el lado izquierdo del panel de administración de enrutamiento y acceso remoto verá un icono con el nombre de su servidor junto a él, haga clic con el botón secundario en esta opción y seleccione "configurar y Habilitar enrutamiento y acceso remoto".
3. En el asistente, seleccione "siguiente", busque "configuración personalizada" en el botón radial y, después, seleccione "siguiente".
4. Active "NAT" y "enrutamiento de LAN" y, a continuación, seleccione "siguiente" y, después, "finalizar". Si le pide que inicie el servicio, hágalo.
5. Ahora, navegue hasta el nodo "IPv4" y expándalo para que el nodo "NAT" esté disponible.
6. Haga clic con el botón secundario en "NAT", seleccione "nueva interfaz...". y selecciona "Ethernet", esta es tu primer NIC con el IP de "10.0.0.4"
7. Ahora necesitamos crear algunas rutas estáticas para forzar el tráfico LAN a la segunda NIC. Para ello, vaya al nodo "rutas estáticas" en "IPv4".
8. Una vez que hayamos creado las siguientes rutas.
    * Ruta 1
        * Interfaz: Ethernet
        * Destino: 10.0.0.0
        * Máscara de red: 255.255.255.0
        * Puerta de enlace: 10.0.0.1
        * Métrica: 256
        * Nota: Aquí incluimos esta opción para permitir que la NIC principal responda a su propia interfaz. Si no hemos tenido esto aquí, la siguiente ruta podría causar que el tráfico para NIC 1 salga a NIC 2. Esto crearía una ruta asimétrica. 10.0.0.1 es la dirección IP que Azure asigna a la subred NAT. Azure usa la primera IP disponible en un rango como puerta de enlace predeterminada. Por tanto, si hubiera usado 192.168.0.0/24 para tu subred NAT, la puerta de enlace sería 192.168.0.1. En el enrutamiento, la ruta más específica gana, lo que significa que esta ruta sustituye a la siguiente ruta.

    * Ruta 2
        * Interfaz: Ethernet 2
        * Destino: 10.0.0.0
        * Máscara de red: 255.255.252.0
        * Puerta de enlace: 10.0.1.1
        * Métrica: 256
        * Nota: esta es una ruta de captura para tráfico dirigido a nuestra VNet de Azure. Forzará el tráfico de la segunda NIC. Tendrá que agregar rutas adicionales para otros rangos a los que desee que tengan acceso las VM anidadas. Por lo tanto, si usted es una red local es 172.16.0.0/22, quiere tener otra ruta para enviar ese tráfico a la segunda NIC de nuestro hipervisor.

## <a name="creating-a-route-table-within-azure"></a>Crear una tabla de rutas en Azure

Consulte [este artículo](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-portal) para obtener más información sobre la creación y la administración de rutas dentro de Azure.

1. Vaya a https://portal.azure.com.
2. En la esquina superior izquierda, seleccione "crear un recurso".
3. En el campo de búsqueda, escriba "tabla de enrutamiento" y presione Entrar.
4. El resultado superior será tabla de rutas, seleccione esta y, a continuación, seleccione "crear".
5. Asigne un nombre a la tabla de rutas, en el caso mi nombre "Routes-for-Nested-VM".
6. Asegúrese de seleccionar la misma suscripción en la que residen los hosts de Hyper-V.
7. Cree un nuevo grupo de recursos o seleccione uno existente y asegúrese de que la región en la que se crea la tabla de enrutamiento es la misma región en la que se encuentra el host de Hyper-V.
8. Seleccione "crear".

## <a name="configuring-the-route-table"></a>Configurar la tabla de rutas

1. Vaya a la tabla de rutas que acabamos de crear. Para ello, busque el nombre de la tabla de enrutamiento en la barra de búsqueda de la parte central superior del portal.
2. Una vez que haya seleccionado la tabla de rutas, vaya a "rutas" desde el blade.
3. Seleccione "agregar".
4. Asigne un nombre a su ruta, fui con "máquinas virtuales anidadas".
5. Para el prefijo de dirección, escribe el intervalo IP de nuestra subred "flotante". En este caso sería 10.0.2.0/24.
6. En "tipo de próximo salto", seleccione "dispositivo virtual" y, a continuación, escriba la dirección IP del segundo NIC de hosts de Hyper-V, que sería 10.0.1.4 y, a continuación, seleccione "Aceptar".
7. Ahora desde dentro del Blade selecciona "subredes", que estará directamente debajo de "rutas".
8. Seleccione "Associate" (asociarse), seleccione nuestra VNet "Nested-Fun" (la que se va a anidar), seleccione la subred "Azure-VMs" y, después, seleccione "Aceptar".
9. Realice este mismo procedimiento para la subred en la que se encuentra el host de Hyper-V y también para cualquier otra subsubred que necesite obtener acceso a las máquinas virtuales anidadas. Si está conectado 

# <a name="end-state-configuration-reference"></a>Referencia de configuración del estado final
El entorno de esta guía tiene las siguientes configuraciones. Esta sección es inteded para usar como referencia.

1. Información de la red virtual de Azure.
    * Configuración de alto nivel de VNet.
        * Nombre: diversión
        * Espacio de direcciones: 10.0.0.0/22
        * Nota: esta estará constituida por cuatro subredes. Además, estos rangos no se establecen en piedra. Si lo desea, no dude en abordar su entorno. 

    * Configuración de alto nivel de subred.
        * Nombre: NAT
        * Espacio de direcciones: 10.0.0.0/24
        * Nota: aquí es donde residen los hosts de la NIC principal. Esto se usará para controlar NAT saliente para las VM anidadas. Será la puerta de enlace a Internet de las máquinas virtuales anidadas.

    * Configuración de alto nivel de subred de la segunda.
        * Nombre: Hyper-V-LAN
        * Espacio de direcciones: 10.0.1.0/24
        * Nota: nuestro host de Hyper-V tendrá una segunda NIC que se usará para controlar el enrutamiento entre la VM anidada y los recursos que no son de Internet externos al host de Hyper-V.

    * Configuración de alto nivel de subred de terceros.
        * Nombre: fantasma
        * Espacio de direcciones: 10.0.2.0/24
        * Nota: esta será una subred "flotante". Nuestras máquinas virtuales anidadas usarán el espacio de direcciones y existirá para administrar los anuncios de ruta de regreso a local. En realidad, no se implementarán máquinas virtuales en esta subred.

    * La cuarta configuración de nivel de subred.
        * Nombre: Azure-VMs
        * Espacio de direcciones: 10.0.3.0/24
        * Nota: subred que contiene máquinas virtuales de Azure.

1. Nuestro host de Hyper-V tiene las siguientes configuraciones de NIC.
    * NIC principal 
        * Dirección IP: 10.0.0.4
        * Máscara de subred: 255.255.255.0
        * Puerta de enlace predeterminada: 10.0.0.1
        * DNS: configurado para DHCP
        * Reenvío IP habilitado: no

    * NIC secundaria
        * Dirección IP: 10.0.1.4
        * Máscara de subred: 255.255.255.0
        * Puerta de enlace predeterminada: Empty
        * DNS: configurado para DHCP
        * Reenvío IP habilitado: sí

    * La NIC creada con Hyper-V para el conmutador virtual interno
        * Dirección IP: 10.0.2.1
        * Máscara de subred: 255.255.255.0
        * Puerta de enlace predeterminada: Empty

3. Nuestra tabla de rutas tendrá una sola regla.
    * Regla 1
        * Nombre: máquinas virtuales anidadas
        * Destino: 10.0.2.0/24
        * Próximo salto: dispositivo virtual-10.0.1.4

## <a name="conclusion"></a>Conclusión

Ahora debe poder implementar una máquina virtual (incluso una VM de 32 bits) en el host de Hyper-V y hacer que sea accesible desde local y dentro de Azure.
