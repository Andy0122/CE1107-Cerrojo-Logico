# Lista de Materiales (BOM) y Distribución de Circuitos Integrados

**Proyecto:** Cerrojo Electrónico en Base 5 con Lógica Combinacional y Secuencial  
**Curso:** CE 1107 — Fundamentos de Arquitectura de Computadores  
**Tecnología:** Disciplina Estática TTL (5 V) — Familia 74LS  

---

## 1. Conteo y Asignación de Circuitos Integrados TTL (15 Chips)

| Circuito Integrado | Cantidad | Descripción Funcional | Módulo y Rol en el Sistema |
| :--- | :---: | :--- | :--- |
| **`74LS14`** | **1** | Séxtuple Inversor con entradas **Schmitt Trigger** | **Acondicionamiento de Sensores:** Convierte las señales analógicas de las LDRs en pulsos digitales de 5 V limpios mediante histéresis, eliminando oscilaciones en la transición de luz a sombra. |
| **`74LS86`** | **1** | Cuádruple compuerta XOR de 2 entradas | **Codificador de Entrada:** Árbol de paridad de 3 compuertas para sintetizar el bit menos significativo ($D_0$) a partir de los 4 dedos. |
| **`74LS04`** | **1** | Séxtuple Inversor NOT estándar | **Lógica Combinacional:** Inversión para el bit $D_1$ del codificador y segmento $d$ del display de 7 segmentos. |
| **`74LS08`** | **2** | Cuádruple compuerta AND de 2 entradas | **Decodificador y Anuncios:** Generación de estados intermedios del display ($S_A, S_C, S_E$), término común y habilitaciones. |
| **`74LS21`** | **2** | Doble compuerta AND de 4 entradas | **Comparador de Claves:** Evaluación de minitérminos de 4 variables para las contraseñas de apertura ($413_5$) y cierre ($142_5$). |
| **`74LS02`** | **2** | Cuádruple compuerta NOR de 2 entradas | **Memoria y Reducción:** 2 compuertas para el Latch SR de retención de la puerta; compuertas restantes para reducción de De Morgan de ceros. |
| **`74LS32`** | **1** | Cuádruple compuerta OR de 2 entradas | **Visualizador de Anuncios:** Suma lógica para los segmentos $a$ y $g$ del display de 7 segmentos. |
| **`74LS153`**| **1** | Doble Multiplexor / Selector de datos 4:1 | **Serializador:** Conversión paralelo a serial de los 3 bits ($D_0, D_1, D_2$) del dígito censado hacia la línea única de transmisión. |
| **`74LS74`** | **2** | Doble Flip-Flop tipo D con Set y Reset | **Temporización y Control:** CI 1 para el contador módulo-3 de bits del serializador; CI 2 para el contador módulo-3 de dígitos (generador de la señal `Listo`). |
| **`74LS164`**| **2** | Registro de desplazamiento SIPO de 8 bits | **Receptor Paralelo:** Conectados en cascada ($8 + 1$ bits) para reconstruir el bus completo de 9 bits ($P_8 \dots P_0$). |

---

## 2. Componentes Discretos, Sensores y Potencia

| Componente | Especificación / Modelo | Cantidad | Función |
| :--- | :---: | :---: | :--- |
| **Fotoresistencias** | LDR 5 mm | 4 | Sensores de presencia de dedos de la mano ($L_0 \dots L_3$). |
| **Optoacopladores** | `PC817` (DIP-4) | 2 | Aislamiento galvánico total entre la lógica TTL (5 V) y el motor (9 V). |
| **Driver de Potencia**| Módulo Puente H `L9110S` | 1 | Inversión de giro del motor de CD en lógica Active-LOW. |
| **Finales de Carrera**| Microswitches con palanca | 2 | Contactos `COM` y `NC` en serie para corte de energía en topes de apertura/cierre. |
| **Capacitores** | $10\,\mu\text{F}$ electrolíticos / cerámicos | 3 | Filtrado antirrebote en pulsadores de Clock y Reset maestro. |
| **Resistencias** | $10\text{ k}\Omega$ (1/4 W) | 10 | Divisores de tensión de LDRs y pull-ups de líneas de control. |
| **Resistencias** | $330\text{ }\Omega$ (1/4 W) | 10 | Limitadoras para LEDs de dedos, optoacopladores y display. |
| **Display 7 Segmentos**| Cátodo Común (Rojo) | 1 | Visualizador de anuncios de estado (`L`, `A`, `C`, `E`). |
| **LEDs indicadores** | 5 mm (Rojos / Verdes) | 6 | 4 LEDs para lectura de dedos y 2 LEDs auxiliares de estado. |
| **Actuador** | Motorreductor CD con caja de engranes | 1 | Tracción mecánica de la puerta en la maqueta. |
| **Fuente Externa** | Batería cuadrada de 9 V | 1 | Alimentación independiente y aislada para la etapa motriz. |