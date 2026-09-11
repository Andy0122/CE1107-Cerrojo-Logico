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

## Sesión 2: Incorporación de Memoria de Estado mediante Latch SR 

### 1. Justificación del Latch SR
Para retener la condición física de la puerta tras la validación de la contraseña, se implementó un biestable Latch SR con compuertas NOR discretas con ecuaciones características:
$$Q_{t+1} = \overline{R + \overline{Q_t}}, \quad \overline{Q_{t+1}} = \overline{S + Q_t}$$
- La señal $Abrir$ excita la entrada $S$ (Set $\rightarrow Q=1$).
- La señal $Cerrar$ excita la entrada $R$ (Reset $\rightarrow Q=0$).

### 2. Exclusión de la Condición Crítica Indeterminada
El estado inválido $S=1, R=1$ queda excluido matemáticamente por diseño: dado que la clave de apertura ($413_5$) y la de cierre ($142_5$) son mutuamente excluyentes a nivel combinacional, resulta físicamente imposible que ambas señales se activen en simultáneo.