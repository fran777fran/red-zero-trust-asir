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
- Nota de diseño: la gestión está de momento en la VLAN 10; se moverá a la VLAN 40 (Gestión) más adelante (mejora Zero Trust).
- Horas: ___ · Commit: ___
