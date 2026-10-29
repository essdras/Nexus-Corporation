# Cambios de diseño sobre Nexus_Proyecto_v6.pkt

Antes de cada cambio se guarda una copia (`v6_antes-cambioN.pkt`). Si una prueba falla, se reabre esa copia y se descartan las modificaciones.

## Cambio 1 · DHCP Administrativo y serverPool — **APLICADO Y VERIFICADO**
| Equipo | Antes | Después |
|---|---|---|
| SRV-DHCP, POOL-ADM | .11–.120 (110) | **172.30.2.4–.126 (123)** |
| SRV-DHCP, serverPool (por defecto) | 172.30.4.0–172.30.5.255, 512, gateway 0.0.0.0 | 172.30.4.62 único, gateway 172.30.4.1. PT no permite borrarlo: lo recrea al reabrir. |

- **Direcciones libres:** se revisaron todas las IP estáticas del .pkt. En 172.30.2.0/25 solo están ocupadas .1 (HSRP), .2 (DSW1) y .3 (DSW2).
- **Prueba:**
  - `ipconfig /release` y `/renew` en PC-ADM-1, PC-ADM-2 y PC-ADM-3. Recibieron .4, .8 y .5, con máscara /25, gateway .1 y DNS .10.
  - Ping hacia SRV-WEB, Occidente y 8.8.8.8: 0 % de pérdida en la 2.ª ronda.
  - Se guardó, se reabrió y los pools siguen igual.
- **Vuelta atrás:** POOL-ADM a inicio .11 y máximo 110.

## Cambio 2 · CIDR en el Core
| Equipo | Comandos |
|---|---|
| R-CORE | `router ospf 1` → `no network 172.30.4.64 0.0.0.31 area 0` · `network 172.30.4.64 0.0.0.15 area 0` · `network 172.30.4.80 0.0.0.3 area 0` · `network 172.30.4.84 0.0.0.3 area 1` · **`area 0 range 172.30.0.0 255.255.248.0`** |
| R-EDGE | `router ospf 1` → redes 172.30.4.84/30 y 172.30.5.0/24 pasan a **area 1** · `no ip route 172.30.0.0 255.255.248.0 172.30.4.85` |

- R-CORE queda como ABR y anuncia hacia R-EDGE una sola ruta, `O IA 172.30.0.0/21`, en lugar de las 14 específicas actuales.
- La ruta por defecto E2 de R-EDGE sigue llegando a toda la red.
- No cambia ninguna dirección.
- **Prueba:**
  - `show ip route` en R-EDGE y R-CORE, antes y después.
  - Ping desde ING, ADM y las tres sucursales hacia 8.8.8.8.
  - Acceso de SRV-INTERNET a la web publicada 200.100.50.10.
  - Invitado → Internet.
- **Falla 3:** se quita `area 0 range` y se muestra qué pasa. Con OSPF reaparecen las rutas específicas y **no** se bloquean las sucursales. Se documenta tal cual y queda como consulta al docente.
- **Vuelta atrás:** se invierten los comandos, o se reabre `v6_antes-cambio2.pkt`.

## Cambio 3 · DHCP y DNS redundantes
| Equipo | Configuración |
|---|---|
| **SRV-DHCP-2** (nuevo) | 172.30.4.16/26, gateway .1, conectado a ASW-DC Fa0/7 (VLAN 40) |
| **SRV-DNS-2** (nuevo) | 172.30.4.17/26, conectado a ASW-DC Fa0/8, con los mismos registros A que SRV-DNS |
| DSW1, DSW2 (SVI 10, 20, 30, 50), R-NORTE, R-SUR, R-OCCIDENTE (LAN) y R-EDGE Gi0/2 | segundo `ip helper-address 172.30.4.16` |
| R-EDGE ACL-INVITADOS | se agrega `permit udp any host 172.30.4.17 eq domain` antes del `deny` |

- **Cómo funciona el respaldo:**
  - PT no tiene failover DHCP, así que se usa *split-scope*. Cada pool se divide en dos tramos sin traslape: SRV-DHCP entrega cerca del 70 % y SRV-DHCP-2 el 30 % restante.
  - Ejemplo en ADM: .4–.89 en el primario y .90–.126 en el secundario.
  - El relay envía cada solicitud a los dos servidores y el cliente acepta la primera oferta. Si cae el primario, el secundario sigue entregando direcciones de su tramo.
  - Los pools de SRV-DHCP-2 entregan DNS 172.30.4.17.
  - Límite de PT: cada pool entrega un solo DNS, así que un cliente servido por el primario solo conoce el DNS1. Queda documentado.
- **Prueba:**
  - Se apaga el servicio DHCP de SRV-DHCP.
  - `ipconfig /renew` en un PC: debe recibir una IP del tramo secundario.
  - Se apaga el DNS1: `nslookup` / `ping` por nombre desde ese PC debe resolver con DNS2.
  - Al final se encienden los dos servicios otra vez.
- **Vuelta atrás:** se borran los dos servidores y la línea `helper-address .16` (o se reabre la copia).

## Cambio 4 · Invitados sin depender de DSW1 (versión simplificada)
- **Situación actual:** AP-GUEST → ASW-ING Fa0/20 → troncales → DSW1 Fa0/10 → R-EDGE Gi0/2. Si cae DSW1, los invitados quedan sin salida.
- **Cambio:** se mueve **un solo cable**. R-EDGE Gi0/2 pasa de DSW1 Fa0/10 a **ASW-ING** (un puerto libre en `switchport access vlan 60`, sin port-security).
  - El tráfico de invitados ya no pasa por la distribución.
  - Dependen solo de ASW-ING y R-EDGE, los mismos de los que ya depende el AP.
  - No hace falta módulo nuevo ni apagar R-EDGE.
- **Aislamiento:** se mantiene igual. El gateway sigue en R-EDGE Gi0/2 (172.30.5.1) con ACL-INVITADOS en una interfaz física, que sí persiste al guardar. No se usan SVI con ACL. El relay DHCP de invitados sale por R-CORE hacia DSW1 o DSW2, así que sobrevive a la caída de DSW1.
- **Prueba:**
  - Se apaga DSW1 por completo.
  - Desde LAP-INVITADO: `ipconfig /renew`, ping a 8.8.8.8 (debe responder) y ping a PC-ING-1 (debe fallar).
  - Se repite tras guardar y reabrir.
- **Vuelta atrás:** se devuelve el cable a DSW1 Fa0/10.

## Cierre
1. Se guarda v6 final y se reabre.
2. Se repiten las pruebas principales: conectividad, DHCP, DNS, aislamiento, SSH y web publicada.
3. Se exportan las configuraciones finales a `configs_final/`.
