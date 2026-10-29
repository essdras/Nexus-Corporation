# Paso 1 · Diagnóstico del estado inicial (27/10/2026, 20:50–21:10)

## Respaldo
- `Nexus_Proyecto_v5_respaldo_20261027.pkt` en `C:/Users/grony/PacketTracerMCP/` (md5 9415ced9…, idéntico al v5 abierto, guardado 19:14).
- Ojo: la copia de `entrega2/Nexus_Proyecto_v5.pkt` (19:09, md5 4c0de88a…) es **anterior** a la del PC. La válida es la del PC.
- Configuraciones exportadas (`show running-config`) de los 15 equipos de red en `revision/configs_estado_inicial/`. Llevan contraseñas tipo 7 (reversibles): compartir solo dentro del grupo.

## Lo que funciona (comprobado hoy)
| Área | Evidencia |
|---|---|
| OSPF área 0 | R-EDGE con vecino 1.1.1.1 FULL; R-CORE aprende las 6 LAN de sede y las 3 sucursales; ruta por defecto O*E2 desde R-EDGE. |
| Conectividad | 2.ª ronda de ping con 0 % de pérdida: ING→Occidente, ADM→8.8.8.8, FIN→SRV-WEB, Invitado→8.8.8.8. Norte→Sur 3/4 (1.ª vez hacia esa IP). |
| Aislamiento invitados | LAP-INVITADO→PC-ING-1: «Destination host unreachable» desde 172.30.5.1 (ACL-INVITADOS). |
| HSRP | DSW1 activo 10/30/50/99, DSW2 activo 20/40, preempt (config). P-10 ya probado. |
| NAT | Estática 200.100.50.10↔172.30.4.12 activa; PAT por Gi0/1. |
| DHCP | Servidor activo con 8 pools + relay `ip helper-address 172.30.4.11` en SVIs. |
| Pruebas previas | P-01…P-12, P-15…P-17, P-20, P-22 (ping), P-23 y línea base: ver informe cap. 8. |

**Pérdidas en la 1.ª ronda** (50 %, 75 %, 25 %): aparecieron justo después de que OSPF reconvergió (rutas con 00:00:01–00:00:55). En la 2.ª ronda desaparecen. Eso es **consistente** con ARP/convergencia, pero no lo prueba: falta capturar la tabla ARP antes y después.

## Lo que falla o no cumple el PDF
1. **DHCP Administrativo:** POOL-ADM entrega .11–.120 = **110** direcciones. El requisito es 120. Ni con .11–.126 alcanza (116). Para llegar sin cambiar la subred: reservar solo .1–.3 (gateway virtual y DSW1/DSW2) y entregar .4–.126 = **123**. No hay equipos estáticos en .4–.10 de VLAN 20.
2. **Pools al máximo de PT:** FIN (110 de 126), SUR (110), OCC (110) y WIFI-CORP (110) cubren la demanda actual. ING (240) y NORTE (240) también.
3. **serverPool por defecto** (172.30.4.0–172.30.5.255, gateway 0.0.0.0, 512) sigue activo y se traslapa con la red de servidores e invitados. Hay que acotarlo o dejarlo sin rango útil.
4. **CIDR no está en el Core:**
   - R-CORE tiene las rutas específicas.
   - R-EDGE tiene la estática 172.30.0.0/21 **y además** 14 rutas OSPF específicas. Por prefijo más largo, la /21 casi no se usa.
   - Por eso, quitarla no bloquea nada, y la frase «reemplaza 16 rutas» no se sostiene.
5. **Invitados sin redundancia:** VLAN 60 depende de DSW1 Fa0/10 → R-EDGE Gi0/2. Si DSW1 cae, invitados quedan sin servicio. Pendiente de probar.
6. **DHCP y DNS sin redundancia:** un solo SRV-DHCP (172.30.4.11) y un solo SRV-DNS (172.30.4.10).
7. **Port Security:** ASW-ING tiene `maximum 2`. Cambiar 1 PC con 1 MAC aprendida no necesariamente provoca violación. Hay que conectar una 3.ª MAC.
8. **Informe:** jitter con (máx−mín)/2, una sola serie de línea base, falta throughput FTP WAN, pruebas de Henry pendientes.

## Sin evidencia todavía
P-13, P-14, P-18, P-19, P-24, la parte web de P-22, la caída completa de DSW1 con efecto en invitados, la persistencia tras guardar y reabrir, la transferencia FTP por WAN y el ensayo de las 8 fallas.

## Limitaciones del simulador (excepciones para consultar al docente)
- No hay WPA3: se usa WPA2-PSK AES.
- No hay portal cautivo: se usa aislamiento por ACL.
- No hay IMAP: se usa POP3/SMTP.
- El pool DHCP de PT admite un solo DNS.
- Las ACL en SVI no persisten al guardar.
- 6 integrantes requeridos y el equipo tiene 4.
- 35 servidores representados por 7 equipos de muestra.
