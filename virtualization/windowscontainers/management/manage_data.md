



# Carpetas compartidas de contenedores

**Esto es contenido preliminar y está sujeto a cambios.**

Las carpetas compartidas permiten que los datos se compartan entre un host de contenedor y un contenedor. Cuando se haya creado la carpeta compartida, estará disponible dentro del contenedor. Los datos que se coloquen en la carpeta compartida desde el host estarán disponibles dentro del contenedor. Los datos que se coloquen en la carpeta compartida desde dentro del contenedor estarán disponibles en el host. Una sola carpeta en el host se puede compartir con muchos contenedores; con esta configuración, los datos se pueden compartir entre contenedores en ejecución.

## Administrar datos: PowerShell

### Crear carpeta compartida

Para crear una carpeta compartida, use el comando `Add-ContainerSharedFolder`. El ejemplo siguiente crea un directorio en el contenedor `c:\shared_data`, que se asigna a un directorio en el host `c:\data_source`.

> Un contenedor debe tener el estado detenido cuando se agrega una carpeta compartida.

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\data_source -DestinationPath c:\shared_data

ContainerName SourcePath       DestinationPath AccessMode
------------- ----------       --------------- ----------
DEMO          c:\data_source   c:\shared_data  ReadWrite
```

### Carpeta compartida de solo lectura

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\sf1 -DestinationPath c:\sf2 -AccessMode ReadOnly

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\sf1     c:\sf2          ReadOnly
```

### Enumerar las carpetas compartidas

Para ver una lista de las carpetas compartidas de un contenedor determinado, use el comando `Get-ContainerSharedFolder`.

```powershell
PS C:\> Get-ContainerSharedFolder -ContainerName DEMO2

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\source  c:\source       ReadWrite
```

### Modificar carpeta compartida

Para modificar la configuración de una carpeta compartida existente, use el comando `Set-ContainerSharedFolder`.

```powershell
PS C:\> Set-ContainerSharedFolder -ContainerName SFRO -SourcePath c:\sf1 -DestinationPath c:\sf1
```

### Quitar carpeta compartida

Para quitar una carpeta compartida, use el comando `Remove-ContainerSharedFolder`.

> Un contenedor debe tener el estado detenido cuando se quita una carpeta compartida.

```powershell
PS C:\> Remove-ContainerSharedFolder -ContainerName DEMO2 -SourcePath c:\source -DestinationPath c:\source
```
## Administrar datos: Docker

### Montar volúmenes

Al administrar contenedores de Windows con Docker, se pueden montar volúmenes con la opción `-v`.

En el ejemplo siguiente, la carpeta de origen es c:\source y la carpeta de destino c:\destination.

```powershell
PS C:\> docker run -it -v c:\source:c:\destination 1f62aaf73140 cmd
```

Para obtener más información sobre la administración de datos en contenedores con Docker, consulte [Volúmenes de Docker en Docker.com](https://docs.docker.com/userguide/dockervolumes/).

## Tutorial en vídeo

<iframe src="https://channel9.msdn.com/Blogs/containers/Container-Fundamentals--Part-3-Shared-Folders/player" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>



<!--HONumber=Feb16_HO3-->
