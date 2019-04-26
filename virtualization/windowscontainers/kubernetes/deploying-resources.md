---
title: Unirse a nodos de Linux
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Implementación de Kubernetes resoureces en un clúster de Kubernetes de sistemas operativos combinados.
keywords: kubernetes, 1.13, windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 7d2f1dd789a96a3ee4898ef196f872e574d6321f
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2019
ms.locfileid: "9574906"
---
# <a name="deploying-kubernetes-resources"></a>Implementación de recursos de Kubernetes #
Si que tienes un clúster de Kubernetes que consta de al menos 1 maestro y 1 trabajadores, estás listo para implementar los recursos de Kubernetes.
> [!TIP] 
> ¿Qué recursos de Kubernetes se admiten actualmente en Windows curiosidad? Consulte [admite oficialmente las características](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features) y [Kubernetes en la guía básica de Windows](https://trello.com/b/rjTqrwjl/windows-k8s-roadmap) para obtener más detalles.


## <a name="running-a-sample-service"></a>Ejecuta un servicio de muestra ##
Implementarás un [servicio web basado en PowerShell](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) muy sencillo para garantizar que has unido el clúster correctamente y que nuestra red está configurada correctamente.

Antes de hacerlo, siempre es una buena idea para asegurarse de que todos los nodos de nuestro están en buenas condiciones.
```bash
kubectl get nodes
```

Si todo parece correcto, puedes descargar y ejecutar el servicio de siguiente:
> [!Important] 
> Antes de `kubectl apply`, realizar seguro a doble-check o modificar la `microsoft/windowsservercore` imagen en el archivo de ejemplo para [una imagen de contenedor es runnable por los nodos](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)!

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Esto creará una implementación y un servicio. El último comando inspección consultas los pods indefinidamente para realizar un seguimiento de su estado; Simplemente presiona `Ctrl+C` para salir de la `watch` comando cuando hecho finalizado.

Si todo ha funcionado correctamente, podrás:

  - Ver 2 contenedores por pod bajo `docker ps` comando en el nodo de Windows
  - ver 2 pods en un comando `kubectl get pods` desde el maestro de Linux
  - `curl` en las IP de *pod* del puerto 80 del maestro de Linux que obtiene una respuesta del servidor web; esto demuestra el nodo adecuado a la comunicación de pod de la red.
  - hacer ping *entre pods* (incluidos entre hosts, si tienes más de un nodo de Windows) a través de `docker exec`; esto demuestra la comunicación adecuada de pod a pod
  - `curl` la *dirección IP de servicio* de virtual (visto en `kubectl get services`) del maestro de Linux y de pods individuales; Esto demuestra servicio adecuada para la comunicación de pod.
  - `curl` el *nombre del servicio* con el Kubernetes [sufijo DNS predeterminado](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services), que muestra la detección de servicios adecuada.
  - `curl` el *NodePort* desde el maestro de Linux o máquinas fuera del clúster. Esto demuestra la conectividad entrante.
  - `curl` direcciones IP externa desde dentro del pod; Esto demuestra la conectividad saliente.

> [!Note]  
> *Hosts de contenedor* de Windows te **no** podrá tener acceso a la dirección IP de servicio de servicios programados en ellos. Esto es una [limitación de plataforma conocida](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) que se mejorará en versiones futuras a Windows Server. Windows *pods* **son** capaces de obtener acceso a la dirección IP de servicio sin embargo.

### <a name="port-mapping"></a>Asignación de puertos ### 
También es posible tener acceso a servicios hospedados en pods a través de sus respectivos nodos mediante la asignación de un puerto en el nodo. Hay [otra muestra YAML disponible](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml) con una asignación de puerto 4444 en el nodo para el puerto 80 en el pod para demostrar esta característica. Para implementarlo, sigue los mismos pasos que antes:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

Ahora debería ser posible la acción `curl` en el IP del *nodo* del puerto 4444 y recibir una respuesta del servidor web. Ten en cuenta que esto limita el escalado a un solo pod por nodo ya que debe aplicar una asignación uno a uno.


## <a name="next-steps"></a>Pasos siguientes ##
En esta sección, hemos visto cómo programar los recursos de Kubernetes en nodos de Windows. Esto concluye a la guía. Si hay algún problema, revisa la sección de solución de problemas:

> [!div class="nextstepaction"]
> [Solución de problemas](./common-problems.md)

De lo contrario, también puede interesarte en marcha componentes de Kubernetes como servicios de Windows:
> [!div class="nextstepaction"]
> [Servicios de Windows](./kube-windows-services.md)