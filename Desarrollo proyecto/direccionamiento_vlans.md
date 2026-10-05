# Direccionamiento y VLANs (R01)

Segmentación en zonas de confianza mediante VLANs sobre un enlace troncal 802.1Q,
terminadas en el cortafuegos pfSense (un subinterfaz por VLAN sobre `em1`).

| Zona | VLAN | Red | Gateway | DHCP | Interfaz pfSense |
|------|------|-----|---------|------|------------------|
| Usuarios | 10 | 10.10.10.0/24 | 10.10.10.1 | .100–.200 | LAN (em1.10) |
| Servidores | 20 | 10.10.20.0/24 | 10.10.20.1 | .100–.200 | OPT1 / SERVIDORES (em1.20) |
| DMZ | 30 | 10.10.30.0/24 | 10.10.30.1 | .100–.200 | OPT2 / DMZ (em1.30) |
| Gestión | 40 | 10.10.40.0/24 | 10.10.40.1 | .100–.200 | OPT3 / GESTION (em1.40) |
| Crítica | 50 | 10.10.50.0/24 | 10.10.50.1 | .100–.200 | OPT4 / CRITICA (em1.50) |
| VPN | — | 10.10.90.0/24 | 10.10.90.1 | — | VPN (tun_wg0) |
| WAN | — | 192.168.42.0/24 | (NAT GNS3) | — | WAN (em0 = 192.168.42.88) |

## Conmutación (GNS3)

- Ethernet switch con el **puerto 0 = dot1q (troncal)** hacia pfSense `em1`: transporta todas las VLANs etiquetadas.
- El resto de puertos en modo **access**, cada uno en su VLAN (p. ej. PC1 en VLAN 10, SRV en VLAN 20).

## Notas

- Las interfaces OPT nacen sin reglas (denegación por defecto) = línea base Zero Trust.
- La estación de gestión se ubica en la VLAN 40 (10.10.40.2) con ruta estática `10.10.0.0/16` vía `10.10.40.1`.
- Administración web del cortafuegos en https://10.10.40.1
