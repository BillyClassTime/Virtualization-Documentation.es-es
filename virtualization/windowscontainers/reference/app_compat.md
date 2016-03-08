



# Compatibilidad de las aplicaciones en los contenedores de Windows

Se trata de una vista previa. Aunque finalmente la aplicación que se ejecuta en Windows también debería ejecutarse en un contenedor, es un buen lugar para ver el estado de compatibilidad de la aplicación actual.

El único propósito de este documento es compartir nuestra experiencia.

¿Falta algo en esta lista? Háganos saber lo que funciona correctamente en su entorno y lo que no a través de [los foros](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).

## Contenedores de Windows Server

Hemos intentado ejecutando las siguientes aplicaciones en un contenedor de Windows Server. Estos resultados no garantizan que la aplicación funcione correctamente.

| **Nombre**| **Versión**| **Imagen base de Windows Server Core**| **Imagen base de Nano Server**| **Comentario**|
|:-----|:-----|:-----|:-----|:-----|
| .NET| 3.5| Sí| Unknown| |
| .NET| 4.6| Sí| Unknown| |
| .NET CLR| 5 beta 6| Sí| Sí| Ambos, x64 y x86|
| Active Python| 3.4.1| Sí| Sí| |
| Apache Cassandra| | Sí| Desconocido|
| Apache CouchDB| 1.6.1| No| No| |
| Apache Hadoop| | Sí| No| |
| Apache HTTPD| 2.4| Sí| Sí| El tiempo de ejecución de VC++ no se instala si se ha cargado el filtro de desduplicación.Descargar la desduplicación mediante `fltmc unload dedup`|
| Apache Tomcat| 8.0.24 x64| Sí| Unknown| |
| ASP.NET| 4.6| Sí| Desconocido| |
| ASP.NET| 5 beta 6| Sí| Sí| Ambos, x64 y x86|
| Django| | Sí| Sí| |
| Go| 1.4.2| Sí| Sí| |
| Internet Information Service| 10.0| Sí| Sí| El tiempo de ejecución de VC++ no se instala si se ha cargado el filtro de desduplicación.Descargar la desduplicación mediante `fltmc unload dedup`|
| Java| 1.8.0_51| Sí| Sí| Utilice la versión de servidor.La versión de cliente no se instala correctamente|
| MongoDB| 3.0.4| Sí| Desconocido| |
| MySQL| 5.6.26| Sí| Sí| |
| NGinx| 1.9.3| Sí| Sí| |
| Node.js| 0.12.6| Parcialmente| Parcialmente| NPM no puede descargar paquetes.|
| Perl| | Sí| Desconocido| |
| PHP| 5.6.11| Sí| Sí| El tiempo de ejecución de VC++ no se instala si se ha cargado el filtro de desduplicación.Descargar la desduplicación mediante `fltmc unload dedup`|
| PostgreSQL| 9.4.4| Sí| Unknown| El tiempo de ejecución de VC++ no se instala si se ha cargado el filtro de desduplicación.Descargar la desduplicación mediante `fltmc unload dedup`|
| Python| 3.4.3| Sí| Sí| |
| R| 3.2.1| No| No| |
| RabbitMQ| 3.5.x| Sí| Unknown| |
| Redis| 2.8.2101| Sí| Sí| |
| Ruby| 2.2.2| Sí| Sí| Ambos, x64 y x86|
| Ruby on Rails| 4.2.3| Sí| Sí| |
| SQLite| 3.8.11.1| Sí| No| |
| SQL Server Express| LocalDB 2014| No| No| |
| Herramientas de Sysinternals| *| Sí| Sí| Solo se intentaron las que no requieren una GUI.PsExec no funciona por el diseño actual|

## Contenedores de Hyper-V

Se han intentado ejecutar las siguientes aplicaciones en un contenedor de Hyper-V. Estos resultados no garantizan que la aplicación funcione correctamente.

| **Nombre**| **Versión**| **Imagen base de Nano Server**| **Comentario**|
|:-----|:-----|:-----|:-----|
| Apache Hadoop| | No| |
| Apache HTTPD| 2.4| Sí| El tiempo de ejecución de VC++ no se instala si se ha cargado el filtro de desduplicación.Descargar la desduplicación mediante `fltmc unload dedup`|
| ASP.NET| 5 beta 6| Sí| Ambos, x64 y x86|
| Django| | Sí| Si la imagen se crea un archivo DockerFile y los archivos binarios de Python se copian como parte de él, Python no funciona.Inicie el contenedor y después copie los archivos binarios de Python.|
| Go| 1.4.2| Sí| |
| Internet Information Service| 10.0| Sí| IIS no se instala mediante dism directamente.Realice una instalación desatendida de IIS mediante los comandos de dism.|
| Java| 1.8.0_51| Sí| Utilice la versión de servidor.La versión de cliente no se instala correctamente|
| MySQL| 5.6.26| Sí| |
| NGinx| 1.9.3| Sí| |
| Node.js| 0.12.6| Parcialmente| NPM no puede descargar paquetes.|
| Python| 3.4.3| Sí| Si la imagen se crea un archivo DockerFile y los archivos binarios de Python se copian como parte de él, Python no funciona.Inicie el contenedor y después copie los archivos binarios de Python.|
| Redis| 2.8.2101| Sí| |
| Ruby| 2.2.2| Sí| Ambos, x64 y x86|
| Ruby on Rails| 4.2.3| Sí| |
| Herramientas de Sysinternals| | Sí| Solo se intentaron las que no requieren una GUI.PsExec no funciona por el diseño actual.|

## Cuéntenos su experiencia

¿Falta algo en esta lista? Háganos saber lo que funciona correctamente en su entorno y lo que no a través de [los foros](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).




<!--HONumber=Feb16_HO3-->
