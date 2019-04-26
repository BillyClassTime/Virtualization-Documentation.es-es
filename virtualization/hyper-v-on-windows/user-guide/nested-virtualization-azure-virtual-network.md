---
title: Configurar máquinas virtuales anidadas para comunicarse directamente con los recursos de una red Virtual de Azure
description: Virtualización anidada
keywords: Windows 10, hyper-v, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: 18ab4d1d87c22f70fe09aae5222a7d125ac9c974
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578676"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Configurar máquinas virtuales anidadas para comunicarse con los recursos de una red virtual de Azure

Las instrucciones original en implementar y configurar las máquinas virtuales anidadas dentro de Azure se necesitan tener acceso a estas máquinas virtuales a través de un conmutador NAT. Esto presenta varias limitaciones:

1. Máquinas virtuales anidadas no pueden acceder a los recursos locales o dentro de una red Virtual de Azure.
2. Recursos locales o recursos de Azure pueden acceder únicamente a las máquinas virtuales anidadas a través de una red NAT, lo que significa que los invitados varios no comparten el mismo puerto.

Este documento le guiará a través de una implementación mediante el cual se usarán RRAS, usuario define las rutas, una subred dedicada a NAT saliente para permitir el acceso de invitado internet y un espacio de direcciones "flotante" para permitir máquinas virtuales anidadas se comportan y se comunique como cualquier otra máquina virtual implementar directamente en un VNet dentro de Azure.

Antes de comenzar a esta guía, consulta:

1. Leer la [guía se proporciona aquí](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/nested-virtualization) en la virtualización anidada.
2. Leer todo el artículo antes de la implementación.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>Introducción de nivel alto de lo que hacemos y por qué
* Vamos a crear una máquina virtual con anidamiento que tiene dos NIC. 
* Una que NIC se usará para proporcionar nuestro máquinas virtuales anidadas con acceso a internet a través de NAT y la otra NIC se usará para enrutar el tráfico desde nuestro conmutador interno a recursos externos para que el hipervisor. Cada NIC tendrá que estar en un dominio de enrutamiento diferente, lo que significa una subred diferente.
* Esto significa que necesitaremos una red Virtual con en un mínimo de tres subredes. Una para NAT, uno para el enrutamiento de LAN y uno que no se usa, pero es "reservado" para nuestras las máquinas virtuales anidadas. Los nombres se usan para las subredes en este documento son, "NAT", "Hyper-V-LAN" y "Ghosted".
* El tamaño de estas subredes es tu criterio, pero hay algunas consideraciones. El tamaño de las subredes "Ghosted" determina el número de direcciones IP que tienes para las máquinas virtuales anidadas. Además, el tamaño de las subredes "NAT" y "Hyper-V-LAN" determina el número de direcciones IP que tienes para hipervisores. Por lo tanto, puede hacer que técnicamente subredes realmente pequeñas aquí si solo se han pensando en tener una o dos hipervisores.
* En segundo plano: Anidada máquinas virtuales no reciben DHCP desde el VNet que su host está conectado a incluso si estableces una interna o un conmutador externo. 
  * Esto significa que el host de Hyper-V debe proporcionar DHCP.
* El host de Hyper-V no es consciente de las concesiones asignadas actualmente en el VNet, por lo que para evitar una situación en la que el host asigna una dirección IP ya existentes debemos asignar un bloque de direcciones IP para su uso solo por el host de Hyper-V. Esto nos permitirá evitar una situación IP duplicada.
  * Corresponde a una subred dentro de la misma VNet que tu Hyper-V se encuentra en el bloque de direcciones IP elegimos.
  * Es el motivo por el que queremos que corresponden a una subred existente controlar los anuncios de BGP volver a través de un ExpressRoute. Si se compone solo un intervalo IP para el host de Hyper-V usar, a continuación, tenemos que crear una serie de rutas estáticas para permitir que los clientes de forma local para comunicarse con las máquinas virtuales anidadas. Esto significa que no es un requisito de disco duro que podría constituyen un intervalo de IP para las máquinas virtuales anidadas y, a continuación, crear todas las rutas necesarias para dirigir a los clientes al host de Hyper-V para ese intervalo.
* Vamos a crear un conmutador interno dentro de Hyper-V y, a continuación, se asignará la interfaz recién creada una dirección IP dentro de un intervalo que se apartan de DHCP. Esta dirección IP se convertirá en la puerta de enlace predeterminada para nuestras las máquinas virtuales anidadas y se pueden usadas para la ruta entre el conmutador interno y la NIC del host que está conectado a nuestro VNet.
* Se instalará el rol de enrutamiento y acceso remoto en el host, que se activará la host en un enrutador.  Esto es necesario para permitir la comunicación entre los recursos externos en el host y nuestras máquinas virtuales anidadas.
* Informaremos otros recursos cómo tener acceso a estas máquinas virtuales anidadas. Esto se necesitan que creamos una tabla de rutas definida por el usuario que contiene una ruta estática para el intervalo de IP que se encuentran en las máquinas virtuales anidadas. Esta ruta estática apuntará a la dirección IP de Hyper-V.
* A continuación, coloque este UDR en la subred de puerta de enlace para que procede local de los clientes sepan cómo llegar a nuestro máquinas virtuales anidadas.
* También se coloca este UDR en cualquier otra subred dentro de Azure, que requiere conectividad a las máquinas virtuales anidadas.
* Para varios hosts de Hyper-V podría crear subredes "flotantes" adicionales y agregar una ruta estática adicional a la UDR.
* Cuando se retira un host de Hyper-V se se delete/reutilizar nuestro subred "flotante" y quitar esa ruta estática de nuestro UDR o si este es el último host de Hyper-V, quitar por completo el UDR.

## <a name="creating-the-host"></a>Crear la host

Se por alto la los valores de configuración que estén hasta preferencias personales, como el nombre de máquina virtual, grupo de recursos, etcetera..

1. Ve a portal.azure.com
2. Haz clic en "Crear un recurso" en la parte superior izquierda
3. Selecciona "Ventana de la máquina virtual del servidor 2016" en la columna Popular
4. En la pestaña "Aspectos básicos" asegúrate de seleccionar un tamaño de máquina virtual que es capaz de virtualización anidada
5. Mover a la pestaña "Redes"
6. Crear una nueva red Virtual con la siguiente configuración
    * Espacio de direcciones de VNet: 10.0.0.0/22
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
        * Nombre: Las VM de Azure
        * Espacio de direcciones: 10.0.3.0/24
7. Asegúrate de que hayas seleccionado la subred NAT para la máquina virtual
8. Ve a "revisión + crear" y selecciona "Crear"

## <a name="create-the-second-network-interface"></a>Crear la segunda interfaz de red
1. Después de la máquina virtual ha terminado de examinar a ella en el Portal Azure de aprovisionamiento
2. Detener la máquina virtual
3. Una vez detenido ir a la "Red" en configuración
4. "Adjuntar la interfaz de red"
5. "Crear la interfaz de red"
6. Asigna un nombre (no importa lo asignarle el nombre, pero Asegúrate de recordar)
7. Selecciona "Hyper-V-LAN" de la subred
8. Asegúrate de que seleccionar el host reside en el mismo grupo de recursos
9. "Crear"
10. Esto te llevará de regreso a la pantalla anterior, asegúrate de seleccionar la interfaz de red recién creado y selecciona "Aceptar"
11. Vuelve al panel de "introducción a" y vuelve a iniciar la máquina virtual cuando se haya completado la acción anterior
12. Ve a la segunda tarjeta que acabamos de crear, puedes encontrarla en el grupo de recursos seleccionado anteriormente
13. Ve a "Configuraciones IP" y alternar "Reenvío de IP" en "Enabled" y, a continuación, guardar el cambio

## <a name="setting-up-hyper-v"></a>Configurar Hyper-V
1. Control remoto en el host
2. Abre un símbolo del sistema con privilegios elevados de PowerShell
3. Ejecuta el siguiente comando `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`
4. Esto reiniciará el host
5. Volver a conectar con el host para continuar con el resto de la configuración

## <a name="creating-our-virtual-switch"></a>Creación de nuestro conmutador virtual

1. Abre PowerShell en modo administrativo.
2. Crear un conmutador interno: `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. Asignar una dirección IP de la interfaz recién creada: `New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>Instalar y configurar DHCP

*Muchas personas piensan este componente cuando este está tratando primero para que funcione la virtualización anidada. A diferencia de en local donde la VM de invitado recibirán DHCP desde la red que se encuentra el host, máquinas virtuales anidadas en Azure deben proporcionarse DHCP a través de la host que se ejecuten en. O necesitas asignar una dirección IP estática para cada máquina virtual anidada, que no es escalable.*

1. Instalar el rol DHCP: `Install-WindowsFeature DHCP -IncludeManagementTools`
2. Crear el ámbito DHCP: `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`
3. Configurar las opciones de DNS y puerta de enlace predeterminada para el ámbito: `Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`
    * Asegúrate de entrada de un servidor DNS válido si quieres que la resolución de nombres para que funcione. En este caso uso [recursiva DNS de Azure](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances).

## <a name="installing-remote-access"></a>Instalación de acceso remoto

1. Abre el administrador del servidor y selecciona "Agregar roles y características".
2. Selecciona "Siguiente" hasta que llegues a "Roles de servidor".
3. Compruebe "Acceso remoto" y haz clic en "Siguiente" hasta que llegues a servicios de rol de"".
4. Comprobar "Enrutamiento", selecciona "Agregar características" y, a continuación, selecciona "Siguiente" y, a continuación, "Install". Completa al asistente y espere a completar la instalación.

## <a name="configuring-remote-access"></a>Configurar el acceso remoto

1. Abre el administrador del servidor y selecciona "Herramientas" y, a continuación, selecciona "Enrutamiento y acceso remoto".
2. En el lado derecho del panel de administración de enrutamiento y acceso remoto se vea un icono con el nombre de los servidores junto a ella, haz clic en este y selecciona "Configurar y habilitar enrutamiento y acceso remoto".
3. En el Asistente para seleccionar "Siguiente", busque el botón radial "Configuración personalizada" y, a continuación, selecciona "Siguiente".
4. Comprobar "NAT" y "Enrutamiento de LAN" y, a continuación, selecciona "siguiente" y, a continuación, "fin". Si se solicita al iniciar el servicio, a continuación, hacerlo.
5. Ahora, navega hasta el nodo de "IPv4" y expandirla para que el nodo de "NAT" se pone a disposición.
6. Haga clic con el botón secundario del mouse en "NAT", selecciona "Nueva interfaz …" y selecciona "Ethernet", este debe ser la primera NIC con la dirección IP de "10.0.0.4"
7. Ahora, debemos crear rutas estáticas para forzar el tráfico de LAN un vistazo a la NIC segundo. Para ello, ve al nodo "Rutas estáticas" en "IPv4".
8. Una vez que haya vamos a crear las siguientes rutas.
    * Ruta de 1
        * Interfaz: Ethernet
        * Destino: 10.0.0.0
        * Máscara de red: 255.255.255.0
        * Puerta de enlace: 10.0.0.1
        * Métrica: 256
        * Nota: Ponemos esto aquí para permitir que la NIC principal responder al tráfico destinado a lo fuera de su propia interfaz. Si no tenemos esto aquí la siguiente ruta provocaría tráfico destinado a NIC 1 alejar la NIC 2. Esto crearía una ruta asimétrica. 10.0.0.1 es la dirección IP que Azure se asigna a la subred NAT. Azure utiliza la primera dirección IP disponible en un intervalo como la puerta de enlace predeterminada. Por lo tanto, si tuviera que han usado 192.168.0.0/24 para la subred NAT, la puerta de enlace sería 192.168.0.1. En la ruta más específica el enrutamiento se reemplazan wins, lo que significa que esta ruta la siguiente ruta.

    * Ruta 2
        * Interfaz: Ethernet 2
        * Destino: 10.0.0.0
        * Máscara de red: 255.255.252.0
        * Puerta de enlace: 10.0.1.1
        * Métrica: 256
        * Nota: Se trata de un problema en todas las rutas de tráfico destinado a nuestro VNet de Azure. Forzará el tráfico de un vistazo a la NIC segundo. Tendrás que agregar rutas adicionales para otros intervalos a que quieres que las máquinas virtuales anidadas para tener acceso. Por tanto, si estás red local es 172.16.0.0/22, a continuación, puede que desee tener otra ruta de acceso para enviar que el tráfico de un vistazo a la segunda NIC de nuestro hipervisor.

## <a name="creating-a-route-table-within-azure"></a>Creación de una tabla de ruta en Azure

Consulta [este artículo](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal) para obtener un más sobre cómo crear y administrar las rutas dentro de Azure de lectura de profundidad.

1. Ve a https://portal.azure.com.
2. En la esquina superior izquierda selecciona "Crear un recurso".
3. En el campo de búsqueda, escribe "Tabla de enrutamiento" y presione ENTRAR.
4. El resultado superior se ser tabla de enrutamiento, seleccione esta opción y, a continuación, selecciona "Crear"
5. Nombre de la tabla de enrutamiento, en mi caso llamado "Rutas-de-anidados-máquinas virtuales".
6. Asegúrate de que seleccionar la mismo suscripción que los hosts de Hyper-V residen en.
7. Crear un nuevo grupo de recursos o seleccione uno existente y se puede estar seguro que la misma región que el host de Hyper-V se encuentra en la región de en que crear la tabla de enrutamiento.
8. Selecciona "Crear".

## <a name="configuring-the-route-table"></a>Configuración de la tabla de enrutamiento

1. Ve a la tabla de enrutamiento que acabamos de crear. Puedes hacerlo si buscas el nombre de la tabla de enrutamiento de la barra de búsqueda en la parte superior central del portal.
2. Una vez que hayas seleccionado la tabla de enrutamiento vaya a "Rutas" desde dentro de la hoja.
3. Selecciona "Agregar".
4. Asigne un nombre a la ruta, ha funcionado con "Máquinas virtuales de anidadas".
5. Dirección de prefijo entrada el intervalo de IP para nuestra subred "flotante". En este caso sería 10.0.2.0/24.
6. Para "Siguiente salto tipo" selecciona "Dispositivo Virtual" y, a continuación, escribe la dirección IP de dirección para Hyper-V hospeda segunda tarjeta de red, lo que sería 10.0.1.4 y, a continuación, selecciona "Aceptar".
7. Ahora desde dentro de la selección de la hoja "Subredes", se trata de justo debajo de "Rutas".
8. Selecciona a "Asociar,", a continuación, selecciona nuestra VNet "Anidadas divertida" y, a continuación, selecciona la subred "VM de Azure" y, a continuación, selecciona "Aceptar".
9. Hacer este mismo proceso para la subred que el host de Hyper-V reside en, así como para cualquier otras subredes que necesitan acceder a las máquinas virtuales anidadas. Si conectado 

# <a name="end-state-configuration-reference"></a>Referencia de configuración de estado final
El entorno en esta guía tiene las configuraciones que aparecen. En esta sección es adecuada para usarse como una referencia.

1. Información de red Virtual de Azure.
    * Configuración de nivel alto VNet.
        * Nombre: Anidados-diversión
        * Espacio de direcciones: 10.0.0.0/22
        * Nota: Esto se se compone de cuatro subredes. Además, no se establecen estos intervalos definitivo. No dudes en el entorno de direcciones como quiera. 

    * Primera alto nivel configuración de subred.
        * Nombre: NAT
        * Espacio de direcciones: 10.0.0.0/24
        * Nota: Esto es donde nuestro Hyper-V hospeda NIC principal reside. Este se usará para controlar NAT saliente para las máquinas virtuales anidadas. Será la puerta de enlace a internet para las máquinas virtuales anidadas.

    * Subred alto nivel configuración del segundo.
        * Nombre: Hyper-V-LAN
        * Espacio de direcciones: 10.0.1.0/24
        * Nota: La host de Hyper-V tendrá una segunda NIC que se usará para controlar el enrutamiento entre los recursos de internet que no sea externos al host de Hyper-V y máquinas virtuales anidadas.

    * Tercera alto nivel configuración de subred.
        * Nombre: fantasma
        * Espacio de direcciones: 10.0.2.0/24
        * Nota: Se trata de una subred "flotante". El espacio de direcciones será usado por nuestro máquinas virtuales anidadas y existe para controlar los anuncios de rutas a local. No hay máquinas virtuales realmente se implementarán en la subred.

    * La cuarta subred de configuración nivel alto.
        * Nombre: Las VM de Azure
        * Espacio de direcciones: 10.0.3.0/24
        * Nota: Subred que contenga VM de Azure.

1. Nuestro host de Hyper-V tiene el debajo de configuraciones de la NIC.
    * NIC principal 
        * Dirección IP: 10.0.0.4
        * La máscara de subred: 255.255.255.0
        * Puerta de enlace predeterminada: 10.0.0.1
        * DNS: Configurado para DHCP
        * Habilitada el reenvío de IP: No

    * NIC secundario
        * Dirección IP: 10.0.1.4
        * La máscara de subred: 255.255.255.0
        * Puerta de enlace predeterminada: vacía
        * DNS: Configurado para DHCP
        * Habilitada el reenvío de IP: Sí

    * Hyper-V creada NIC para el conmutador Virtual
        * Dirección IP: 10.0.2.1
        * La máscara de subred: 255.255.255.0
        * Puerta de enlace predeterminada: vacía

3. La tabla de rutas tendrá una única regla.
    * Regla de 1
        * Nombre: Anidados-máquinas virtuales
        * Destino: 10.0.2.0/24
        * Siguiente salto: Dispositivo Virtual-10.0.1.4

## <a name="conclusion"></a>Conclusión

Ahora debe ser capaz de implementar una máquina virtual (incluso una VM de 32 bits!) del host de Hyper-V y hacer que sea accesible desde locales y en Azure.
