
# <a name="using-insider-container-images"></a>Uso de imágenes del contenedor de Insider

Este ejercicio te llevará por la implementación y el uso de la función de contenedor de Windows en la última compilación para Insider de Windows Server desde el programa Windows Insider Preview. Durante este ejercicio, tendrás que instalar el rol de contenedor e implementar una edición de vista previa de las imágenes de sistema operativo base. Si necesitas familiarizarte con los contenedores, encontrarás esta información en [Acerca de los contenedores](../about/index.md).

Este inicio rápido es específico de los contenedores de WindowsServer en el programa WindowsServerInsiderPreview. Familiarízate con el programa antes de continuar con este inicio rápido.

## <a name="prerequisites"></a>Requisitos previos:

- Forma parte del [Programa Windows Insider](https://insider.windows.com/GettingStarted) y revisa los términos de uso.
- Un sistema del equipo (físico o virtual) que ejecute la última versión de Windows Server desde el programa Windows Insider y/o la compilación más reciente de Windows 10 del programa Windows Insider.

> [!IMPORTANT]
> Windows requiere que la versión del sistema operativo del host coincida con la versión del sistema operativo del contenedor. Si desea ejecutar un contenedor basado en una compilación de Windows más reciente, asegúrese de que tiene una compilación de host equivalente. De lo contrario, puede usar el aislamiento de Hyper-V para ejecutar contenedores más antiguos en nuevas compilaciones de host. Puede obtener más información sobre la compatibilidad de la versión del contenedor de Windows en nuestros documentos de contenedor.

## <a name="install-docker-enterprise-edition-ee"></a>Instalar DockerEnterpriseEdition (EE)

Para trabajar con contenedores de Windows es necesario DockerEE. DockerEE consta de motor y de cliente.

Para instalar DockerEE, usaremos el módulo de PowerShell del proveedor OneGet. El proveedor habilitará la característica de contenedores en la máquina e instalará DockerEE, lo que requerirá un reinicio. Abre una sesión de PowerShell con privilegios elevados y ejecuta los comandos siguientes.

> [!NOTE]
> Instalar el acoplador EE con compilaciones de Insider de Windows Server requiere un proveedor de OneGet diferente al usado para las compilaciones que no son de Insider. Si el proveedor de DockerMsftProviderOneGet y DockerEE ya están instalados, quítalos antes de continuar.

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

Antes de trabajar con los contenedores de Windows, debe instalarse una imagen base. Al formar parte del programa Windows Insider, también puedes probar nuestras últimas compilaciones para las imágenes base. Con las imágenes básicas de Insider, ahora hay 6 imágenes de base disponibles basadas en Windows Server. Consulta la tabla siguiente para comprobar los fines para los que debe usarse cada una:

| Imagen base del sistema operativo                       | Uso                      |
|-------------------------------------|----------------------------|
| mcr.microsoft.com/windows/servercore         | Producción y desarrollo |
| mcr.microsoft.com/windows/nanoserver              | Producción y desarrollo |
| mcr.microsoft.com/windows/              | Producción y desarrollo |
| mcr.microsoft.com/windows/servercore/insider | Solo desarrollo           |
| mcr.microsoft.com/windows/nanoserver/insider        | Solo desarrollo           |
| mcr.microsoft.com/windows/insider        | Solo desarrollo           |

Para extraer la imagen de Server Core Sider base, consulte las marcas destacadas en el [repositorio de Hub del Docker de Server Core](https://hub.docker.com/_/microsoft-windows-servercore-insider) para usar el siguiente formato:

```console
docker pull mcr.microsoft.com/windows/servercore/insider:10.0.{build}.{revision}
```

Para extraer la imagen de la base de nano Server Insider, consulta las marcas destacadas en el [repositorio del Hub de nano Server Insider](https://store.docker.com/_/microsoft-windows-nanoserver-insider) para usar el siguiente formato:

```console
docker pull mcr.microsoft.com/windows/nanoserver/insider:10.0.{build}.{revision}
```

Para extraer la imagen de Windows Insider, consulte las etiquetas destacadas en el [repositorio del concentrador de acoplamiento de Windows Insider](https://store.docker.com/_/microsoft-windows-insider) para usar el siguiente formato:

```console
docker pull mcr.microsoft.com/windows/insider:10.0.{build}.{revision}
```

> [!IMPORTANT]
> Lea el [CLUF](../EULA.md ) de la imagen de Windows Containers y las [condiciones de uso](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)del programa Windows Insider.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Compilar y ejecutar una aplicación de ejemplo](./Nano-RS3-.NET-Core-and-PS.md)
