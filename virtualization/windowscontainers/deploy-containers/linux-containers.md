# <a name="linux-containers"></a>Contenedores de Linux

Esta característica usa [Aislamiento de Hyper-V](../manage-containers/hyperv-container.md) para ejecutar un kernel de Linux con suficiente sistema operativo para admitir contenedores. Los cambios en Hyper-V y Windows para crear esto comenzaron en la actualización _Windows 10 Fall Creators Update_ y _Windows Server, versión 1709_, pero para lograrlo también debimos trabajar con el código abierto [proyecto Moby](https://www.github.com/moby/moby) que se usa para crear la tecnología de Docker, así como el kernel de Linux. 

![Vídeo de versión preliminar de contenedores de Linux](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

Para probar esto, necesitas lo siguiente:

- Windows 10 o Windows Server Insider Preview, compilación 16267 o posterior
- Una compilación del demonio Docker basada en la rama maestra de Moby, que se ejecuta con la marca `--experimental`
- La imagen de Linux compatible que selecciones

Las siguientes guías de introducción están disponibles para esta versión preliminar:

- [Docker Enterprise Edition Preview](https://blog.docker.com/2017/09/docker-windows-server-1709/) incluye un sistema LinuxKit y una versión preliminar de Docker EE que pueden ejecutar contenedores de Linux. Para conocer más detalles, consulta también [Preview Linux Containers on Windows using LinuxKit](https://go.microsoft.com/fwlink/?linkid=857061) en el blog de Docker (disponible en inglés)
- [Running Ubuntu Containers with Hyper-V Isolation on Windows 10 and Windows Server (disponible en inglés)](https://go.microsoft.com/fwlink/?linkid=857067)


## <a name="work-in-progress"></a>Trabajo en curso

Se puede hacer el seguimiento del trabajo en curso del proyecto Moby en [GitHub](https://github.com/moby/moby/issues/33850)


### <a name="known-app-issues"></a>Problemas conocidos de las aplicaciones

Todas las aplicaciones siguientes requieren la asignación de volumen, lo que tiene algunas limitaciones que se describen en la sección [Enlazar montajes](#Bind-mounts). No se iniciarán ni se ejecutarán correctamente.

- Mysql
- Postgress
- Wordpress
- Jenkins
- Mariadb
- Rabbitmq


### <a name="bind-mounts"></a>Enlazar montajes

Al enlazar volúmenes de montaje con `docker run -v ...`, se almacenan los archivos en el sistema de archivos NTFS de Windows, por lo que es necesaria una traducción de las operaciones de POSIX. Algunas operaciones del sistema de archivos están implementadas parcialmente o no se han implementado, lo que puede provocar problemas de compatibilidad con algunas aplicaciones.

Las siguientes operaciones no funcionan actualmente para volúmenes montados mediante enlace:

- MkNod
- XAttrWalk
- XAttrCreate
- Lock
- Getlock
- Auth
- Flush
- INotify

También hay algunas que no se han implementado completamente:

- GetAttr: el recuento de Nlink siempre se notifica como 2
- Open: solo se implementan las marcas ReadWrite, WriteOnly y ReadOnly