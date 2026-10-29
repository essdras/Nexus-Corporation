# Verificación final sobre el archivo definitivo (28/10/2026, 06:15–06:55 UTC)

## Archivo probado

| Dato | Valor |
|---|---|
| Archivo | `01_PKT/Nexus_Proyecto_Final.pkt` (277 393 bytes) |
| MD5 | `4fbc73cbdc35736905827d69ed734600` |
| SHA-256 | `519dfe7a1c89b9861bcebcf172e86494725c283ea82abda9cb82df5b16117e07` |
| Copia de trabajo en el PC | `C:\Users\grony\PacketTracerMCP\verificacion_final\Nexus_Proyecto_Final.pkt` (mismo MD5) |
| Copia para pruebas de falla | `Nexus_Proyecto_Final_COPIA_FALLAS.pkt` (mismo MD5, sin usar) |

- El archivo se abrió **sin guardar** durante las pruebas. Al terminar, su MD5 seguía igual.
- Antes de probar se comprobó la convergencia OSPF. R-CORE tenía 6 vecinos FULL: DSW1, DSW2, R-EDGE y las tres sucursales.
- Las capturas son PNG de la ventana real de Packet Tracer, a la resolución nativa de la pantalla. Están en `capturas/`.

## 1. Ingeniería con DHCP de respaldo

**Procedimiento:**
1. Se apagó el servicio DHCP de SRV-DHCP (172.30.4.11).
2. En PC-ING-1 se ejecutó `ipconfig /renew`.
3. Se comprobaron la resolución de nombres y el acceso web.
4. Se volvió a encender el servicio DHCP.

| Campo | Resultado |
|---|---|
| IP | **172.30.7.187** (pool POOL-ING-B de SRV-DHCP-2, rango .181–.250) |
| Máscara | 255.255.255.0 |
| Gateway | 172.30.7.1 |
| DNS | 172.30.4.17 (SRV-DNS-2) |
| Servidor DHCP | **172.30.4.16** (antes 172.30.4.11) |
| nslookup www.nexus.com | Dos *DNS request timed out* en la primera consulta. En el reintento resolvió **172.30.4.12**, y una segunda consulta respondió directamente. |
| ping www.nexus.com | 3/4. La primera solicitud se perdió. |
| Web http://www.nexus.com | Carga «Nexus Corporation – Portal corporativo interno» |

- **Capturas:** `V1a`, `V1b` y `V1c`.
- **Restauración:** SRV-DHCP quedó otra vez habilitado.
- **Observación:** la línea `ipconfig /release` aparece como *Invalid Command* porque se cortó un carácter. La concesión nueva igual demuestra el respaldo: el campo *DHCP Servers* cambió de .11 a .16.

## 2. Seguridad tras renumerar Ingeniería

| Prueba | Origen | Destino | Resultado |
|---|---|---|---|
| SSH autorizado | PC-ING-1 **172.30.7.187** | DSW1-SEDE 172.30.4.78 (`ssh -l admin`) | Entra a `DSW1-SEDE#`. Lo permite ACL-GESTION, línea 20 (`permit 172.30.7.0 0.0.0.255`). |
| Bloqueo a SRV-DB | PC-ING-1 **172.30.7.187** | SRV-DB 172.30.4.15 | 0/4 (100 % de pérdida) |
| Acceso permitido a SRV-DB | PC-FIN-1 **172.30.3.218** | SRV-DB 172.30.4.15 | 3/4 en el primer intento y **4/4** en la repetición |

- **Firewall de SRV-DB en este archivo:** ACL 101 con `permit ip 172.30.3.128 0.0.0.127 any` y `permit ip 172.30.6.128 0.0.0.31 any`. Todo lo demás queda denegado de forma implícita.
- **Capturas:** `V2a` y `V2b`.

## 3. Conectividad estable (50 pings)

**Preparación:**
- Se verificaron los vecinos OSPF y la ruta 172.30.7.0/24 en R-SUR y R-OCCIDENTE: O, métrica 66, por Serial0/0/0.
- Luego se enviaron 4 pings de preparación y se ejecutó `arp -a`.

| Prueba | Preparación (4 pings) | 50 pings |
|---|---|---|
| PC-SUR-1 (172.30.2.140) → 172.30.7.1 | 3/4. Se perdió el **2.º** paquete. | **50/50 (0 %)**. Mín. 1, máx. 30, prom. 5 ms |
| PC-ING-1 (172.30.7.187) → PC-OCC-1 (172.30.3.12) | 3/4. Se perdió el **1.º** paquete. | **50/50 (0 %)**. Mín. 0, máx. 30, prom. 2 ms |

**Análisis de las pérdidas de preparación.** Cada causa se comprobó con evidencia; no se atribuyeron automáticamente a ARP.

- **SUR → 172.30.7.1.** La primera respuesta llegó, así que la pérdida no se debe al ARP del PC.
  - R-CORE tiene **dos rutas de igual costo** a 172.30.7.0/24: vía DSW1 (172.30.4.78) y vía DSW2 (172.30.4.82). Reparte los paquetes entre ellas.
  - DSW2 está en *Standby* para el grupo HSRP 10, por lo que no posee la IP virtual. Para entregar el paquete a 172.30.7.1 tuvo que resolverla por ARP.
  - Su tabla ARP mostraba la entrada 172.30.7.1 → 0000.0C07.AC0A con **edad de 2 minutos**, es decir, creada justo durante la preparación.
  - El 2.º paquete fue el primero que tomó el camino por DSW2 y se perdió mientras se resolvía ese ARP.
- **ING → OCC.** Se perdió el 1.er paquete.
  - R-OCCIDENTE mostraba 172.30.3.12 en su tabla ARP con **edad de 2 minutos**, creada durante la preparación.
  - La pérdida corresponde a la resolución ARP del último salto, en el router de la sucursal.
- Una vez creadas esas entradas, las dos series de 50 no tuvieron pérdidas.

**Capturas:** `V3a`, `V3b`, `V3c` y `V3d`.

## 4. Comprobación del archivo definitivo

| Punto | Resultado en este .pkt | Captura |
|---|---|---|
| Resumen CIDR | R-EDGE: `O IA 172.30.0.0/21 [110/2] via 172.30.4.85`, vecino 1.1.1.1 FULL. Se volvió a ver después de reabrir el archivo al final. | `V4a` |
| DNS principal | PC-ADM-1 (172.30.2.90, DHCP .11, DNS .10). `nslookup www.nexus.com` → 172.30.4.12; la primera consulta tuvo un *timeout*. | `V4b` |
| HTTP interno | PC-ADM-1 carga `http://www.nexus.com` | `V4c` |
| HTTPS interno | PC-ADM-1 carga `https://www.nexus.com` (HttpClient con `isHttps=true`) | `V4d` |
| HTTP externo | SRV-INTERNET carga `http://200.100.50.10` | `V4e` |
| HTTPS externo | SRV-INTERNET carga `https://200.100.50.10` (`isHttps=true`) | `V4f` |
| Wi-Fi corporativo | LAP-CORP conectado a NEXUS-CORP con WPA2-Personal. IP 172.30.6.87/25, gateway 172.30.6.1, DNS 172.30.4.17. El ping al gateway dio 4/4. | `V4g`, `V4h` |
| Aislamiento de invitados | LAP-INVITADO (172.30.5.11/24, gateway 172.30.5.1) hacia 172.30.4.12, 172.30.7.14 y 172.30.7.187: *Destination host unreachable* desde 172.30.5.1 (0/4 cada uno). Hacia 8.8.8.8: 4/4. | `V4i` |
| Web interna desde invitados | `http://www.nexus.com` → *Request Timeout* | `V4j` |

### Comparación con el archivo de las capturas anteriores

Las capturas C1–C16 y P-01/03/04/05/10 de `05_Evidencias/capturas_finales` se tomaron sobre `Nexus_Proyecto_v6.pkt`, no sobre este archivo. Ese .pkt se volvió a guardar el 28/10 a las 01:07.

Para compararlos se abrieron los dos archivos y se leyó por script el estado de los equipos que no son IOS:
- **Servidores:** pools y parámetros DHCP, estado y número de registros DNS, HTTP/HTTPS, reglas del firewall de host e IP/máscara/gateway/DNS de los nueve servidores.
- **Wi-Fi:** SSID, autenticación y cifrado de las dos laptops.

**Resultado:** los nueve servidores son **idénticos** en ambos archivos. Coinciden la huella de pools DHCP, DNS (encendido, 10 registros), HTTP/HTTPS, el firewall de SRV-DB (2 reglas) y la configuración de puertos. Las laptops tienen el mismo SSID, autenticación 4 (WPA2-PSK) y cifrado 4 (AES): NEXUS-CORP en LAP-CORP y NEXUS-INVITADOS en LAP-INVITADO.

Solo varía el estado dinámico: la IP obtenida por DHCP y si la laptop ya estaba asociada en el momento de leer. Las configuraciones IOS ya se habían comparado antes y son idénticas a `02_Configs`.

Además, **todas las pruebas de esta sección se repitieron sobre el archivo definitivo**, así que las capturas `V4*` pueden sustituir a C1–C5, C8, C9 y C11 si se quiere que todo provenga del mismo .pkt.

**Limitación:** la comparación de registros DNS cubre el estado del servicio y la cantidad de registros, no el texto de cada registro, porque la API de PT no lo expone. La resolución correcta de `www.nexus.com` → 172.30.4.12 se comprobó funcionalmente en este archivo, con los dos servidores DNS.

## Estado final

- SRV-DHCP quedó otra vez habilitado. Ningún otro servicio se modificó.
- No se guardó ningún .pkt: el archivo definitivo conserva el MD5 `4fbc73cb…`.
- Packet Tracer quedó con `Nexus_Proyecto_Final.pkt` abierto, recién recargado desde disco. OSPF convergido: 6 vecinos FULL en R-CORE y `O IA 172.30.0.0/21` en R-EDGE.
- No se repitieron la línea base ni el ensayo de fallos, porque no se cambió ninguna configuración.
