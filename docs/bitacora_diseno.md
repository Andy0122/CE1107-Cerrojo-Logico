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

## Sesión 3: Síntesis del Decodificador de Anuncios y Visualización 

### 1. Estandarización de Señales de Control
Se renombró la señal original de control temporal `FIN` a `Listo`, alineándose con la semántica del sistema para indicar que el usuario ha terminado de ingresar la contraseña.

### 2. Definición de Variables de Estado Intermedias (*One-Hot*)
Para evitar una síntesis combinacional compleja de 7 salidas independientes a partir de 3 entradas no alineadas (`Listo`, `Abrir`, `Cerrar`), se decodifican tres variables intermedias mutuamente excluyentes:
- $S_A = Abrir \cdot Listo$ (Condición de apertura confirmada)
- $S_C = Cerrar \cdot Listo$ (Condición de cierre confirmada)
- $S_E = \overline{(Abrir + Cerrar)} \cdot Listo$ (Condición de clave inválida / error)

Cuando $Listo = 0$, el sistema se encuentra en reposo o lectura ($S_A = 0, S_C = 0, S_E = 0$).

---

### 3. Tabla de Verdad y Mapeo hacia Segmentos (Cátodo Común)

Para representar los caracteres requeridos (`L`, `A`, `C`, `E`), se construyó la siguiente tabla de mapeo de encendido binario (1 = encendido, 0 = apagado):

| Estado | Carácter | Listo | Abrir | Cerrar | $S_A$ | $S_C$ | $S_E$ | a | b | c | d | e | f | g |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Leyendo** | **L** | 0 | X | X | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 1 | 1 | 0 |
| **Abriendo** | **A** | 1 | 1 | 0 | 1 | 0 | 0 | 1 | 1 | 1 | 0 | 1 | 1 | 1 |
| **Cerrando** | **C** | 1 | 0 | 1 | 0 | 1 | 0 | 1 | 0 | 0 | 1 | 1 | 1 | 0 |
| **Inválida** | **E** | 1 | 0 | 0 | 0 | 0 | 1 | 1 | 0 | 0 | 1 | 1 | 1 | 1 |

---

### 4. Deducción Analítica Columna por Columna

Analizando el comportamiento binario de cada segmento a partir de la tabla anterior:

1. **Segmentos $e$ y $f$:**  
   Sus columnas son un vector de solo unos: `[1, 1, 1, 1]^T`. Al ser constantes lógicas en todos los estados requeridos, no demandan compuertas:
   $$e = 1 \; (V_{DD}), \quad f = 1 \; (V_{DD})$$

2. **Segmentos $b$ y $c$:**  
   Sus columnas presentan el vector `[0, 1, 0, 0]^T`, el cual es estrictamente idéntico a la columna de la variable de estado $S_A$:
   $$b = S_A, \quad c = S_A$$

3. **Segmento $d$:**  
   Presenta el vector `[1, 0, 1, 1]^T`. Se observa que es exactamente el complemento lógico (inverso) de la columna $S_A$ (`[0, 1, 0, 0]^T`):
   $$d = \overline{S_A}$$

4. **Segmento $g$:**  
   Presenta el vector `[0, 1, 0, 1]^T`. Solo está activo cuando $S_A = 1$ (estado 'A') o cuando $S_E = 1$ (estado 'E'):
   $$g = S_A + S_E$$

5. **Segmento $a$:**  
   Presenta el vector `[0, 1, 1, 1]^T`. Se enciende en cualquiera de los tres estados activos tras confirmación:
   $$a = S_A + S_C + S_E$$

### 5. Conclusión de Recursos
Esta deducción matemática reduce el costo de implementación del módulo a:
- 1 compuerta NOR de 2 entradas y 3 compuertas AND de 2 entradas (para generar $S_A, S_C, S_E$).
- 1 compuerta NOT (para el segmento $d$).
- 3 compuertas OR de 2 entradas (para los segmentos $g$ y $a$).
- Conexión directa a $V_{DD}$ para $e$ y $f$.