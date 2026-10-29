# P-18 · Traza de simulación HTTP (TCP) — 27/10/2026 22:40, copia v6 restaurada

**Método:** modo Simulation, reset de la lista de eventos. `HttpClient.go("http://172.30.4.12")` en PC-ADM-1 (por IP, para no mezclar DNS). 20 pasos y lectura de la lista de eventos con el lector de trazas de la extensión MCP. PT clasifica las tramas TCP como `type1` y la de la aplicación HTTP como `type17`.

| # evento | Tiempo sim. | Equipo / acción | Tipo | Interpretación |
|---|---|---|---|---|
| 0 | 555585 | PC-ADM-1, en búfer hacia 172.30.4.12 | TCP | Segmento retenido mientras se resuelve ARP |
| 1–15 | 555585–555588 | ARP broadcast desde PC-ADM-1 por ASW-ADM → DSW2-SEDE / DSW1-SEDE | ARP | Resolución del gateway (.1). PC-ADM-1 acepta la respuesta (evento 11) |
| 16 | 555589 | PC-ADM-1 envía a 172.30.4.12 | TCP | **1.er segmento: SYN** |
| 17–23 | 555589–555591 | ASW-ADM → DSW2-SEDE → ASW-DC (Fa0/3) | TCP | Enrutado por la SVI de DSW2-SEDE hacia VLAN 40 |
| 24 | 555592 | SRV-WEB responde | TCP | **2.º segmento: SYN-ACK** |
| 29–32 | 555593–555596 | ASW-DC → DSW2-SEDE → ASW-ADM → PC-ADM-1 (aceptado) | TCP | Llega el SYN-ACK |
| 34 | 555597 | PC-ADM-1 envía a 172.30.4.12 | TCP | **3.er segmento: ACK** |
| 35 | 555598 | PC-ADM-1 envía la petición | HTTP | Petición GET, después del saludo |

- **Conclusión:** antes de la primera trama HTTP hay tres segmentos TCP en el orden cliente → servidor → cliente. Es el saludo de tres vías. En DNS (P-18, parte UDP) la consulta salió sin saludo previo.
- **Límite de la evidencia:** el lector de trazas no devolvió las banderas TCP. El orden y la dirección de las tramas coinciden con SYN, SYN-ACK y ACK, pero ver las banderas exige abrir *PDU Details* en la interfaz de PT. Esa captura PNG la tiene que tomar una persona.
- Las tramas «dropped» en ASW-ING, ASW-FIN y ASW-DC son copias de *flooding* de unicast desconocido y de broadcast ARP descartadas en puertos sin destino. No son pérdidas del flujo.
