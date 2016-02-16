# Administración de recursos de contenedores

**Esto es contenido preliminar y está sujeto a cambios.**

Los contenedores de Windows incluyen la capacidad de administrar los recursos de CPU, entrada y salida de disco, red y memoria pueden consumir los contenedores. Si se restringen los consumos de recursos, los recursos del host se utilizarán de forma eficiente y se evitará el consumo excesivo. En este documento se detallará la administración de recursos de contenedor con PowerShell y Docker.

## Administrar recursos con PowerShell

### Memory

Se pueden establecer límites de memoria de los contenedores cuando se crea un contenedor mediante el parámetro `-MaximumMemoryBytes` del comando `New-Container`. Este ejemplo establece la cantidad máxima de memoria a 256 MB.

```powershell
PS C:\> New-Container –Name TestContainer –MaximumMemoryBytes 256MB -ContainerimageName WindowsServerCore
```
También puede establecer el límite de memoria de un contenedor existente con el cmdlet `Set-ContainerMemory`.

```powershell
PS C:\> Set-ContainerMemory -ContainerName TestContainer -MaximumBytes 256mb
```

### Ancho de banda de la red

Se pueden establecer límites de ancho de banda de la red en un contenedor existente. Para ello, asegúrese de que el contenedor tenga un adaptador de red mediante el comando `Get-ContainerNetworkAdapter`. Si no existe un adaptador de red, use el comando `Add-ContainerNetworkAdapter` para crear uno. Por último, use el comando `Set-ContainerNetworkAdapter` para limitar el ancho de banda de la red de salida máximo del contenedor.

En el ejemplo siguiente, el ancho de banda máximo es 100 Mbps.

```powershell
PS C:\> Set-ContainerNetworkAdapter –ContainerName TestContainer –MaximumBandwidth 100000000
```

### CPU

Puede limitar la cantidad de procesos que un contenedor puede utilizar estableciendo un porcentaje máximo de CPU o un peso relativo para el contenedor. Aunque combinar estos dos esquemas de administración de CPU no está prohibido, no se recomienda. De forma predeterminada, todos los contenedores disponen de uso completo del procesador (un máximo de 100 %) y un peso relativo de 100.

Lo siguiente establece el peso relativo del contenedor a 1000. El peso predeterminado de un contenedor es 100, por lo que este contenedor tendrá 10 veces la prioridad de un contenedor con el valor predeterminado. El valor máximo es 10000.

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 –RelativeWeight 10000
```

También puede establecer un límite en la cantidad de CPU que un contenedor puede utilizar, en términos de porcentaje de tiempo de CPU. De forma predeterminada, un contenedor puede utilizar el 100 % de la CPU. Lo siguiente establece el porcentaje máximo de una CPU que puede utilizar un contenedor al 30 %. Si se usa la marca –Maximum, se establece automáticamente el valor de RelativeWeight en 100.

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 -Maximum 30
```

### E/S de almacenamiento

Puede limitar la cantidad de E/S que un contenedor puede usar en términos de ancho de banda (bytes por segundo) o E/S por segundo normalizada de 8k. Estos dos parámetros se pueden establecer conjuntamente. La limitación se produce cuando se alcanza el primer límite.

```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumBandwidth 1000000
```
```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumIOPS 32
```

## Administrar recursos con Docker

Se ofrece la capacidad de administrar un subconjunto de recursos del contenedor a través de Docker. Concretamente, se permiten que los usuarios especifiquen cómo se debe compartir la CPU entre contenedores.

### CPU

Es posible administrar los recursos compartidos de la CPU entre contenedores en tiempo de ejecución mediante la marca --cpu-shares. De forma predeterminada, todos los contenedores tienen la misma proporción de tiempo de CPU. Para cambiar el recurso compartido relativo de la CPU que usan los contenedores, ejecute la marca --cpu-shares con un valor de 1-10000. De forma predeterminada, todos los contenedores reciben un peso de 5000. Para más información sobre la limitación de uso compartido de la CPU, consulte [Docker Run Reference](https://docs.docker.com/engine/reference/run/#cpu-share-constraint) (Referencia de ejecución de Docker).

```powershell 
C:\> docker run –it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## Problemas conocidos

- Actualmente, no se admiten los controles de recursos de E/S y CPU con contenedores de Hyper-V.
- Los controles de recursos de E/S no se admiten actualmente con las carpetas compartidas de contenedores.

## Tutorial en vídeo

<iframe src="https://channel9.msdn.com/Blogs/containers/Container-Fundamentals--Part-4-Resource-Management/player#ccLang=es" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>





<!--HONumber=Feb16_HO1-->
