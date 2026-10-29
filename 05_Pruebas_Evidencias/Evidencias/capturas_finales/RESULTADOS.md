# Capturas finales sobre el .pkt final (28/10/2026, 05:20–05:45 UTC)

**Archivo:** `C:/Users/grony/PacketTracerMCP/Nexus_Proyecto_v6.pkt`, abierto sin guardar.
- Su md5 difiere del de `01_PKT/Nexus_Proyecto_Final.pkt` porque se volvió a guardar el 28/10 a las 01:07.
- Las *startup-config* de los 15 equipos IOS son idénticas a `02_Configs`. Se comprobó con un diff antes de las pruebas.
- Se esperaron unos 7 minutos a que OSPF convergiera.

Las capturas se tomaron a resolución nativa de la pantalla y se recortaron a la región de interés.

## Pruebas de Gabriel repetidas

| Prueba | Captura | Resultado obtenido |
|---|---|---|
| P-01 | `P01_DSW1_ip_int_brief.png`, `P01_DSW2_ip_int_brief.png` | Vlan10 en 172.30.7.2 (DSW1) y 172.30.7.3 (DSW2), up/up. Todas las SVI están up/up. |
| P-03 / P-04 | `P03_P04_R-EDGE_rutas_OSPF.png` | R-EDGE aprende `O IA 172.30.0.0/21` por OSPF y tiene un vecino FULL (1.1.1.1). La estática flotante (AD 120) no aparece porque OSPF gana. |
| P-04 | `P04_R-CORE_vecinos_rutas_OSPF.png` | R-CORE tiene 6 vecinos FULL: DSW1, DSW2, R-EDGE y las tres sucursales. Ve las rutas O de todas las VLAN, incluida 172.30.7.0, y la ruta por defecto O*E2. |
| P-05 | `P05a_…primer_intento.png`, `P05b_…repeticion.png` | PC-ING-1 (172.30.7.17) hace ping a PC-ADM-2 (172.30.2.90) y a PC-FIN-1 (172.30.3.141) con TTL 127. En el primer intento hubo 1 y 2 pérdidas iniciales, ocurridas justo después de reactivar la Vlan10 de DSW1 en la prueba P-10. En la repetición fue 4/4 y 4/4. |
| P-10 | `P10a`…`P10d` | **Antes:** DSW1 es Active en el grupo 10 (VIP 172.30.7.1). **Falla:** `shutdown` en la Vlan10 de DSW1. **Después:** DSW2 pasa a Active en el grupo 10, y PC-ING-1 hace ping a 172.30.7.1 con 8/8 respuestas. El ping se lanzó después del cambio de rol, así que no mide la pérdida durante la conmutación. Al final se ejecutó `no shutdown`. |

## Capturas C1 a C16

| # | Captura | Resultado obtenido |
|---|---|---|
| C1 | `C1_P13_http_interno.png` | PC-ADM-2 carga `http://www.nexus.com`: «Nexus Corporation – Portal corporativo interno». Se usó PC-ADM-2 en lugar de PC-ADM-1; es la misma VLAN. |
| C2 | `C2_P13_https_interno.png` | La misma página carga por `https://www.nexus.com`. Antes se forzó un *Request Timeout* para comprobar que la página no venía de la caché. |
| C3 | `C3_P13_invitado_bloqueado.png` | LAP-INVITADO no carga `http://www.nexus.com` (*Request Timeout*). |
| C4 | `C4_P14_web_publica_http.png` | SRV-INTERNET carga `http://200.100.50.10`. Las dos primeras solicitudes terminaron en *Request Timeout*; la tercera cargó. |
| C5 | `C5_P14_web_publica_https.png` | SRV-INTERNET carga `https://200.100.50.10`. |
| C6 | `C6_P22_ftp_e_icmp_bloqueados.png` | Desde SRV-INTERNET, el ping a 200.100.50.10 recibe *Destination host unreachable* y `ftp 200.100.50.10` termina en *Timed out*. |
| C7 | `C7_P22_contadores_acl.png` | ACL-INTERNET-IN: `eq www` 51, `eq 443` 10, `established` 129, `echo-reply` 10 y `deny ip any any` 41 coincidencias. |
| C8 | `C8_P19_wifi_ok.png` | LAP-CORP asociado a NEXUS-CORP, WPA2-Personal, IP 172.30.6.87, gateway 172.30.6.1. |
| C9 | `C9_P19_wifi_ip.png` | `ipconfig` muestra 172.30.6.87/25, y el ping a 172.30.6.1 recibe 4/4 respuestas. |
| C10 | `C10a_…sin_asociacion.png`, `C10b_…ipconfig.png` | Con la clave errónea (aplicada por script y reiniciando la laptop), aparece «No association with access point». La IP queda en 0.0.0.0 y el ping a 172.30.6.1 tiene 0/4 respuestas. Después se restauró la clave y la laptop recuperó 172.30.6.87. |
| C11 | `C11_P19_wifi_invitado.png` | LAP-INVITADO tiene 172.30.5.11 con gateway 172.30.5.1. El ping a 8.8.8.8 dio 2/4 en el primer intento y 4/4 en la repetición. |
| C12 | `C12_P18_syn.png` | Outbound PDU de PC-ADM-2: TCP 1032 → 80, SEQ 0, ACK 0, FLAGS 0b00000010 (**SYN**). |
| C13 | `C13_P18_synack.png` | Outbound PDU de SRV-WEB: TCP 80 → 1032, SEQ 0, ACK 1, FLAGS 0b00010010 (**SYN + ACK**). |
| C14 | `C14_P18_ack.png` | Outbound PDU de PC-ADM-2: SEQ 1, ACK 1, FLAGS 0b00010000 (**ACK**). |
| C15 | `C15_P18_dns_udp.png` | PC-ADM-1 envía una consulta DNS tipo A con capa 4 **UDP**, puerto origen 1025 y destino 53, hacia 172.30.4.10. No hay saludo previo. |
| C16 | `C16_P18_event_list.png` | Lista de eventos con filtro TCP+HTTP: primero los segmentos TCP del saludo y después el primer evento **HTTP** desde PC-ADM-2. También aparecen copias inundadas por los switches (*unknown unicast flooding*). |

**Bits de FLAGS en PT** (de derecha a izquierda): FIN, SYN, RST, PSH, ACK. Por eso 0b10 es SYN, 0b10010 es SYN+ACK y 0b10000 es ACK.
