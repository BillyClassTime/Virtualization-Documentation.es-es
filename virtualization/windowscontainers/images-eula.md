# <a name="microsoft-software-supplemental-license-for-windows-container-base-image"></a>SOFTWARE DE MICROSOFT CON LICENCIA COMPLEMENTARIA PARA CONTENEDOR DE WINDOWS

Esta licencia complementaria es para la imagen base del contenedor de Windows ("imagen del contenedor"). Si cumple con las condiciones de esta licencia complementaria, puede usar la imagen de contenedor tal como se describe a continuación.

## <a name="container-os-image"></a>IMAGEN SE SISTEMA OPERATIVO DE CONTENEDOR
La imagen del contenedor solo se puede usar con una copia con licencia válida de:

Software Windows Server Standard o Windows Server Datacenter (colectivamente "software de host de servidor") o el sistema operativo Microsoft Windows (versión 10) ("software host de cliente") o Windows 10 IoT Enterprise y Windows 10 IoT Core (colectivamente "IoT Software de host ").
El software de host de servidor, el software de host de cliente y el software de host de IoT son denominados colectivamente como "software de host" y una licencia para software de host es una "licencia de host".

No puede usar la imagen del contenedor si no tiene la versión y la edición correspondientes de la licencia de host. Se pueden aplicar determinadas restricciones y condiciones adicionales, que se describen en el presente documento. Si los términos de licencia aquí entran en conflicto con la licencia de hospedaje, esta licencia complementaria regirá por la imagen del contenedor. AL ACEPTAR ESTA LICENCIA COMPLEMENTARIA O USAR LA IMAGEN DE CONTENEDOR, ACEPTAS TODAS ESTAS CONDICIONES. SI NO ACEPTA Y CUMPLE CON ESTAS CONDICIONES, NO PODRÁ USAR LA IMAGEN DEL CONTENEDOR.

## <a name="definitions"></a>LAS
**Windows Server Container** (sin el aislamiento de Hyper-V) es una característica del software de servidor de Microsoft Windows.

**Contenedor de Windows Server con aislamiento de Hyper-V.** La sección 2 (k) de los términos de licencia de Microsoft Windows Server se elimina por completo y se reemplaza por las condiciones revisadas, tal como se muestra en la "actualización" a continuación.

Última actualización: el contenedor de Windows Server con aislamiento de Hyper-V (anteriormente denominado contenedor de Hyper-V) es una tecnología de contenedor de Windows Server que usa un entorno de sistema operativo virtual para hospedar uno o varios contenedores de Windows Server. Cada instancia de aislamiento de Hyper-V que se usa para hospedar uno o varios contenedores de Windows Server se considera un entorno de sistema operativo virtual.

## <a name="license-terms"></a>TÉRMINOS DE LICENCIA
**Licencia de host.** Los términos de licencia de hospedaje se aplican a su uso de la imagen del contenedor y a cualquier contenedor de Windows creado con la imagen del contenedor, que son distintos y se separan de una máquina virtual.

**Derechos de uso.** La imagen de contenedor puede usarse para crear un entorno de sistema operativo Windows virtualizado aislado que incluya al menos una aplicación que agregue funcionalidades principales y significativas. Puede usar la imagen del contenedor solo para crear, compilar y ejecutar contenedores de Windows en el software del host. Las actualizaciones del software de host no pueden actualizar la imagen de contenedor para que pueda volver a crear cualquier contenedor de Windows basado en una imagen de contenedor actualizada.

**Puesta.** No puede quitar este archivo de documento de licencia complementaria de la imagen del contenedor. No se puede habilitar el acceso remoto a las aplicaciones que se ejecutan en su contenedor para evitar las cuotas de licencia aplicables. No puede utilizar técnicas de ingeniería inversa, descompilar o desensamblar la imagen del contenedor, o intentar hacerlo, excepto y únicamente en la medida en que lo requieran los términos de licencia de terceros que rigen el uso de ciertos componentes de código abierto que se pueden incluir con el software. Es posible que se apliquen restricciones adicionales en la licencia de host.

## <a name="additional-terms"></a>CONDICIONES ADICIONALES
**Software de host de cliente.** Al ejecutar una imagen de contenedor en el software de host de cliente, puede ejecutar cualquier número de la imagen de contenedor creada como contenedores de Windows con fines de prueba o de desarrollo. No puede usar estos contenedores de Windows en un entorno de producción en software de host de cliente.

**Software de hospedaje de IoT.** Al ejecutar una imagen de contenedor en el software de host de IoT, puede ejecutar cualquier número de la imagen de contenedor creada como contenedores de Windows con fines de prueba o de desarrollo. Solo puede usar la imagen del contenedor en un entorno de producción si ha aceptado las condiciones de uso comercial de Microsoft para las imágenes en tiempo de ejecución de Windows 10 Core o la licencia de dispositivo de Windows 10 IoT Enterprise ("contrato comercial de Windows IoT"). Las cláusulas y restricciones adicionales de los contratos de Windows IoT Commercial rigen para el uso de la imagen del contenedor en un entorno de producción.

**Software de terceros.** La imagen del contenedor puede incluir aplicaciones de terceros a las que se le concede licencia en esta licencia complementaria o según sus condiciones. Los términos de licencia, los avisos y las confirmaciones, si las hay, de las aplicaciones de terceros pueden ser http://aka.ms/thirdpartynotices accesibles en línea en o en el archivo de avisos que lo acompañan. Incluso si dichas aplicaciones se rigen por otros acuerdos, la renuncia, las limitaciones y exclusiones de daños en la licencia de hospedaje se aplicarán en la medida en que lo permitan las leyes vigentes.

**Componentes de código abierto.** La imagen del contenedor puede contener un software protegido por terceros con licencia de licencias de origen abierto con obligaciones de disponibilidad de códigos de origen. Las copias de esas licencias se incluyen en el archivo ThirdPartyNotices o en otros avisos de acompañamiento. Puede obtener el código fuente completo correspondiente de Microsoft si y según sea necesario bajo la licencia de origen abierta pertinente mediante el envío de un giro postal o un cheque por $5,00: equipo de cumplimiento de código fuente, Microsoft Corporation, 1 Microsoft Way, Redmond, WA 98052, Estados Unidos. Incluya el nombre "Microsoft Software Supplemental License para la base del contenedor de Windows" Image, el nombre del componente de código abierto y el número de versión en la línea de memo de su pago. También puede encontrar una copia del origen en http://aka.ms/getsource.
