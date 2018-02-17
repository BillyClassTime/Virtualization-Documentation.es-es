# <a name="host-gateway-mode"></a>Modo Host-Gateway #
Una de las opciones disponibles para redes Kubernetes es el *modo host-gateway*, lo que conlleva a la configuración de rutas estáticas entre subredes de pods en todos los nodos.


## <a name="configuring-static-routes--linux"></a>Configurar rutas estáticas | Linux ##
Para ello, usamos `iptables`. Sustituye (o establece) la variable `$CLUSTER_PREFIX` con la subred abreviada que todos los pods usarán:

```bash
CLUSTER_PREFIX="192.168"
sudo iptables -t nat -F
sudo iptables -t nat -A POSTROUTING ! -d $CLUSTER_PREFIX.0.0/16 \
              -m addrtype ! --dst-type LOCAL -j MASQUERADE
sudo sysctl -w net.ipv4.ip_forward=1
```

Esto solo configura la red NAT básica para los pods. Ahora, necesitamos que todo el tráfico destinado a pods vaya a través de la interfaz principal. De nuevo, sustituye la variable `$CLUSTER_PREFIX`, según sea necesario, y `eth0` si procede:

```bash
sudo route add -net $CLUSTER_PREFIX.0.0 netmask 255.255.0.0 dev eth0
```

Por último, tenemos que agregar la puerta de enlace de salto siguiente **por nodo**. Por ejemplo, si el primer nodo es un nodo de Windows en `192.168.1.0/16`:

```bash
sudo route add -net $CLUSTER_PREFIX.1.0 netmask 255.255.255.0 gw $CLUSTER_PREFIX.1.2 dev eth0
```

Se debe agregar una ruta similar *para* cada nodo del clúster, *en* cada nodo del clúster.


<a name="explanation-2-suffix"></a>
> [!Important]  
> Para nodos de Windows **únicamente**, la puerta de enlace es `.2` de la subred. Para Linux, es probablemente siempre `.1`. Esta anomalía se debe a que la dirección `.1` se reserva como la puerta de enlace para el adaptador de red que une la red de host y la red de pod virtual.


## <a name="configuring-static-routes--windows"></a>Configurar rutas estáticas | Windows ##
Para ello, usamos `New-NetRoute`. Hay un script automatizado disponible, `AddRoutes.ps1`, en [este repositorio](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/AddRoutes.ps1). Tendrás que conocer la dirección IP del *maestro de Linux*y la puerta de enlace predeterminada del adoptador *externo* del nodo de Windows (no la puerta de enlace de pod). En ese caso:

```powershell
$url = "https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/windows/AddRoutes.ps1"
wget $url -o AddRoutes.ps1
./AddRoutes.ps1 -MasterIp 10.1.2.3 -Gateway 10.1.3.1
```
