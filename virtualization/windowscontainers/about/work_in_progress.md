---
title: Trabajo en curso de contenedores de Windows
description: Trabajo en curso de contenedores de Windows
keywords: docker, containers
author: scooley
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5d9f1cb4-ffb3-433d-8096-b085113a9d7b
redirect_url: ../containers_welcome
translationtype: Human Translation
ms.sourcegitcommit: e3f5535594123f6b4f8931e41a91d92f3b837814
ms.openlocfilehash: 085bb8c0158aedf4270cf2423114ec1901af1ebd

---

# Trabajo en curso

Si no ve que su problema se trate aquí o tiene preguntas, publique lo que necesite en el [foro](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).

-----------------------

## Funcionalidad general

### Los números de compilación del contenedor y el host deben coincidir
Un contenedor de Windows requiere una imagen de sistema operativo que coincida con el host del contenedor a nivel de compilación y revisión. Una discrepancia dará lugar a una posible inestabilidad o un comportamiento impredecible del contenedor o el host.

Si instala las actualizaciones en el sistema operativo del host del contenedor de Windows, deberá actualizar la imagen del sistema operativo base del contenedor para obtener las actualizaciones que coincidan.
<!-- Can we give examples of behavior or errors?  Makes it more searchable -->

**Solución alternativa:**   
Descargue e instale una imagen base de contenedor que coincida con el nivel de revisión y la versión del sistema operativo del host del contenedor.

### Comportamiento del firewall predeterminado
En un entorno de contenedores y host del contenedor, solo tiene el firewall del host del contenedor. Todas las reglas del firewall configuradas en el host del contenedor se propagarán a todos sus contenedores.

### Los contenedores de Windows se inician con lentitud
Si el contenedor tarda más de 30 segundos en iniciarse, puede estar realizando varias exploraciones duplicadas de virus.

Muchas soluciones antimalware, como Windows Defender, pueden estar analizando innecesariamente los archivos de dentro de las imágenes de contenedor, incluidos todos los archivos y binarios del sistema operativo de la imagen del sistema operativo del contenedor.  Esto se produce cuando se crea un nuevo contenedor y, desde la perspectiva del software antimalware, todos los "archivos del contenedor" parecen archivos nuevos que no se han analizado anteriormente.  De esta manera, si los procesos dentro del contenedor intentan leer estos archivos, los componentes de software antimalware los examinan primero antes de permitir el acceso a los archivos.  En realidad, estos archivos ya se analizaron cuando la imagen del contenedor se importó o se extrajo al servidor. A partir de Windows Server Technical Preview 5, se incluye una infraestructura que permite que las soluciones antimalware, incluido Windows Defender, puedan conocer estas situaciones y actuar en consecuencia para evitar varios análisis. Las soluciones antimalware pueden actualizar su solución gracias a los pasos descritos [aquí](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers) y evitar varios análisis. 

### A veces se producen errores al iniciar o detener si la memoria está restringida a < 48 MB
Los contenedores de Windows experimentan errores incoherentes y aleatorios si la memoria está restringida a menos de 48 MB.

La ejecución del siguiente PowerShell y repetir la acción de inicio y detención varias veces producirán errores al iniciar o detener.

```PowerShell
new-container "Test" -containerimagename "WindowsServerCore" -MaximumBytes 32MB
start-container test
stop-container test
```

**Solución alternativa:**  
Cambie el valor de memoria a 48 MB. 


### Se produce un error de Start-container si el número de procesadores es 1 o 2 en una máquina virtual de 4 núcleos

Los contenedores de Windows no se iniciaron por el error:  
`failed to start: This operation returned because the timeout period expired. (0x800705B4).`

Esto sucede si el número de procesadores se establece en 1 o 2 en una máquina virtual de 4 núcleos.

``` PowerShell
new-container "Test2" -containerimagename "WindowsServerCore"
Set-ContainerProcessor -ContainerName test2 -Maximum 2
Start-Container test2

Start-Container : 'test2' failed to start.
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4).
'test2' failed to start. (Container ID 133E9DBB-CA23-4473-B49C-441C60ADCE44)
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4). (Container ID
133E9DBB-CA23-4473-B49C-441C60ADCE44)
At line:1 char:1
+ Start-Container test2
+ ~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationTimeout: (:) [Start-Container], VirtualizationException
    + FullyQualifiedErrorId : OperationTimeout,Microsoft.Containers.PowerShell.Cmdlets.StartContainer
PS C:\> Set-ContainerProcessor -ContainerName test2 -Maximum 3
PS C:\> Start-Container test2
```

**Solución alternativa:**  
Aumente los procesadores disponibles para el contenedor, sin especificar explícitamente los procesadores disponibles para el contenedor ni reducir los procesadores disponibles para la máquina virtual.

--------------------------

## Redes

### Aislamiento del compartimento de red e implicaciones
Cada contenedor usa un compartimento de red para proporcionar aislamiento. Todos los adaptadores de red del contenedor (puntos de conexión) asociados a un contenedor determinado residirán en el mismo compartimento de red. Según el modo de red (controlador) usado, es posible que no pueda acceder a dos puntos de conexión distintos del contenedor con la misma dirección IP o puerto. Además, las reglas de firewall de Windows no son compatibles con los compartimentos ni los contenedores, así que cualquier regla de firewall asociada se aplicará a todos los contenedores del host de contenedor, independientemente de un punto de conexión determinado.

*** Red transparente ***


*** Red NAT *** Puede exponer varios puntos de conexión asociados a un único contenedor con las reglas de reenvío de puerto NAT que se aplican por puntos de conexión. Estas reglas de reenvío deben usar distintos puertos externos (en el host de contenedor) cuando se asignan al mismo puerto interno (en el contenedor).  Pero, como se indicó anteriormente, las reglas de firewall asociadas tendrán ámbito global en el host de contenedor.



### Las asignaciones estáticas de NAT podrían entrar en conflicto con las asignaciones de puerto a través de Docker
A partir de Windows Server Technical Preview 5, las reglas de creación NAT y asignación de puertos están integradas en los cmdlets *ContainerNetwork* y los comandos Docker. Windows Host Network Service (HNS) administrará NAT en su nombre. Pero es posible que un cliente externo pueda probar y crear una regla de asignación de puertos duplicada con la misma NAT creada por HNS.


Este es un ejemplo de un conflicto con una asignación estática en el puerto 80 y del error notificado por Docker cuando se produce.
```
C:\Users\Administrator>docker run -it -p 80:80 windowsservercore cmd
docker: Error response from daemon: failed to create endpoint berserk_bassi on network nat: hnsCall failed in Win32: The remote procedure call failed. (0x6be).
```


***Mitigación*** En general, es muy poco probable que esto ocurra, ya que HNS administra NAT. Todas las reglas de reenvío y asignación de puertos deben crearse con `docker run -p <external>:<internal>` o Add-ContainerNetworkAdapterStaticMapping. Pero, si HNS no limpia automáticamente las asignaciones, este error puede resolverse mediante la eliminación de la asignación de puertos con PowerShell. Esta acción quitará el conflicto del puerto 80 del ejemplo anterior.
```powershell
Get-NetNatStaticMapping | ? ExternalPort -eq 80 | Remove-NetNatStaticMapping
```


### Los contenedores de Windows no obtienen direcciones IP
Si se conecta a los contenedores de Windows con conmutadores de máquina virtual DHCP, es posible que el host del contenedor reciba una dirección IP, pero los contenedores no.

Los contenedores obtienen un 169.254.***.*** Dirección IP APIPA.

**Solución alternativa:** Se trata de un efecto secundario del uso compartido del kernel.  Todos los contenedores tienen efectivamente la misma dirección MAC.

Habilite la suplantación de direcciones MAC en el host físico que hospeda la máquina virtual del host de contenedor.

Esto puede lograrse mediante PowerShell
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```
--------------------------

## Compatibilidad de aplicaciones

Hay tantas preguntas sobre las aplicaciones que funcionan y las que no en los contenedores de Windows, que se ha decidido separar la información sobre compatibilidad de aplicaciones en [su propio artículo](../reference/app_compat.md).

Algunos de los problemas más comunes se encuentran aquí también.

### Los registros de eventos no están disponibles dentro del contenedor

Los comandos de PowerShell como `get-eventlog Application` y las API que consultan el registro de eventos devolverán un error similar al siguiente:
```
get-eventlog : Cannot open log Application on machine .. Windows has not provided an error code.
At line:1 char:1
```

Como solución alternativa, puede agregar este paso a un Dockerfile. Las imágenes creadas con este paso incluido tendrán habilitado el registro de eventos.
```
RUN powershell.exe -command Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\WMI\Autologger\EventLog-Application Start 1
```


### Se produjo un error inesperado en una llamada al método de API de una instancia de localdb
Se produjo un error inesperado en una llamada al método de API de una instancia de localdb

### RTerm no funciona
RTerm se instala, pero no se inicia en un contenedor de Windows Server.

Error:  
```
'C:\Program' is not recognized as an internal or external command,
operable program or batch file.
```


### Contenedor: el tiempo de ejecución de Visual C++ x64/x86 2015 no se instala

Comportamiento observado: En un contenedor:
```
C:\build\vcredist_2015_x64.exe /q /norestart
C:\build>echo %errorlevel%
0
C:\build>wmic product get
No Instance(s) Available.
```

Se trata de un problema de interoperabilidad con el filtro de desduplicación. La desduplicación comprueba el destino del cambio de nombre para ver si se trata de un archivo desduplicado. Se produce un error en la emisión de create it con `STATUS_IO_REPARSE_TAG_NOT_HANDLED` porque el filtro de contenedor de Windows Server se encuentra por encima de la desduplicación.


Consulte el [artículo de compatibilidad de aplicaciones](../reference/app_compat.md) para obtener más información sobre las aplicaciones que se pueden incluir en contenedores.

--------------------------


## Administración de Docker

### No todos los comandos de Docker funcionan
* Se produce un error en la ejecución de Docker en los contenedores de Hyper-V.

Si se produce un error relacionado con algo que no está en esta lista (o si un comando genera un error diferente al esperado), háganoslo saber a través de [los foros](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).

### Hay un límite de 50 caracteres al pegar los comandos en una sesión de Docker interactiva
Hay un límite de 50 caracteres cuando se pegan comandos en una sesión de Docker interactiva.  
Si copia una línea de comandos en una sesión de Docker interactiva, actualmente hay una limitación de 50 caracteres. Simplemente se trunca la cadena pegada.

Esto no es así por diseño, estamos trabajando para eliminar la restricción.

### Errores de net use
Net use devuelve el error de sistema 1223 en lugar de solicitar el nombre de usuario o la contraseña

**Solución alternativa:**  
Especifique el nombre de usuario y la contraseña cuando ejecute net use.

``` PowerShell
net use S: \\your\sources\here /User:shareuser [yourpassword]
``` 


--------------------------



## Escritorio remoto 

No es posible administrar los contenedores de Windows ni interactuar con ellos a través de una sesión RDP en TP5.

--------------------------

### La salida de un contenedor en un host de contenedor Nano Server no se puede realizar con "exit"
Si intenta salir de un contenedor que se encuentra en un host de contenedor Nano Server, con "exit", se desconectará de dicho host y no saldrá del contenedor.

**Solución alternativa:** Use en su lugar Exit-PSSession para salir del contenedor.

No dude en solicitar características en [los foros](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers). 


--------------------------


## Usuarios y dominios

### Usuarios locales
Se pueden crear cuentas de usuario locales y usarse para ejecutar aplicaciones y servicios de Windows en contenedores.


### Pertenencia al dominio
Los contenedores no pueden unirse a dominios de Active Directory y no pueden ejecutar servicios ni aplicaciones como usuarios de dominio, cuentas de servicio o cuentas de equipo. 

Los contenedores están diseñados para comenzar rápidamente en un estado coherente conocido que es en gran medida independiente del entorno. Unirse a un dominio y aplicar configuraciones de directiva de grupo desde el dominio aumentarán el tiempo que se tarda en iniciar un contenedor, cambiarán su funcionamiento con el tiempo y limitarán la posibilidad de mover o compartir imágenes entre desarrolladores e implementaciones.

Estamos considerando cuidadosamente los comentarios que nos llegan sobre cómo las aplicaciones y los servicios usan Active Directory y la intersección de implementar estos en los contenedores. Si tiene información sobre lo que funcionaría mejor en su caso, compártalo con nosotros en [los foros](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers). 

Estamos buscando activamente soluciones que admitan estos tipos de escenarios.



<!--HONumber=Jun16_HO5-->


