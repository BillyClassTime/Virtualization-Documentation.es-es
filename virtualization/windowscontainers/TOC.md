# [Documentación de contenedores en Windows](index.md) 

# Introducción
## [Acerca de los contenedores de Windows](about/index.md)
## [Contenedores contra máquinas virtuales](about/containers-vs-vm.md)
## [Requisitos del sistema](deploy-containers/system-requirements.md)
## [Preguntas más frecuentes](about/faq.md)

# Comenzar
## [Configurar el entorno](quick-start/set-up-environment.md)
## [Ejecutar su primer contenedor](quick-start/run-your-first-container.md)
## [Incluir una aplicación de ejemplo en un contenedor](quick-start/building-sample-app.md)

# Tutoriales
## Crear un contenedor de Windows
### [Escribir un archivo Dockerfile](manage-docker/manage-windows-dockerfile.md)
### [Optimizar un archivo Dockerfile](manage-docker/optimize-windows-dockerfile.md)
## Ejecutar en Azure Kubernetes Service
### [Crear un clúster de contenedores de Windows en AKS](/azure/aks/windows-container-cli)
### [Limitaciones actuales](/azure/aks/windows-node-limitations)
## Ejecutar en Service Fabric
### [Implementar su primer contenedor](/azure/service-fabric/service-fabric-quickstart-containers)
### [Implementar una aplicación .NET en un contenedor de Windows](/azure/service-fabric/service-fabric-host-app-in-a-container)
## Ejecutar en Azure App Service
### [Inicio rápido de Azure App Service](/azure/app-service/app-service-web-get-started-windows-container)
### [Migración de una aplicación de ASP.NET con contenedores de Windows y Azure App Service](/azure/app-service/app-service-web-tutorial-windows-containers-custom-fonts)
## Contenedores de Linux en Windows
### [Introducción](deploy-containers/linux-containers.md)
### [Ejecutar su primer contenedor LCOW](quick-start/quick-start-windows-10-linux.md)
## Usar contenedores con el Programa Windows Insider
### [Introducción](deploy-containers/insider-overview.md)

# Conceptos
## Conceptos básicos de los contenedores de Windows
### [Imágenes base de contenedor](manage-containers/container-base-images.md)
### [Modos de aislamiento](manage-containers/hyperv-container.md)
### [Compatibilidad de versiones](deploy-containers/version-compatibility.md)
### [Controles de recursos](manage-containers/resource-controls.md)
## Docker
### [Docker Engine en Windows](manage-docker/configure-docker-daemon.md)
### [Administración remota de un host de Windows Docker](management/manage_remotehost.md)
## Orquestación de contenedores
### [Introducción](about/overview-container-orchestrators.md)
### Kubernetes en Windows
#### [Kubernetes en Windows](kubernetes/getting-started-kubernetes-windows.md)
#### [Crear un maestro de Kubernetes](kubernetes/creating-a-linux-master.md)
#### [Elegir una solución de red](kubernetes/network-topologies.md)
#### [Unir trabajos de Windows](kubernetes/joining-windows-workers.md)
#### [Unir trabajos de Linux](kubernetes/joining-linux-workers.md)
#### [Implementar recursos de Kubernetes](kubernetes/deploying-resources.md)
#### [Solución de problemas](kubernetes/common-problems.md)
#### [Kubernetes como un servicio de Windows](kubernetes/kube-windows-services.md)
#### [Compilar archivos binarios de Kubernetes](kubernetes/compiling-kubernetes-binaries.md)
### Service Fabric
#### [Service Fabric y contenedores](/azure/service-fabric/service-fabric-containers-overview)
#### [Gobernanza de recursos](/azure/service-fabric/service-fabric-resource-governance)
### Docker Swarm
#### [Modo Swarm](manage-containers/swarm-mode.md)
## Cargas de trabajo
### Cuentas de servicio administradas de grupo
#### [Crear una gMSA](manage-containers/manage-serviceaccounts.md)
#### [Configurar la aplicación para usar una gMSA](manage-containers/gmsa-configure-app.md)
#### [Ejecutar un contenedor con una gMSA](manage-containers/gmsa-run-container.md)
#### [Orquestar contenedores con una gMSA](manage-containers/gmsa-orchestrate-containers.md)
#### [Solucionar problemas de gMSA](manage-containers/gmsa-troubleshooting.md)
### [Servicios de impresora](deploy-containers/print-spooler.md)
## Funciones de red
### [Introducción](container-networking/architecture.md)
### [Topologías y controladores de redes](container-networking/network-drivers-topologies.md)
### [Aislamiento y seguridad de red](container-networking/network-isolation-security.md)
### [Configurar opciones redes avanzadas](container-networking/advanced.md)
## Almacenamiento
### [Introducción](manage-containers/container-storage.md)
### [Almacenamiento persistente](manage-containers/persistent-storage.md)
## Dispositivos
### [Dispositivos de hardware](deploy-containers/hardware-devices-in-containers.md)
### [Aceleración de GPU](deploy-containers/gpu-acceleration.md)

# Referencia
## [Ciclos de vida de mantenimiento de imágenes base](deploy-containers/base-image-lifecycle.md)
## [Optimización de antivirus](https://docs.microsoft.com/windows-hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)
## [Herramientas de plataforma de contenedor](deploy-containers/containerd.md)
## [Términos de licencia de la imagen de sistema operativo del contenedor](Images_EULA.md)

# Recursos
## [Ejemplos de contenedor](samples.md)
## [Solución de problemas](troubleshooting.md)
## [Foro acerca de los contenedores](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)
## [Blogs y vídeos de la comunidad](communitylinks.md)
