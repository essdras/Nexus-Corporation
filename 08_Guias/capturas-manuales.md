# Capturas que debe tomar una persona (Nexus v6)

**Reglas:**
- Abrir `01_PKT/Nexus_Proyecto_Final.pkt` y esperar **5 minutos** a que OSPF converja.
- No guardar el archivo.
- Capturar con la herramienta de recortes (Win+Shift+S) y guardar en PNG con el nombre indicado.
- Si el resultado no coincide con lo esperado, capturar lo que salga y anotarlo. No repetir hasta que «salga bien» sin decirlo.

Las claves están en `credenciales-laboratorio.md` (solo para el grupo). No deben verse en las capturas.

## Prioridad 1: HTTP/HTTPS interno (P-13)
| # | Archivo | Equipo | Acción | Debe verse |
|---|---|---|---|---|
| C1 | `P13_http_interno.png` | PC-ADM-1 › Desktop › Web Browser | URL `http://www.nexus.com` › Go | Página «Nexus Corporation – Portal corporativo interno» con la URL visible |
| C2 | `P13_https_interno.png` | PC-NORTE-1 › Web Browser | URL `https://www.nexus.com` | Misma página, con `https://` en la barra |
| C3 | `P13_invitado_bloqueado.png` | LAP-INVITADO › Web Browser | `http://www.nexus.com` | La página no carga (*Request Timeout* o sin respuesta) |

## Prioridad 2: web externa publicada (P-14 / P-22)
| # | Archivo | Equipo | Acción | Debe verse |
|---|---|---|---|---|
| C4 | `P14_web_publica_http.png` | SRV-INTERNET › Desktop › Web Browser | `http://200.100.50.10` | Página de la empresa |
| C5 | `P14_web_publica_https.png` | SRV-INTERNET › Web Browser | `https://200.100.50.10` | Página de la empresa |
| C6 | `P22_ftp_bloqueado.png` | SRV-INTERNET › Command Prompt | `ftp 200.100.50.10` | Sin conexión (*timed out*) |
| C7 | `P22_contadores_acl.png` | R-EDGE › CLI (enable) | `show access-lists ACL-INTERNET-IN` después de C4–C6 | Coincidencias en las líneas `eq www` / `eq 443` y en `deny ip any any` |

## Prioridad 3: Wi-Fi (P-19)
| # | Archivo | Equipo | Acción | Debe verse |
|---|---|---|---|---|
| C8 | `P19_wifi_ok.png` | LAP-CORP › Desktop › PC Wireless › Link Information | Ver estado | Asociado a NEXUS-CORP, con señal |
| C9 | `P19_wifi_ip.png` | LAP-CORP › Command Prompt | `ipconfig` y `ping 172.30.6.1` | IP 172.30.6.x y respuestas 4/4 |
| C10 | `P19_wifi_clave_mala.png` | LAP-CORP › PC Wireless › Profiles | Editar el perfil con una clave errónea › Connect; luego `ipconfig` | Sin asociación, IP 0.0.0.0 o 169.254.x. **Después restaurar la clave correcta y comprobar el ping** |
| C11 | `P19_wifi_invitado.png` | LAP-INVITADO › Command Prompt | `ipconfig` y `ping 8.8.8.8` | IP 172.30.5.x y 4/4 |

## Prioridad 4: banderas TCP (P-18)
**Preparación:** pasar a modo **Simulation**. En *Edit Filters*, dejar solo TCP y HTTP (y DNS para C15).

| # | Archivo | Equipo | Acción | Debe verse |
|---|---|---|---|---|
| C12 | `P18_syn.png` | PC-ADM-1 › Web Browser `http://172.30.4.12`, luego *Capture/Forward* hasta que salga el 1.er TCP | Clic en el sobre (PDU) › *Outbound PDU Details* | Capa 4 TCP, puerto destino 80, bandera **SYN** |
| C13 | `P18_synack.png` | El PDU de vuelta desde SRV-WEB | *Inbound PDU Details* | Bandera **SYN + ACK** |
| C14 | `P18_ack.png` | El 3.er TCP desde PC-ADM-1 | PDU Details | Bandera **ACK**. Luego, la petición HTTP |
| C15 | `P18_dns_udp.png` | PC-ADM-1 › Command Prompt `ping www.nexus.com`, con el filtro DNS | PDU Details de la consulta | Capa 4 **UDP**, puerto 53, sin saludo previo |
| C16 | `P18_event_list.png` | Panel *Simulation* | Lista de eventos completa | Orden TCP, TCP, TCP y luego HTTP |

## Prioridad 5: fallos 5 y 6 (opcional, sobre una copia del .pkt)
Seguir la `guia-8-fallos.md`. Capturar el síntoma y la recuperación (`F5_sintoma.png`, `F5_recuperado.png`, `F6_sintoma.png`, `F6_recuperado.png`).
