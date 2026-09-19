# Bitácora de Diseño: Serializador con CI TTL 74153 y Contador Módulo-3 Síncrono

**Curso:** CE 1107 — Fundamentos de Arquitectura de Computadores  
**Fecha:** 19 de septiembre de 2026  
**Autores:** Juan Pablo Araya Rodríguez, Dylan Elizondo Picado  
**Módulo:** Conversión Paralelo-Serial e Indexación Temporal de Dígitos  

---

## 1. Justificación de la Arquitectura y Selección del Integrado

La especificación del proyecto impone una interfaz de comunicación estrictamente serial (un solo cable) entre la etapa de censado y el decodificador combinatorio. Para convertir los 3 bits paralelos del dígito ($D_2 D_1 D_0$) a una trama temporal:

### Decisión de Diseño: Descarte del MUX Nativo de Logisim y Adopción del CI `74153`
Inicialmente se evaluó el bloque multiplexor genérico provisto por Logisim Evolution. No obstante, su manejo de buses de selección y polarizaciones abstractas no se correspondía con la topología de cableado físico en protoboard. 

Para asegurar una correspondencia 1:1 entre el diseño esquemático y el montaje final, se adoptó directamente el circuito integrado de la librería TTL: **`74LS153`** (Doble Multiplexor / Selector de datos de 4 a 1 líneas con encapsulado DIP de 16 pines).

---

## 2. Contador Módulo-3 Síncrono (2 Flip-Flops D + 1 NOR + 1 AND)

Para evitar la saturación de pines de conexión hacia otros submódulos, el contador indexador se implementó dentro del mismo lienzo del serializador. El circuito requiere recorrer cíclicamente tres estados para seleccionar ordenadamente $D_0$, luego $D_1$, y finalmente $D_2$:
$$\text{Secuencia de Conteo: } 00_2 \rightarrow 01_2 \rightarrow 10_2 \rightarrow 00_2$$

### A. Tabla de Excitación y Transición de Estados
Utilizando dos flip-flops tipo D ($FF_1, FF_0$) que representan los bits de estado ($Q_1, Q_0$):

| Estado Presente ($Q_1 Q_0$) | Bit Indexado | Estado Siguiente ($Q_1^+ Q_0^+$) | Entrada $D_1$ requerida | Entrada $D_0$ requerida |
| :---: | :---: | :---: | :---: | :---: |
| **`00`** | $D_0$ (Bit 0) | **`01`** | 0 | 1 |
| **`01`** | $D_1$ (Bit 1) | **`10`** | 1 | 0 |
| **`10`** | $D_2$ (Bit 2) | **`00`** | 0 | 0 |
| **`11`** *(Estado Ilegal)* | — | **`00`** *(Recuperación)*| 0 | 0 |

### B. Ecuaciones Lógicas Mínimas de Realimentación
A partir de la tabla, se sintetizó la lógica de control con solo dos compuertas discretas:

1. **Para la entrada $D_0$ (Compuerta NOR de 2 entradas):**  
   $D_0$ debe ponerse en alto únicamente cuando el estado actual es `00`:
   $$D_0 = \overline{Q_1 + Q_0}$$
2. **Para la entrada $D_1$ (Compuerta AND de 2 entradas):**  
   $D_1$ se activa únicamente cuando se transita desde el estado `01`:
   $$D_1 = Q_0 \cdot \overline{Q_1}$$

### C. Ventaja de Robustez (Auto-Recuperación)
Si por transitorios de encendido el contador cayera en el estado no utilizado `11`:
- $D_0 = \overline{1 + 1} = 0$
- $D_1 = 1 \cdot \overline{1} = 0$
El siguiente pulso de reloj forzará al contador inmediatamente al estado válido `00`, eliminando cualquier posibilidad de bloqueo (*lock-up*).

---

## 3. Configuración y Conexionado del Circuito Integrado `74153`

Se utilizó únicamente la primera sección (Sección 1) del integrado. Para garantizar el correcto funcionamiento del chip en lógica TTL y evitar fallos por líneas flotantes, se aplicó la siguiente disciplina de conexión:

```text
                           74LS153
                     +-------v-------+
   (GND) 1/G (Strobe)| 1          16 | VCC (+5V)
    (Q0) B (Select 0)| 2          15 | 2/G (Strobe Canal 2 -> VCC)
 (GND) 1C3 (Dato 3)  | 3          14 | A (Select 1) (Q1)
  (D2) 1C2 (Dato 2)  | 4          13 | 2C3 (No usado -> GND)
  (D1) 1C1 (Dato 1)  | 5          12 | 2C2 (No usado -> GND)
  (D0) 1C0 (Dato 0)  | 6          11 | 2C1 (No usado -> GND)
 (Línea Serial) 1Y   | 7          10 | 2C0 (No usado -> GND)
                GND  | 8           9 | 2Y  (No usado)
                     +---------------+
```

### Justificación de Terminales Críticos:
1. **Pin 1 ($1\overline{G}$ - Strobe / Enable del Canal 1):**  
   Este pin opera con **lógica activa en bajo**. Es fundamental conectarlo permanentemente a **Tierra (GND)**. Si se deja flotando o a nivel alto, el multiplexor inhibe sus compuertas internas y fuerza la salida serial a `0` constante.
2. **Pin 3 ($1C_3$ - Cuarta entrada de datos):**  
   Dado que solo se serializan 3 bits ($D_0, D_1, D_2$), la cuarta entrada no se utiliza. En tecnología TTL dejar entradas al aire puede provocar conmutaciones espurias; por tanto, **se derivó directamente a GND (0 lógico)**.
3. **Pines de Selección (Pin 14: Select B, Pin 2: Select A):**  
   Controlados por los bits del contador: Select A conectado a $Q_0$ y Select B conectado a $Q_1$.
4. **Pines de Alimentación:**  
   Pin 16 a $V_{CC}$ (5 V) y Pin 8 a GND.
5. **Sección 2 no utilizada:**  
   Para evitar consumo innecesario y ruido térmico en la pastilla de silicio, el pin $2\overline{G}$ (Pin 15) se deshabilitó conectándolo a $V_{CC}$, y sus entradas de datos a tierra.

---

## 4. Dinámica de Salida Serial en Línea $1Y$

A medida que el botón de reloj inyecta pulsos en los flip-flops:
- **Ciclo 0 ($Q_1 Q_0 = 00$):** Selectores en `00` $\rightarrow$ Salida $1Y = D_0$.
- **Ciclo 1 ($Q_1 Q_0 = 01$):** Selectores en `01` $\rightarrow$ Salida $1Y = D_1$.
- **Ciclo 2 ($Q_1 Q_0 = 10$):** Selectores en `10` $\rightarrow$ Salida $1Y = D_2$.

Adicionalmente, la salida combinada $Q_1 \cdot \overline{Q_0}$ se extrae como la línea de acarreo `Fin_Digito`, la cual servirá de disparo de reloj para el segundo contador acumulador de 9 bits.