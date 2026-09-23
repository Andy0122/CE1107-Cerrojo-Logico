# Bitácora de Diseño: Desacople Galvánico Óptico (PC817) y Driver de Potencia L9110S

**Curso:** CE 1107 — Fundamentos de Arquitectura de Computadores  
**Fecha:** 23 de septiembre de 2026  
**Autores:** Juan Pablo Araya Rodríguez, Dylan Elizondo Picado  
**Módulo:** Aislamiento Galvánico, Conmutación Active-LOW y Etapa de Potencia  

---

## 1. Discrepancia entre Simulación Digital y Realidad Física

En herramientas EDA como Logisim Evolution, las señales se modelan bajo abstracciones booleanas ideales (0 V y 5 V) con impedancias infinitas y ausencia de demanda de corriente. 

No obstante, la integración de un actuador electromecánico (motor de CD) introduce severas restricciones físicas que no pueden emularse en el simulador:
1. **Incompatibilidad de Corriente:** Las compuertas TTL estándar de la serie `74LS` entregan una corriente de salida máxima de $I_{OH} \approx 0.4\text{ mA}$ y absorben $I_{OL} \approx 8\text{ mA}$. Un micromotor de CD demanda entre $150\text{ mA}$ y $400\text{ mA}$ en régimen transitorio, lo que destruiría inmediatamente las etapas de salida de silicio del circuito lógico.
2. **Ruido Inductivo y Rebote de Tierra (*Ground Bounce*):** La conmutación de las bobinas del rotor genera fuerzas contraelectromotrices de decenas de voltios e interferencia electromagnética (EMI). Si el motor compartiera el plano de tierra ($GND$) con los biestables del circuito, los transitorios provocarían el reinicio espurio del Latch SR y los contadores `74LS74`.

Por esta razón, la arquitectura física implementa una **etapa de desacople eléctrico verdadero (aislamiento galvánico)** gobernada por fotones.

---

## 2. Aislamiento Óptico mediante Optoacopladores `PC817`

Se utilizaron dos optoacopladores `PC817` de encapsulado DIP-4 para desacoplar de forma absoluta el dominio digital del dominio de potencia:

```text
 DOMINIO LÓGICO TTL (5V)                 BARRERA ÓPTICA                DOMINIO MOTOR (9V)
                                       +----------------+
Señal Puerta (5V) ---[ R: 330 Ω ]----->| 1 (Ánodo)    4 |------- Pin Entrada L9110S
                                       |   (LED IR)     |        (Con pull-up interno a 9V)
                                       |       |        |
Tierra Lógica 5V --------------------->| 2 (Cátodo)   3 |------- Tierra Batería 9V (GND_9V)
(GND_5V)                               +----------------+        (CERO conexión con GND_5V)
```

### A. Parámetros de Polarización en el Emisor (Lado 5V):
Para una caída típica en el diodo emisor infrarrojo de $V_F \approx 1.2\text{ V}$, se calculó la resistencia limitadora de excitación:
$$I_F = \frac{V_{CC} - V_F}{R} = \frac{5.0\text{ V} - 1.2\text{ V}}{330\text{ }\Omega} \approx 11.51\text{ mA}$$
Esta corriente satura plenamente el LED interno sin exceder el límite de disipación térmica del componente.

### B. Aislamiento Galvánico Total:
- El circuito lógico opera con su propia fuente regulada de $5\text{ V}$ y su plano de tierra $GND_{5V}$.
- El motor opera exclusivamente con los bornes de la batería de $9\text{ V}$ ($V_{BAT}$ y $GND_{9V}$).
- **No existe ningún conductor metálico uniendo ambos dominios.** El acoplamiento se efectúa exclusivamente mediante radiación óptica infrarroja en el espacio interno sellado del chip.

---

## 3. Integración con el Módulo Puente H `L9110S`

Para comandar la inversión de giro del motor se empleó el módulo `L9110S`, aprovechando sus características internas:

### A. Operación en Lógica Activa en Bajo (*Active LOW*):
Las pruebas de laboratorio revelaron que los pines de control `A-1A` y `A-1B` cuentan con polarización resistiva interna hacia la línea de alimentación de $9\text{ V}$. Esto define la siguiente tabla de estados físicos:

| Entrada `A-1A` | Entrada `A-1B` | Estado Interno Puente H | Comportamiento del Motor |
| :---: | :---: | :---: | :---: |
| **$9\text{ V}$ (Alto)** | **$9\text{ V}$ (Alto)** | Ambas salidas en alta impedancia / reposo | **Detenido / Freno pasivo** |
| **$0\text{ V}$ (Bajo)** | **$9\text{ V}$ (Alto)** | Conducción directa canal A | **Giro Apertura (Adelante)** |
| **$9\text{ V}$ (Alto)** | **$0\text{ V}$ (Bajo)** | Conducción inversa canal A | **Giro Cierre (Reversa)** |
| **$0\text{ V}$ (Bajo)** | **$0\text{ V}$ (Bajo)** | Salidas a potencial común | **Frenado dinámico** |

### B. Conmutación por Colector Abierto:
El fototransistor del `PC817` se conectó en configuración *pull-down* directo:
- **Reposo (Lógica 5V en 0):** El LED interno está apagado $\rightarrow$ El fototransistor está en corte $\rightarrow$ La entrada del `L9110S` flota a $9\text{ V}$ por su pull-up interno $\rightarrow$ **Motor apagado**.
- **Activación (Lógica 5V en 1):** El LED enciende $\rightarrow$ El fototransistor satura $\rightarrow$ Conecta la entrada del `L9110S` directamente a $GND_{9V}$ (imponiendo el cero lógico) $\rightarrow$ **Motor en marcha**.

Esta topología elimina la necesidad de resistencias externas de pull-down o etapas adicionales con transistores bipolares discretos, logrando una interfaz limpia y libre de disipación parásita.