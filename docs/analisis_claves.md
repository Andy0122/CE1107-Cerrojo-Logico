# Análisis de Codificación de Claves en Base 5

**Proyecto:** Cerrojo Digital con Lógica Combinacional  
**Curso:** CE 1107 — Fundamentos de Arquitectura de Computadores  

---

## 1. Criterio de Selección de Contraseñas (No Trivialidad)

La especificación del proyecto establece el uso de un código de 3 dígitos ingresado mediante el conteo de dedos de una mano ($0$ a $4$ dedos), lo que define un espacio de estados en **base 5** con $5^3 = 125$ combinaciones posibles.

Para cumplir con la directriz de que la contraseña sea **poco trivial**, se descartaron patrones predecibles:
- Combinaciones monótonas o repetitivas ($000_5, 111_5, 222_5, \dots$).
- Secuencias aritméticas ascendentes o descendentes ($012_5, 123_5, 234_5, 432_5$).
- Patrones simétricos o tipo espejo ($121_5, 242_5$).

### Claves Seleccionadas:
1. **Clave de Apertura:** $413_5$ (4 dedos $\rightarrow$ 1 dedo $\rightarrow$ 3 dedos)
2. **Clave de Cierre:** $142_5$ (1 dedo $\rightarrow$ 4 dedos $\rightarrow$ 2 dedos)

### Justificación de Seguridad y Robustez Lógica:
- **Dispersión:** No comparten el dígito inicial ni final.
- **Distancia de Hamming:** Presentan una separación en binario de 6 bits diferentes entre sí, lo que evita conmutaciones espurias o falsos disparos por rebotes de un solo bit.
- **Exclusión Mutua:** Garantiza matemáticamente que las señales $Abrir$ y $Cerrar$ nunca puedan activarse de forma simultánea, protegiendo el Latch SR de caer en el estado prohibido ($S=1, R=1$).

---

## 2. Mapeo Posicional y Codificación Binaria

Dado que 5 estados ($0 \dots 4$) requieren al menos 3 bits ($2^3 = 8$), cada dígito en base 5 se codifica individualmente en binario natural ($D_2 D_1 D_0$). El bus completo de la contraseña consta de 9 bits ($P_8 \dots P_0$), organizados en orden de llegada (Dígito 1 = menos significativo, Dígito 3 = más significativo):

### Clave de Apertura ($413_5$):
- Dígito 1 ($4_{10}$): $100_2 \rightarrow P_2 P_1 P_0$
- Dígito 2 ($1_{10}$): $001_2 \rightarrow P_5 P_4 P_3$
- Dígito 3 ($3_{10}$): $011_2 \rightarrow P_8 P_7 P_6$
- **Vector Binario ($P_8 \dots P_0$):** `0 1 1 0 0 1 1 0 0`

### Clave de Cierre ($142_5$):
- Dígito 1 ($1_{10}$): $001_2 \rightarrow P_2 P_1 P_0$
- Dígito 2 ($4_{10}$): $100_2 \rightarrow P_5 P_4 P_3$
- Dígito 3 ($2_{10}$): $010_2 \rightarrow P_8 P_7 P_6$
- **Vector Binario ($P_8 \dots P_0$):** `0 1 0 1 0 0 0 0 1`

---

## 3. Minitérminos Canónicos Iniciales

Antes de cualquier proceso de reducción algebraica, las funciones de detección canónicas corresponden a compuertas AND de 9 variables:

- $Abrir = \overline{P_8} \cdot P_7 \cdot P_6 \cdot \overline{P_5} \cdot \overline{P_4} \cdot P_3 \cdot P_2 \cdot \overline{P_1} \cdot \overline{P_0}$
- $Cerrar = \overline{P_8} \cdot P_7 \cdot \overline{P_6} \cdot P_5 \cdot \overline{P_4} \cdot \overline{P_3} \cdot \overline{P_2} \cdot \overline{P_1} \cdot P_0$