# <a name="using-insider-container-images"></a>Uso de imágenes del contenedor de Insider

Este ejercicio te llevará por la implementación y el uso de la función de contenedor de Windows en la última compilación para Insider de Windows Server desde el programa Windows Insider Preview. Durante este ejercicio, tendrás que instalar el rol de contenedor e implementar una edición de vista previa de las imágenes de sistema operativo base. Si necesitas familiarizarte con los contenedores, encontrarás esta información en [Acerca de los contenedores](../about/index.md).

Este inicio rápido es específico de los contenedores de WindowsServer en el programa WindowsServerInsiderPreview. Familiarízate con el programa antes de continuar con este inicio rápido.

**Requisitos previos:**

- Forma parte del [Programa Windows Insider](https://insider.windows.com/GettingStarted) y revisa los términos de uso.
- Un sistema del equipo (físico o virtual) que ejecute la última versión de Windows Server desde el programa Windows Insider y/o la compilación más reciente de Windows 10 del programa Windows Insider.

>Es necesario que uses una compilación de Windows Server desde el programa Windows Server Insider Preview, o una compilación de Windows 10 del programa Windows Insider Preview, para poder usar la imagen base que se describe a continuación. Si no estás utilizando una de estas compilaciones, el uso de estas imágenes base dará como resultado errores al iniciar un contenedor.

## <a name="install-docker"></a>Instalar Docker
Para trabajar con contenedores de Windows es necesario Docker. Docker consta de motor y cliente. También necesitarás una versión de Docker que admita compilaciones multietapa para conseguir la mejor experiencia al usar la imagen de Nano Server optimizada para contenedores.

Para instalar Docker, usaremos el módulo de PowerShell del proveedor OneGet. El proveedor habilitará la característica de contenedores en la máquina e instalará Docker, lo que requerirá un reinicio. Ten en cuenta que hay varios canales con diferentes versiones de docker para usarlas en diferentes casos. En este ejercicio, vamos a usar la versión más reciente de Community Edition de Docker del canal estable. También está disponible un canal avanzado, por si quieres probar los últimos avances en Docker.

Abre una sesión de PowerShell con privilegios elevados y ejecuta los comandos siguientes.

>Nota: Instalar Docker en las compilaciones de Insider requiere un proveedor distinto del que se usa normalmente en la actualidad. Ten en cuenta la diferencia que se señala a continuación.

Instala el módulo OneGet de PowerShell.
```powershell
Install-Module -Name DockerMsftProviderInsider -Repository PSGallery -Force
```
Usa OneGet para instalar la versión más reciente de Docker.
```powershell
Install-Package -Name docker -ProviderName DockerMsftProviderInsider -RequiredVersion 17.06.0-ce
```
Cuando la instalación se haya completado, reinicia el equipo.
```
Restart-Computer -Force
```

## <a name="install-base-container-image"></a>Instalar la imagen del contenedor de base

Antes de trabajar con los contenedores de Windows, debe instalarse una imagen base. Al formar parte del programa Windows Insider, también puedes probar nuestras últimas compilaciones para las imágenes base. Con las imágenes base de Insider, ahora existen 4 imágenes base disponibles basadas en Windows Server. Consulta la tabla siguiente para comprobar los fines para los que debe usarse cada una:

| Imagen base del sistema operativo                       | Uso                      |
|-------------------------------------|----------------------------|
| microsoft/windowsservercore         | Producción y desarrollo |
| microsoft/nanoserver                | Producción y desarrollo |
| microsoft/windowsservercore-insider | Solo desarrollo           |
| microsoft/nanoserver-insider        | Solo desarrollo           |

Para recuperar la imagen base de Insider de Nano Server, ejecuta lo siguiente:

```
docker pull microsoft/nanoserver-insider
```

Para recuperar la imagen base de Windows Server Core Insider, ejecuta lo siguiente:

```
docker pull microsoft/windowsservercore-insider
```

Consulta el CLUF de imágenes de sistema operativo de contenedores de Windows que encontrarás aquí: [EULA](../EULA.md ) y los términos de uso del programa Windows Insider, que se encuentran aquí: [Términos de uso](https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewserver).

## <a name="next-steps"></a>Pasos siguientes

[Compilar y ejecutar una aplicación con o sin .NET Core 2.0 o PowerShell Core 6](./Nano-RS3-.NET-Core-and-PS.md)
