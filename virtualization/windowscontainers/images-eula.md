# <a name="microsoft-software-supplemental-license-for-windows-container-base-image"></a>IMAGEN DEL CONTENEDOR DE LA LICENCIA COMPLEMENTARIA DE SOFTWARE DE MICROSOFT PARA WINDOWS

Esta licencia complementaria es para la imagen base del contenedor de Windows ("imagen de contenedor"). Si cumple con los términos de esta licencia complementaria, puede usar la imagen de contenedor como se describe a continuación.

## <a name="container-os-image"></a>IMAGEN SE SISTEMA OPERATIVO DE CONTENEDOR
La imagen de contenedor solo se puede usar con una copia con licencia válida de:

Software de Windows Server Standard o Windows Server Datacenter (colectivamente "software de host de servidor") o el software del sistema operativo Microsoft Windows (versión 10) ("software de host de cliente") o Windows 10 IoT Enterprise y Windows 10 IoT Core (colectivamente, IoT) Software de host ").
El software de host de servidor, el software de host de cliente y el software de host de IoT se conocen colectivamente como "software de host" y una licencia de software de host es una "licencia de host".

No puede usar la imagen de contenedor si no tiene una versión y edición correspondientes de la licencia de host. Se pueden aplicar ciertas restricciones y términos adicionales, que se describen en este artículo. Si los términos de licencia aquí están en conflicto con la licencia de host, esta licencia complementaria se regirá con respecto a la imagen de contenedor. AL ACEPTAR ESTA LICENCIA COMPLEMENTARIA O USAR LA IMAGEN DE CONTENEDOR, ACEPTA TODOS ESTOS TÉRMINOS. SI NO ACEPTA Y CUMPLE ESTOS TÉRMINOS, NO PUEDE USAR LA IMAGEN DE CONTENEDOR.

## <a name="definitions"></a>DEFINICIONES
El **contenedor de Windows Server** (sin aislamiento de Hyper-V) es una característica del software de Microsoft Windows Server.

**Contenedor de Windows Server con aislamiento de Hyper-V.** La sección 2 (k) de los términos de licencia de Microsoft Windows Server se elimina por completo y se reemplaza por los términos revisados, tal como se muestra en "actualizado" a continuación.

ACTUALIZADO: el contenedor de Windows Server con el aislamiento de Hyper-V (anteriormente conocido como contenedor de Hyper-V) es una tecnología de contenedor de Windows Server que emplea un entorno de sistema operativo virtual para hospedar uno o varios contenedores de Windows Server. Cada instancia de aislamiento de Hyper-V que se usa para hospedar uno o varios contenedores de Windows Server se considera un entorno de sistema operativo virtual.

## <a name="license-terms"></a>TÉRMINOS DE LICENCIA
**Licencia de host.** Los términos de la licencia de host se aplican al uso de la imagen de contenedor y a los contenedores de Windows creados con la imagen de contenedor, que son distintos y separados de una máquina virtual.

**Derechos de uso.** La imagen de contenedor se puede usar para crear un entorno de sistema operativo Windows virtualizado aislado que incluye al menos una aplicación que agrega funcionalidad principal y importante. Puede usar la imagen de contenedor solo para crear, compilar y ejecutar contenedores de Windows en software de host. Es posible que las actualizaciones del software del host no actualicen la imagen del contenedor, por lo que puede volver a crear los contenedores de Windows en función de una imagen de contenedor actualizada.

**Conocer.** No puede quitar este archivo de documento de licencia adicional de la imagen de contenedor. No se puede habilitar el acceso remoto a las aplicaciones que se ejecutan en el contenedor para evitar las cuotas de licencia aplicables. No se puede aplicar ingeniería inversa, descompilar o desensamblar la imagen de contenedor, o intentar hacerlo, excepto y únicamente en la medida requerida por los términos de licencia de terceros que rigen el uso de determinados componentes de código abierto que se pueden incluir con el software. Es posible que se apliquen restricciones adicionales en la licencia del host.

## <a name="additional-terms"></a>TÉRMINOS ADICIONALES
**Software del host del cliente.** Al ejecutar una imagen de contenedor en el software de host de cliente, puede ejecutar cualquier número de la imagen de contenedor con instancias creadas como contenedores de Windows solo con fines de prueba o desarrollo. No puede usar estos contenedores de Windows en un entorno de producción en el software de host del cliente.

**Software de host de IoT.** Al ejecutar una imagen de contenedor en el software de host de IoT, puede ejecutar cualquier número de la imagen de contenedor con instancias creadas como contenedores de Windows solo con fines de prueba o desarrollo. Solo puede usar la imagen de contenedor en un entorno de producción si ha aceptado los términos de uso comerciales de Microsoft para imágenes de tiempo de ejecución de Windows 10 Core o la licencia de dispositivo de Windows 10 IoT Enterprise ("contrato comercial de Windows IoT"). Se aplican términos y restricciones adicionales en los acuerdos comerciales de IoT Windows al uso de la imagen de contenedor en un entorno de producción.

**Third Party Software.** La imagen de contenedor puede incluir aplicaciones de terceros que se le conceden bajo esta licencia complementaria o bajo sus propios términos. Es posible que se pueda tener acceso a los términos de licencia, los avisos y las confirmaciones de las aplicaciones de terceros en http://aka.ms/thirdpartynotices o en un archivo de avisos adjunto. Aunque dichas aplicaciones se rijan por otros contratos, la exención de responsabilidad, las limitaciones y las exclusiones de daños en la licencia de host también se aplican en la medida que lo permita la ley aplicable.

**Componentes de código abierto.** La imagen de contenedor puede contener software de terceros protegidos con licencia bajo licencias de código abierto con obligaciones de disponibilidad del código fuente. Las copias de esas licencias se incluyen en el archivo ThirdPartyNotices u otro archivo de avisos que lo acompaña. Puede obtener el código fuente completo correspondiente de Microsoft si y, según sea necesario, en la licencia de código abierto correspondiente mediante el envío de un pedido monetario o una comprobación de $5,00 a: equipo de cumplimiento del código fuente, Microsoft Corporation, 1 Microsoft Way, Redmond, WA 98052, EE. UU. Incluya el nombre "licencia complementaria de software de Microsoft para la imagen base del contenedor de Windows", el nombre del componente de código abierto y el número de versión en la línea de memorando del pago. También puede encontrar una copia del origen en http://aka.ms/getsource.
