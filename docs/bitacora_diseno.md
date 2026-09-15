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


## Sesión 4: Optimización Booleana y Arquitectura NOR-AND sin Inversores 

### 1. Motivación y Análisis de Redundancia Posicional
La síntesis canónica de la Sesión 1 requería dos redes disjuntas de 9 entradas con compuertas AND inexistentes en la familia TTL comercial y un exceso de 11 inversores discretos.

Al comparar bit a bit el vector de Apertura ($413_5 = 011001100_2$) y el de Cierre ($142_5 = 010100001_2$):

| Variable | $P_8$ | $P_7$ | $P_6$ | $P_5$ | $P_4$ | $P_3$ | $P_2$ | $P_1$ | $P_0$ |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Abrir** | 0 | 1 | 1 | 0 | 0 | 1 | 1 | 0 | 0 |
| **Cerrar** | 0 | 1 | 0 | 1 | 0 | 0 | 0 | 0 | 1 |
| **Coincidencia** | **0** | **1** | $\neq$ | $\neq$ | **0** | $\neq$ | $\neq$ | **0** | $\neq$ |

Se identificaron cuatro literales comunes idénticos:
$$COMUN = \overline{P_8} \cdot \overline{P_4} \cdot \overline{P_1} \cdot P_7$$

Aplicando el **Teorema de De Morgan** a las tres variables negadas:
$$COMUN = \overline{(P_8 + P_4 + P_1)} \cdot P_7$$
Sintetizado físicamente con **1 compuerta NOR de 3 entradas** y **1 compuerta AND de 2 entradas**.

---

### 2. Eliminación Total de Inversores mediante Topología NOR-AND
Para evitar el uso de compuertas NOT en los literales restantes de cada clave, se aplicó sistemáticamente el principio de De Morgan ($\overline{X} \cdot \overline{Y} = \overline{X+Y}$), dividiendo la evaluación de cada clave en dos sub-bloques balanceados:

#### A. Rama de Apertura (`Abrir`):
- **Ceros restantes ($P_5, P_0$):** Se agrupan directamente en una compuerta NOR de 2 entradas:
  $$\text{NOR}_{abrir} = \overline{P_5 + P_0} = \overline{P_5} \cdot \overline{P_0}$$
- **Unos restantes ($P_6, P_3, P_2$):** Se agrupan en una compuerta AND de 3 entradas:
  $$\text{AND}_{abrir} = P_6 \cdot P_3 \cdot P_2$$
- **Etapa de Coincidencia Final:** Una compuerta AND multiplica las tres etapas:
  $$Abrir = COMUN \cdot \text{NOR}_{abrir} \cdot \text{AND}_{abrir}$$

#### B. Rama de Cierre (`Cerrar`):
- **Ceros restantes ($P_6, P_3, P_2$):** Se agrupan directamente en una compuerta NOR de 3 entradas:
  $$\text{NOR}_{cerrar} = \overline{P_6 + P_3 + P_2} = \overline{P_6} \cdot \overline{P_3} \cdot \overline{P_2}$$
- **Unos restantes ($P_5, P_0$):** Se agrupan en una compuerta AND de 2 entradas:
  $$\text{AND}_{cerrar} = P_5 \cdot P_0$$
- **Etapa de Coincidencia Final:** Una compuerta AND multiplica las tres etapas:
  $$Cerrar = COMUN \cdot \text{NOR}_{cerrar} \cdot \text{AND}_{cerrar}$$

---

### 3. Balance de Recursos y Ventajas de Implementación
1. **Cero inversores requeridos:** Se eliminaron los 11 chips NOT que exigía la forma canónica, reduciendo el ruido de conmutación y el enrutamiento en protoboard.
2. **Compatibilidad TTL directa:** El circuito se implementa en su totalidad con compuertas estándar de bajo conteo de entradas: