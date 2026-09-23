# Red Zero Trust ASIR

**Diseño e implementación de una arquitectura de red segura basada en el modelo Zero Trust para la protección de infraestructuras críticas.**

Proyecto Intermodular del Ciclo Formativo de Grado Superior en Administración de Sistemas Informáticos en Red (ASIR). Se diseña, despliega y valida en un laboratorio virtualizado una red corporativa segmentada según el modelo **Zero Trust** (NIST SP 800-207), demostrando —con datos reproducibles— que reduce la superficie de ataque y contiene el movimiento lateral frente a una red plana tradicional.

---

## Arquitectura

La red se divide en zonas de confianza mediante VLANs sobre un enlace troncal 802.1Q, gobernadas por un cortafuegos que actúa como **punto de aplicación de políticas (PEP)** con denegación por defecto y reglas explícitas de mínimo privilegio.

| Zona | VLAN | Red | Función |
|------|------|-----|---------|
| Usuarios | 10 | 10.10.10.0/24 | Puestos de trabajo internos |
| Servidores | 20 | 10.10.20.0/24 | Servicios internos / BD |
| DMZ | 30 | 10.10.30.0/24 | Servicios publicados |
| Gestión | 40 | 10.10.40.0/24 | Administración |
| Crítica | 50 | 10.10.50.0/24 | Activos sensibles |
| VPN | — | 10.10.90.0/24 | Acceso remoto (WireGuard) |

---

## Requisitos del proyecto (RFTP)

| Código | Requisito |
|--------|-----------|
| **R01** | Segmentación en zonas mediante VLANs (802.1Q) |
| **R02** | Microsegmentación: denegación por defecto y mínimo privilegio |
| **R03** | Acceso remoto seguro por VPN cifrada, acotado a un segmento |
| **R04** | Detección de intrusiones y registro (IDS) |
| **R05** | Contención de la zona crítica (aislamiento del movimiento lateral) |
| **R06** | Validación y comparativa (red segmentada frente a red plana) |

**Resultado clave (R06):** desde una posición de atacante, la red plana permite descubrir y alcanzar las 6 zonas (6/6), incluida la zona crítica; con segmentación Zero Trust el atacante queda confinado a la única zona autorizada (2/6). La segmentación reduce drásticamente la superficie de reconocimiento y contiene el movimiento lateral.

---

## Tecnologías

- **GNS3 3.0.6** — emulación de la topología de red.
- **pfSense CE 2.9.0** — cortafuegos / PEP (VLANs, reglas, VPN, IDS).
- **WireGuard** (integrado en pfSense) — VPN de acceso remoto.
- **Suricata** (paquete de pfSense) — IDS sobre la zona de Servidores.
- **Kali Linux** — cliente VPN y equipo de auditoría.
- **nmap 7.99** — descubrimiento de hosts y escaneo de puertos.
- **draw.io** — diagramas de arquitectura y de red.
- **GanttProject** — planificación y diagrama de Gantt.
- **Git / GitHub** — control de versiones y trazabilidad.

---

## Estructura del repositorio

```
├── Anteproyecto/      Anteproyecto y documentación inicial
├── Prememorias/       Versiones de la memoria en elaboración (Pre_x.docx)
├── Desarrollo proyecto/  Material de desarrollo de la solución
├── Contenido/         Contenidos y borradores de la memoria
├── docs/              Documentación (diagramas, notas técnicas)
├── configs/           Configuraciones exportadas (sanitizadas, sin secretos)
├── evidencias/        Evidencias de las pruebas
├── imagenes/          Capturas utilizadas en la memoria
├── pruebas/           Resultados de pruebas (salidas nmap, alertas IDS)
├── BITACORA.md        Bitácora de desarrollo por tarea RFTP
├── PROGRESO.md        Seguimiento del proyecto (estado y porcentajes)
└── .gitignore         Exclusión de material sensible
```

---

## Trazabilidad y metodología

- El desarrollo se registra en **[BITACORA.md](BITACORA.md)** con una ficha por tarea (objetivo, configuración, evidencia, prueba e incidencias), y el avance global en **[PROGRESO.md](PROGRESO.md)**.
- Los **commits** se corresponden con las tareas del RFTP, de modo que el historial de Git es contrastable con la planificación (Gantt) y los casos de uso, como exige la metodología del proyecto.

## Seguridad del repositorio

El fichero `.gitignore` excluye del control de versiones cualquier material sensible: claves privadas (`*.key`, `*_privatekey*`, `*.pem`), configuraciones de WireGuard con secretos (`wg-*.conf`), la carpeta `secrets/` y los ficheros de credenciales. Las configuraciones publicadas están sanitizadas.

---

## Autor

**Francisco de Asís Sierra de la Rosa** — CFGS Administración de Sistemas Informáticos en Red (modalidad online).

Proyecto académico con fines educativos. El laboratorio es un entorno virtualizado y aislado; las direcciones, claves y credenciales mostradas corresponden únicamente a dicho entorno de pruebas.
