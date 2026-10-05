# Desarrollo del proyecto — artefactos técnicos

Ficheros "fuente" generados durante el desarrollo que respaldan la memoria y permiten
**reproducir** la solución. Cada artefacto corresponde a uno o varios requisitos (RFTP).

| Fichero | Contenido | Requisito |
|---------|-----------|-----------|
| `direccionamiento_vlans.md` | Plan de VLANs, direccionamiento, DHCP e interfaces; troncal 802.1Q | R01 |
| `matriz_flujos_reglas.md` | Matriz de flujos permitidos + denegación por defecto | R02 / R05 |
| `suricata_custom.rules` | Reglas propias del IDS (SID 1000001 escaneo, SID 1000010 ICMP) | R04 |
| `wg0.conf.ejemplo` | Config del cliente VPN (sanitizada, sin clave privada) | R03 |
| `wireguard_servidor.md` | Parámetros del túnel y reglas en pfSense | R03 |
| `nmap_r06_segmentada.txt` | Salida de nmap en red segmentada (2/6 hosts; Crítica bloqueada) | R06 |
| `nmap_r06_plana.txt` | Salida de nmap en red plana (6/6 hosts; Crítica accesible) | R06 |
| `suricata_alertas_muestra.txt` | Muestra de alertas del IDS (ICMP + escaneo detectado) | R04 |

## Seguridad

- El `wg0.conf.ejemplo` **no contiene la clave privada** (marcador en su lugar).
- No se incluyen claves, hashes de contraseñas ni material sensible. El `.gitignore` del repositorio excluye además `*.key`, `*.pem`, `wg-*.conf`, `secrets/` y ficheros de credenciales.
