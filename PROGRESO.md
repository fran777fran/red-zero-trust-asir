# Seguimiento del proyecto — Red Zero Trust ASIR

> Documento vivo de control. Se actualiza en cada hito. Última actualización: **2026-09-21**.

## Resumen de porcentaje

| Ámbito | % completado |
|---|---|
| **Desarrollo técnico (R01–R06)** | **~70%** |
| **Memoria** | ~40% |
| **Proyecto completo** (técnico + memoria + defensa) | **~60%** |

## Estado por requisito (RFTP)

| Requisito | Estado | % | Qué falta |
|---|---|---|---|
| Fase 0 · Diseño / Anteproyecto | ✅ | 100% | — |
| R01 · Segmentación (VLANs 10–50) | ✅ | 100% | — |
| R02 · Microsegmentación (reglas) | ⏳ | 85% | Salida a internet por zona + publicación WAN→DMZ (necesita servicios reales) |
| R03 · VPN (WireGuard) | ✅ | 100% | — |
| R04 · IDS (Suricata) | ⬜ | 0% | Instalar/activar Suricata + detección de escaneo |
| R05 · Contención zona crítica | ✅ | 95% | Demostrado; opcional: escenario "simular compromiso" más formal |
| R06 · Validación (nmap) | ⬜ | 0% | Escaneos nmap segmentada vs plana + tabla comparativa |

## Estado no técnico

| Bloque | Estado | % | Qué falta |
|---|---|---|---|
| Memoria | ⏳ | ~40% | Ensamblar Word completo, secciones finales (conclusiones, trabajos futuros, evolución), maquetación, índices, ≥50 págs, cambiar "2.8.1"→"2.9.0" |
| Defensa | ⬜ | 0% | Presentación + guion + ensayo (≤15 min) |

## Pendientes transversales (para NO olvidar)

- [ ] **R02 refinamientos**: reglas de salida a internet por zona + publicación WAN→DMZ (NAT port-forward).
- [ ] **Servicios reales en el lab**: servidor web en DMZ, servicio/BD en Servidores → habilita el R02 pendiente y enriquece R06.
- [ ] **Cambiar la contraseña de admin** de pfSense (sigue en `pfsense`) — punto de hardening.
- [ ] **Horas reales** en la bitácora (planificación real) + análisis de desviaciones.
- [ ] **Capturas** en `evidencias/` y renumerar las "Ilustración X".
- [ ] **Gantt** fechado en GanttProject.
- [ ] **Diagramas** dibujados por ti en draw.io (arquitectura, red, casos de uso).
- [ ] Ensamblar el `desarrollo_despliegue.docx` con todo lo nuevo (R02, R05, R03).

## Calendario (entregas)

| Hito | Fecha | Estado |
|---|---|---|
| Entrega anteproyecto | 25 sep | ✅ |
| 1ª entrega parcial (R01) | 9 oct | ✅ (adelantado) |
| 2ª entrega parcial (R02–R05) | 20 nov | En curso: R02 núcleo/R03/R05 ✅; falta R04 y refinamientos R02 |
| Depósito final | 7 dic | — |
| Defensa | 14–20 dic | — |

## Hecho hasta ahora (resumen)

- Entorno: GNS3 + GNS3 VM + pfSense CE 2.9.0 (emulación, KVM off). Acceso web de gestión en VLAN 40 (https://10.10.40.1).
- R01: VLANs 10 (Usuarios), 20 (Servidores), 30 (DMZ), 40 (Gestión), 50 (Crítica) sobre troncal 802.1Q; direccionamiento 10.10.X0.1/24 + DHCP.
- R02 (núcleo): default-deny + reglas DMZ→Servidores:5432, Gestión→todo, Usuarios→DMZ:443. Gestión movida a VLAN 40.
- R03: VPN WireGuard; cliente Kali (integrada en GNS3, lado WAN); acceso restringido a Servidores, bloqueado a Crítica.
- R05: contención probada (Usuarios→Crítica bloqueado; Gestión→Crítica permitido).
