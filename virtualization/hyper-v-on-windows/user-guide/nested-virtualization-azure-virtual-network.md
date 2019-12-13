---
title: Configuración de máquinas virtuales anidadas para comunicarse directamente con los recursos de una Virtual Network de Azure
description: Virtualización anidada
keywords: Windows 10, Hyper-v, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: efd180c458457da1cea6b379e21ba3a37083d15a
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910965"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Configuración de máquinas virtuales anidadas para comunicarse con recursos en una red virtual de Azure

La orientación original sobre la implementación y configuración de máquinas virtuales anidadas dentro de Azure requiere que tenga acceso a estas máquinas virtuales a través de un conmutador NAT. Esto presenta varias limitaciones:

1. Las máquinas virtuales anidadas no pueden tener acceso a recursos locales o dentro de una Virtual Network de Azure.
2. Los recursos o recursos locales dentro de Azure solo pueden acceder a las máquinas virtuales anidadas a través de una NAT, lo que significa que varios invitados no pueden compartir el mismo puerto.

Este documento le guiará a través de una implementación en la que usamos RRAS, rutas definidas por el usuario, una subred dedicada a NAT de salida para permitir el acceso a Internet de invitados y un espacio de direcciones "flotante" para permitir que las máquinas virtuales anidadas se comporten y se comuniquen como cualquier otra máquina virtual. implementado directamente en una red virtual dentro de Azure.

Antes de empezar esta guía, haga lo siguiente:

1. Lea las [instrucciones que se proporcionan aquí](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization) en la virtualización anidada.
2. Lea este artículo completo antes de la implementación.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>Información general de alto nivel sobre lo que estamos haciendo y por qué
* Vamos a crear una máquina virtual compatible con el anidamiento que tiene dos NIC. 
* Se usará una NIC para proporcionar a nuestras máquinas virtuales anidadas acceso a Internet a través de NAT y la otra NIC se usará para enrutar el tráfico desde nuestro conmutador interno a los recursos externos al hipervisor. Cada NIC deberá estar en un dominio de enrutamiento distinto, lo que significa que se trata de una subred diferente.
* Esto significa que necesitaremos un Virtual Network con como mínimo tres subredes. Una para NAT, una para el enrutamiento de LAN y otra que no se usa, pero que está "reservada" para nuestras máquinas virtuales anidadas. Los nombres que usamos para las subredes de este documento son "NAT", "Hyper-V-LAN" y "fantasma".
* El tamaño de estas subredes depende de su discreción, pero hay algunas consideraciones. El tamaño de las subredes "fantasma" determina el número de direcciones IP que tiene para las máquinas virtuales anidadas. Además, el tamaño de las subredes "NAT" y "Hyper-V-LAN" determina el número de direcciones IP que tiene para los hipervisores. Por lo tanto, puede realizar técnicamente subredes realmente pequeñas si solo estaba planeando tener uno o dos hipervisores.
* Segundo plano: las máquinas virtuales anidadas no recibirán DHCP de la red virtual a la que está conectado su host aunque configure un conmutador interno o externo. 
  * Esto significa que el host de Hyper-V debe proporcionar DHCP.
* El host de Hyper-V no es consciente de las concesiones asignadas actualmente en la red virtual, por lo que para evitar una situación en la que el host asigna una IP ya existente, se debe asignar un bloque de direcciones IP para su uso solo por el host de Hyper-V. Esto nos permitirá evitar un escenario de IP duplicado.
  * El bloque de direcciones IP que elijas se corresponderá con una subred dentro de la misma red virtual en la que se encuentra Hyper-V.
  * La razón por la que queremos que esto se corresponda con una subred existente es controlar los anuncios BGP de nuevo en ExpressRoute. Si se acaba de crear un intervalo de direcciones IP para el host de Hyper-V que se va a usar, tendrá que crear una serie de rutas estáticas para permitir que los clientes locales se comuniquen con las máquinas virtuales anidadas. Esto significa que esto no es un requisito estricto, ya que podría conformar un intervalo de direcciones IP para las máquinas virtuales anidadas y, después, crear todas las rutas necesarias para dirigir a los clientes al host de Hyper-V para ese intervalo.
* Vamos a crear un conmutador interno en Hyper-V y, a continuación, asignaremos a la interfaz recién creada una dirección IP dentro de un intervalo que se reserva para DHCP. Esta dirección IP se convertirá en la puerta de enlace predeterminada para nuestras máquinas virtuales anidadas y se usará para la ruta entre el conmutador interno y la NIC del host que está conectado a la red virtual.
* Se instalará el rol de enrutamiento y acceso remoto en el host, que convertirá el host en un enrutador.  Esto es necesario para permitir la comunicación entre los recursos externos al host y nuestras máquinas virtuales anidadas.
* Le indicaremos a otros recursos cómo acceder a estas máquinas virtuales anidadas. Esto requiere que se cree una tabla de rutas definida por el usuario que contiene una ruta estática para el intervalo de direcciones IP en el que residen las máquinas virtuales anidadas. Esta ruta estática apuntará a la dirección IP de Hyper-V.
* A continuación, colocará este UDR en la subred de puerta de enlace para que los clientes procedentes de local sepan cómo llegar a nuestras máquinas virtuales anidadas.
* También colocará esta UDR en cualquier otra subred de Azure que requiera conectividad con las máquinas virtuales anidadas.
* En el caso de varios hosts de Hyper-V, se crean subredes "flotantes" adicionales y se agrega una ruta estática adicional a UDR.
* Al retirar un host de Hyper-V, eliminará o reasignará la subred "flotante" y quitará la ruta estática de nuestro UDR, o bien, si se trata del último host de Hyper-V, quite el UDR por completo.

## <a name="creating-the-host"></a>Creación del host

Explicaré los valores de configuración que tengan preferencia personal, como el nombre de la máquina virtual, el grupo de recursos, etc.

1. Vaya a portal.azure.com
2. Haga clic en "crear un recurso" en la parte superior izquierda
3. Seleccione "Windows Server 2016 VM" en la columna popular
4. En la pestaña "conceptos básicos", asegúrese de seleccionar un tamaño de máquina virtual que sea capaz de la virtualización anidada
5. Vaya a la pestaña "redes"
6. Cree una nueva Virtual Network con la configuración siguiente
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
        * Nombre: Azure-VM
        * Espacio de direcciones: 10.0.3.0/24
7. Asegúrese de que ha seleccionado la subred NAT para la máquina virtual
8. Vaya a "revisar + crear" y seleccione "crear".

## <a name="create-the-second-network-interface"></a>Creación de la segunda interfaz de red
1. Una vez que la máquina virtual ha finalizado el aprovisionamiento, búsquelo en Azure portal.
2. detener la máquina virtual.
3. Una vez detenido, vaya a "redes" en configuración.
4. "Asociar interfaz de red"
5. "Crear interfaz de red"
6. Asígnele un nombre (no importa cuál sea su nombre, pero no olvide recordarlo)
7. Seleccione "Hyper-V-LAN" para la subred
8. Asegúrese de seleccionar el mismo grupo de recursos en el que reside el host
9. A
10. Esto le llevará a la pantalla anterior, asegúrese de seleccionar la interfaz de red que acaba de crear y seleccione "Aceptar".
11. Vuelva al panel "información general" e inicie la máquina virtual de nuevo una vez completada la acción anterior.
12. Navegue hasta la segunda NIC que acabamos de crear, puede encontrarla en el grupo de recursos que seleccionó anteriormente.
13. Vaya a "configuraciones de IP" y cambie "reenvío IP" a "habilitado" y guarde el cambio.

## <a name="setting-up-hyper-v"></a>Configuración de Hyper-V
1. Remota en el host
2. Abra un símbolo del sistema de PowerShell con privilegios elevados
3. Ejecute el siguiente comando `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`
4. Se reiniciará el host
5. Vuelva a conectarse al host para continuar con el resto de la configuración.

## <a name="creating-our-virtual-switch"></a>Creación de nuestro conmutador virtual

1. Abra PowerShell en modo administrativo.
2. Crear un conmutador interno: `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. Asigne a la interfaz recién creada una dirección IP: `New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>Instalación y configuración de DHCP

*Muchas personas pierden este componente cuando están intentando obtener la virtualización anidada en funcionamiento. A diferencia de lo que ocurre en el entorno local, en el que las máquinas virtuales invitadas recibirán DHCP de la red en la que reside el host, las máquinas virtuales anidadas en Azure deben proporcionarse a través del host en el que se ejecutan. O bien, debe asignar estáticamente una dirección IP a cada máquina virtual anidada, que no es escalable.*

1. Instale el rol DHCP: `Install-WindowsFeature DHCP -IncludeManagementTools`
2. Crear el ámbito DHCP: `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`
3. Configure las opciones DNS y puerta de enlace predeterminada para el ámbito: `Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`
    * Asegúrese de especificar un servidor DNS válido si desea que funcione la resolución de nombres. En este caso, estoy usando [DNS recursivo de Azure](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances).

## <a name="installing-remote-access"></a>Instalación de acceso remoto

1. Abra Administrador del servidor y seleccione "Agregar roles y características".
2. Seleccione "siguiente" hasta llegar a "roles de servidor".
3. Active "acceso remoto" y haga clic en "siguiente" hasta llegar a "servicios de rol".
4. Active "enrutamiento", seleccione "agregar características" y, a continuación, seleccione "siguiente" y, a continuación, "instalar". Complete el asistente y espere a que se complete la instalación.

## <a name="configuring-remote-access"></a>Configuración del acceso remoto

1. Abra Administrador del servidor y seleccione "herramientas" y, a continuación, seleccione "enrutamiento y acceso remoto".
2. En el lado izquierdo del panel de administración de enrutamiento y acceso remoto, verá un icono con el nombre de los servidores junto a él. Haga clic con el botón derecho en él y seleccione "configurar y Habilitar enrutamiento y acceso remoto".
3. En el asistente, seleccione "siguiente", marque el botón radial para "configuración personalizada" y, a continuación, seleccione "siguiente".
4. Compruebe "NAT" y "enrutamiento de LAN" y, a continuación, seleccione "siguiente" y "finalizar". Si le pide que inicie el servicio, hágalo.
5. Ahora, vaya al nodo "IPv4" y expándalo para que el nodo "NAT" esté disponible.
6. Haga clic con el botón derecho en "NAT" y seleccione "nueva interfaz..." y seleccione "Ethernet", que debe ser su primera NIC con la dirección IP de "10.0.0.4".
7. Ahora es necesario crear algunas rutas estáticas para forzar el tráfico LAN fuera de la segunda NIC. Para ello, vaya al nodo "rutas estáticas" en "IPv4".
8. Una vez allí, crearemos las siguientes rutas.
    * Ruta 1
        * Interfaz: Ethernet
        * Destino: 10.0.0.0
        * Máscara de red: 255.255.255.0
        * Puerta de enlace: 10.0.0.1
        * Métrica: 256
        * Nota: aquí se incluye para permitir que la NIC principal responda al tráfico destinado a ella fuera de su propia interfaz. Si no lo hemos hecho aquí, la siguiente ruta podría provocar que el tráfico destinado a la NIC 1 salga de la NIC 2. Esto crearía una ruta asimétrica. 10.0.0.1 es la dirección IP que Azure asigna a la subred NAT. Azure usa la primera dirección IP disponible en un intervalo como la puerta de enlace predeterminada. Por lo tanto, si fuera a usar 192.168.0.0/24 para la subred NAT, la puerta de enlace sería 192.168.0.1. En el enrutamiento de la ruta más específica gana, lo que significa que esta ruta sustituirá a la ruta siguiente.

    * Ruta 2
        * Interfaz: Ethernet 2
        * Destino: 10.0.0.0
        * Máscara de red: 255.255.252.0
        * Puerta de enlace: 10.0.1.1
        * Métrica: 256
        * Nota: se trata de una ruta de detección de todo el tráfico destinado a la red virtual de Azure. Forzará el tráfico fuera de la segunda NIC. Tendrá que agregar rutas adicionales para otros intervalos a los que desee que tengan acceso las máquinas virtuales anidadas. Por tanto, si la red local es 172.16.0.0/22, querrá tener otra ruta para enviar ese tráfico fuera de la segunda NIC de nuestro hipervisor.

## <a name="creating-a-route-table-within-azure"></a>Creación de una tabla de rutas en Azure

Consulte [este artículo](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-portal) para obtener más información detallada sobre la creación y la administración de rutas en Azure.

1. Vaya a https://portal.azure.com.
2. En la esquina superior izquierda, seleccione "crear un recurso".
3. En el campo de búsqueda, escriba "tabla de rutas" y presione Entrar.
4. El resultado superior será tabla de rutas, seleccione esta y, a continuación, seleccione "crear".
5. Asigne un nombre a la tabla de rutas, en el caso de que se le denomine "Routes-for-Nested-VM".
6. Asegúrese de seleccionar la misma suscripción en la que residen los hosts de Hyper-V.
7. Cree un nuevo grupo de recursos o seleccione uno existente y asegúrese de que la región en la que se crea la tabla de rutas es la misma región en la que reside el host de Hyper-V.
8. Seleccione "Crear".

## <a name="configuring-the-route-table"></a>Configuración de la tabla de rutas

1. Vaya a la tabla de rutas que acabamos de crear. Para ello, busque el nombre de la tabla de rutas en la barra de búsqueda de la parte superior central del portal.
2. Después de seleccionar la tabla de rutas, vaya a "rutas" dentro de la hoja.
3. Seleccione "Agregar".
4. Asigne un nombre a la ruta, me he "anidado-máquinas virtuales".
5. Para la entrada de prefijo de dirección, el intervalo de direcciones IP de nuestra subred "flotante". En este caso, sería 10.0.2.0/24.
6. En "tipo de próximo salto", seleccione "aplicación virtual" y, a continuación, escriba la dirección IP de la segunda NIC de hosts de Hyper-V, que sería 10.0.1.4 y, a continuación, seleccione "Aceptar".
7. Ahora, en la hoja, seleccione "subredes", que se encuentra justo debajo de "rutas".
8. Seleccione "asociar" y, a continuación, seleccione la red virtual "Nesta-Fun" y luego seleccione la subred "Azure-VMs" y, a continuación, seleccione "Aceptar".
9. Realice el mismo proceso para la subred en la que reside el host de Hyper-V, así como para las demás subredes que necesiten tener acceso a las máquinas virtuales anidadas. Si está conectado 

# <a name="end-state-configuration-reference"></a>Referencia de configuración de estado final
El entorno de esta guía tiene las siguientes configuraciones. Esta sección es inteded que se va a usar como referencia.

1. Información de Virtual Network de Azure.
    * Configuración de alto nivel de VNet.
        * Nombre: diversión anidada
        * Espacio de direcciones: 10.0.0.0/22
        * Nota: esto se compone de cuatro subredes. Además, estos intervalos no se establecen en piedra. No dude en abordar su entorno sin embargo, si lo desea. 

    * Configuración de alto nivel de la primera subred.
        * Nombre: NAT
        * Espacio de direcciones: 10.0.0.0/24
        * Nota: aquí es donde reside la NIC principal de hosts de Hyper-V. Se usará para administrar la NAT de salida para las máquinas virtuales anidadas. Será la puerta de enlace a Internet para las máquinas virtuales anidadas.

    * Configuración de nivel superior de la segunda subred.
        * Nombre: Hyper-V-LAN
        * Espacio de direcciones: 10.0.1.0/24
        * Nota: nuestro host de Hyper-V tendrá una segunda NIC que se usará para controlar el enrutamiento entre las máquinas virtuales anidadas y los recursos que no son de Internet externos al host de Hyper-V.

    * Configuración de alto nivel de la tercera subred.
        * Nombre: fantasma
        * Espacio de direcciones: 10.0.2.0/24
        * Nota: se trata de una subred "flotante". El espacio de direcciones lo consumen nuestras máquinas virtuales anidadas y existe para controlar los anuncios de ruta de vuelta al entorno local. En realidad, no se implementará ninguna máquina virtual en esta subred.

    * Configuración de la cuarta subred de nivel superior.
        * Nombre: Azure-VM
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
        * Puerta de enlace predeterminada: vacía
        * DNS: configurado para DHCP
        * Reenvío IP habilitado: sí

    * NIC creada por Hyper-V para el conmutador virtual interno
        * Dirección IP: 10.0.2.1
        * Máscara de subred: 255.255.255.0
        * Puerta de enlace predeterminada: vacía

3. Nuestra tabla de rutas tendrá una sola regla.
    * Regla 1
        * Nombre: máquinas virtuales anidadas
        * Destino: 10.0.2.0/24
        * Próximo salto: aplicación virtual-10.0.1.4

## <a name="conclusion"></a>Conclusión

Ahora debería poder implementar una máquina virtual (incluso una VM de 32 bits) en el host de Hyper-V y hacer que sea accesible desde el entorno local y dentro de Azure.
