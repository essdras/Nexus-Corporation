# Registro de cambios v5 → v6 (27/10/2026)

**Archivo:** `entrega2/Nexus_Proyecto_v6.pkt` (md5 c4e1a6c6…)

**Copias de vuelta atrás** en `C:/Users/grony/PacketTracerMCP/`:
- `Nexus_Proyecto_v5_respaldo_20261027.pkt`
- `v6_antes-cambio2.pkt`
- `v6_antes-cambio3.pkt`
- `v6_antes-cambio4.pkt`

**Configuraciones finales:** `revision/configs_final/`, con las tablas de rutas en los archivos `*_rutas.txt`.

## 1. DHCP Administrativo y serverPool
- **Qué se cambió:**
  - POOL-ADM pasa de .11–.120 (110 direcciones) a **.4–.126 (123)**.
  - serverPool queda acotado a 172.30.4.62 (PT lo recrea al abrir, no se puede borrar).
- **Por qué:** el PDF pide 120 usuarios en Administrativo.
- **Cómo se comprobó:**
  - PC-ADM-1, PC-ADM-2 y PC-ADM-3 renovaron y recibieron .4, .8 y .5, con gateway .1 y DNS .10.
  - Ping a la web, a Occidente y a Internet con 0 % de pérdida en la 2.ª ronda.
  - Se guardó, se reabrió y la configuración persiste.
- Con el cambio 3 el rango se reparte entre dos servidores: SRV-DHCP .4–.89 (86) y SRV-DHCP-2 .90–.126 (37). Siguen siendo 123 en total.

## 2. CIDR en el Core
- **Qué se cambió:**
  - **R-CORE (ABR):**
    - Gi0/2 (172.30.4.84/30) pasa al **área 1**.
    - Se agrega `area 0 range 172.30.0.0 255.255.248.0`.
  - **R-EDGE:**
    - Sus redes pasan al área 1.
    - La estática /21 se reemplaza por una **flotante con distancia administrativa 120**.
- **Antes:** R-EDGE tenía la estática /21 más 14 rutas OSPF específicas. La /21 casi no se usaba.
- **Después:** R-EDGE solo tiene `O IA 172.30.0.0/21` desde R-CORE, más sus redes conectadas y la ruta por defecto. R-CORE instala `172.30.0.0/21 is a summary, Null0`.
- **Cómo se comprobó:**
  - Ping a 8.8.8.8 desde ING, ADM, FIN, NORTE, SUR y OCC: 4/4 (salvo la 1.ª ronda tras un cambio).
  - Invitado a Internet: 4/4.
- **Límite de PT 9:**
  - Al reabrir el archivo, R-CORE tarda unos minutos en regenerar la LSA resumen. Durante ese tiempo R-EDGE ve 172.30.0.0/24 en lugar de /21.
  - La estática flotante cubre ese intervalo. Se comprobó dos veces: tras reabrir, la ruta usada fue la estática y luego volvió `O IA /21`.
  - `clear ip ospf process` en R-CORE lo corrige al instante.
- **Falla 3 (quitar la sumarización):** con OSPF reaparecen las rutas específicas y no se bloquean las sucursales. Falta ensayarla y documentarla. Es consulta para el docente.

## 3. DHCP y DNS redundantes
- **Qué se cambió:**
  - **SRV-DHCP-2:** 172.30.4.16, conectado en ASW-DC Fa0/7.
  - **SRV-DNS-2:** 172.30.4.17, conectado en Fa0/8, con los 8 registros A del DNS principal más `dhcp2` y `dns2`.
  - **Split-scope:** el primario entrega cerca del 70 % de cada pool y el secundario el 30 %. Los pools del secundario entregan DNS .17.
  - **Relay:** segundo `ip helper-address 172.30.4.16` en DSW1 y DSW2 (VLAN 10, 20, 30 y 50), R-NORTE, R-SUR, R-OCCIDENTE y R-EDGE Gi0/2.
  - **ACL-INVITADOS:** se agrega la línea 25 `permit udp any host 172.30.4.17 eq domain`.
- **Cómo se comprobó:**
  - Con el servicio DHCP de SRV-DHCP apagado, renovaron PC-ADM-2 (.2.90), PC-NORTE-2 (.1.181) y LAP-INVITADO (.5.181), todos con DNS .17.
  - Con SRV-DNS apagado:
    - `ping www.nexus.com` resolvió a 172.30.4.12 desde ADM y NORTE.
    - `nslookup internet.com` respondió desde el invitado usando el servidor 172.30.4.17.
  - Al final se volvieron a encender los dos servicios.
- **Límite de PT:** cada pool entrega un solo DNS. Un cliente servido por el primario no resolvió nombres mientras SRV-DNS estuvo apagado (PC-ADM-1, sin respuesta en 12 s).

## 4. Invitados sin depender de DSW1
- **Qué se cambió:**
  - El cable de R-EDGE Gi0/2 pasa de DSW1 Fa0/10 a **ASW-ING Fa0/21** (acceso VLAN 60, portfast y bpduguard).
  - DSW1 Fa0/10 queda apagado.
- **Aislamiento:** no cambia. El gateway sigue en R-EDGE Gi0/2 con ACL-INVITADOS en una interfaz física, que persiste al guardar.
- **Prueba de caída completa de DSW1 (equipo apagado):**
  - HSRP: DSW2 pasó a activo en las VLAN 10, 20, 30, 40, 50 y 99.
  - ING a Internet: 4/4.
  - FIN a Norte: 3/4.
  - NORTE a la web: 4/4.
  - Invitado: renovó DHCP (.5.181) y llegó a Internet 4/4. Hacia ING, *Destination host unreachable* desde 172.30.5.1.
  - Después se reabrió v6 para volver a encender DSW1 sin guardar el apagado.

## Pruebas principales repetidas tras reabrir v6
- ING→Occidente: 4/4.
- ADM→www.nexus.com: 4/4.
- SUR→web: 4/4.
- FIN, OCC, NORTE y ADM→Internet: 3/4 a 4/4 en la 2.ª ronda, después de la convergencia de OSPF.
- Invitado→Internet: 4/4.
- Invitado→interno: bloqueado.
- SRV-INTERNET→200.100.50.10 por ICMP: bloqueado por ACL-INTERNET-IN, como está diseñado. **Falta probar HTTP** (P-22 web).

## Hallazgo adicional
Al abrir el archivo, a veces DSW1 tarda en instalar las rutas OSPF. Esto es consistente con las pérdidas de la 1.ª ronda observadas en las pruebas. Por eso las pruebas se repiten después de la convergencia.

## Pendiente
- Clusters.
- Totales de VLSM y Excel.
- Plan de crecimiento a 3 años.
- Ensayo de las 8 fallas.
- Port Security con una 3.ª MAC.
- FTP por WAN.
- P-13, P-14, P-18, P-19, P-24 y P-22 web.
- Línea base de 3 series y jitter corregido.
- Informe y portada.
- Consultas al docente: WPA3, portal cautivo, IMAP, 6 integrantes, 35 servidores representados y falla 3.
