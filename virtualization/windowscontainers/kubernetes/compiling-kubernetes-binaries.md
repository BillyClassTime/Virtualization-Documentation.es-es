---
title: Compilar archivos binarios de Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Compilación y compilación cruzada de archivos binarios de Kubernetes desde el origen.
keywords: kubernetes, 1.12, linux, compilar
ms.openlocfilehash: 40bf7e65a8910cdab095abb269aa0a92508189cd
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178878"
---
# <a name="compiling-kubernetes-binaries"></a>Compilar archivos binarios de Kubernetes #
La compilación de Kubernetes requiere un entorno de trabajo Go. En esta página se muestran varias formas de compilación de archivos binarios de Linux y de compilación cruzada de archivos binarios de Windows.
> [!NOTE] 
> Esta página es voluntaria y solo incluye para desarrolladores de Kubernetes interesados que quieran experimentar con el código fuente más recientes y más amplio.

> [!tip]
> Para recibir notificaciones acerca de los avances más recientes puede suscribirse a [@kubernetes-announce](https://groups.google.com/forum/#!forum/kubernetes-announce).

## <a name="installing-go"></a>Instalar Go ##
Para hacerlo más sencillo, este proceso pasa por la instalación de Go en una ubicación temporal y personalizada:

```bash
cd ~
wget https://redirector.gvt1.com/edgedl/go/go1.11.1.linux-amd64.tar.gz -O go1.11.1.tar.gz
tar -vxzf go1.11.1.tar.gz
mkdir gopath
export GOROOT="$HOME/go"
export GOPATH="$HOME/gopath"
export PATH="$GOROOT/bin:$PATH"
```

> [!Note]  
> Estas opciones establecen las variables de entorno de tu sesión. Agrega `export`s a tu `~/.profile` para una configuración permanente.

Ejecuta `go env` para garantizar que se han establecido correctamente las rutas de acceso. Hay varias opciones para la compilación de archivos binarios de Kubernetes:

  - Compílalos [localmente](#build-locally).
  - Genera los archivos binarios con [Vagrant](#build-with-vagrant).
  - Saca partido de los [scripts de compilaciones en contenedores estándar](https://github.com/kubernetes/kubernetes/tree/master/build#key-scripts) del proyecto Kubernetes. Para ello, sigue los pasos para [compilar localmente](#build-locally) hasta los pasos de `make` y luego sigue las instrucciones vinculadas.

Para copiar archivos binarios de Windows en sus respectivas nodos, usa una herramienta visual como [WinSCP](https://winscp.net/eng/download.php) o una herramienta de línea de comandos como [pscp](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) para transferirlos al directorio `C:\k`.


## <a name="building-locally"></a>Compilar localmente ##
> [!Tip]  
> Si te encuentras con errores de "permiso denegado", estos pueden evitarse con la compilación de `kubelet` de Linux en primer lugar, según la nota indicada en [`acs-engine`](https://github.com/Azure/acs-engine/blob/master/scripts/build-windows-k8s.sh#L176):
>  
> _Debido a lo que parece un error en el sistema de compilación de Windows de Kubernetes, se debe compilar primero un archivo binario de Linux para generar `_output/bin/deepcopy-gen`. Compilar en Windows sin hacer este paso generará un `deepcopy-gen` vacío._

En primer lugar, recupera el repositorio de Kubernetes:

```bash
KUBEREPO="k8s.io/kubernetes"
go get -d $KUBEREPO
# Note: the above command may spit out a message about 
#       "no Go files in...", but it can be safely ignored!
cd $GOPATH/src/$KUBEREPO
```

Ahora echa un vistazo a múltiples versiones desde las que compilar y compila el archivo binario `kubelet` de Linux. Esto es necesario para evitar los errores de compilación de Windows indicados anteriormente. Aquí, usaremos `v1.12.2`. Después del `git checkout` puedes aplicar revisiones y PR pendientes, o realizar otras modificaciones en los archivos binarios personalizados.

```bash
git checkout tags/v1.12.2
make clean && make WHAT=cmd/kubelet
```

Por último, compila los archivos binarios de cliente Windows necesarios (el último paso puede variar, en función del lugar desde donde los archivos binarios de Windows deben recuperarse más adelante):

```bash
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubectl
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 make WHAT=cmd/kube-proxy
cp _output/local/bin/windows/amd64/kube*.exe ~/kube-win/
```

Los pasos para compilar archivos binarios de Linux son idénticos; tan solo debes omitir el prefijo `KUBE_BUILD_PLATFORMS=windows/amd64` de los comandos. En su lugar, el directorio de salida será `_output/.../linux/amd64`.


## <a name="build-with-vagrant"></a>Compilar con Vagrant ##
Hay una configuración de Vagrant disponible [aquí](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux/vagrant). Úsala para preparar una VM Vagrant y luego ejecuta estos comandos en su interior:

```bash
DIST_DIR="${HOME}/kube/"
SRC_DIR="${HOME}/src/k8s-main/"
mkdir ${DIST_DIR}
mkdir -p "${SRC_DIR}"

git clone https://github.com/kubernetes/kubernetes.git ${SRC_DIR}

cd ${SRC_DIR}
git checkout tags/v1.12.2
KUBE_BUILD_PLATFORMS=linux/amd64   build/run.sh make WHAT=cmd/kubelet
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kubelet 
KUBE_BUILD_PLATFORMS=windows/amd64 build/run.sh make WHAT=cmd/kube-proxy 
cp _output/dockerized/bin/windows/amd64/kube*.exe ${DIST_DIR}

ls ${DIST_DIR}
```

