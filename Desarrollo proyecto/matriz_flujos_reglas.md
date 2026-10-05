# Matriz de flujos y reglas del cortafuegos (R02 / R03 / R05)

Política de **denegación por defecto**: todo lo que no aparezca en esta matriz queda bloqueado.
En pfSense las reglas se evalúan en la **interfaz de origen**, de arriba abajo, con deny implícito al final.

## Flujos permitidos (lista blanca)

| Origen | Destino | Protocolo / Puerto | Interfaz (regla) | Requisito |
|--------|---------|--------------------|------------------|-----------|
| Usuarios (LAN) | DMZ | TCP 443 | LAN | R02 |
| DMZ | Servidores | TCP 5432 | DMZ | R02 |
| Gestión | Cualquier zona | Any | GESTION | R02 / R05 |
| Cliente VPN (10.10.90.0/24) | Servidores | (según regla, mínimo privilegio) | VPN | R03 |
| Exterior (WAN) | pfSense (WAN address) | UDP 51820 (WireGuard) | WAN | R03 |

## Denegado (por defecto)

- Usuarios → Servidores / Gestión / Crítica → **bloqueado**.
- Cliente VPN → DMZ / Gestión / Crítica → **bloqueado** (solo alcanza Servidores).
- Cualquier zona de menor confianza → Crítica → **bloqueado** (solo Gestión accede).

## Cambios aplicados para el default-deny

- Eliminadas las reglas "Default allow LAN to any" (IPv4 e IPv6).
- Desactivado el anti-lockout de la LAN.
- Verificado que las pestañas **Floating** y del grupo **WireGuard** están vacías (el bloqueo es por diseño, no por regla residual).
