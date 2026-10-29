# Pruebas de Gabriel: qué se usa y qué se repite

El original está en `05_Evidencias/pruebas_gabriel/PRUEBAS_Gabriel_original.docx`. Sus capturas se tomaron sobre la versión 5, con ING en 172.30.0.x y la ruta estática /21 en R-EDGE. El .pkt final usa ING en 172.30.7.0/24 y el resumen /21 se aprende por OSPF.

## Se usan tal como están
- P-02, P-06, P-07, P-08 y P-09: no dependen del direccionamiento de ING.
- FTP LAN vs WAN (1,558.6 kbps y 1,614.8 kbps, archivo asa842-k8.bin de 5,571,584 bytes). Se reporta como resultado funcional del simulador. No sirve como medición de rendimiento, porque PT ignora los 64 kbps del enlace.

## No se usa
- El jitter de la línea base 3.1, calculado como (máx − mín)/2. Se usa la línea base corregida de 900 muestras que ya está en el informe.

## Se repiten sobre `01_PKT/Nexus_Proyecto_Final.pkt` (abrir sin guardar)
| Prueba | Equipo | Comando o acción | Resultado esperado |
|---|---|---|---|
| P-01 | DSW1-SEDE y DSW2-SEDE | `show ip interface brief` | Vlan10 en 172.30.7.2 (DSW1) y 172.30.7.3 (DSW2), up/up |
| P-03 | R-EDGE | `show ip route` | `O IA 172.30.0.0/21` y la estática flotante con AD 120 que no aparece activa |
| P-04 | R-CORE y R-EDGE | `show ip ospf neighbor` y `show ip route ospf` | Vecinos en FULL y rutas O / O IA |
| P-10 | DSW1-SEDE, PC de ING | `show standby brief`, luego `shutdown` en Vlan10 de DSW1, ping continuo de la PC a 172.30.7.1 | DSW2 pasa a Active y el ping se recupera tras unas pocas pérdidas. Al final ejecutar `no shutdown` y **no guardar** |
| P-05 | Solo si usó una PC de ING | Repetir el mismo procedimiento con una PC que tenga IP 172.30.7.x | El mismo resultado que obtuvo Gabriel, con la IP nueva |

Cada captura se guarda en PNG en `05_Evidencias/pruebas_gabriel/` con el nombre `P-01.png`, `P-03.png`, etc.
