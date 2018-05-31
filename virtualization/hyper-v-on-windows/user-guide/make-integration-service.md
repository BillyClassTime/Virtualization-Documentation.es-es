---
title: Cree sus propios servicios de integración
description: Servicios de integración de Windows 10.
keywords: windows 10, hyper-v, HVSocket, AF_HYPERV
author: scooley
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 1ef8f18c-3d76-4c06-87e4-11d8d4e31aea
ms.openlocfilehash: 966ca3ff267e03e8c380391281c8dde723e4b1dd
ms.sourcegitcommit: f1a08252087e72791ac4d12517c02e39f41c33f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2018
ms.locfileid: "1723641"
---
# <a name="make-your-own-integration-services"></a>Cree sus propios servicios de integración

A partir de la Actualización de aniversario de Windows10, cualquiera puede crear las aplicaciones que se comuniquen entre el host de Hyper-V y sus máquinas virtuales con sockets de Hyper-V: un socket de Windows con una nueva familia de direcciones y un punto de conexión especializado para las máquinas virtuales de destino.  Todas las comunicaciones mediante sockets de Hyper-V se ejecutan sin usar redes y todos los datos permanecen en la misma memoria física.   Las aplicaciones que usan sockets de Hyper-V son similares a los servicios de integración de Hyper-V.

Este documento explica paso a paso la creación de un programa simple basado en sockets de Hyper-V.

**Sistemas operativos host admitidos**
* Windows 10 y versiones posteriores
* Windows Server 2016 y versiones posteriores

**Sistemas operativos invitado admitidos**
* Windows 10 y versiones posteriores
* Windows Server 2016 y versiones posteriores
* Invitados de Linux con servicios de integración de Linux (consulte [Supported Linux and FreeBSD virtual machines for Hyper-V on Windows](https://technet.microsoft.com/library/dn531030.aspx) [Máquinas virtuales de Linux y FreeBSD soportadas para Hyper-V en Windows])
> **Nota:** Un invitado Linux compatible debe ser compatible con kernel en:
> ```bash
> CONFIG_VSOCKET=y
> CONFIG_HYPERV_VSOCKETS=y
> ```

**Capacidades y limitaciones**
* Admite acciones de modo de usuario o modo kernel
* Solo el flujo de datos
* Sin memoria de bloque (no se recomienda para copias de seguridad ni vídeo)

--------------

## <a name="getting-started"></a>Introducción

Requisitos:
* Compilador de C/C++.  Si no tienes ninguno, echa un vistazo a la [comunidad de Visual Studio](https://aka.ms/vs).
* [SDK de Windows10](https://developer.microsoft.com/windows/downloads/windows-10-sdk): Preinstalado en Visual Studio2015 con la actualización 3 y versiones posteriores.
* Un equipo con uno de los sistemas operativos host mencionados anteriormente y con al menos una máquina virtual. Esto es para probar la aplicación.

> **Nota:** La API para los sockets de Hyper-V pasó a estar disponible públicamente en Windows10 un poco después.  Las aplicaciones que usan HVSocket se ejecutarán en cualquier host e invitado con Windows10, pero solo se pueden desarrollar con un Windows SDK posterior a la compilación 14290.

## <a name="register-a-new-application"></a>Registro de una nueva aplicación
Para poder utilizar sockets de Hyper-V, la aplicación debe registrarse con el Registro del host de Hyper-V.

Al registrar el servicio en el Registro, obtendrá:
*  Administración de WMI para habilitar, deshabilitar y enumerar los servicios disponibles.
*  Permiso para comunicarse directamente con las máquinas virtuales.

El siguiente PowerShell registrará una nueva aplicación denominada "HV Socket Demo".  Se debe ejecutar como administrador.  Las instrucciones manuales se encuentran a continuación.

``` PowerShell
$friendlyName = "HV Socket Demo"

# Create a new random GUID.  Add it to the services list
$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ((New-Guid).Guid)

# Set a friendly name
$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```


**Información y ubicación del Registro:**
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```
En esta ubicación del Registro, verás varios GUID.  Se trata de nuestros servicios incluidos.

Información del Registro por cada servicio:
* `Service GUID`
    * `ElementName (REG_SZ)` es el nombre descriptivo del servicio.

Para registrar su propio servicio, cree una nueva clave del Registro con su propio GUID y un nombre descriptivo.

El nombre descriptivo se asociará con la nueva aplicación.  Aparecerá en los contadores de rendimiento y en otros lugares donde un GUID no es adecuado.

La entrada del Registro tendrá este aspecto:
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName REG_SZ  VM Session Service
    YourGUID\
        ElementName REG_SZ  Your Service Friendly Name
```

> **Nota:** el GUID de servicio de invitado Linux utiliza el protocolo VSOCK qué direcciona a través de un `svm_cid` y `svm_port` en lugar de un guid. Para superar esta incoherencia con Windows, se usa el GUID bien conocido como plantilla de servicio en el host que traduce a un puerto en el invitado. Para personalizar el GUID de servicio, simplemente cambia el primer valor "00000000" al número de puerto deseado. Por ejemplo: "00000ac9" es el puerto 2761.
> ```C++
> // Hyper-V Socket Linux guest VSOCK template GUID
> struct __declspec(uuid("00000000-facb-11e6-bd58-64006a7986d3")) VSockTemplate{};
>
>  /*
>   * GUID example = __uuidof(VSockTemplate);
>   * example.Data1 = 2761; // 0x00000AC9
>   */
> ```
>

> **Sugerencia:**  Para generar un GUID en PowerShell y copiarlo en el Portapapeles, ejecuta:
>``` PowerShell
>(New-Guid).Guid | clip.exe
>```

## <a name="create-a-hyper-v-socket"></a>Creación de un socket de Hyper-V

En el caso más básico, la definición de un socket requiere una familia de direcciones, un tipo de conexión y un protocolo.

Esta es una sencilla [definición de socket](
https://msdn.microsoft.com/en-us/library/windows/desktop/ms740506(v=vs.85).aspx
)

``` C
// Windows
SOCKET WSAAPI socket(
  _In_ int af,
  _In_ int type,
  _In_ int protocol
);

// Linux guest
int socket(int domain, int type, int protocol);
```

Para un socket de Hyper-V:
* Familia de direcciones: `AF_HYPERV` (Windows) o `AF_VSOCK` (invitado en Linux)
* tipo: `SOCK_STREAM`
* protocolo: `HV_PROTOCOL_RAW` (Windows) o `0` (invitado en Linux)


Este es un ejemplo de declaración o creación de instancia:
``` C
// Windows
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);

// Linux guest
int sock = socket(AF_VSOCK, SOCK_STREAM, 0);
```

## <a name="bind-to-a-hyper-v-socket"></a>Enlace a un socket de Hyper-V

El enlace asocia un socket con la información de conexión.

La definición de función está copiada a continuación para su comodidad. Puede obtener más información sobre los enlaces [aquí](https://msdn.microsoft.com/en-us/library/windows/desktop/ms737550.aspx).

``` C
// Windows
int bind(
  _In_ SOCKET                s,
  _In_ const struct sockaddr *name,
  _In_ int                   namelen
);

// Linux guest
int bind(int sockfd, const struct sockaddr *addr,
         socklen_t addrlen);
```

A diferencia de la dirección de socket (sockaddr) de una familia de direcciones de protocolo de Internet estándar (`AF_INET`), que consta de la dirección IP del equipo host y un número de puerto en ese host, la dirección del socket de `AF_HYPERV` usa el identificador de la máquina virtual y el identificador de aplicación definidos anteriormente para establecer una conexión. Si el enlace desde un invitado en Linux `AF_VSOCK` usa el `svm_cid` y el `svm_port`.

Como los sockets de Hyper-V no dependen de una pila de red, TCP/IP, DNS, etc., el punto de conexión del socket necesita un formato que no sea de IP ni nombre de host y aun así describa la conexión de forma inequívoca.

Aquí está la definición de la dirección de socket de un socket de Hyper-V:

``` C
// Windows
struct SOCKADDR_HV
{
     ADDRESS_FAMILY Family;
     USHORT Reserved;
     GUID VmId;
     GUID ServiceId;
};

// Linux guest
// See include/uapi/linux/vm_sockets.h for more information.
struct sockaddr_vm {
    __kernel_sa_family_t svm_family;
    unsigned short svm_reserved1;
    unsigned int svm_port;
    unsigned int svm_cid;
    unsigned char svm_zero[sizeof(struct sockaddr) -
                   sizeof(sa_family_t) -
                   sizeof(unsigned short) -
                   sizeof(unsigned int) - sizeof(unsigned int)];
};
```

En lugar de una IP o un nombre de host, los puntos de conexión AF_HYPERV dependen en gran medida de dos GUID:
* Id. de máquina virtual (VMID): este es el identificador único asignado por máquina virtual.  Un identificador de máquina virtual se encuentra mediante el siguiente fragmento de código de PowerShell.
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* Id. de servicio (GUID), [descrito anteriormente](#RegisterANewApplication), con el cual se registra la aplicación en el Registro del host de Hyper-V.

Hay también un conjunto de caracteres comodín de VMID disponibles cuando no se trata de una conexión a una máquina virtual específica.

### <a name="vmid-wildcards"></a>Caracteres comodín de VMID

| Nombre | GUID | Descripción |
|:-----|:-----|:-----|
| HV_GUID_ZERO | 00000000-0000-0000-0000-000000000000 | Las escuchas deben enlazarse a este elemento VmId para aceptar la conexión de todas las particiones. |
| HV_GUID_WILDCARD | 00000000-0000-0000-0000-000000000000 | Las escuchas deben enlazarse a este elemento VmId para aceptar la conexión de todas las particiones. |
| HV_GUID_BROADCAST | FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF | |
| HV_GUID_CHILDREN | 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd | Dirección comodín de los elementos secundarios. Las escuchas deben enlazarse a este elemento VmId para aceptar la conexión de sus elementos secundarios. |
| HV_GUID_LOOPBACK | e0e16197-dd56-4a10-9195-5ee7a155a838 | La dirección de bucle invertido. Al usar este elemento VmId, se conecta a la misma partición que el conector. |
| HV_GUID_PARENT | a42e7cda-d03f-480c-9cc2-a4de20abb878 | Dirección del elemento primario. Al usar este elemento VmId, se conecta a la partición primaria del conector.* |


\* `HV_GUID_PARENT` El elemento primario de una máquina virtual es su host.  El elemento primario de un contenedor es el host del contenedor.
La conexión desde un contenedor que se ejecuta en una máquina virtual hará que se conecte a la máquina virtual que hospeda el contenedor.
Escuchar este VmId acepta la conexión de: (contenedores Inside): host del contenedor.
(Dentro de la máquina virtual: host contenedor / ningún contenedor): host de la máquina virtual.
(Fuera de la máquina virtual: host contenedor / ningún contenedor): no se admite.

## <a name="supported-socket-commands"></a>Comandos de socket admitidos

Socket() Bind() Connect() Send() Listen() Accept()

## <a name="useful-links"></a>Vínculos útiles
[API WinSock completa](https://msdn.microsoft.com/en-us/library/windows/desktop/ms741394.aspx)

[Referencia de los servicios de integración de Hyper-V](../reference/integration-services.md)