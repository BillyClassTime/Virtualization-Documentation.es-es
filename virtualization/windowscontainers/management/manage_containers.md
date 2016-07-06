---
author: neilpeterson
redirect_url: ../quick_start/manage_docker
translationtype: Human Translation
ms.sourcegitcommit: 2b85875eae1dcf1e50162e69c53dbf1ac7463450
ms.openlocfilehash: 8921cbd910bf657ddc4998e4214c1e9f9c3a01e9

---

# Administración de contenedores de Windows Server

**Esto es contenido preliminar y está sujeto a cambios.** 

El ciclo de vida del contenedor incluye acciones tales como iniciar, detener y quitar contenedores. Al realizar estas acciones, puede que también tenga que recuperar una lista de imágenes de contenedores, administrar redes de contenedores y limitar recursos de contenedores. En este documento se detallan las tareas básicas de administración de contenedores mediante Docker y también se incluyen vínculos a artículos detallados. 

## Administración de contenedores

### Crear un contenedor

Use `docker run` para crear un contenedor con Docker.

```none
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Para más información sobre el comando Docker `run`, consulte [Docker run reference]( https://docs.docker.com/engine/reference/run/) (Referencia de Docker run).

### Detiene un contenedor

Use el comando `docker stop` para detener un contenedor con Docker.

```none
PS C:\> docker stop tender_panini

tender_panini
```

Este ejemplo detiene todos los contenedores en ejecución con Docker.

```none
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### Quitar contenedor

Para quitar un contenedor con Docker, use el comando `docker rm`.

```none
PS C:\> docker rm prickly_pike

prickly_pike
``` 

Para quitar todos los contenedores con Docker.

```none
PS C:\> docker rm $(docker ps -aq)

dc3e282c064d
2230b0433370
```

Para más información sobre el comando docker rm, consulte la [referencia de docker rm](https://docs.docker.com/engine/reference/commandline/rm/).



<!--HONumber=Jun16_HO4-->


