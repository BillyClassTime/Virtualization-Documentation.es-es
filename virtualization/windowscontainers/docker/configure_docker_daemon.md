---
title: Configuración de Docker en Windows
description: Configuración de Docker en Windows
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
---

El motor de Docker no se incluye con Windows y deberá instalarse y configurarse por separado. Además, el demonio de Docker puede aceptar varias configuraciones personalizadas. Algunos ejemplos incluyen la configuración de cómo acepta el demonio las solicitudes entrantes, las opciones de red predeterminadas y la configuración de registro y depuración. En Windows, estas configuraciones pueden especificarse en un archivo de configuración o mediante el Administrador de control de servicios de Windows. En este documento se detallará cómo instalar y configurar el demonio de Docker. Asimismo, se proporcionarán algunos ejemplos de configuraciones frecuentes.

## Instalar Docker

Para trabajar con contenedores de Windows es necesario Docker. Docker consta de motor y cliente. En este ejercicio se instalarán ambos.

Cree una carpeta para los ejecutables de Docker.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Descargue el demonio de Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Descargue el cliente de Docker.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

Agregue el directorio de Docker a la ruta de acceso del sistema. Una vez hecho, reinicie la sesión de PowerShell para que reconozca la ruta de acceso modificada.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Para instalar Docker como un servicio de Windows, ejecute lo siguiente.

```none
dockerd --register-service
```

Una vez instalado, puede iniciar el servicio.

```none
Start-Service Docker
```

Tendrán que instalarse imágenes de contenedor antes de poder usar Docker. Para obtener más información, vea [Administrar imágenes de contenedores](../management/manage_images.md).

## Archivo de configuración de Docker

El método preferido para configurar el demonio de Docker en Windows es usar un archivo de configuración. Puede encontrar el archivo de configuración en “c:\ProgramData\docker\config\daemon.json”. Si este archivo no existe, se puede crear.

Nota: no todas las opciones de configuración de Docker disponibles se pueden aplicar a Docker en Windows. El siguiente ejemplo muestra cuáles son. Para obtener documentación completa sobre la configuración del demonio de Docker, incluida para Linux, consulte [Demonio de Docker]( https://docs.docker.com/v1.10/engine/reference/commandline/daemon/).

```none
{
    "authorization-plugins": [],
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "storage-driver": "",
    "storage-opts": [],
    "labels": [],
    "log-driver": "", 
    "mtu": 0,
    "pidfile": "",
    "graph": "",
    "cluster-store": "",
    "cluster-advertise": "",
    "debug": true,
    "hosts": [],
    "log-level": "",
    "tlsverify": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "group": "",
    "default-ulimits": {},
    "bridge": "",
    "fixed-cidr": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "insecure-registries": [],
    "disable-legacy-registry": false
}
```

Solo se necesitan agregar los cambios de configuración deseados al archivo de configuración. Por ejemplo, en esta muestra se configura el demonio de Docker para que acepte las conexiones entrantes a través del puerto TCP 2375. Las demás opciones de configuración usarán los valores predeterminados.

```none
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

De la misma forma, en esta muestra se configura el demonio de Docker para aceptar únicamente conexiones seguras a través del puerto 2376.

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```



## administrador de control de servicios

El demonio de Docker también se puede configurar modificando el servicio de Docker con `sc config`. Con este método, se establecen marcas de demonio de Docker directamente en el servicio de Docker.


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

## Configuración común

Los siguientes ejemplos de archivos de configuración muestran configuraciones de Docker comunes. Esta configuración se puede combinar en un único archivo de configuración.

### Creación de red predeterminada 

Para configurar el demonio de Docker para que no se cree la red NAT predeterminada, use lo siguiente. Para obtener más información, vea [Administrar redes de Docker](../management/container_networking.md).

```none
{
    "bridge" : "none"
}
```

### Definir el grupo de seguridad de Docker

Si ha iniciado sesión en el host de Docker y ejecuta comandos de Docker de forma local, estos se ejecutan a través de una canalización con nombre. De forma predeterminada, solo los miembros del grupo de administradores pueden acceder al demonio de Docker a través de la canalización con nombre. Para especificar un grupo de seguridad que tiene este acceso, use la marca `group`.

```none
{
    "group" : "docker"
}
```


<!--HONumber=Jun16_HO2-->


