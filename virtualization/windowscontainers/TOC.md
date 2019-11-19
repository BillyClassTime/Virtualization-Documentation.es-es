# [Contenedores en la documentación de Windows](index.md) 

# Introducción
## [Acerca de los contenedores de Windows](about/index.md)
## [Contenedores frente a máquinas virtuales](about/containers-vs-vm.md)
## [Requisitos del sistema](deploy-containers/system-requirements.md)
## [Preguntas frecuentes](about/faq.md)

# Introducción
## [Configurar el entorno](quick-start/set-up-environment.md)
## [Ejecutar el primer contenedor](quick-start/run-your-first-container.md)
## [Descontenedorar una aplicación de muestra](quick-start/building-sample-app.md)

# Tutoriales
## Crear un contenedor de Windows
### [Escribir un Dockerfile](manage-docker/manage-windows-dockerfile.md)
### [Optimizar un Dockerfile](manage-docker/optimize-windows-dockerfile.md)
## Ejecutar en el servicio Kubernetes de Azure
### [Crear un clúster de contenedores de Windows en AKS](/azure/aks/windows-container-cli)
### [Limitaciones actuales](/azure/aks/windows-node-limitations)
## Ejecutar en Service fabric
### [Implementa tu primer contenedor.](/azure/service-fabric/service-fabric-quickstart-containers)
### [Implementar una aplicación .NET en un contenedor de Windows](/azure/service-fabric/service-fabric-host-app-in-a-container)
## Ejecutar en el servicio de aplicaciones de Azure
### [Inicio rápido de servicio de aplicaciones de Azure](/azure/app-service/app-service-web-get-started-windows-container)
### [Migrar una aplicación de ASP.NET con contenedores de Windows y el servicio de aplicaciones de Azure](/azure/app-service/app-service-web-tutorial-windows-containers-custom-fonts)
## Contenedores de Linux en Windows
### [Introducción](deploy-containers/linux-containers.md)
### [Ejecutar el primer contenedor LCOW](quick-start/quick-start-windows-10-linux.md)
## Usar contenedores con el programa Windows Insider
### [Introducción](deploy-containers/insider-overview.md)

# Conceptos
## Aspectos esenciales del contenedor de Windows
### [Imágenes base del contenedor](manage-containers/container-base-images.md)
### [Modos de aislamiento](manage-containers/hyperv-container.md)
### [Compatibilidad de versiones](deploy-containers/version-compatibility.md)
### [Controles de recursos](manage-containers/resource-controls.md)
## Docker
### [Docker Engine en Windows](manage-docker/configure-docker-daemon.md)
### [Administración remota de un host de acoplador de Windows](management/manage_remotehost.md)
## Orquestación de contenedor
### [Introducción](about/overview-container-orchestrators.md)
### Kubernetes en Windows
#### [Kubernetes en Windows](kubernetes/getting-started-kubernetes-windows.md)
#### [Crear un patrón de Kubernetes](kubernetes/creating-a-linux-master.md)
#### [Elegir una solución de red](kubernetes/network-topologies.md)
#### [Unirse a trabajadores de Windows](kubernetes/joining-windows-workers.md)
#### [Unirse a trabajadores de Linux](kubernetes/joining-linux-workers.md)
#### [Implementar recursos de Kubernetes](kubernetes/deploying-resources.md)
#### [Solución de problemas](kubernetes/common-problems.md)
#### [Kubernetes como servicio de Windows](kubernetes/kube-windows-services.md)
#### [Compilar binarios de Kubernetes](kubernetes/compiling-kubernetes-binaries.md)
### Fabric de servicio
#### [Fabric y contenedores de servicios](/azure/service-fabric/service-fabric-containers-overview)
#### [Gobierno de recursos](/azure/service-fabric/service-fabric-resource-governance)
### Swarm de acoplamiento
#### [Modo Swarm](manage-containers/swarm-mode.md)
## Cargas
### Cuentas de servicio administradas de grupo
#### [Crear una gMSA](manage-containers/manage-serviceaccounts.md)
#### [Configurar la aplicación para que use un gMSA](manage-containers/gmsa-configure-app.md)
#### [Ejecutar un contenedor con un gMSA](manage-containers/gmsa-run-container.md)
#### [Orquestar contenedores con un gMSA](manage-containers/gmsa-orchestrate-containers.md)
#### [Solución de problemas de gMSAs](manage-containers/gmsa-troubleshooting.md)
### [Servicios de impresora](deploy-containers/print-spooler.md)
## Redes
### [Introducción](container-networking/architecture.md)
### [Topologías y drivers de red](container-networking/network-drivers-topologies.md)
### [Aislamiento de redes y seguridad](container-networking/network-isolation-security.md)
### [Configurar opciones avanzadas de red](container-networking/advanced.md)
## Almacenamiento
### [Introducción](manage-containers/container-storage.md)
### [Almacenamiento persistente](manage-containers/persistent-storage.md)
## Dispositivos
### [Dispositivos de hardware](deploy-containers/hardware-devices-in-containers.md)
### [Aceleración de GPU](deploy-containers/gpu-acceleration.md)

# Referencia
## [Ciclos de vida de mantenimiento de imágenes básicos](deploy-containers/base-image-lifecycle.md)
## [Optimización antivirus](https://docs.microsoft.com/windows-hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)
## [Herramientas de plataforma de contenedor](deploy-containers/containerd.md)
## [CLUF de la imagen de sistema operativo](Images_EULA.md)

# Recursos
## [Muestras de contenedor](samples.md)
## [Solución de problemas](troubleshooting.md)
## [Foro de contenedores](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)
## [Vídeos y blogs de la comunidad](communitylinks.md)
