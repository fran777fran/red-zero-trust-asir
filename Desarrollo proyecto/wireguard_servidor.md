# Servidor WireGuard en pfSense (R03)

Configuración del túnel y de la restricción de acceso del cliente VPN.

## Túnel

- Paquete **WireGuard** instalado desde System > Package Manager.
- Túnel (VPN > WireGuard > Tunnels): escucha en **UDP 51820**, red de túnel **10.10.90.0/24**, pfSense = **10.10.90.1**. Par de claves del servidor generado. WireGuard habilitado en Settings.
- Túnel **asignado como interfaz** (tun_wg0), descripción **"VPN"**.
- **Gotcha clave:** en esta versión del paquete, la dirección del túnel se configura en la **interfaz asignada**, no en el túnel. La interfaz VPN se fija a **IPv4 Static 10.10.90.1/24**; si se deja en None falta la ruta de 10.10.90.0/24 y el tráfico de retorno se pierde (handshake OK pero sin respuesta).

## Peer (cliente)

- Peer **"Cliente Kali"**: Allowed IPs **10.10.90.2/32**, clave pública del cliente `DInv2WuCDGGeaDZ213RiFL3aSEMtk0TVmU2xVOBFODo=`.

## Reglas asociadas

- **WAN:** permitir entrante **UDP 51820** hacia la WAN address.
- Desactivado **"Block private networks"** en la WAN (la red de salida del laboratorio es privada, 192.168.42.0/24; WAN de pfSense = 192.168.42.88).
- **Interfaz VPN:** permitir **10.10.90.0/24 → SERVIDORES** (mínimo privilegio). El resto, incluida la zona crítica, bloqueado por defecto.

## Prueba (P03)

Con el túnel activo (`wg show` con handshake), desde Kali: ping a 10.10.20.1 (Servidores) responde; ping a 10.10.50.1 (Crítica) bloqueado.
