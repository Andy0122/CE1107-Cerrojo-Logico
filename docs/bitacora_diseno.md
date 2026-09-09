# Bitácora de Diseño y Pruebas de Laboratorio

## Sesión 1: Deducción Canónica de Claves

### 1. Implementación Inicial en Logisim
Se sintetizaron de forma canónica las funciones para las dos contraseñas en módulos independientes (`Decodificador_Abrir` y `Decodificador_Cerrar`):
- **Apertura ($413_5$):** Vector `011001100`. Se implementó un minitérmino con inversores para los bits en bajo ($P_8, P_5, P_4, P_1, P_0$) conectados a una compuerta AND de 9 entradas.
- **Cierre ($142_5$):** Vector `010100001`. Se implementó el minitérmino negando ($P_8, P_6, P_4, P_3, P_2, P_1$) hacia una compuerta AND de 9 entradas.

### 2. Limitaciones Identificadas (Problema de Ingeniería)
1. **Inexistencia física en TTL:** No existen comercialmente compuertas AND de 9 entradas en la serie 74LS (el circuito integrado con mayor cantidad de entradas es el `74LS21`, que cuenta únicamente con 4 entradas por compuerta).
2. **Sobrecarga de integrados:** Utilizar compuertas NOT discretas para cada cero demandaría 11 inversores (2 chips `74LS04` completos), lo que saturaría el espacio de protoboard e incrementaría el riesgo de conexiones falsas.
*Conclusión de la sesión:* Se requiere una optimización algebraica posterior mediante el teorema de De Morgan y extracción de términos comunes.