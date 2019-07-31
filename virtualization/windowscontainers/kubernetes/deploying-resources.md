---
title: Unirse a nodos de Linux
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Implementación de Kubernetes resoureces en un clúster de Kubernetes de sistema operativo mixto.
keywords: kubernetes, 1,14, Windows, introducción
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 8c21581433f672a22a247db6643a19168eedea6c
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883198"
---
# <a name="deploying-kubernetes-resources"></a>Implementación de recursos de Kubernetes #
Suponiendo que tiene un clúster de Kubernetes que consta de al menos 1 maestro y 1 trabajador, está listo para implementar los recursos de Kubernetes.
> [!TIP] 
> ¿Quiere saber qué recursos de Kubernetes se admiten hoy en Windows? Para obtener más información, consulta [las características](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) y Kubernetess oficialmente admitidas [en la guía básica de Windows](https://github.com/orgs/kubernetes/projects/8) .


## <a name="running-a-sample-service"></a>Ejecutar un servicio de ejemplo ##
Implementarás un [servicio web basado en PowerShell](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) muy sencillo para garantizar que has unido el clúster correctamente y que nuestra red está configurada correctamente.

Antes de hacerlo, siempre es una buena idea asegurarse de que todos nuestros nodos estén en buen estado.
```bash
kubectl get nodes
```

Si todo es correcto, puede descargar y ejecutar el siguiente servicio:
> [!Important] 
> Antes `kubectl apply`, asegúrese de volver a activar/modificar la `microsoft/windowsservercore` imagen en el archivo de ejemplo en [una imagen de contenedor que pueda ejecutar un nodo](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions).

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Esto crea una implementación y un servicio. El último comando Watch consulta a los pods de forma indefinida para hacer un seguimiento de su estado; simplemente pulse `Ctrl+C` para salir del `watch` comando cuando haya acabado de observar.

Si todo ha funcionado correctamente, podrás:

  - ver 2 contenedores por Pod bajo `docker ps` comando en el nodo de Windows
  - ver 2 pods en un comando `kubectl get pods` desde el maestro de Linux
  - `curl` en las IP de *pod* del puerto 80 del maestro de Linux que obtiene una respuesta del servidor web; esto demuestra el nodo adecuado a la comunicación de pod de la red.
  - hacer ping *entre pods* (incluidos entre hosts, si tienes más de un nodo de Windows) a través de `docker exec`; esto demuestra la comunicación adecuada de pod a pod
  - `curl` el *IP de servicio* virtual (visto `kubectl get services`en) desde el maestro de Linux y desde los pods individuales; Esto muestra el servicio adecuado para la comunicación del Pod.
  - `curl` el *nombre del servicio* con el [sufijo DNS predeterminado](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)Kubernetes, que muestra la detección de servicios adecuada.
  - `curl` el *NodePort* del maestro Linux o de las máquinas fuera del clúster; Esto muestra la conectividad entrante.
  - `curl` IP externas desde dentro de la caja Pod; Esto demuestra la conectividad saliente.

> [!Note]  
> Los *hosts contenedores* de Windows **no** podrán acceder a la dirección IP del servicio desde los servicios programados en ellas. Esta es una [limitación de plataforma conocida](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) que se mejorará en versiones futuras a Windows Server. Sin embargo, los *pods* **** de Windows pueden acceder a la IP del servicio.

### <a name="port-mapping"></a>Asignación de puertos ### 
También es posible tener acceso a servicios hospedados en pods a través de sus respectivos nodos mediante la asignación de un puerto en el nodo. Hay [otra muestra YAML disponible](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml) con una asignación de puerto 4444 en el nodo para el puerto 80 en el pod para demostrar esta característica. Para implementarlo, sigue los mismos pasos que antes:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

Ahora debería ser posible la acción `curl` en el IP del *nodo* del puerto 4444 y recibir una respuesta del servidor web. Ten en cuenta que esto limita el escalado a un solo pod por nodo ya que debe aplicar una asignación uno a uno.


## <a name="next-steps"></a>Pasos siguientes ##
En esta sección, hemos explicado cómo programar recursos de Kubernetes en nodos de Windows. Esto concluye la guía. Si ha habido problemas, revise la sección de solución de problemas:

> [!div class="nextstepaction"]
> [Solución de problemas](./common-problems.md)

De lo contrario, también puede estar interesado en ejecutar componentes de Kubernetes como servicios de Windows:
> [!div class="nextstepaction"]
> [Servicios de Windows](./kube-windows-services.md)
