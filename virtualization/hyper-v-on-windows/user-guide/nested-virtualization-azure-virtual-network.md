# <a name="configuring-nested-vms-to-communicate-directly-with-resources-in-an-azure-virtual-network"></a>Configurar máquinas virtuales anidadas para comunicarse directamente con los recursos de una red Virtual de Azure
Las instrucciones original en implementar y configurar las máquinas virtuales anidadas dentro de Azure se necesitan tener acceso a estas máquinas virtuales a través de un conmutador NAT. Esto presenta varias limitaciones:

1. Máquinas virtuales anidadas no pueden acceder a los recursos locales o dentro de una red Virtual de Azure.
2. Recursos locales o recursos de Azure pueden acceder únicamente a las máquinas virtuales anidadas a través de una red NAT, lo que significa que los invitados varios no comparten el mismo puerto.

Este documento le guiará a través de una implementación mediante el cual se usarán RRAS, algunas rutas definidas de usuario y un espacio de direcciones "flotante" para permitir máquinas virtuales anidadas se comportan y se comunique como cualquier otra máquina virtual implementar directamente en un VNet dentro de Azure. 

Antes de comenzar a esta guía, consulta:
1. Leer la [guía se proporciona aquí](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/nested-virtualization) en la virtualización anidada, crear las máquinas virtuales capaces anidamiento e instalar el rol de Hyper-V dentro de las máquinas virtuales. No continúes más allá de configurar el rol de Hyper-V.
2. Leer todo el artículo antes de la implementación.

Esta guía hace que las siguientes suposiciones sobre el entorno de destino:
1. Estamos trabajando en una [topología radial](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke), con nuestro hub está conectado a una ExpressRoute.
2. Nuestra red radial se le asigna el espacio de direcciones de 10.0.0.0/23, que se extrae en dos /24 subredes.
  * 10.0.0.0/24: subred donde se encuentra nuestro host de Hyper-V.
  * 10.0.1.0/24: se trata de una subred "flotante". Este espacio de direcciones se consumirse en nuestro máquinas virtuales anidadas y existe para controlar los anuncios de ruta a local.
  * Los radios VNet inteligentemente se denomina "Radial".
3. Nuestro intervalo IP de redes de concentrador no es relevante, pero sabes que su nombre es "Hub".
4. Nuestra Hyper-V se asigna la dirección de 10.0.0.4/24.
5. Tenemos un servidor DNS en 10.0.0.10/24, esto no es un requisito, pero una suposición de nuestro tutorial. 
 
## <a name="high-level-overview-of-what-were-doing-and-why"></a>Introducción de nivel alto de lo que estamos haciendo y por qué

* En segundo plano: Anidada máquinas virtuales no reciben DHCP desde el VNet que su host está conectado a, incluso si estableces una interna o un conmutador externo. 
  * Esto significa que el host de Hyper-V debe proporcionar DHCP.
* Se asignará un bloque de direcciones IP para su uso JUST por el host de Hyper-V.  El host de Hyper-V no es consciente de la concesiones asignadas actualmente en el VNet, por lo que para evitar una situación en la que el host asigna una dirección IP ya existentes debemos asignar un bloque de direcciones IP para su uso solo por el host de Hyper-V. Esto nos permitirá evitar una situación IP duplicada. 
  * El bloque de direcciones IP elegimos se corresponde a una subred dentro de la VNet que el host de Hyper-V se encuentra en.
  * El motivo por el que queremos que corresponden a una subred existente es controlar anuncios de BGP vuelve a través de la ExpressRoute. Si se compone solo un intervalo IP para el host de Hyper-V usar, a continuación, tenemos que crear una serie de rutas estáticas para permitir que los clientes de forma local para comunicarse con las máquinas virtuales anidadas. Esto significa que no es un requisito de disco duro que podría constituyen un intervalo de IP para las máquinas virtuales anidadas y, a continuación, crear todas las rutas necesarias para dirigir a los clientes en el host de Hyper-V para ese intervalo.
* Vamos a crear un conmutador interno dentro de Hyper-V y, a continuación, se asignará la interfaz recién creada una dirección IP dentro de un intervalo que se apartan de DHCP. Esta dirección IP se convertirá en la puerta de enlace predeterminada para nuestras las máquinas virtuales anidadas y se pueden usar para la ruta entre el conmutador interno y la NIC del host que está conectado a nuestro VNet.
* Se instalará el rol de enrutamiento y acceso remoto en el host, que se activará nuestro host en un enrutador.  Esto es necesario para permitir la comunicación entre los recursos externos para el host y nuestras máquinas virtuales anidadas.
* Realizaremos lo siguiente para indicar a otros recursos cómo tener acceso a estas máquinas virtuales anidadas. Esto se necesitan que creamos una tabla de ruta definida por el usuario que contiene una ruta estática para el intervalo de IP que se encuentran en las máquinas virtuales anidadas. Esta ruta estática apuntará a la dirección IP de Hyper-V.
* A continuación, se coloca este UDR en la subred de puerta de enlace para que los clientes que procede local sepan cómo llegar a nuestro máquinas virtuales anidadas.
* También se coloca este UDR en cualquier otra subred dentro de Azure, que requiere conectividad a las máquinas virtuales anidadas.
* Para varios hosts de Hyper-V podría crear subredes "flotantes" adicionales y agregar una ruta estática adicional a la UDR.
* Cuando se retira un host de Hyper-V se se eliminar o reutilizar nuestro subred "flotante" y quitar esa ruta estática de nuestro UDR o si este es el último host de Hyper-V, quitar por completo el UDR.
 
## <a name="creating-our-virtual-switch"></a>Creación de nuestra conmutador Virtual
1. Abre PowerShell en modo administrativo.
2. Crear un conmutador interno: `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. Asignar una dirección IP de la interfaz recién creada: `New-NetIPAddress –IPAddress 10.0.1.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`
 
## <a name="install-and-configure-dhcp"></a>Instalar y configurar DHCP
*Muchas personas pase por alto este componente cuando este está tratando primero para que funcione la virtualización anidada. A diferencia de en local donde la VM de invitado recibirán DHCP desde la red que se encuentra el host, máquinas virtuales anidadas en Azure deben proporcionarse DHCP mediante el host que se ejecuten en. O bien que asignar una dirección IP estática para cada máquina virtual anidada, que no es escalable.*

1. Instalar el rol DHCP: `Install-WindowsFeature DHCP -IncludeManagementTools`
2. Crear el ámbito DHCP: `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.1.2 -EndRange 10.0.1.254 -SubnetMask 255.255.255.0`
3. Configurar las opciones de DNS y puerta de enlace predeterminada para el ámbito: `Set-DhcpServerV4OptionValue -DnsServer 10.0.0.10 -Router 10.0.1.1`
    * Asegúrate de un servidor DNS válido de entrada. En este caso producirse dispone de un servidor en la red 10.0.0.0/24 que ofrecen de DNS de Windows.
 
## <a name="installing-remote-access"></a>Instalación de acceso remoto
* Abre el administrador del servidor y selecciona "Agregar roles y características".
* Selecciona "Siguiente" hasta que llegues a "Roles de servidor".
* Compruebe "Acceso remoto" y haz clic en "Siguiente" hasta que llegues a servicios de rol de"".
* Comprobar "Enrutamiento", selecciona "Agregar características" y, a continuación, selecciona "Siguiente" y, a continuación, "Install". Completa al asistente y esperar completar la instalación.
 
## <a name="configuring-remote-access"></a>Configurar el acceso remoto
* Abre el administrador del servidor y selecciona "Herramientas" y, a continuación, selecciona "Enrutamiento y acceso remoto".
* En el lado derecho del panel de administración de enrutamiento y acceso remoto se vea un icono con el nombre de los servidores junto a ella, haz clic en este y selecciona "Configurar y habilitar enrutamiento y acceso remoto".
* En el Asistente para seleccionar "Siguiente", comprueba el botón radial para "Conexión segura entre dos redes privadas" y, a continuación, selecciona "Siguiente".
* Selecciona el botón radial para "No" cuando se le pregunte si quieres usar conexiones de marcado a petición y, a continuación, selecciona "Siguiente" y, a continuación, selecciona "Finalizar".
 
## <a name="creating-a-route-table-within-azure"></a>Creación de una tabla de ruta en Azure
Consulta [este artículo](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal) para obtener un más sobre cómo crear y administrar las rutas dentro de Azure de lectura de profundidad. 
* Ve a https://portal.azure.com.
* En la esquina superior izquierda selecciona "Crear un recurso".
* En el campo de búsqueda, escribe "Tabla de enrutamiento" y escribe el posicionamiento.
* El resultado superior se ser tabla de ruta, selecciona esto y, a continuación, selecciona "Crear"
* Nombre de la tabla de ruta, en mi caso llamado "Rutas-de-anidado-máquinas virtuales".
* Asegúrate de que seleccionar la mismo suscripción que los hosts de Hyper-V residen en.
* Crear un nuevo grupo de recursos o seleccione uno existente y asegúrate de que la región de en que crear la tabla de ruta es la misma región que el host de Hyper-V se encuentra en.
* Selecciona "Crear".
 
## <a name="configuring-the-route-table"></a>Configuración de la tabla de ruta
* Ve a la tabla de enrutamiento que acabamos de crear. Puedes hacerlo si buscas el nombre de la tabla de ruta de la barra de búsqueda en la parte superior central del portal.
* Una vez que hayas seleccionado la tabla de enrutamiento vaya a "Rutas" desde dentro de la hoja.
* Selecciona "Agregar".
* Asigne un nombre a la ruta, ha funcionado con "Máquinas virtuales de anidadas".
* Dirección de prefijo entrada el intervalo de IP para nuestra subred "flotante". En este caso sería 10.0.1.0/24.
* Para "Tipo siguiente salto" selecciona "Dispositivo Virtual" y, a continuación, escribe la dirección IP para el host de Hyper-V, que sería 10.0.0.4 y, a continuación, selecciona "Aceptar".
* Ahora desde dentro de la selección de la hoja "Subredes", será justo debajo de "Rutas".
* Selecciona "Asociar", a continuación, selecciona nuestra red Virtual de "Hub" y, a continuación, selecciona el "GatewaySubnet" y, a continuación, selecciona "Aceptar".
* Hacer este mismo proceso para la subred que nuestro host de Hyper-V se encuentra en, así que para cualquier otras subredes que necesitan acceder a las máquinas virtuales anidadas.
 
## <a name="conclusion"></a>Conclusión
Ahora podrás implementar una máquina virtual (y posiblemente incluso una VM de 32 bits!) del host de Hyper-V y hacer que sea accesible desde locales y en Azure.
