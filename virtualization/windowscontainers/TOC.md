# [Contenedores en la documentación de Windows](index.md) 

# Introducción
## [Acerca de los contenedores de Windows](about/index.md)
## [Requisitos del sistema](deploy-containers/system-requirements.md)
## [Preguntas más frecuentes](about/faq.md)

# Introducción
## Windows 10
### [Ejecutar el primer contenedor WCOW](quick-start/quick-start-windows-10.md)
### [Crear una aplicación de ejemplo](quick-start/building-sample-app.md)
## Windows Server
### [Ejecutar el primer contenedor de Windows Server](quick-start/quick-start-windows-server.md)
### [Automatizar las compilaciones de contenedores](quick-start/quick-start-images.md)
## WindowsInsider
### [Usar las imágenes de Insider](quick-start/Using-Insider-Container-Images.md)
### [Crear y ejecutar una aplicación](quick-start/Nano-RS3-.NET-Core-and-PS.md)

# Tutoriales
## Crear un contenedor de Windows
### [Escribir un Dockerfile](manage-docker/manage-windows-dockerfile.md)
### [Optimizar un Dockerfile](manage-docker/optimize-windows-dockerfile.md)
## Kubernetes en Windows
### [Kubernetes en Windows](kubernetes/getting-started-kubernetes-windows.md)
### [Crear un patrón de Kubernetes](kubernetes/creating-a-linux-master.md)
### [Elegir una solución de red](kubernetes/network-topologies.md)
### [Unirse a trabajadores de Windows](kubernetes/joining-windows-workers.md)
### [Unirse a trabajadores de Linux](kubernetes/joining-linux-workers.md)
### [Implementar recursos de Kubernetes](kubernetes/deploying-resources.md)
### [Solución de problemas](kubernetes/common-problems.md)
### [Servicios de Windows en Kubernetes](kubernetes/kube-windows-services.md)
### [Compilar binarios de Kubernetes](kubernetes/compiling-kubernetes-binaries.md)
## Service fabric en Windows
### [Implementa tu primer contenedor.](/azure/service-fabric/service-fabric-quickstart-containers)
### [Implementar una aplicación .NET en un contenedor de Windows](/azure/service-fabric/service-fabric-host-app-in-a-container)
## Contenedores de Linux en Windows
### [Introducción](deploy-containers/linux-containers.md)
### [Ejecutar el primer contenedor LCOW](quick-start/quick-start-windows-10-linux.md)

# Conceptos
## Aspectos esenciales del contenedor de Windows
### [Controles de recursos](manage-containers/resource-controls.md)
### [Aislamiento de Hyper-V](manage-containers/hyperv-container.md)
### [Compatibilidad de versiones](deploy-containers/version-compatibility.md)
### [Imágenes base del contenedor](manage-containers/container-base-images.md)
## Docker
### [Docker Engine en Windows](manage-docker/configure-docker-daemon.md)
### [Swarm de acoplamiento](manage-containers/swarm-mode.md)
### [Administración remota de un host de acoplador de Windows](management/manage_remotehost.md)
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