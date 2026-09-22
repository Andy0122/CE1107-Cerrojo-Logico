# Bitácora de Diseño: Módulo de Entrada, Sensores y Codificador a 3 Bits

**Fecha:** 18 de septiembre de 2026  
**Autores:** Juan Pablo Araya Rodríguez, Dylan Elizondo Picado  
**Módulo:** Captura Sensorial y Codificación Combinatoria  

---

## 1. Modelo Sensorial y Naturaleza Acumulativa (Código Termómetro)

El sistema captura la contraseña mediante el conteo físico de dedos extendidos ($0$ a $4$) sobre una matriz de 4 fotoresistencias (LDRs, $L_0 \dots L_3$). Cada LDR opera en un divisor de tensión con resistencia de pull-up de $10\text{ k}\Omega$ a $5\text{ V}$.

A nivel biomecánico y ergonómico, el conteo manual presenta una naturaleza acumulativa (*thermometer code*):
- 0 dedos: ningún sensor cubierto.
- 1 dedo: se cubre $L_0$.
- 2 dedos: se cubren $L_0$ y $L_1$.
- 3 dedos: se cubren $L_0, L_1$ y $L_2$.
- 4 dedos: se cubren $L_0, L_1, L_2$ y $L_3$.

### Visualizador Auxiliar de Entrada:
En paralelo con la salida de cada sensor, se conectó un LED con resistencia limitadora de $330\text{ }\Omega$, permitiendo validar gráficamente la postura de la mano antes de enviar el dato.

---

## 2. Tabla de Verdad del Mapeo Sensorial

La traducción del vector de 4 líneas sensoriales ($L_3 L_2 L_1 L_0$) al código binario natural de 3 bits ($D_2 D_1 D_0$) para representar la base 5 ($0 \dots 4$) se rige por la siguiente tabla de estados:

| Dedos extendidos | $L_3$ | $L_2$ | $L_1$ | $L_0$ | Valor Decimal | $D_2$ | $D_1$ | $D_0$ |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **0 dedos** | 0 | 0 | 0 | 0 | **0** | 0 | 0 | 0 |
| **1 dedo**  | 0 | 0 | 0 | 1 | **1** | 0 | 0 | 1 |
| **2 dedos** | 0 | 0 | 1 | 1 | **2** | 0 | 1 | 0 |
| **3 dedos** | 0 | 1 | 1 | 1 | **3** | 0 | 1 | 1 |
| **4 dedos** | 1 | 1 | 1 | 1 | **4** | 1 | 0 | 0 |

---

## 3. Deducción Algebraica y Síntesis por Bits

En lugar de recurrir a mapas de Karnaugh con don't cares que sobrecargarían el uso de compuertas dispersas, se dedujeron las funciones mínimas aprovechando las propiedades funcionales de la serie TTL:

### A. Bit Menos Significativo ($D_0$) mediante Árbol de Paridad XOR
Al analizar la secuencia de la columna $D_0$:
$$D_0 \in \{0, 1, 0, 1, 0\}$$
Se evidencia que $D_0$ se activa exclusivamente cuando la cantidad de unos presentes en el vector de entrada es impar (1 o 3 dedos activos). 

Aprovechando la propiedad asociativa de la función XOR (detector de paridad impar):
$$D_0 = L_0 \oplus L_1 \oplus L_2 \oplus L_3$$

Dado que no existen compuertas XOR comerciales de 4 entradas en la serie 74LS, se sintetizó mediante un árbol de tres compuertas XOR de 2 entradas disponibles dentro de un único integrado `74LS86`:
$$D_0 = (L_0 \oplus L_1) \oplus (L_2 \oplus L_3)$$

- **Evaluación analítica:**
  - 0 dedos (`0000`): $0 \oplus 0 = \mathbf{0}$.
  - 1 dedo (`0001`): $1 \oplus 0 = \mathbf{1}$.
  - 2 dedos (`0011`): $0 \oplus 0 = \mathbf{0}$.
  - 3 dedos (`0111`): $0 \oplus 1 = \mathbf{1}$.
  - 4 dedos (`1111`): $0 \oplus 0 = \mathbf{0}$.

### B. Bit Medio ($D_1$) mediante Inhibición Lógica
La columna $D_1$ presenta la secuencia $\{0, 0, 1, 1, 0\}$, debiendo estar activa únicamente para 2 y 3 dedos.
Bajo la condición termométrica:
- El estado mínimo de activación lo marca la presencia del segundo dedo ($L_1 = 1$).
- El estado debe extinguirse cuando se alcanzan los 4 dedos ($L_3 = 1$).

Por tanto, se formula como la habilitación por $L_1$ inhibida por la presencia de $L_3$:
$$D_1 = L_1 \cdot \overline{L_3}$$
Sintetizada con **1 inversor NOT** (`74LS04`) y **1 compuerta AND de 2 entradas** (`74LS08`).

### C. Bit Más Significativo ($D_2$) mediante Coincidencia Total
El bit $D_2$ opera exclusivamente en el valor decimal 4 ($D_2 = 1$). Por tanto, responde a la conjunción total de todos los sensores activados:
$$D_2 = L_0 \cdot L_1 \cdot L_2 \cdot L_3$$
Sintetizada mediante una compuerta AND de 4 entradas (`74LS21`) o dos compuertas AND en cascada.

---

## 4. Recursos de Hardware Empleados
- 3 compuertas XOR de 2 entradas (utiliza 3 de las 4 compuertas del integrado `74LS86`).
- 1 compuerta NOT (utiliza 1 de 6 inversores del `74LS04`).
- 1 compuerta AND de 2 entradas (utiliza 1 de 4 compuertas del `74LS08`).
- 1 compuerta AND de 4 entradas.