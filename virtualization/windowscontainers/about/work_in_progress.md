# Trabajo en curso

Si no ve que su problema se trate aquí o tiene preguntas, publique lo que necesite en el [foro](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).

-----------------------


## Funcionalidad general

### Los números de compilación del contenedor y el host deben coincidir

Un contenedor de Windows requiere una imagen de sistema operativo que coincida con el host del contenedor a nivel de compilación y revisión. Una discrepancia dará lugar a una posible inestabilidad o un comportamiento impredecible del contenedor o el host.

Si instala las actualizaciones en el sistema operativo del host del contenedor de Windows, deberá actualizar la imagen del sistema operativo base del contenedor para obtener las actualizaciones que coincidan.


**Solución alternativa:**   
Descargue e instale una imagen base de contenedor que coincida con el nivel de revisión y la versión del sistema operativo del host del contenedor.

### Todas las unidades excepto C:/ son visibles en los contenedores

Todas las unidades distintas de C:/ que estén disponibles para el host del contenedor se asignan automáticamente a los nuevos contenedores de Windows en ejecución.

En este momento, no hay ninguna manera de asignar carpetas selectivamente en un contenedor, así que como solución provisional, las unidades se asignan automáticamente.

**Solución alternativa: **  
Estamos trabajando en ella. En el futuro habrá uso compartido de carpetas.

### Comportamiento del firewall predeterminado

En un entorno de contenedores y host del contenedor, solo tiene el firewall del host del contenedor. Todas las reglas del firewall configuradas en el host del contenedor se propagarán a todos sus contenedores.

### Los contenedores de Windows se inician con lentitud

Si el contenedor tarda más de 30 segundos en iniciarse, puede estar realizando varias exploraciones duplicadas de virus.

Muchas soluciones antimalware, como Windows Defender, pueden estar analizando innecesariamente los archivos dentro de las imágenes de los contenedores que incluyen todos los archivos y binarios de sistema operativo en la imagen de sistema operativo del contenedor. Esto se produce cuando se crea un nuevo contenedor y, desde la perspectiva del software antimalware, todos los "archivos del contenedor" parecen archivos nuevos que no se ha analizado anteriormente. De esta manera, si los procesos dentro del contenedor intentan leer estos archivos, los componentes de software antimalware los examinan primero antes de permitir el acceso a los archivos. En realidad, estos archivos ya se analizaron cuando la imagen del contenedor se importó o se extrajo al servidor. En futuras vistas previas, se incluirá infraestructura de forma que las soluciones antimalware, entre las que se incluyen Windows Defender, puedan conocer estas situaciones y pueden actuar en consecuencia para evitar varias exploraciones.

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
"no se pudo iniciar: se devolvió esta operación porque se agotó el tiempo de espera. (0x800705B4)".

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


## Funciones de red

### Compartimientos de red limitados

En esta versión se admite un compartimiento de red por contenedor. Esto significa que si tiene un contenedor con varios adaptadores de red, no puede acceder al mismo puerto de red en cada adaptador (por ejemplo, 192.168.0.1:80 y 192.168.0.2:80 que pertenezcan al mismo contenedor).

**Solución alternativa: **  
Si un contenedor debe exponer varios puntos de conexión, utilice la asignación de puertos NAT.

### Los contenedores de Windows no obtienen direcciones IP

Si se conecta a los contenedores de Windows con conmutadores de máquina virtual DHCP, es posible que el host del contenedor reciba una dirección IP, pero los contenedores no.

Los contenedores obtienen una dirección IP de APIPA 169.254.***.***.

**Solución alternativa:**
Se trata de un efecto secundario de compartir el kernel. Todos los contenedores tienen efectivamente la misma dirección MAC.

Habilite la suplantación de direcciones MAC en el host del contenedor.

Esto puede lograrse mediante PowerShell
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```
### No se admiten HTTPS y TLS

Los contenedores de Windows Server y de Hyper-V no admiten HTTPS ni TLS. Estamos trabajando para que lo hagan en el futuro.

--------------------------


## Compatibilidad de aplicaciones

Hay tantas preguntas sobre las aplicaciones que funcionan y las que no en los contenedores de Windows, que hemos decidido dividir la información de compatibilidad de aplicaciones en [su propio artículo](../reference/app_compat.md).

Algunos de los problemas más comunes se encuentran aquí también.

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

Comportamiento observado:
En un contenedor:
```
C:\build\vcredist_2015_x64.exe /q /norestart
C:\build>echo %errorlevel%
0
C:\build>wmic product get
No Instance(s) Available.
```

Se trata de un problema de interoperabilidad con el filtro de desduplicación. La desduplicación comprueba el destino del cambio de nombre para ver si se trata de un archivo desduplicado. Se produce un error en la emisión de create it con el mensaje `STATUS_IO_REPARSE_TAG_NOT_HANDLED` porque el filtro de contenedor de Windows Server se encuentra por encima de la desduplicación.


Consulte el [artículo de compatibilidad de aplicaciones](../reference/app_compat.md) para más información sobre las aplicaciones que se pueden incluir en contenedores.

--------------------------



## Administración de Docker

### Clientes de Docker no seguros

En esta versión preliminar, la comunicación con Docker es pública si sabe dónde buscar.

### No todos los comandos de Docker funcionan

* Se produce un error en la ejecución de Docker en los contenedores de Hyper-V.
* Los comandos relacionados con DockerHub aún no se admiten.

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

No es posible administrar los contenedores de Windows ni interactuar con ellos a través de una sesión RDP en TP4.

--------------------------


## Administración de PowerShell

### No todos los elementos *-PSSession tienen un argumento containerid

Esto es correcto. Está planificada una compatibilidad completa con cimsession en el futuro.

### La salida de un contenedor en un host de contenedor Nano Server no se puede realizar con "exit"

Si intenta salir de un contenedor que se encuentra en un host de contenedor Nano Server, con "exit", se desconectará de dicho host y no saldrá del contenedor.

**Solución alternativa:**
Utilice en su lugar Exit-PSSession para salir del contenedor.

No dude en solicitar características en [los foros](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).


--------------------------



## Usuarios y dominios

### Usuarios locales

Se pueden crear cuentas de usuario locales y usarse para ejecutar aplicaciones y servicios de Windows en contenedores.


### Pertenencia al dominio

Los contenedores no pueden unirse a dominios de Active Directory y no pueden ejecutar servicios ni aplicaciones como usuarios de dominio, cuentas de servicio o cuentas de equipo.

Los contenedores están diseñados para comenzar rápidamente en un estado coherente conocido que es en gran medida independiente del entorno. Unirse a un dominio y aplicar configuraciones de directiva de grupo desde el dominio aumentarán el tiempo que se tarda en iniciar un contenedor, cambiarán su funcionamiento con el tiempo y limitarán la posibilidad de mover o compartir imágenes entre desarrolladores e implementaciones.

Estamos considerando cuidadosamente los comentarios que nos llegan sobre cómo las aplicaciones y los servicios usan Active Directory y la intersección de implementar estos en los contenedores. Si tiene información sobre lo que funcionaría mejor en su caso, compártalo con nosotros en [los foros](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers). Estamos buscando activamente soluciones que admitan estos tipos de escenarios.




<!--HONumber=Jan16_HO3-->
