---
author: scooley
redirect_url: ../quick_start/manage_docker
translationtype: Human Translation
ms.sourcegitcommit: e56aa08067fb18caa28224ddc0577a677f09ded3
ms.openlocfilehash: 5a10afe0f0adcfa86fe9776efa45cfb935ca1beb

---


# Comparación de PowerShell frente a Docker para administrar contenedores de Windows

Hay muchas maneras de administrar contenedores de Windows con herramientas incluidas con Windows (en esta vista previa, PowerShell) y herramientas de administración de código abierto como Docker.  
Hay guías disponibles que explican cada una de ellas de manera individual, aquí:
* [Administrar contenedores de Windows con Docker](../quick_start/manage_docker.md)
* [Administrar contenedores de Windows con PowerShell](../quick_start/manage_powershell.md) 

Esta página es una referencia que compara en más profundidad las herramientas de Docker y las herramientas de administración de PowerShell.

## PowerShell para contenedores frente a máquinas virtuales de Hyper-V
Puede crear, ejecutar e interactuar con contenedores de Windows mediante cmdlets de PowerShell. Todo lo que necesita para empezar está disponible de forma predeterminada.

Si ha usado PowerShell de Hyper-V, el diseño de los cmdlets debe resultarle bastante familiar. Gran parte del flujo de trabajo es similar a la forma de administrar una máquina virtual con el módulo de Hyper-V. En lugar de `New-VM`, `Get-VM`, `Start-VM`, `Stop-VM`, tiene `New-Container`, `Get-Container`, `Start-Container`, `Stop-Container`.  Hay una gran cantidad de parámetros y cmdlets específicos de contenedores, pero la administración y el ciclo de vida general de un contenedor de Windows es relativamente similar al de una máquina virtual de Hyper-V.

## Comparativa de la administración de PowerShell frente a Docker 
Los cmdlets de PowerShell de contenedores exponen una API que no es exactamente igual a Docker; como regla general, los cmdlets son más granulares en operación. Algunos comandos de Docker tienen equivalentes muy claros en PowerShell:

| Comando de Docker |  Cmdlet de PowerShell |
|----|----|
| `docker ps -a`    | `Get-Container` |
| `docker images`   | `Get-ContainerImage` |
| `docker rm`   | `Remove-Container` |
| `docker rmi` | `Remove-ContainerImage` |
| `docker create`   | `New-Container` |
| `docker commit <container ID>` | `New-ContainerImage -Container <container>` |
| `docker load <tarball>` | `Import-ContainerImage <AppX package>` |
| `docker save` |   `Export-ContainerImage` |
| `docker start` |  `Start-Container` |
| `docker stop` |   `Stop-Container` |

Los cmdlets de PowerShell no son equivalentes perfectamente exactos y hay un número importante de comandos para los que no se ofrecen reemplazos de PowerShell* (en concreto, `docker build` y `docker cp`). Pero lo que probablemente más le llame la atención es que no hay sustituto de línea única para `docker run`.

\* Sujeto a cambios.

### Pero necesito docker run. ¿Qué está ocurriendo?  
Hay un par de aspectos con los que trabajamos aquí para ofrecer un modelo de interacción ligeramente más familiar para los usuarios que ya conocen bien PowerShell. Por supuesto, si está acostumbrado a la manera en que funciona Docker, implicará un pequeño cambio de mentalidad.

1.  El ciclo de vida de un contenedor en el modelo de PowerShell es ligeramente diferente. En el módulo de PowerShell de contenedores, se exponen las operaciones más granulares `New-Container` (que crea un contenedor nuevo que está detenido) y `Start-Container`.
  
  Entre la creación y el inicio del contenedor, también puede definir la configuración del contenedor; para TP3, la única otra configuración que tenemos intención de exponer es la capacidad para establecer la conexión de red del contenedor mediante los cmdlets (Add/Remove/Connect/Disconnect/Get/Set)-ContainerNetworkAdapter.

2.  Actualmente no se puede pasar un comando para que se ejecute dentro del contenedor al inicio. Sin embargo, todavía puede obtener una sesión de PowerShell interactiva en un contenedor en ejecución mediante `Enter-PSSession -ContainerId <ID of a running container>`. Asimismo, puede ejecutar un comando dentro de un contenedor en ejecución mediante `Invoke-Command -ContainerId <container id> -ScriptBlock { code to run inside the container }` o `Invoke-Command -ContainerId <container id> -FilePath <path to script>`.  
Estos dos comandos permiten la marca opcional `-RunAsAdministrator` para las acciones con privilegios elevados.  


## Advertencias y problemas conocidos
1.  En este momento, los cmdlets de contenedores no tienen información sobre los contenedores o las imágenes creadas mediante Docker y Docker no sabe nada sobre los contenedores y las imágenes creadas mediante PowerShell. Si se creó en Docker, se debe administrar con Docker; si se ha creado mediante PowerShell, se debe administrar con PowerShell.

2.  Hay una gran cantidad de trabajo que nos gustaría realizar para mejorar la experiencia del usuario final: mejores mensajes de error mejorados, informes de progreso mejorados, cadenas de eventos no válidos, etc. Si se encontrara en alguna situación en la que le gustaría recibir más información o más detallada, no dude en enviar sus sugerencias a los foros.

## Un rápido repaso
Este es un tutorial sobre algunos flujos de trabajo comunes.

Se supone que ha instalado una imagen de contenedor de sistema operativo llamada "ServerDatacenterCore" y ha creado un conmutador virtual denominado "Virtual Switch" (mediante New-VMSwitch).

``` PowerShell
### 1. Enumerating images
# At this point, you can enumerate the images on the system:
Get-ContainerImage

# Get-ContainerImage also accepts filters.
# For example, this will return all container images whose Name starts with S (case-insensitive):
Get-ContainerImage -Name S*

# You can save the results of this to a variable.
# (If you're not familiar with PowerShell, the "$" denotes a variable.)
$baseImage = Get-ContainerImage -Name ServerDatacenterCore
$baseImage

### 2. Creating and enumerating containers
# Now, we can create a new container using this image:
New-Container -Name "MyContainer" -ContainerImage $baseImage -SwitchName "Virtual Switch"

# Now we can enumerate all containers.
Get-Container

# Similarly, we can save this container to a variable:
$container1 = Get-Container -Name "MyContainer"

### 3. Starting containers, interacting with running containers, and stopping containers
# Now let's go ahead and start the container.
Start-Container -Name "MyContainer"

# (We could've also started this container using "Start-Container -Container $container1".)

# With the container now running, let's go ahead and enter an interactive PowerShell session:
Enter-PSSession -ContainerId $container1.Id

# This should eventually bring up a PowerShell prompt from inside the container.
# You can try all the things that you did in the interactive cmd prompt given by "docker run -it".
# For now, just to prove we've been here, we can create a new file:
cd \
mkdir Test
cd Test
echo "hello world" > hello.txt
exit

# Now we should be back in the outside world. Even though we've exited the PowerShell session,
# the container itself is still running, as you can see by printing out the container again:
$container1

# Before we can commit this container to a new image, we need to stop the container.
# Let's do that now.
Stop-Container -Container $container1

### 4. Creating a new container image
# And now let's commit it to a new image.
$image1 = New-ContainerImage -Container $container1 -Publisher Test -Name Image1 -Version 1.0

# Enumerate all the images again, for sanity's sake:
Get-ContainerImage

# Rinse and repeat! Make another container based on the new image.
$container2 = New-Container -Name "MySecondContainer" -ContainerImage $image1 -SwitchName "Virtual Switch"

# (If you like, you can start the second container and verify that the new file
# "\Test\hello.txt" is there as expected.)

### 5. Removing a container
# The first container we created is now stopped. Let's get rid of it:
Remove-Container -Container $container1

# And confirm that it's gone:
Get-Container

### 6. Exporting, removing, and importing images
# For images that aren't the base OS image, we can export them into an .appx package file.
Export-ContainerImage -Image $image1 -Path "C:\exports"
# This should create a .appx file in the C:\exports folder.
# If you've given your image the same publisher, name, and version we used earlier,
# you'd expect the resulting .appx to be named "CN=Test_Image1_1.0.0.0.appx".

# Before we can try importing the image again, we need to remove the image.
# (If you have any running containers that depend on this image, you'll want to stop them first.)
Remove-ContainerImage -Image $image1

# Now let's import the image again:
Import-ContainerImage -Path C:\exports\CN=Test_Image1_1.0.0.0.appx

# We'd previously created a container dependent on this image. You should be able to start it:
Start-Container -Container $container2 
```

### Compilar su propio ejemplo
Puede ver todos los cmdlets Container mediante `Get-Command -Module Containers`.  Hay muchos otros cmdlets que no se describen aquí, que dejaremos para que se informe por su cuenta.    
**Nota** No se devolverán los cmdlets `Enter-PSSession` e `Invoke-Command`, que forman parte del núcleo de PowerShell.

También puede obtener ayuda sobre el uso de cualquier cmdlet `Get-Help [cmdlet name]`, o de forma equivalente `[cmdlet name] -?`.  En la actualidad, el resultado de la ayuda se genera automáticamente y simplemente indica la sintaxis de los comandos; iremos agregando más documentación a medida que nos acerquemos a finalizar el diseño de los cmdlets.

Una forma más adecuada de conocer la sintaxis es PowerShell ISE, que puede que nunca haya visto si no ha utilizado mucho PowerShell. Si está ejecutando en una SKU que lo permita, pruebe a iniciar el ISE, abra el panel Comandos y elija el módulo "Contenedores", que mostrará una representación gráfica de los cmdlets y sus conjuntos de parámetros.

PS: Para demostrar que es posible, aquí se muestra una función de PowerShell que forma parte de algunos de los cmdlets que vimos anteriormente en una imitación de `docker run`. (Como aclaración, cabe destacar que se trata de una prueba de concepto, no está en desarrollo activo).

``` PowerShell
function Run-Container ([string]$ContainerImageName, [string]$Name="fancy_name", [switch]$Remove, [switch]$Interactive, [scriptblock]$Command) {
    $image = Get-ContainerImage -Name $ContainerImageName
    $container = New-Container -Name $Name -ContainerImage $image
    Start-Container $container

    if ($Interactive) {
         Start-Process powershell ("-NoExit", "-c", "Enter-PSSession -ContainerId $($container.Id)") -Wait
    } else {
        Invoke-Command -ContainerId $container.Id -ScriptBlock $Command
    }

    Stop-Container $container

    if ($Remove) {
        Remove-Container $container -Force
    }
} 
```

## Docker
Los contenedores de Windows se pueden administrar con comandos de Docker.  Aunque los contenedores de Windows deberían ser comparables a sus homólogos de Linux y tener la misma experiencia de administración mediante Docker, hay algunos comandos de Docker que simplemente no tienen sentido con un contenedor de Windows.  Otros simplemente no se han probado (pronto lo haremos).

En un esfuerzo por no duplicar la documentación de API disponible en Docker, <a href="https://docs.docker.com/engine/reference/commandline/cli/" >aquí</a> se muestra un vínculo a sus API de administración.  Sus tutoriales son fantásticos.

Estamos realizando un seguimiento lo que funciona y lo que no en las API de Docker en nuestro documento de trabajo en curso.



<!--HONumber=Jun16_HO4-->


