# Solucionar problemas

¿Tiene problemas para configurar el equipo o para ejecutar un contenedor? Hemos creado un script de PowerShell para comprobar los problemas comunes. Pruébelo primero para ver lo que encuentra y compartir los resultados.

```PowerShell
Invoke-WebRequest https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/windows-server-container-tools/Debug-ContainerHost/Debug-ContainerHost.ps1 | Invoke-Expression
```
Puede ver una lista de todas las pruebas que se ejecutan junto con soluciones comunes en el [archivo Léame](https://github.com/Microsoft/Virtualization-Documentation/blob/master/windows-server-container-tools/Debug-ContainerHost/README.md) del script.

Si eso no ayuda encontrar el origen del problema, publique la salida del script en el [Foro del contenedor](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers). Este es el mejor lugar para obtener ayuda de la comunidad, incluidos los desarrolladores e Insiders de Windows.


## Buscar registros
Hay varios servicios que se usan para administrar contenedores de Windows. En las secciones siguientes se muestra dónde obtener los registros de cada servicio.

### Motor de Docker
El motor de Docker registra en el registro de eventos "Application" de Windows, en lugar de en un archivo. Estos registros se pueden leer, ordenar y filtrar muy fácilmente con Windows PowerShell.

Por ejemplo, esto mostrará los registros del motor de Docker de los últimos 5 minutos, empezando por los más antiguos.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

Esto también se podría canalizar fácilmente en un archivo CSV para que otra herramienta u hoja de cálculo pueda leerlo.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

#### Habilitar el registro de depuración
También puede habilitar el registro de depuración en el motor de Docker. Esto puede resultar útil para solucionar problemas si los registros normales no tienen información suficiente.

En primer lugar, abra un símbolo del sistema con privilegios elevados, ejecute `sc.exe qc docker` para obtener la línea de comandos para el servicio Docker.
Ejemplo:
```none
C:\> sc.exe qc docker
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: docker
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\Docker\dockerd.exe" --run-service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Docker Engine
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

Tome la `BINARY_PATH_NAME` actual y modifíquela:
- Agregue -D al final
- Escape las "con \
- Escriba el comando completo entre "

A continuación, ejecute `sc.exe config docker binpath= ` seguido de la nueva cadena. Por ejemplo: 
```none
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


Reinicie el servicio Docker
```none
sc.exe stop docker
sc.exe start docker
```

Esto registrará mucha más información en el registro de eventos de aplicación, por lo que es mejor quitar la opción `-D` una vez que haya terminado de solucionar los problemas. Use los mismos pasos anteriores sin `-D` y reinicie el servicio para deshabilitar el registro de depuración.


### Servicio de contenedor de host
El motor de Docker depende de un servicio de contenedor de host específico de Windows. Tiene registros independientes: 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

Son visibles en el Visor de eventos y también pueden consultarse con PowerShell.

Por ejemplo:
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```



<!--HONumber=Oct16_HO3-->


