# Verificación del .pkt final (27/10/2026, 23:35–23:42)

**Archivo final:** `entrega_final/01_PKT/Nexus_Proyecto_Final.pkt`
- Es `Nexus_Proyecto_v6.pkt` con un solo cambio: Ingeniería en 172.30.7.0/24 (md5 4fbc73cb…).
- La v6 original queda como respaldo en `respaldos/Nexus_Proyecto_v6_respaldo_cierre.pkt` (md5 c4e1a6c6…).

## Cambio aplicado sobre v6
| Elemento | Antes | Después |
|---|---|---|
| Vlan10 en DSW1-SEDE / DSW2-SEDE | 172.30.0.2 / .3, VIP HSRP .0.1 | 172.30.7.2 / .3, VIP 172.30.7.1 |
| POOL-ING (SRV-DHCP) | 172.30.0.11, 170 | 172.30.7.11, 170, gw .7.1, DNS .10 |
| POOL-ING-B (SRV-DHCP-2) | 172.30.0.181, 70 | 172.30.7.181, 70, gw .7.1, DNS .17 |
| ACL-GESTION, línea 20, en 14 equipos | permit 172.30.0.0/24 | permit 172.30.7.0/24 |

- **172.30.0.0/24 queda reservada.** Si se usara, chocaría con el identificador del resumen 172.30.0.0/21 en PT 9.
- Se ejecutó `write memory` en todos los equipos. Las configuraciones están en `02_Configs`, exportadas del startup-config.

## Resultados tras guardar, cerrar y reabrir, sin comandos de reparación
| Prueba | Resultado | Evidencia |
|---|---|---|
| R-EDGE `show ip route ospf` | Solo `O IA 172.30.0.0/21`, sin rutas específicas | `05_Evidencias/cidr/final_cidr_reabrir.txt` |
| R-EDGE, LSA tipo 3 del área 1 | Una sola: 172.30.0.0 /21, ADV 1.1.1.1 | ídem |
| ~5 min: ING, ADM → 8.8.8.8 | 4/6; los primeros paquetes se pierden en la convergencia | ídem |
| ~5 min: SUR → 8.8.8.8 | 5/6 | ídem |
| ~5 min: OCC → 8.8.8.8, invitado → 8.8.8.8, FIN → gateway de ING | 6/6 | ídem |
| ~7 min: ING → 8.8.8.8, ADM → 8.8.8.8, NORTE → www.nexus.com | 6/6 | `final_cidr_reabrir_r2.txt` |
| ~7 min: SUR → gateway ING 172.30.7.1 | 5/6 (se perdió el 2.º paquete) | ídem |
| ~7 min: ING → OCC | 5/6 (1.er paquete, ARP) | ídem |

## Prueba sin la estática de respaldo y fallo 3
- Se hicieron en una copia de prueba con la **misma configuración de enrutamiento** (v6 + ING en .7.0/24, sin la estática). Esa copia se borró después.
- **Sin la estática:** todas las sedes llegaron a Internet y la web publicada cargó.
- **Fallo 3:** al quitar `area range`, R-EDGE recibe 14 rutas específicas y la conectividad sigue (6/6). Al reponerlo, la /21 vuelve en 45 s.
- Evidencia: `05_Evidencias/cidr/cidr_sinEstatica_*.txt` y `v7_fallo3_*.txt` (el nombre «v7» es el de la copia de prueba).

## Pruebas que no se repitieron
DHCP/DNS de respaldo, invitados, P-19, P-24, la línea base y FTP no dependen de la red de Ingeniería, o se probaron en v6 con la misma configuración. La evidencia está en `05_Evidencias/v6_pruebas/`. Las pruebas pendientes están en `PENDIENTES.md`.
