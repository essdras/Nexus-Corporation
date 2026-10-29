# P-24 Port Security sobre copia del archivo definitivo (28/10/2026)
Archivo: Nexus_Proyecto_Final_COPIA_FALLAS.pkt (mismo MD5 que 01_PKT/Nexus_Proyecto_Final.pkt), abierto sin guardar. ASW-FIN Fa0/1, PC-FIN-1 (172.30.3.141, MAC 0040.0B41.1161).
- P24a: Secure-up, máximo 2, 1 sticky, 0 violaciones.
- P24b: 2.ª MAC 0001.0001.0001, ping al gateway 4/4, 2 sticky, 0 violaciones.
- P24c/P24d/P24g: 3.ª MAC 0002.0002.0002 → PSECURE_VIOLATION, Secure-shutdown, err-disabled, Violation Count 1, ping 0/4.
- P24e/P24f/P24h: MAC original, sticky 0001.0001.0001 borrada, shutdown/no shutdown → Secure-up, 1 sticky, 0 violaciones, connected; ping gateway 4/4; SRV-WEB 2/4 y luego 4/4.
