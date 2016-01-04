# PowerShell para contenedores

## Install-ContainerOSImage

**NOMBRE**      
Install-ContainerOSImage

**RESUMEN**  
Instala el archivo WIM especificado como una imagen del sistema operativo del contenedor para su uso con Windows Server o con contenedores de Hyper-V.


**SINTAXIS**
``` PowerShell  
Install-ContainerOSImage [-WimPath] <String> [-Force] [< CommonParameters >]
```

**DESCRIPCIÓN**  
Instala una imagen base a partir de un archivo WIM en el almacén de imágenes central compartido para la característica de contenedores de Hyper-V y Windows Server.

**PARÁMETROS**
``` PowerShell
    -WimPath <String>
        A path to the WIM file that will be installed.

        Required?                    true
        Position?                    1
        Default value
        Accept pipeline input?       false
        Accept wildcard characters?  false

    -Force [<SwitchParameter>]

        Required?                    false
        Position?                    named
        Default value                False
        Accept pipeline input?       false
        Accept wildcard characters?  false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**      
Ninguno

**SALIDAS**  
Ninguno

**ALIAS**  
Ninguno

-------------------------- EJEMPLO 1 --------------------------

``` PowerShell
PS C:\>Install-ContainerOSImage c:\baseimage.wim
```

## Uninstall-ContainerOSImage

**NOMBRE**  
Uninstall-ContainerOSImage

**RESUMEN**  
Quita una imagen de sistema operativo del contenedor instalada previamente

**SINTAXIS**
```PowerShell
Uninstall-ContainerOSImage [-ImageName] <string> [-Force]  [< CommonParameters >]

Uninstall-ContainerOSImage [-ContainerImage] <Object> [-Force]  [< CommonParameters >]
```

**PARÁMETROS**
``` PowerShell
    -ContainerImage <Object>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           ByContainerImage
        Aliases                      None
        Dynamic?                     false

    -Force

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -ImageName <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           ByName
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Ninguno


**SALIDAS**  
System.Object

**ALIAS**  
Ninguno

## Add-ContainerNetworkAdapter

**NOMBRE**  
Add-ContainerNetworkAdapter

**RESUMEN**  
Agrega un nuevo adaptador de red a un contenedor existente

**SINTAXIS**
``` PowerShell  
Add-ContainerNetworkAdapter [-ContainerName] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>]
    [-Credential <pscredential[]>] [-SwitchName <string>] [-Name <string>] [-DynamicMacAddress] [-StaticMacAddress
    <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Add-ContainerNetworkAdapter [-Container] <Container[]> [-SwitchName <string>] [-Name <string>]
    [-DynamicMacAddress] [-StaticMacAddress <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -DynamicMacAddress

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -StaticMacAddress <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -SwitchName <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```


**ENTRADAS**  
System.String\[\]  
Microsoft.Containers.PowerShell.Objects.Container\[\]


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**ALIAS**  
Ninguno

## Connect-ContainerNetworkAdapter

**NOMBRE**  
Connect-ContainerNetworkAdapter

**RESUMEN**  
Conecta un adaptador de red del contenedor a un conmutador virtual

**SINTAXIS**
``` PowerShell
    Connect-ContainerNetworkAdapter [-ContainerName] <string[]> [[-Name] <string[]>] [-SwitchName] <string>
    [-Passthru] [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>] [-WhatIf]
    [-Confirm]  [<CommonParameters>]

    Connect-ContainerNetworkAdapter [-NetworkAdapter] <ContainerNetworkAdapter[]> [-SwitchName] <string> [-Passthru]
    [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -NetworkAdapter <ContainerNetworkAdapter[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Object_SwitchName
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -SwitchName <string>

        Required?                    true
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Object_SwitchName, Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter\[\]


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**ALIAS**  
Ninguno

## Disconnect-ContainerNetworkAdapter

**NOMBRE**  
Disconnect-ContainerNetworkAdapter

**RESUMEN**  
Desconecta un adaptador de red del contenedor de un conmutador virtual

**SINTAXIS**
``` PowerShell
    Disconnect-ContainerNetworkAdapter [-ContainerName] <string[]> [[-Name] <string[]>] [-CimSession <CimSession[]>]
    [-ComputerName <string[]>] [-Credential <pscredential[]>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Disconnect-ContainerNetworkAdapter [-NetworkAdapter] <ContainerNetworkAdapter[]> [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -NetworkAdapter <ContainerNetworkAdapter[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           ResourceObject
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter\[\]


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**ALIAS**  
Ninguno

## Export-ContainerImage

**NOMBRE**  
Export-ContainerImage

**RESUMEN**  
Copia una imagen de contenedor fuera del almacén local

**SINTAXIS**
``` PowerShell
    Export-ContainerImage [[-Name] <string>] [-Path] <string> [[-Version] <version>] [-CimSession <CimSession[]>]
    [-ComputerName <string[]>] [-Credential <pscredential[]>] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]

    Export-ContainerImage [-Image] <ContainerImage[]> [-Path] <string> [-AsJob] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Image <ContainerImage[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Image Object
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Path <string>

        Required?                    true
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    false
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Microsoft.Containers.PowerShell.Objects.ContainerImage\[\]


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**ALIAS**  
Ninguno

## Get-Container

**NOMBRE**  
Get-Container

**RESUMEN**  
Enumera los contenedores del sistema actual

**SINTAXIS**
``` PowerShell
    Get-Container [[-Name] <string[]>] [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>]  [<CommonParameters>]

    Get-Container [[-Id] <guid>] [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>]  [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Id, Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Id, Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name, Id
        Aliases                      None
        Dynamic?                     false

    -Id <guid>

        Required?                    false
        Position?                    0
        Accept pipeline input?       true (ByValue, ByPropertyName)
        Parameter set name           Id
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    false
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
System.String\[\]  
System.Guid


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.Container


**ALIAS**  
Ninguno

## Get-ContainerHost

**NOMBRE**  
Get-ContainerHost

**RESUMEN**  
Obtiene el objeto host del host de contenedor

**SINTAXIS**
``` PowerShell
    Get-ContainerHost [[-ComputerName] <string[]>] [[-Credential] <pscredential[]>]  [<CommonParameters>]

    Get-ContainerHost [-CimSession] <CimSession[]>  [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByPropertyName)
        Parameter set name           CimSession
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           ComputerName
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    1
        Accept pipeline input?       true (ByValue)
        Parameter set name           ComputerName
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Microsoft.Management.Infrastructure.CimSession\[\]  
System.String\[\]  
System.Management.Automation.PSCredential\[]


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.ContainerHost


**ALIAS**  
Ninguno

## Get-ContainerImage

**NOMBRE**  
Get-ContainerImage

**RESUMEN**  
Enumera las imágenes de contenedores en el host del contenedor

**SINTAXIS**
``` PowerShell
Get-ContainerImage [[-Name] <string>] [[-Publisher] <string>] [[-Version] <version>] [-ChildOf <ContainerImage>]
[-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>]  [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -ChildOf <ContainerImage>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    false
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Ninguno


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**ALIAS**  
Ninguno

## Get-ContainerNetworkAdapter

**NOMBRE**  
Get-ContainerNetworkAdapter

**RESUMEN**  
Enumera los adaptadores de red asociados a un contenedor

**SINTAXIS**
``` PowerShell
    Get-ContainerNetworkAdapter [-ContainerName] <string[]> [[-Name] <string>] [-CimSession <CimSession[]>]
    [-ComputerName <string[]>] [-Credential <pscredential[]>]  [<CommonParameters>]

    Get-ContainerNetworkAdapter [-Container] <Container[]> [[-Name] <string>]  [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Microsoft.Containers.PowerShell.Objects.Container\[\]  
System.String\[\]


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**ALIAS**  
Ninguno

## Import-ContainerImage

**NOMBRE**  
Import-ContainerImage

**RESUMEN**  
Importa una imagen de contenedor que se exportó desde otro equipo

**SINTAXIS**
``` PowerShell
    Import-ContainerImage [-Path] <string> [-AsJob] [-CimSession <CimSession[]>] [-ComputerName <string[]>]
    [-Credential <pscredential[]>] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Path <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
System.String


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**ALIAS**  
Ninguno

## Move-ContainerImageRepository

**NOMBRE**  
Move-ContainerImageRepository

**RESUMEN**  
Cambia la ubicación donde se almacenan las imágenes de contenedores. Debe ser una ubicación en un disco local. Solo puede cambiarse cuando no haya imágenes presentes en el sistema.

**SINTAXIS**
``` PowerShell
    Move-ContainerImageRepository [-Path] <string> [-AsJob] [-Passthru] [-CimSession <CimSession[]>] [-ComputerName
    <string[]>] [-Credential <pscredential[]>] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Path <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Ninguno


**SALIDAS**  
Microsoft.HyperV.PowerShell.VMHost


**ALIAS**
Ninguno

## New-Container

**NOMBRE**  
New-Container

**RESUMEN**  
Crear un contenedor nuevo

**SINTAXIS**
``` PowerShell
    New-Container [[-Name] <string>] -ContainerImageName <string> [-ContainerImagePublisher <string>]
    [-ContainerImageVersion <version>] [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>] [-MemoryStartupBytes <long>] [-SwitchName <string>] [-Path <string>] [-AsJob] [-WhatIf]
    [-Confirm]  [<CommonParameters>]

    New-Container [[-Name] <string>] -ContainerImage <ContainerImage> [-MemoryStartupBytes <long>] [-SwitchName
    <string>] [-Path <string>] [-AsJob] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -ContainerImage <ContainerImage>

        Required?                    true
        Position?                    Named
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Image Object
        Aliases                      None
        Dynamic?                     false

    -ContainerImageName <string>

        Required?                    true
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -ContainerImagePublisher <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -ContainerImageVersion <version>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -MemoryStartupBytes <long>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers, Container Image Object
        Aliases                      None
        Dynamic?                     false

    -Path <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -SwitchName <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.Container


**ALIAS**  
Ninguno

## New-ContainerImage

**NOMBRE**  
New-ContainerImage

**RESUMEN**  
Crear una nueva imagen de contenedor a partir de un contenedor existente

**SINTAXIS**
``` PowerShell
    New-ContainerImage [-ContainerName] <string> [-Name] <string> [-Publisher] <string> [-Version] <version>
    [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>] [-WhatIf] [-Confirm]
    [<CommonParameters>]

    New-ContainerImage [-Container] <Container> [-Name] <string> [-Publisher] <string> [-Version] <version> [-WhatIf]
    [-Confirm]  [<CommonParameters>]

    New-ContainerImage [-ContainerId] <guid> [-Name] <string> [-Publisher] <string> [-Version] <version> [-CimSession
    <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Id, Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name, Container Id
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerId <guid>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Container Id
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name, Container Id
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    true
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    true
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    true
        Position?                    3
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Microsoft.Containers.PowerShell.Objects.Container


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**ALIAS**  
Ninguno

## Remove-Container

**NOMBRE**  
Remove-Container

**RESUMEN**  
Quitar un contenedor existente del sistema

**SINTAXIS**
``` PowerShell
    Remove-Container [-Name] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>] [-Force] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Remove-Container [-Container] <Container[]> [-Force] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Force

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
System.String\[\]  
Microsoft.Containers.PowerShell.Objects.Container\[\]


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.Container


**ALIAS**  
Ninguno

## Remove-ContainerImage

**NOMBRE**  
Remove-ContainerImage

**RESUMEN**  
Quitar una imagen de contenedor del host del contenedor

**SINTAXIS**
``` PowerShell
    Remove-ContainerImage [[-Name] <string>] [[-Publisher] <string>] [[-Version] <version>] [-CimSession
    <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>] [-Force] [-WhatIf] [-Confirm]
    [<CommonParameters>]

    Remove-ContainerImage [-Image] <ContainerImage> [-Force] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Force

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Image <ContainerImage>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Image Object
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    false
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**SALIDAS**  
System.Object

**ALIAS**  
Ninguno

## Remove-ContainerNetworkAdapter

**NOMBRE**  
Remove-ContainerNetworkAdapter

**RESUMEN**  
Quita un adaptador de red de un contenedor

**SINTAXIS**
``` PowerShell
    Remove-ContainerNetworkAdapter [-ContainerName] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>]
    [-Credential <pscredential[]>] [-Name <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Remove-ContainerNetworkAdapter [-NetworkAdapter] <ContainerNetworkAdapter[]> [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]

    Remove-ContainerNetworkAdapter [-Container] <Container[]> [-Name <string>] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name, Container Object
        Aliases                      None
        Dynamic?                     false

    -NetworkAdapter <ContainerNetworkAdapter[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           ResourceObject
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter\[\]  
System.String\[\]  
Microsoft.Containers.PowerShell.Objects.Container\[\]


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**ALIAS**  
Ninguno

## Set-ContainerNetworkAdapter

**NOMBRE**  
Set-ContainerNetworkAdapter

**RESUMEN**  
Establece la dirección MAC en un adaptador de red de un contenedor

**SINTAXIS**
``` PowerShell
    Set-ContainerNetworkAdapter [-ContainerName] <string> [-CimSession <CimSession[]>] [-ComputerName <string[]>]
    [-Credential <pscredential[]>] [-Name <string>] [-DynamicMacAddress] [-StaticMacAddress <string>] [-Passthru]
    [-WhatIf] [-Confirm]  [<CommonParameters>]

    Set-ContainerNetworkAdapter [-NetworkAdapter] <ContainerNetworkAdapter> [-DynamicMacAddress] [-StaticMacAddress
    <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Set-ContainerNetworkAdapter [-Container] <Container> [-Name <string>] [-DynamicMacAddress] [-StaticMacAddress
    <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -DynamicMacAddress

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           ResourceObject, Container Object, Container Name
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Object, Container Name
        Aliases                      None
        Dynamic?                     false

    -NetworkAdapter <ContainerNetworkAdapter>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           ResourceObject
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -StaticMacAddress <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           ResourceObject, Container Object, Container Name
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
System.String  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter  
Microsoft.Containers.PowerShell.Objects.Container


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**ALIAS**  
Ninguno

## Start-Container

**NOMBRE**  
Start-Container

**RESUMEN**  
Inicia un contenedor

**SINTAXIS**
``` PowerShell
    Start-Container [-Name] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Start-Container [-Container] <Container[]> [-AsJob] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Microsoft.Containers.PowerShell.Objects.Container\[\]  
System.String\[\]


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.Container


**ALIAS**  
Ninguno

## Stop-Container

**NOMBRE**  
Stop-Container

**RESUMEN**  
Detiene un contenedor

**SINTAXIS**
``` PowerShell
    Stop-Container [-Name] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>] [-TurnOff] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Stop-Container [-Container] <Container[]> [-TurnOff] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -TurnOff

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Microsoft.Containers.PowerShell.Objects.Container\[\]  
System.String\[\]


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.Container


**ALIAS**  
Ninguno

## Test-ContainerImage

**NOMBRE**  
Test-ContainerImage

**RESUMEN**  
Validar una imagen de contenedor en el sistema de host del contenedor

**SINTAXIS**
``` PowerShell
    Test-ContainerImage [[-Name] <string>] [[-Publisher] <string>] [[-Version] <version>] [-CimSession <CimSession[]>]
    [-ComputerName <string[]>] [-Credential <pscredential[]>] [-AsJob] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Test-ContainerImage [-Image] <ContainerImage> [-AsJob] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**PARÁMETROS**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Image <ContainerImage>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Image Object
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    false
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**ENTRADAS**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**SALIDAS**  
Microsoft.Containers.PowerShell.Objects.ContainerImageReport


**ALIAS**  
Ninguno



