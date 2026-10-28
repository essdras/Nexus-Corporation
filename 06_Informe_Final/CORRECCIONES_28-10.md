# Corrección final del informe (28/10/2026)

Base: versión de 88 páginas (respaldo en respaldos/informe_antes_correccion_final/). Resultado: 96 páginas.
No se modificaron el .pkt ni las configuraciones, y no se repitió ninguna prueba.

## Contenido
- Resumen ejecutivo: usa el texto pedido sobre v5, v6 y el archivo definitivo; FLSM = 8 bloques (6 enlaces en uso y 2 de reserva); 9 servidores (8 del DC + SRV-INTERNET).
- 3.3 y conclusión 1: HSRP mantiene el gateway; quedan puntos únicos en el núcleo y el perímetro.
- 3.5 y tabla 8: separa la caída completa de DSW1 (v6, registro en texto) de P-10 (solo la SVI Vlan10, figuras 14–16).
- 8.5: 3 × 50 × 6 = 900 paquetes, rotación PC-ING-1/2/3, 3 flujos, tiempos simulados, «<1 ms» = 0. Variación media entre RTT consecutivos como aproximación del jitter; ecuación de Word; exclusión de pares con pérdida (48/47 pares, comprobado con el CSV). Pérdidas reales: respuesta 1 (ADM), 3 (NORTE) y 1 (OCC). Figuras 29–34 = serie previa, solo de referencia.
- 9.1, 9.2 y conclusiones: 1.ª y 2.ª ronda, 7 de 8 con síntoma, fallo 3 sin corte, fallo 4 sin pérdida documentada; ensayo identificado como copia de v6.
- Restauración DHCP: 172.30.0.11/170 y .181/70 (v6) y 172.30.7.11/170 y .181/70 (final).
- Anexo C fallo 7: procedimiento probado en 6 pasos. Anexo A-8: la violación ocurre al aparecer una MAC no permitida con el máximo alcanzado.
- Anexo D: ensayo frente a defensa presencial; figuras 41–45 para DHCP/DNS de respaldo y SRV-DB; Port Security sin captura.

## Precisión
- ARP: «compatible con» y separación entre observaciones e hipótesis (8.7, figs. 46 y 48, 4.3, 8.5, P-21, P-23).
- FTP: frase pedida sobre el caudal de 64 kbps (1.4 y 8.5).
- SRV-DB: «acceso IP/ICMP» (P-21, 8.7, figs. 44–45, conclusiones).
- DNS: solo se comparó el estado del servicio y la cantidad de registros; se nombra www.nexus.com como el nombre probado.
- WPA2, portal cautivo e IMAP/POP3 se describen como sustitutos parciales, no equivalentes.
- Las capturas C1–C16 y P-01/03/04/05/10 se identifican como tomadas en la «copia de trabajo con la configuración final» (v6.pkt guardado de nuevo 28/10 01:07).
- Se corrigieron las referencias desfasadas de P-11 a P-17 (figuras 17–23) y las evidencias de P-20, P-22 y P-23 (antes nombres de .jpg).

## Formato
- Ecuaciones LaTeX → ecuaciones de Word (4.5 y 8.5).
- Tablas 8.1–8.5 y del capítulo 9 en páginas horizontales con columnas más anchas: ya no se cortan IPs, máscaras ni URLs; IP y máscara unidas con espacio no separable.
- Títulos APA «Tabla N» + título en cursiva para las 35 tablas; referencias «tabla 4.1/9.1» actualizadas.
- Número de página arriba a la derecha (no tenía).
- «Nota.» en cursiva; título, imagen y nota se mantienen juntos; 4 capturas altas reducidas para que no quede el título solo en una página.
- Anexo B: rutas que coinciden con el .zip; nota sobre entrega_final/ = Nexus_Entrega_Final/.

## Pendiente (sin inventar)
- Aporte de Esdras (portada y matriz): sigue «—» / «Sin tareas asignadas».
- Versión del .pkt usada por Henry en las figuras 17–23: no registrada.
- Responsables del ensayo de fallos en la bitácora (dice «Gabriel y Henry»): confirmar.
- Tiempos de resolución de la tabla 32: no se cronometraron.
- Capturas faltantes o de baja resolución: P-24 (Port Security), P-21 en v6, caída completa de DSW1; las figuras 24–34 (v6 y serie previa) son de baja resolución.

# Cierre (28/10/2026, 16:40) — 100 páginas
- Esdras: «Elaboración del informe técnico, junto con José Andres Chinchilla García» (portada y matriz), según andres.
- Bitácora: el ensayo de fallos se atribuye a «Todo el equipo» (andres: «al final todos participamos»).
- Figuras 17–23 de Henry: «archivo final, según indicó el equipo» (andres confirmó que usaron la última versión).
- Referencias con sangría francesa de 0.5 in; el Anexo A empieza en una página nueva.
- Las filas de las tablas de pruebas ya no se parten entre páginas.
- Port Security (P-24): capturas tomadas el 28/10 sobre una copia del archivo definitivo (sin guardar), figuras 60–62 y fila nueva en la tabla 30; configuración restaurada y ping 4/4. PNG en 05_Evidencias/capturas_finales/P24/. Informe: 102 páginas.

## Capturas rehechas (28/10/2026, 17:15)
- Figuras 24 a 34 reemplazadas por capturas nítidas de José Andres Chinchilla García sobre una copia del archivo definitivo (no guardada): P-20, P-22 (ping/ftp y contadores ACL), P-23 (SSH permitido y rechazado) y series de 50 pings a los 6 destinos. Originales en 05_Evidencias/capturas_finales/rehechas/ (recortes usados en el informe: rehechas/informe/).
- Notas, títulos de las figuras 29 a 34, texto de 8.5 (fuente de las figuras), 8.6 y Tabla 26 (P-20, P-22, P-23) actualizados con los valores de las nuevas capturas. La tabla de línea base sigue siendo la de 900 pings de v6.
- Respaldo previo: respaldos/informe_antes_rehechas/. Informe: 108 páginas.

## Tabla P-24 (28/10/2026, 17:25)
- Fila P-24 (Tabla 26, pág. 36): resultado histórico de v6 conservado y separado de la repetición sobre copia del archivo definitivo (figuras 60 a 62); evidencia separada por ensayo. Fila completa en una página, sin desborde. Respaldo: respaldos/informe_antes_p24_tabla/.

## Formato APA 7 (28/10/2026, 17:45)
- Texto a doble espacio sin espacio extra entre párrafos, sangría de 1,27 cm en la primera línea, títulos nivel 1 centrados en negrita y nivel 2 a la izquierda en negrita (mismo tamaño que el texto), portada centrada sin el rótulo «Portada», resumen en página nueva, Referencias y cada anexo en página nueva. Tablas a espacio sencillo (APA lo permite). Informe: 118 páginas. Respaldo: respaldos/informe_antes_apa/.
