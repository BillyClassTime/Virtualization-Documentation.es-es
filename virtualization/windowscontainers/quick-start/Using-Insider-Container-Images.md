
# <a name="using-insider-container-images"></a>Uso de imágenes del contenedor de Insider

Este ejercicio te llevará por la implementación y el uso de la función de contenedor de Windows en la última compilación para Insider de Windows Server desde el programa Windows Insider Preview. Durante este ejercicio, tendrás que instalar el rol de contenedor e implementar una edición de vista previa de las imágenes de sistema operativo base. Si necesitas familiarizarte con los contenedores, encontrarás esta información en [Acerca de los contenedores](../about/index.md).

Este inicio rápido es específico de los contenedores de WindowsServer en el programa WindowsServerInsiderPreview. Familiarízate con el programa antes de continuar con este inicio rápido.

## <a name="prerequisites"></a>Requisitos previos:

- Forma parte del [Programa Windows Insider](https://insider.windows.com/GettingStarted) y revisa los términos de uso.
- Un sistema del equipo (físico o virtual) que ejecute la última versión de Windows Server desde el programa Windows Insider y/o la compilación más reciente de Windows 10 del programa Windows Insider.

> [!IMPORTANT]
> Debes usar una compilación de Windows Server desde el programa Windows Server Insider Preview o una compilación de Windows 10 desde el programa Windows Insider Preview para usar la imagen base que se describe a continuación. Si no estás utilizando una de estas compilaciones, el uso de estas imágenes base dará como resultado errores al iniciar un contenedor.

## <a name="install-docker-enterprise-edition-ee"></a>Instalar DockerEnterpriseEdition (EE)

Para trabajar con contenedores de Windows es necesario DockerEE. DockerEE consta de motor y de cliente.

Para instalar DockerEE, usaremos el módulo de PowerShell del proveedor OneGet. El proveedor habilitará la característica de contenedores en la máquina e instalará DockerEE, lo que requerirá un reinicio. Abre una sesión de PowerShell con privilegios elevados y ejecuta los comandos siguientes.

> [!NOTE]
> Instalar Docker EE con compilaciones de Windows Server Insider requiere otro proveedor de OneGet del que se usa para las compilaciones que no son de Insider. Si el proveedor de DockerMsftProviderOneGet y DockerEE ya están instalados, quítalos antes de continuar.

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

Instala el módulo OneGetPowerShell para usarlo con compilaciones de WindowsInsider.

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

Usa OneGet para instalar la última versión de DockerEEPreview.

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

Cuando la instalación se haya completado, reinicia el equipo.

```powershell
Restart-Computer -Force
```

## <a name="install-base-container-image"></a>Instalar la imagen del contenedor de base

Antes de trabajar con los contenedores de Windows, debe instalarse una imagen base. Al formar parte del programa Windows Insider, también puedes probar nuestras últimas compilaciones para las imágenes base. Con las imágenes base de Insider, ahora existen 4 imágenes base disponibles basadas en Windows Server. Consulta la tabla siguiente para comprobar los fines para los que debe usarse cada una:

| Imagen base del sistema operativo                       | Uso                      |
|-------------------------------------|----------------------------|
| MCR.Microsoft.com/Windows/servercore         | Producción y desarrollo |
| MCR.Microsoft.com/Windows/nanoserver              | Producción y desarrollo |
| MCR.Microsoft.com/Windows/servercore/Insider | Solo desarrollo           |
| MCR.Microsoft.com/Windows/nanoserver/Insider        | Solo desarrollo           |

Para recuperar la imagen base de Insider de Nano Server, ejecuta lo siguiente:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Para recuperar la imagen base de Windows Server Core Insider, ejecuta lo siguiente:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> Lee el [CLUF](../EULA.md ) de imagen de sistema operativo de contenedores de Windows y los [Términos de uso](https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewserver)del programa Windows Insider.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Compilar y ejecutar una aplicación de ejemplo](./Nano-RS3-.NET-Core-and-PS.md)