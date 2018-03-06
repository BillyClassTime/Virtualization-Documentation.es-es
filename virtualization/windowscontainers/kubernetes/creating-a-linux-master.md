---
title: Maestro de Kubernetes desde cero
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "Crear un maestro de clúster de Kubernetes desde cero."
keywords: kubernetes, 1.9, maestro, linux
ms.openlocfilehash: 3ea338f7af3dd921731fce0ec5a8b2cf8c4fef0c
ms.sourcegitcommit: f542e8c95b5bb31b05b7c88f598f00f76779b519
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/01/2018
---
# <a name="kubernetes-master--from-scratch"></a>Maestro de Kubernetes desde cero #
Esta página te guiará a través de una implementación manual de un maestro de Kubernetes de principio a fin.

Se necesita una máquina Linux, similar a Ubuntu, recientemente actualizada. Windows no se incluye en absoluto en la imagen; los archivos binarios se compilan desde Linux.


> [!Warning]  
> Dada la volatilidad de Kubernetes según la versión, esta guía puede asumir que no son verdaderos en el futuro.


## <a name="preparing-the-master"></a>Preparar el maestro ##
En primer lugar, instala todos los requisitos previos:

```bash
sudo apt-get install curl git build-essential docker.io conntrack python2.7
```

Si estás conectado a un servidor proxy, define las variables de entorno para la sesión actual:
```bash
HTTP_PROXY=http://proxy.example.com:80/
HTTPS_PROXY=http://proxy.example.com:443/
http_proxy=http://proxy.example.com:80/
https_proxy=http://proxy.example.com:443/
```
O si quieres establecer esta configuración de forma permanente, añade las variables al /etc./entorno (para aplicar los cambios es necesario cerrar sesión e iniciarla de nuevo).

Hay una colección de scripts en [este repositorio](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux), que ayudarán en el proceso de configuración. Echa un vistazo a `~/kube/`; todo el directorio se montará para muchos de los contenedores de Docker en los pasos futuros, así que mantén su estructura de la misma forma que se ha indicado en la guía.

```bash
mkdir ~/kube
mkdir ~/kube/bin
git clone https://github.com/Microsoft/SDN /tmp/k8s 
cd /tmp/k8s/Kubernetes/linux
chmod -R +x *.sh
chmod +x manifest/generate.py
mv * ~/kube/
```


### <a name="installing-the-linux-binaries"></a>Instalar los archivos binarios de Linux ###

> [!Note]  
> Para incluir las revisiones o usar el código de Kubernetes de vanguardia en lugar de descargar archivos binarios integrados previamente, consulta [esta página](./compiling-kubernetes-binaries.md).

Descarga e instala los archivos binarios oficiales de Linux en la [línea principal de Kubernetes](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1) e instálalos de la siguiente manera:

```bash
wget -O kubernetes.tar.gz https://github.com/kubernetes/kubernetes/releases/download/v1.9.1/kubernetes.tar.gz
tar -vxzf kubernetes.tar.gz 
cd kubernetes/cluster 
# follow the prompts from this command, the defaults are generally fine:
./get-kube-binaries.sh
cd ../server
tar -vxzf kubernetes-server-linux-amd64.tar.gz 
cd kubernetes/server/bin
cp hyperkube kubectl ~/kube/bin/
```

Agrega los archivos binarios a `$PATH`, de modo que podemos ejecutarlos en cualquier lugar. Ten en cuenta que esto solo establece la ruta de acceso de la sesión; agrega esta línea a `~/.profile` para una configuración permanente.

```bash
$ PATH="$HOME/kube/bin:$PATH"
```

### <a name="install-cni-plugins"></a>Instalar complementos CNI ###
Los complementos CNI básicos son necesarios para redes Kubernetes. Se pueden descargar [aquí](https://github.com/containernetworking/plugins/releases) y deben extraerse en `/opt/cni/bin/`:

```bash
DOWNLOAD_DIR="${HOME}/kube/cni-plugins"
CNI_BIN="/opt/cni/bin/"
mkdir ${DOWNLOAD_DIR}
cd $DOWNLOAD_DIR
curl -L $(curl -s https://api.github.com/repos/containernetworking/plugins/releases/latest | grep browser_download_url | grep 'amd64.*tgz' | head -n 1 | cut -d '"' -f 4) -o cni-plugins-amd64.tgz
tar -xvzf cni-plugins-amd64.tgz
sudo mkdir -p ${CNI_BIN}
sudo cp -r !(*.tgz) ${CNI_BIN}
ls ${CNI_BIN}
```


### <a name="certificates"></a>Certificados ###
En primer lugar, adquiere la dirección IP local, ya sea a través de `ifconfig` o:

```bash
$ ip addr show dev eth0
```

si el nombre de interfaz es conocido. Se mencionará en muchas ocasiones durante todo este proceso; configurarlo en una variable de entorno simplifica mucho las cosas. El siguiente fragmento de código lo establece temporalmente; si la sesión finaliza o el shell se cierra, debe volver a configurarse.

```bash
$ MASTER_IP=10.123.45.67   # example! replace
```

Prepara los certificados que se usarán para que los nodos se comuniquen en el clúster:

```bash
cd ~/kube/certs
chmod u+x generate-certs.sh
./generate-certs.sh $MASTER_IP
```

### <a name="prepare-manifests--addons"></a>Preparar los manifiestos y complementos ###
Genera un conjunto de archivos YAML que especifican pods del sistema Kubernetes pasando la dirección IP del maestro y CIDR de clúster *completo* al script Python en la carpeta `manifest`:

```bash
cd ~/kube/manifest
./generate.py $MASTER_IP --cluster-cidr 192.168.0.0/16
```

Vuelve a mover el script Python para que Kubernetes no lo confunda por un manifiesto; esto provocará problemas más adelante si no se hace.

> [!Important]  
> Si la versión de Kubernetes difiere de esta guía, usa los indicadores de control de versiones del scritp (como por ejemplo, `--api-version`) para [personalizar la imagen](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL/hyperkube-amd64) que los pods implementan. No todos los manifiestos usan la misma imagen y tienen distintos esquemas de control de versiones (en especial, `etcd` y el administrador de complementos).


#### <a name="manifest-customization"></a>Personalización de manifiestos ####
En este punto, pueden ser convenientes los cambios específicos de la configuración. Por ejemplo, puede haber una necesidad de asignar manualmente subredes a nodos, en lugar de permitirles administrarse mediante Kubernetes automáticamente. Esta configuración específica tiene una opción en el script (consulta `--help` para ver una explicación del parámetro `--im-sure`):

```bash
./generate.py $MASTER_IP --im-sure
```

Las demás opciones de configuración personalizadas requerirán una modificación manual de los manifiestos generados.


### <a name="configure--run-kubernetes"></a>Configurar y ejecutar Kubernetes ###
Configura Kubernetes para usar los certificados generados. Esto creará una configuración en `~/.kube/config`:

```bash
cd ~/kube
./configure-kubectl.sh $MASTER_IP
```

Ahora, copia el archivo donde se espera que los pods estarán más adelante:

```bash
mkdir ~/kube/kubelet
sudo cp ~/.kube/config ~/kube/kubelet/
```

El "cliente" de Kubernetes, `kubelet`, está listo para iniciarse. Los scripts siguientes se ejecutan indefinidamente; abre otra sesión de terminal después de cada uno de ellos para seguir funcionando:

```bash
cd ~/kube
sudo ./start-kubelet.sh
```

Ejecuta el script Kubeproxy, pasando el clúster parcial CIDR:

```bash
cd ~/kube
sudo ./start-kubeproxy.sh 192.168
```


> [!Important]  
> Será el 16 CIDR/esperado *completo* en el que se incluirán los nodos, *aunque haya tráfico que no proceda de Kubernetes en dicho CIDR.* Kubeproxy *solo* se aplica al tráfico de Kubernetes a la subred de *servicio*, por lo que no debe interferir con el tráfico de otros hosts.

> [!Note]  
> Estos scripts pueden ejecutarse como un demonio. Esta guía solo cubre su ejecución manual, ya que es muy útil durante la instalación para detectar errores.


## <a name="verifying-the-master"></a>Comprobar el maestro ##
Después de unos minutos, el sistema debe estar en el estado siguiente:

  - En `docker ps`, habrá ~23 contenedores de trabajo y de pod.
  - Si llamas a `kubectl cluster-info`, mostrará información sobre el servidor de API del maestro de Kubernetes, además de complementos DNS y Heapster.
  - `ifconfig` se mostrará una nueva interfaz `cbr0` con el CIDR del clúster elegido.

