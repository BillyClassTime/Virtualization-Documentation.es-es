# Interoperabilidad de administración

**Esto es contenido preliminar y está sujeto a cambios.**

En su mayor parte, los contenedores de Windows creados con PowerShell deben administrarse con PowerShell y los creados con Docker deben administrarse con Docker. Dicho eso, el módulo de PowerShell de informática de host ofrece la capacidad de detectar y detener contenedores **en ejecución**, independientemente de cómo se hayan creado. Este módulo funciona como un "administrador de tareas" para los contenedores que se ejecutan en un host de contenedor.

## Mostrar todos los contenedores

Para devolver una lista de contenedores, use el comando `Get-ComputeProcess`.

```powershell
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## Detener un contenedor

Para detener un contenedor independientemente de si se creó con PowerShell o Docker, use el comando `Stop-ComputeProcess`.

> En el momento de la escritura, el servicio de VMMS deberá reiniciarse para que los contenedores se muestren como detenidos cuando se use el comando `Get-Container`.

```powershell
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```




