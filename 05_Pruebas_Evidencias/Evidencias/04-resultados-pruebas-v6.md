# Resultados de pruebas sobre v6 (27/10/2026, después de reabrir el archivo)

## DHCP secundario: serverPool documentado
| Servidor | serverPool | Gateway / DNS |
|---|---|---|
| SRV-DHCP (172.30.4.11) | 172.30.4.62, 1 dirección | .1 / .10 |
| SRV-DHCP-2 (172.30.4.16) | 172.30.4.61, 1 dirección | .1 / .17 |

- **Direcciones usadas en 172.30.4.0/26:**
  - .1 (HSRP), .2 (DSW1) y .3 (DSW2).
  - .10 a .15 (servidores) y .16, .17 (respaldo).
- Ningún equipo de VLAN 40 usa DHCP. Los serverPool no se traslapan entre sí ni con direcciones usadas.
- Los pools de usuarios están divididos sin traslape (ver registro de cambios).
- Esto es un **respaldo parcial**:
  - Si cae el primario, el secundario solo tiene ~30 % de cada pool.
  - Los clientes del primario reciben un solo DNS (.10), porque PT no permite dos DNS por pool.

## P-13 · HTTP y HTTPS internos (TCP 80 / 443)
- **Método:** proceso HttpClient del navegador de PT. El navegador no se puede capturar por control de pantalla.

| Origen | URL | Resultado |
|---|---|---|
| PC-ADM-1 | http://www.nexus.com | Página «Nexus Corporation – Portal corporativo interno» |
| PC-OCC-1 | http://www.nexus.com | Misma página |
| PC-NORTE-1 | https://www.nexus.com | Misma página (HTTPS) |
| PC-ING-1 | https://www.nexus.com | 1.er intento sin respuesta en 20 s; 2.º intento, página cargada |
| LAP-INVITADO | http://www.nexus.com | Sin contenido (ACL-INVITADOS bloquea la red interna) |
| LAP-INVITADO | http://internet.com | Página de SRV-INTERNET (salida a Internet permitida) |

**Estado: CUMPLE.** Falta la captura de pantalla del navegador, que debe tomar una persona.

## P-14 y P-22 (parte web) · Publicación por NAT estático y perímetro
| Prueba desde SRV-INTERNET | Resultado | Contador de ACL-INTERNET-IN |
|---|---|---|
| http://200.100.50.10 | Carga la página de la empresa | eq www: 7 coincidencias |
| https://200.100.50.10 | Carga la página | eq 443: de 5 a 10 durante la prueba |
| ping 200.100.50.2 | *Destination host unreachable*, 0/4 | — |
| ftp 200.100.50.10 | *Timed out* | deny ip any any: de 4 a 15 |

**Estado: CUMPLE.** Desde Internet solo se publican 80 y 443 del servidor web, y el resto cae en el deny final.

## Conectividad estable después de reabrir
- **Al reabrir el archivo, los PC vuelven a pedir DHCP y cambian de IP.** Muchos reciben dirección del tramo del SRV-DHCP-2.
  - Por eso las pruebas deben usar las IP actuales (`ipconfig`) o nombres DNS.
  - Tres series iniciales fallaron 50/50 porque apuntaban a IP antiguas que ya no existían. Repetidas con las IP correctas: 0 % de pérdida.
- **10 series de 50 pings después de la convergencia, 500 paquetes en total, 0 % de pérdida:**
  - ADM→web.
  - FIN→Internet.
  - OCC→Internet.
  - Invitado→Internet.
  - ING→OCC.
  - NORTE→SUR.
  - SUR→NORTE.
  - Más 3 series adicionales de prueba.
- Evidencia: `evidencias_v6/series_v6_s1*.txt`. Tablas de rutas: `configs_final/*_rutas.txt`.
- **Pérdidas que se investigaron:**
  - En la línea base, el **1.er paquete** de 3 series se perdió, y solo ese.
  - En modo simulación, la traza mostró que DSW2 **descartó la primera consulta DNS** de PC-ADM-1 y en seguida emitió un ARP para 172.30.4.10. Es decir, el primer paquete se pierde mientras el equipo de capa 3 resuelve ARP. Queda **verificado en un caso**.
  - Las demás pérdidas aparecieron solo durante la convergencia de OSPF tras abrir el archivo o tras un cambio.
- **Observación pendiente de causa confirmada:**
  - Al reabrir el archivo o al volver a aplicar `area 0 range`, R-EDGE puede quedarse sin la ruta `O IA 172.30.0.0/21`, aunque R-CORE sí muestra `172.30.0.0/21 is a summary, Null0`.
  - Hipótesis: choque del identificador de LSA 172.30.0.0, compartido con la LAN de Ingeniería, en la implementación de PT 9.
  - La estática flotante (AD 120) mantiene la conectividad. **No se usa `clear ip ospf process` como paso requerido.**

## Línea base corregida (3 series × 6 destinos × 50 pings = 900 paquetes)
- **Condiciones de la medición:**
  - Origen: VLAN 10, con PC-ING-1, PC-ING-2 y PC-ING-3 en rotación.
  - 3 flujos ICMP simultáneos.
  - Sin otro tráfico generado.
  - Tiempo real de PT tras más de 10 min de convergencia.
- **Jitter** = media de |RTTᵢ − RTTᵢ₋₁| en cada serie, promediado entre las 3 series.
- **«<1 ms» se registra como 0.**
- Muestras completas en `evidencias_v6/linea_base_muestras.csv`. Resumen por serie en `linea_base_resumen.csv`.

| Destino | Enviados | Perdidos | Mín (ms) | Prom (ms) | Máx (ms) | Jitter (ms) |
|---|---|---|---|---|---|---|
| ADM (PC-ADM-1) | 150 | 1 | 0 | 1.91 | 32 | 3.01 |
| WEB (SRV-WEB) | 150 | 0 | 0 | 2.23 | 24 | 3.02 |
| NORTE (PC-NORTE-1) | 150 | 1 | 1 | 4.74 | 107 | 5.52 |
| SUR (PC-SUR-1) | 150 | 0 | 0 | 3.31 | 27 | 3.72 |
| OCC (PC-OCC-1) | 150 | 1 | 0 | 3.01 | 26 | 3.73 |
| Internet (8.8.8.8) | 150 | 0 | 0 | 3.29 | 36 | 5.01 |

**Limitación:** los tiempos de PT son simulados. Sirven para comparar entre destinos, no como medida física.

## FTP por WAN (PC-NORTE-1 → SRV-FTP 172.30.4.13)
### a) Transferencia funcional
- `get c2950-i6q4l2-mz.121-22.EA4.bin` por el enlace serie R-NORTE–R-CORE: **completa, 3,058,048 bytes**, igual al tamaño del archivo en el servidor. Usuario FTP autenticado (230 Logged in).
- **CUMPLE** como prueba funcional. Se hizo dos veces (con y sin *Fast Forward*) y las dos terminaron sin error.

### b) Medición de rendimiento (no representativa)
| Corrida | Fast Forward | Tiempo según PT | Caudal según PT |
|---|---|---|---|
| 1 (27/10, 1.ª ronda) | Sí | 20.521 s | 149,020 B/s (≈1.19 Mbps) |
| 2 (27/10 22:35, copia v6 restaurada) | **No** | 18.517 s | 165,148 B/s (≈1.32 Mbps) |

- A 64 kbps reales, 3,058,048 bytes tardarían ≈382 s. PT reporta ≈20 veces más rápido: **no aplica el `clock rate 64000` al caudal TCP**.
- El tiempo que reporta PT es tiempo simulado. En la corrida 2, el reloj del equipo anfitrión registró entre 27 s y 131 s de espera, sin precisión mayor.
- Conclusión: la cifra solo sirve como resultado del simulador. No se compara con un enlace físico ni con el FTP LAN anterior (374.6 kbps).
- Evidencia: `evidencias_v6/ftp_wan_sin_ff.txt`.

## P-19 · Autenticación Wi-Fi (WPA2-PSK AES)
| Estado | Resultado |
|---|---|
| Clave correcta (antes) | LAP-CORP asociada al AP 0002.1733.11AE, IP 172.30.6.87 |
| Clave incorrecta | Sin AP asociado, IP 0.0.0.0, ping a 172.30.6.1 0/4 |
| Clave restaurada | Asociada al mismo AP, IP 172.30.6.14, ping a 172.30.6.1 4/4 |

Es WPA2 y no WPA3: PT 9 no ofrece WPA3 en este AP. Queda como excepción a consultar.

## P-24 · Port Security (máximo 2, sticky, modo shutdown) en ASW-FIN Fa0/1
| Paso | Resultado |
|---|---|
| Inicial | Secure-up, 1 MAC aprendida (la de PC-FIN-1) |
| 2.ª MAC (0001.0001.0001) | Permitida: ping 4/4, Total MAC 2, sin violación (coincide con el máximo de 2) |
| 3.ª MAC (0002.0002.0002) | **Secure-shutdown, err-disabled**, Violation Count 1 |
| Restaurar | MAC original, se quita la sticky de prueba, `shutdown` / `no shutdown`; PC-FIN-1 vuelve a responder (3/4) |

## P-18 · TCP contra UDP
- **UDP (DNS):** la traza de simulación confirmó que la consulta DNS sale **sin saludo previo**. En un caso, la primera consulta se descartó por ARP.
- **TCP (HTTP):** con PC-ADM-1 → http://172.30.4.12 en modo Simulation, se ven tres segmentos TCP antes de la petición HTTP: cliente → SRV-WEB, SRV-WEB → cliente y cliente → SRV-WEB. El orden coincide con SYN / SYN-ACK / ACK. Detalle por evento en `evidencias_v6/p18_traza_http_tcp.md`.
- **Falta:** captura PNG de *PDU Details* con las banderas TCP. El lector remoto no expone las banderas y la captura de ventanas de PT no está disponible a distancia.

## Ensayo de los 8 fallos
**Archivo:** copia `v6_ensayo_fallos.pkt`, no se guardó. Se restauró y comprobó después de cada fallo.

| # | Fallo provocado | Síntoma observado | Diagnóstico | Solución | Verificación |
|---|---|---|---|---|---|
| 1 | DSW2 VLAN 20 con /26 | PC-ADM-1 (.5) 7/8; PC-ADM-2 (.94, fuera de la /26) **0/8** hacia la web | `show running-config` / `show ip interface brief` en DSW2 | Máscara /25 | PC-ADM-2 8/8 |
| 2 | PC-NORTE-1 estática .50, gateway .254 | Ping a su gateway real 4/4; a SRV-WEB **0/4** | `ipconfig` | Volver a DHCP + `/renew` | IP .181, gateway .1, ping 4/4 |
| 3 | R-CORE `no area 0 range` | **No bloquea.** R-EDGE pasa a 13 rutas O IA específicas y la ING sale por la estática flotante. SUR→Internet 4/4 | `show ip route ospf` en R-EDGE | Volver a `area 0 range` | R-CORE muestra el resumen; R-EDGE no reinstaló la /21 y siguió por la estática (ver observación) |
| 4 | ASW-ING Gi0/1 con nativa 1 | `%CDP-4-NATIVE_VLAN_MISMATCH` en ASW-ING y DSW1 | `show interfaces trunk` (Gi0/1 nativa 1) | Nativa 999 | Corregido |
| 5 | ACL-MALA (deny DNS a .10 **y a .17**) entrante en Vlan10 de DSW1-SEDE y DSW2-SEDE (2.ª ronda; en la 1.ª solo .10 quedó enmascarado por el DNS de respaldo) | PC-ING-1 (172.30.0.12): `ping www.nexus.com` → *could not find host*; `ping 172.30.4.12` **4/4**. Solo falla la resolución | `show access-lists ACL-MALA`: deny .10 con 2→4 coincidencias | `no ip access-group` en las dos SVI y borrar la ACL | `ping www.nexus.com` 2/2; Vlan10 sin ACL. Evidencia `evidencias_v6/fallos/falla5_*.txt` |
| 6 | POOL-ING (SRV-DHCP) y POOL-ING-B (SRV-DHCP-2) con inicio .1 y 3 direcciones, que chocan con HSRP .1, DSW1 .2 y DSW2 .3 (2.ª ronda; en la 1.ª solo se cambió el primario) | PC-ING-1 `ipconfig /release` + `/renew`: **DHCP request failed** (dos intentos), IP 0.0.0.0 | Revisar los pools en los dos servidores (inicio .1, máx. 3) | Restaurar .11/170 y .181/70 | `/renew` da 172.30.0.181, DNS .17; `ping www.nexus.com` 4/4. Evidencia `falla6_*.txt` |
| 7 | Cambiar 2 veces la MAC de PC-FIN-1 | Fa0/1 err-disabled, 1 violación | `show port-security interface fa0/1` | MAC original, quitar la sticky de prueba, shut / no shut | Ping 3/4 |
| 8 | ACL-INTERNET-IN línea 5 deny www | La web 200.100.50.10 no carga; la línea 5 suma 11 coincidencias | `show access-lists ACL-INTERNET-IN` | `no 5` | La web carga |

**Fallos 5 y 6 (2.ª ronda, 27/10 22:30):** con DHCP/DNS de respaldo hay que provocar el fallo en los dos servidores. Así ya producen síntoma observable. En el fallo 6, PT no entregó las direcciones del gateway y los switches: el síntoma es que el cliente se queda sin IP, no un conflicto de IP duplicada.

**Fallo 3 frente al enunciado:** el PDF espera que quitar la sumarización deje sin acceso a las sucursales. Con OSPF, al quitar `area range` se anuncian las rutas específicas y la conectividad sigue. Se mantiene como diferencia explícita y consulta al docente. Ver también `06-investigacion-ospf.md`.
