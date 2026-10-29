# Investigación: el resumen OSPF 172.30.0.0/21 no se reinstala de forma consistente (v6)

> **Actualización (23:40):** esta investigación llevó a la solución. Ingeniería se movió a 172.30.7.0/24 y el CIDR quedó cumplido en el .pkt final. Ver `07-verificacion-final.md`. Las «opciones» de abajo eran el estado antes de la decisión.

**Estado: CIDR NO cerrado.** La conectividad se mantiene por la estática flotante (AD 120), pero R-EDGE no recibe la /21 por OSPF después de reabrir el archivo.

## Pruebas (archivo v6 sin modificar, reabierto, sin `clear ip ospf process`)
| Ensayo | Momento | LSA tipo 3 en área 1 (R-CORE origina) | Lo que guarda R-EDGE | Ruta usada en R-EDGE |
|---|---|---|---|---|
| Reapertura 1 | ~2 min | Dos LSA con **el mismo Link State ID 172.30.0.0**: /21 (seq 8000002a, métrica 1) y /24 (seq 80000030, métrica 2) | Solo la /24 (seq 80000030) | 172.30.0.0/24 por OSPF; 172.30.4.10 (ADM/DC) por la **estática /21 AD 120** |
| Reapertura 1 | ~4 min | Igual | Solo la /24 (edad 234 s) | Igual: sin cambio |
| Reapertura 2 | ~2 min | — | Solo la /24 (seq 8000002d) | 172.30.0.0/24 por OSPF; 172.30.4.10 por la estática |
| Sesión anterior (antes de reabrir) | — | 1 LSA /24 (seq 8000004a) + 2 LSA /21 (seq 80000053) | Solo la /21 | `O IA 172.30.0.0/21` |

Evidencia original: `evidencias_v6/ospf/ospf1.txt`, `ospf2.txt`, `ospf_reopen1*.txt`, `ospf_reopen2.txt`.

## Causa (comprobada en las salidas, no supuesta)
1. Con `area 0 range 172.30.0.0 255.255.248.0`, R-CORE (ABR) **no suprime** la LSA de la red componente 172.30.0.0/24 (Ingeniería, VLAN 10): origina la /21 y también la /24 hacia el área 1. En IOS real la /24 se suprimiría.
2. Las dos LSA usan el **mismo Link State ID 172.30.0.0**. IOS real aplica el apéndice E del RFC 2328 (pone bits de host en el LSID para distinguir misma red con distinta máscara); PT 9 no lo hace.
3. En el área 1 queda como válida la instancia con número de secuencia más alto. Tras reabrir, la /24 se origina después y gana; R-EDGE descarta la /21. Por eso el resultado depende del orden de arranque y no es consistente.
4. Cualquier resumen que cubra Ingeniería empieza en 172.30.0.0, así que choca con la /24 de Ingeniería mientras esa red sea 172.30.0.0/24.

## Lo que se descartó
- `area 0 range 172.30.0.0 255.255.255.0 not-advertise`: PT responde *Invalid input*.
- Área 1 stub/NSSA: R-EDGE es ASBR (redistribuye la ruta por defecto); no aplica.
- `clear ip ospf process`: lo corrige al momento, pero el usuario lo prohibió como requisito, y además no es determinista en la siguiente reapertura.

## Opciones (decisión del equipo)
- **A. Mantener v6 y documentarlo como limitación de PT 9** (consulta al docente), con la estática flotante como respaldo de diseño. Sin riesgo para lo que ya funciona. La sumarización existe en R-CORE (`172.30.0.0/21 is a summary, Null0`), pero R-EDGE no siempre la aprende.
- **B. Renumerar Ingeniería** fuera de 172.30.0.0/24 (p. ej. dejar 172.30.0.0/24 libre o usarla para una red no OSPF). Elimina el choque, pero cambia SVI/HSRP de DSW1/DSW2, pools DHCP en los dos servidores, ACL, Excel e informe. Requiere nueva ronda de pruebas.
