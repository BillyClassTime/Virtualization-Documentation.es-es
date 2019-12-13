---
title: Unión de nodos de Linux
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Implementación de Kubernetes resoureces en un clúster Kubernetes de sistema operativo mixto.
keywords: kubernetes, 1,14, Windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: e6c569ae8d5bf50e24ea0fc7a6dd04734b60a863
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909965"
---
# <a name="deploying-kubernetes-resources"></a>Implementación de recursos de Kubernetes #
Suponiendo que tiene un clúster de Kubernetes que consta de al menos 1 maestro y un trabajo, está listo para implementar los recursos de Kubernetes.
> [!TIP] 
> ¿Qué recursos Kubernetes se admiten actualmente en Windows? Consulte [las características admitidas oficialmente](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) y [Kubernetes en el mapa de ruta de Windows](https://github.com/orgs/kubernetes/projects/8) para obtener más detalles.


## <a name="running-a-sample-service"></a>Ejecutar un servicio de ejemplo ##
Implementarás un [servicio web basado en PowerShell](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) muy sencillo para garantizar que has unido el clúster correctamente y que nuestra red está configurada correctamente.

Antes de hacerlo, siempre es una buena idea asegurarse de que todos los nodos están en buen estado.
```bash
kubectl get nodes
```

Si todo parece correcto, puede descargar y ejecutar el siguiente servicio:
> [!Important] 
> Antes de `kubectl apply`, asegúrese de comprobar o modificar la imagen de `microsoft/windowsservercore` del archivo de ejemplo en [una imagen de contenedor que se](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)pueda ejecutar en los nodos.

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Esto crea una implementación y un servicio. El último comando Watch consulta los pods indefinidamente para realizar el seguimiento de su estado; simplemente presione `Ctrl+C` para salir del comando `watch` cuando haya terminado de observar.

Si todo ha funcionado correctamente, podrás:

  - Consulte 2 contenedores por Pod en `docker ps` comando del nodo Windows.
  - ver 2 pods en un comando `kubectl get pods` desde el maestro de Linux
  - `curl` en las direcciones IP del *Pod* en el puerto 80 desde el maestro de Linux obtiene una respuesta del servidor Web; Esto muestra el nodo adecuado para establecer la comunicación a través de la red.
  - hacer ping *entre pods* (incluidos entre hosts, si tienes más de un nodo de Windows) a través de `docker exec`; esto demuestra la comunicación adecuada de pod a pod
  - `curl` la *dirección IP del servicio* virtual (que se presenta en `kubectl get services`) desde el maestro de Linux y desde los pods individuales; Esto muestra el servicio adecuado para la comunicación del Pod.
  - `curl` el *nombre del servicio* con el [sufijo DNS predeterminado](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)Kubernetes, mostrando la detección de servicios adecuada.
  - `curl` el *NodePort* del maestro de Linux o de las máquinas fuera del clúster; Esto muestra la conectividad entrante.
  - `curl` IP externas desde dentro del Pod; Esto muestra la conectividad saliente.

> [!Note]  
> Los *hosts de contenedor* de Windows **no** podrán tener acceso a la dirección IP del servicio desde los servicios programados en ellos. Se trata de una [limitación conocida](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) de la plataforma que se mejorará en versiones futuras a Windows Server. Sin embargo, los *pods* de **Windows pueden tener** acceso a la dirección IP del servicio.

## <a name="next-steps"></a>Pasos siguientes ##
En esta sección, se ha explicado cómo programar recursos Kubernetes en nodos de Windows. Esto concluye la guía. Si hay algún problema, consulte la sección de solución de problemas:

> [!div class="nextstepaction"]
> [Solución de problemas](./common-problems.md)

De lo contrario, puede que también le interese ejecutar componentes de Kubernetes como servicios de Windows:
> [!div class="nextstepaction"]
> [Servicios de Windows](./kube-windows-services.md)
