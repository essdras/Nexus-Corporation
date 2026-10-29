# Guía definitiva: ensayo de los 8 fallos (Nexus v6)

**Reglas generales:**
- Trabajar sobre una **copia** (`Nexus_ensayo.pkt`) y nunca guardar el fallo en el archivo final.
- Antes de empezar, esperar 5 minutos de convergencia y anotar `ipconfig` de los PC a usar (las IP cambian al reabrir).
- Después de cada fallo, restaurar y comprobar antes de pasar al siguiente.
- Los resultados entre corchetes son los que se obtuvieron el 27/10/2026 (evidencia en `05_Evidencias/`).

| # | Provocar | Síntoma esperado | Diagnóstico | Restaurar | Comprobar |
|---|---|---|---|---|---|
| 1 | DSW2-SEDE: `interface Vlan20` › `ip address 172.30.2.3 255.255.255.192` | Los PC de ADM fuera de .0–.63 (p. ej. .94) no salen de su red [PC-ADM-2 0/8; PC-ADM-1 7/8] | `show ip interface brief` / `show running-config` en DSW2-SEDE: máscara /26 | `ip address 172.30.2.3 255.255.255.128` | PC-ADM-2 `ping 172.30.4.12` [8/8] |
| 2 | PC-NORTE-1: IP estática 172.30.1.50/24, gateway 172.30.1.254 | Llega a su LAN pero no a SRV-WEB [0/4] | `ipconfig` en el PC: gateway incorrecto | Volver a DHCP y `ipconfig /renew` | IP 172.30.1.x, gateway .1, `ping 172.30.4.12` [4/4] |
| 3 | R-CORE: `router ospf 1` › `no area 0 range 172.30.0.0 255.255.248.0` | **Según el PDF:** las sucursales quedan sin acceso. **En el diseño final no ocurre:** OSPF anuncia las rutas específicas y el tráfico sigue [SUR e ING→Internet 6/6] | `show ip route ospf` en R-EDGE: 14 rutas `O IA` específicas en lugar de la /21 | `area 0 range 172.30.0.0 255.255.248.0` | R-EDGE vuelve a mostrar solo `O IA 172.30.0.0/21` [en 45 s]. Ping a Internet desde una sucursal |
| 4 | ASW-ING: `interface Gi0/1` › `switchport trunk native vlan 1` | Mensajes `%CDP-4-NATIVE_VLAN_MISMATCH` en ASW-ING y DSW1-SEDE | `show interfaces trunk` (nativa 1 contra 999) | `switchport trunk native vlan 999` | Ya no aparece el aviso; `show interfaces trunk` muestra 999 en los dos extremos |
| 5 | DSW1-SEDE **y** DSW2-SEDE: `ip access-list extended ACL-MALA` con `deny udp any host 172.30.4.10 eq domain`, `deny udp any host 172.30.4.17 eq domain` y `permit ip any any`; luego `interface Vlan10` › `ip access-group ACL-MALA in` | PC-ING-1 (172.30.7.x): `ping www.nexus.com` → *could not find host*, mientras `ping 172.30.4.12` responde [4/4]. Solo falla DNS | `show access-lists ACL-MALA` (suben las coincidencias del deny) y `show ip interface Vlan10` | En las dos SVI `no ip access-group ACL-MALA in` y luego `no ip access-list extended ACL-MALA` | `ping www.nexus.com` [2/2]; `show ip interface Vlan10`: *Inbound access list is not set* |
| 6 | SRV-DHCP › DHCP › POOL-ING: inicio 172.30.7.1, máximo 3. SRV-DHCP-2 › POOL-ING-B: igual. **Guardar en los dos.** Luego, en PC-ING-1, `ipconfig /release` y `ipconfig /renew` | *DHCP request failed*, IP 0.0.0.0. El pool solo tiene .1–.3, que son el gateway HSRP y los switches | Revisar los pools en los dos servidores | POOL-ING inicio 172.30.7.11, máx. 170; POOL-ING-B inicio 172.30.7.181, máx. 70 | `ipconfig /renew` da 172.30.7.x; `ping www.nexus.com` [4/4] |
| 7 | PC-FIN-1 › Config › FastEthernet0: cambiar la MAC a 0001.0001.0001 y hacer ping; luego cambiarla a 0002.0002.0002 y hacer ping | ASW-FIN Fa0/1 pasa a **err-disabled**, *Violation Count 1* | `show port-security interface fa0/1` | Devolver la MAC original al PC. En ASW-FIN: `no switchport port-security mac-address sticky 0001.0001.0001`, luego `shutdown` / `no shutdown` en Fa0/1 | Fa0/1 *Secure-up*; PC-FIN-1 hace ping a su gateway [3/4] |
| 8 | R-EDGE: `ip access-list extended ACL-INTERNET-IN` › `5 deny tcp any host 200.100.50.10 eq www` | SRV-INTERNET no carga `http://200.100.50.10` | `show access-lists ACL-INTERNET-IN`: la línea 5 suma coincidencias [11] | `ip access-list extended ACL-INTERNET-IN` › `no 5` | La web carga desde SRV-INTERNET |

## Diferencia pendiente del fallo 3
- El enunciado espera que quitar la sumarización bloquee las sucursales. En OSPF eso no pasa, porque el ABR vuelve a anunciar las redes específicas, y la conectividad sigue (evidencia `05_Evidencias/cidr/v7_fallo3_*.txt`, copia de prueba con la misma configuración de enrutamiento).
- Lo que sí se observa es que la tabla de R-EDGE pasa de 1 ruta a 14. Es el efecto real de perder la sumarización.
- **Requiere aceptación del docente.** Si pide un corte, habría que quitar también la estática flotante de R-EDGE, y aun así OSPF mantiene las rutas específicas. No se fuerza un corte con otro cambio, porque eso ya no sería el fallo 3.
