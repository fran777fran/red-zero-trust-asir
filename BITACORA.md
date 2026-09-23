# Bitácora de desarrollo — Red Zero Trust ASIR

Registro por tarea del RFTP. Cada entrada alimenta el apartado "Desarrollo de la solución" y la planificación real de horas.

## Plantilla (copiar por cada tarea)

### R0xF0xT0x — [título]
- Objetivo:
- Configuración realizada:
- Evidencia: evidencias/xxxx.png (Ilustración X)
- Prueba (P0x): [qué] -> Resultado:
- Incidencias: ninguna / [problema + solución]
- Horas:
- Commit:

---

## Registro

### Preparación del entorno de laboratorio (Despliegue) — 2026-09-15/16
- Objetivo: montar el entorno de emulación de red (GNS3) y desplegar el cortafuegos pfSense, base de la arquitectura Zero Trust.
- Configuración realizada:
  - Instalación de GNS3 (GUI) + GNS3 VM en VirtualBox (6 GB RAM, 2 vCPU, virtualización anidada activada).
  - Añadida la plantilla de pfSense (appliance QEMU) e instalación de pfSense CE 2.9.0.
  - Nodo pfSense con 6 interfaces (em0–em5): em0 = WAN conectada a un nodo NAT (salida a internet); em1 = LAN/troncal (192.168.1.1/24).
- Evidencia: GNS3 VM en verde; instalador de pfSense (versión 2.9.0 / Install CE); topología inicial. (evidencias/…)
- Incidencias (controladas):
  1. "DHCP server already exists" al habilitar el GNS3 VM (conflicto en el adaptador host-only de VirtualBox). Solución: eliminado el DHCP duplicado con `VBoxManage dhcpserver remove` para que GNS3 crease el suyo.
  2. Sin aceleración KVM (`/dev/kvm` inexistente; sin virtualización anidada efectiva en el host). Solución: `enable_kvm = false` en `gns3_server.conf`; pfSense se ejecuta por emulación (más lento pero funcional).
  3. El instalador de pfSense 2.9.0 es online y requería conexión con Netgate. Solución: WAN (em0) conectada al nodo NAT para dar internet; instalada la edición CE (sin suscripción Plus).
- Requisitos mínimos: VirtualBox + GNS3 3.x, GNS3 VM (6 GB / 2 vCPU), pfSense CE 2.9.0, ~8 GB de RAM libres.
- Horas: ___ (completar)
- Commit: ___ (hash)

### R01F01T02 — Etiquetado 802.1Q en el conmutador — 2026-09-16
- Objetivo: transportar las VLANs de las zonas de confianza hacia el cortafuegos por un enlace troncal 802.1Q.
- Configuración realizada:
  - Añadido un Ethernet switch (Switch1) en GNS3.
  - Puerto hacia pfSense (em1): tipo dot1q (troncal) — transporta todas las VLANs etiquetadas.
  - Puerto hacia PC1 (VPCS): tipo access, VLAN 10 (zona Usuarios).
- Evidencia: configuración de puertos del switch en GNS3 (evidencias/…) → Ilustración X.
- Prueba (P): pendiente; se validará con el ping de PC1 a su gateway tras crear la VLAN 10 en pfSense (ver R01F01T01).
- Incidencias: ninguna.
- Horas: ___ (completar)
- Commit: ___ (hash)

### R01F01T01 — Crear VLANs y direccionamiento en pfSense — VLANs 10–50 completadas — 2026-09-16/17
- Objetivo: crear las VLANs sobre em1 y asignarles direccionamiento (10.10.X0.1/24). Completada la VLAN 10 (Usuarios).
- Configuración realizada (VLAN 10):
  - pfSense (consola) opción 1 "Assign Interfaces": VLAN con parent em1, tag 10, asignada como LAN (em1.10).
  - Opción 2 "Set interface IP": em1.10 = 10.10.10.1/24; servidor DHCP habilitado (rango 10.10.10.100–200).
  - Switch (GNS3): port 0 → pfSense em1 = dot1q VLAN 1 (troncal); port 1 → PC1 = access VLAN 10.
- Evidencia: config de puertos del switch; salida de `ip dhcp` y `ping` en PC1 (evidencias/…) → Ilustración X.
- Prueba (P): PC1 (VPCS) obtiene IP por DHCP (10.10.10.100) y hace ping a 10.10.10.1 → Resultado: OK (~3–5 ms).
- Incidencias (controladas):
  1. IP puesta como cliente DHCP por error ("y" en vez de "n" en "via DHCP?"); la LAN quedó sin IP fija. Solución: repetir opción 2 con "n" y fijar 10.10.10.1/24 + rango DHCP.
  2. Sin conectividad inicial: los puertos del switch estaban en access VLAN 1 por defecto y, al configurarlos, quedaron invertidos (dot1q/access en el puerto equivocado). Detectado con el tooltip del enlace (pfSense em1 = switch port 0; PC1 = switch port 1). Solución: port 0 = dot1q VLAN 1, port 1 = access VLAN 10.
- Horas: ___ (completar)
- Commit: ___ (hash)
- VLANs 20–50 (por interfaz web): creadas en Interfaces → Assignments → VLANs (parent em1, tags 20/30/40/50, descripciones SERVIDORES/DMZ/GESTION/CRITICA); asignadas como OPT1–OPT4; cada interfaz habilitada con IP estática (10.10.20.1 / 10.10.30.1 / 10.10.40.1 / 10.10.50.1, /24, gateway None) y servidor DHCP (rango .100–.200).
- Incidencia (controlada): al asignar, por una descripción ambigua se cambió temporalmente la interfaz LAN a la VLAN 20 (riesgo de autobloqueo, porque la gestión entra por la VLAN 10). Detectado antes de guardar; solución: LAN devuelta a la VLAN 10 y las nuevas VLANs añadidas como OPT vía "Available network ports → Add".
- Estado: R01F01 COMPLETO (VLANs 10–50 creadas, direccionadas y con DHCP). Las interfaces OPT nacen sin reglas (default-deny) = línea base Zero Trust; la conectividad por zona se validará en R02.

### Acceso web de gestión (Despliegue) — 2026-09-16
- Objetivo: acceder al webConfigurator de pfSense desde el navegador de Windows para configurar el resto (VLANs, reglas) de forma más ágil que por consola.
- Configuración realizada:
  - Adaptador host-only dedicado en VirtualBox (#2 = 10.10.10.2/24, DHCP desactivado); en Windows es "Ethernet 4".
  - 3ª tarjeta de red en el GNS3 VM conectada a ese host-only #2 (modo promiscuo "permitir todo").
  - Nodo Cloud dentro del GNS3 VM (eth2) conectado a Switch1 en un puerto access VLAN 10.
  - Acceso a https://10.10.10.1 (admin) desde Windows.
- Incidencia (controlada): primero se creó el Cloud en el servidor local; el enlace cruzaba al Switch (en el GNS3 VM) por un túnel entre servidores que no pasaba tráfico (PC1 sí pingaba, Windows no). Solución: mover el Cloud al GNS3 VM con una tarjeta host-only #2 dedicada. Detectado comparando el ping de PC1 (OK) con el de Windows (fallo) y verificando el adaptador correcto (10.10.10.2 = Ethernet 4, no Ethernet 2).
- Nota de diseño: la gestión estaba al principio en la VLAN 10; se movió a la VLAN 40 (Gestión) durante R02 (ver más abajo).
- Horas: ___ · Commit: ___

### R02 — Microsegmentación (denegación por defecto + reglas de mínimo privilegio) — 2026-09-17/18
- Objetivo: aplicar denegación por defecto y permitir solo los flujos necesarios entre zonas (mínimo privilegio).
- Configuración realizada:
  - En pfSense las reglas se aplican en la interfaz de ORIGEN, de arriba abajo, con deny implícito al final.
  - Regla DMZ → SERVIDORES, TCP 5432 (interfaz DMZ).
  - Regla GESTION → any, protocolo Any (zona de administración).
  - Regla LAN (Usuarios) → DMZ, TCP 443; eliminadas las reglas "Default allow LAN to any" (IPv4 e IPv6) para que rija el deny implícito; anti-lockout de LAN desactivado.
  - Traslado de la gestión a la VLAN 40: switch port 2 → access VLAN 40; adaptador Windows a 10.10.40.2 + ruta estática (`route -p add 10.10.0.0 mask 255.255.0.0 10.10.40.1`). Administración ahora en https://10.10.40.1.
- Incidencias (controladas):
  1. Regla de Usuarios creada por error en la interfaz WAN y con puerto 433; corregida a interfaz LAN y puerto 443.
  2. La regla Usuarios→DMZ quedaba por debajo de "Default allow LAN to any"; se eliminó la allow-all para que aplicara el orden correcto.
  3. Ping desde Gestión fallaba: (a) la regla estaba en TCP → cambiada a Any; (b) el PC de gestión no tenía ruta a otras subredes → añadida ruta estática 10.10.0.0/16 vía 10.10.40.1.
- Estado: R02 núcleo completo (default-deny + flujos DMZ→Servidores, Gestión→todo, Usuarios→DMZ). Refinamientos pendientes: accesos a internet por zona y publicación WAN→DMZ (según alcance / trabajo futuro).
- Horas: ___ · Commit: ___

### R05 — Contención de la zona crítica (prueba de aislamiento) — 2026-09-18
- Objetivo: verificar que un host de otra zona NO alcanza la zona crítica y que solo Gestión puede.
- Prueba (P):
  - Desde PC1 (Usuarios, 10.10.10.100): ping a 10.10.50.1 (Crítica) y a 10.10.20.1 (Servidores) → timeout = BLOQUEADO.
  - Desde el PC de Gestión (10.10.40.2): ping a 10.10.50.1 (Crítica) → responde = PERMITIDO.
- Resultado: el mismo destino (10.10.50.1) queda bloqueado desde Usuarios y permitido desde Gestión → demostración directa de la contención Zero Trust (sin movimiento lateral hacia la zona crítica) y del acceso de administración segregado.
- Evidencia: capturas de los pings (PC1 timeout / Windows respuesta) → Ilustración X.
- Horas: ___ · Commit: ___

### R03 — VPN de acceso remoto (WireGuard) — COMPLETADO — 2026-09-21
- Objetivo: proporcionar acceso remoto seguro mediante VPN (WireGuard), sujeto también a las reglas Zero Trust.
- Configuración realizada (lado servidor):
  - Instalado el paquete WireGuard (System → Package Manager).
  - Túnel creado (VPN → WireGuard → Tunnels): escucha en UDP 51820, red de túnel 10.10.90.0/24 (pfSense = 10.10.90.1), claves del servidor generadas. WireGuard habilitado en Settings.
  - Túnel asignado como interfaz (tun_wg0), descripción "VPN", IPv4 None, interfaz habilitada.
  - Regla WAN: permitir UDP 51820 hacia la WAN address.
  - Desactivado "Block private networks and loopback addresses" en la WAN (la "internet" del laboratorio es una red privada, 192.168.42.0/24; WAN de pfSense = 192.168.42.88).
- Incidencias (controladas):
  1. Al asignar la interfaz se añadió por error em1 (la troncal) como OPT5; se eliminó —em1 no debe asignarse, es el padre de las VLANs— y se asignó tun_wg0.
  2. La descripción "WIREGUARD" chocaba con el grupo de interfaces homónimo que crea el paquete; se usó "VPN".
- Cliente y prueba (completado):
  - Kali integrada en GNS3 en el segmento WAN (switch nuevo entre pfSense em0 y el NAT; Kali con IP 192.168.42.x). Reutilizable como escáner en R06.
  - Claves del cliente generadas en Kali (wg genkey / wg pubkey). Peer creado en pfSense (Cliente Kali, clave pública del cliente, Allowed IPs 10.10.90.2/32, endpoint dinámico).
  - Config del cliente en Kali (/etc/wireguard/wg0.conf): PrivateKey del cliente, Address 10.10.90.2/24; [Peer] PublicKey del servidor, Endpoint 192.168.42.88:51820, AllowedIPs 10.10.0.0/16, PersistentKeepalive 25.
  - Regla en la interfaz VPN: permitir 10.10.90.0/24 → SERVIDORES (mínimo privilegio); el resto (incluida Crítica) queda bloqueado por defecto.
- Incidencia (controlada): el cliente conectaba (handshake OK, tráfico saliente) pero no recibía respuesta. Causa: en esta versión del paquete WireGuard la dirección del túnel se configura en la INTERFAZ asignada (no en el túnel), y estaba en IPv4=None → faltaba la ruta 10.10.90.0/24 en pfSense (Diagnostics → Routes no la mostraba). Solución: interfaz VPN → IPv4 Static 10.10.90.1/24 → aparece la ruta y el tráfico de retorno funciona.
- Prueba (P): con el túnel establecido (wg show con "latest handshake"), desde Kali: ping a 10.10.20.1 (Servidores) → RESPONDE (~6 ms); ping a 10.10.50.1 (Crítica) → 100% pérdida (BLOQUEADO).
- Resultado: R03 COMPLETO. Acceso remoto por VPN cifrada operativo y sujeto al modelo Zero Trust (alcanza lo autorizado, no la zona crítica).
- Horas: ___ · Commit: ___

### R04 — Sistema de detección de intrusiones (Suricata IDS) — COMPLETADO — 2026-09-22
- Objetivo: desplegar un IDS (Suricata) que inspeccione el tráfico del segmento Servidores y detecte reconocimiento/escaneo de puertos, generando alertas registrables.
- Configuración realizada:
  - Instalado el paquete Suricata (System → Package Manager).
  - Instancia creada sobre la interfaz SERVIDORES (em1.20) — "IDS-Servidores" — en modo legacy (detección + registro).
  - Reglas ET Open descargadas y habilitadas (categorías de escaneo, etc.) vía Global Settings + Updates.
  - Reglas propias (Category Selection → custom.rules):
    - SID 1000010 — `alert icmp any any -> any any` — regla de prueba para verificar que el motor inspecciona y carga reglas custom.
    - SID 1000001 (rev 2) — `alert tcp 10.10.90.0/24 any -> 10.10.20.0/24 any (flags:S,12; threshold:type both, track by_src, count 5, seconds 60; classtype:attempted-recon; ...)` — detección de escaneo SYN desde la VPN hacia Servidores.
- Evidencia: Logs View → alerts.log y pestaña Alerts (filtro Source 10.10.90.2) → capturas de las alertas ICMP (SID 1000010) y de escaneo (SID 1000001) → Ilustración X. Ruta del log: /var/log/suricata/suricata_em1.<id>/alerts.log.
- Prueba (P04): con el túnel WireGuard activo, desde Kali (cliente VPN 10.10.90.2):
  1. `ping -c 4 10.10.20.100` → Suricata registra la alerta SID 1000010 "ICMP detectado (test IDS)" con Src 10.10.90.2 (confirma que el IDS ve el tráfico de la VPN).
  2. `sudo nmap -sS -p 1-200 10.10.20.100` → Suricata dispara la alerta **SID 1000001 "Escaneo de puertos SYN desde VPN hacia Servidores"** (Prioridad 2, clase "Attempted Information Leak", Src 10.10.90.2 → Dst 10.10.20.100). Detección de reconocimiento CORRECTA.
- Incidencias (controladas):
  1. Las reglas ET SCAN no disparaban en un escaneo interno porque HOME_NET incluye todas las redes privadas y esas firmas van en sentido $EXTERNAL_NET→$HOME_NET. Solución: escribir una regla propia orientada al flujo VPN→Servidores.
  2. Ruido de decodificador "SURICATA TCPv4 invalid checksum" por el offloading de la emulación. Solución: System → Advanced → Networking, desactivados los 3 ajustes de hardware offloading + reboot de pfSense.
  3. La descarga de reglas fallaba ("Could not resolve host"). Solución: fijar DNS 8.8.8.8/1.1.1.1, desmarcar "DNS Server Override" y activar el modo forwarding del DNS Resolver.
  4. El escaneo no se detectaba porque el túnel WireGuard estaba caído tras reiniciar la topología (el filtro por Src 10.10.90.2 salía vacío). Solución: `sudo wg-quick up wg0` en Kali; confirmado con la alerta ICMP.
  5. La primera versión de la regla (flags:S estricto; umbral 15 SYN/10 s) no saltaba. Solución: `flags:S,12` (ignora bits reservados) y umbral 5 SYN/60 s (rev 2).
- Resultado: R04 COMPLETO. IDS desplegado, inspeccionando el segmento Servidores y detectando/registrando el escaneo de reconocimiento procedente de la VPN.
- Horas: ___ · Commit: ___

### R06 — Validación: comparativa red segmentada vs red plana — COMPLETADO — 2026-09-23
- Objetivo: validar empíricamente que la segmentación Zero Trust reduce la superficie de ataque y contiene el reconocimiento, comparándola con una red plana equivalente.
- Metodología: experimento controlado desde el mismo atacante (Kali, cliente VPN 10.10.90.2 → modelo de amenaza "credenciales VPN comprometidas"), mismos objetivos, variando ÚNICAMENTE la política del cortafuegos.
- Escenario A — Red segmentada (estado real Zero Trust): solo la regla "VPN → Servidores" + denegación implícita. Verificado que las pestañas Floating y WireGuard están vacías → el bloqueo es por diseño, no por casualidad.
  - `nmap -sn` (una IP por zona): 2 de 6 hosts descubiertos (solo Servidores: 10.10.20.1 y 10.10.20.100).
  - `nmap -sS -p 1-200 10.10.20.100` (Servidores, autorizada): host up y enumerable (el nodo VPCS responde SYN/ACK a todos los puertos → artefacto de simulación, no servicios reales).
  - `nmap -sS -p 1-200 10.10.50.1` (Crítica): host down = BLOQUEADO.
  - Efecto cruzado: el escaneo a Servidores vuelve a disparar la alerta del IDS (R04, SID 1000001).
- Escenario B — Red plana (simulada): regla temporal allow-all en la interfaz VPN (10.10.90.0/24 → any); ELIMINADA tras la prueba para restaurar el modelo.
  - `nmap -sn`: 6 de 6 hosts descubiertos (todas las zonas visibles).
  - `nmap -sS -p 1-200 10.10.50.1` (Crítica): host up, 53/tcp y 80/tcp abiertos.
  - `nmap -sS -p 1-200 10.10.40.1` (Gestión): host up, 53/tcp y 80/tcp abiertos.
- Resultado / comparativa: la segmentación reduce los hosts descubiertos de 6/6 (plana) a 2/6 (segmentada, solo la zona autorizada). En red plana, un atacante con credenciales VPN comprometidas alcanza TODA la infraestructura, incluidas la zona crítica y los servicios de gestión del propio firewall (DNS/web); con segmentación Zero Trust queda confinado a la única zona autorizada.
- Incidencias (controladas):
  1. Los primeros escaneos salían "host down" en todo: el túnel WireGuard estaba a medio levantar (`wg show` = 0 B received, sin handshake). Solución: `sudo wg-quick down wg0; sudo wg-quick up wg0` y confirmar "latest handshake" + bytes recibidos > 0.
  2. Tras reiniciar, el VPCS SRV perdió su IP → `ip dhcp` para recuperar 10.10.20.100.
  3. En una prueba previa se observó acceso a Crítica desde la VPN; se comprobó que era un artefacto de estados residuales de la sesión anterior (cortafuegos stateful), no una regla mal configurada. Lección aprendida: validar siempre desde un estado limpio tras reiniciar.
- Evidencia: salidas de nmap de ambos escenarios (A y B) → Ilustración X.
- Horas: ___ · Commit: ___
