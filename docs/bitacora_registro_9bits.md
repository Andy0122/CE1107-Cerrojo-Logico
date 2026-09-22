# Bitácora de Diseño: Registro de Desplazamiento de 9 Bits y Contador de Control 'Listo'

**Curso:** CE 1107 — Fundamentos de Arquitectura de Computadores  
**Fecha:** 21 de septiembre de 2026  
**Autores:** Juan Pablo Araya Rodríguez, Dylan Elizondo Picado  
**Módulo:** Reconstrucción Serial-Paralelo y Generación Modular de Control  

---

## 1. Separación Arquitectónica: Ruta de Datos vs Unidad de Control

Para mantener un diseño desacoplado y modular, se evitó mezclar la lógica de conteo dentro del registro de almacenamiento. El sistema divide la recepción en dos bloques independientes:
1. **Ruta de Datos (`Registro_Desplazamiento_9bits`):** Encargada exclusivamente de capturar la trama serial y exponer el bus de 9 variables ($P_8 \dots P_0$).
2. **Unidad de Control (`Contador_Digitos`):** Encargada de supervisar el progreso de la transmisión y activar la bandera de estado `Listo`.

---

## 2. Módulo de Almacenamiento: Registro de 9 Bits

Dado que la contraseña consta de 3 dígitos en base 5 ($3 \times 3 \text{ bits} = 9 \text{ bits totales}$), se configuró una cadena de registro de desplazamiento de entrada serial y salida paralela (SIPO):
- **Entrada Serial:** Recibe el bit transmitido por la salida $1Y$ del multiplexor `74LS153`.
- **Reloj de Desplazamiento (`CLK_Bit`):** Sincroniza la entrada de cada bit individual.
- **Salida Paralela ($P_8 \dots P_0$):** Conforme ingresan los pulsos, los bits se van recorriendo a lo largo de las 9 posiciones de memoria.
- **Reset Maestro:** La línea asíncrona de borrado reinicia todo el bus a `000000000` antes de iniciar un nuevo intento de apertura o cierre.

---

## 3. Módulo de Control: Segundo Contador Módulo-3 (`Contador_Digitos`)

Este submódulo opera de manera autónoma como un contador síncrono de eventos de orden superior utilizando los dos flip-flops D del segundo circuito integrado `74LS74` ($Q_3 Q_2$):

### A. Jerarquía de Reloj:
A diferencia del registro de datos (que avanza bit por bit con `CLK_Bit`), este contador **únicamente incrementa cuando se completa un dígito entero**. Su entrada de reloj está conectada a la línea `Fin_Digito` generada por el primer contador del serializador:
- Pulso 1 de `Fin_Digito`: Dígito 1 recibido (3 bits en registro).
- Pulso 2 de `Fin_Digito`: Dígito 2 recibido (6 bits en registro).
- Pulso 3 de `Fin_Digito`: Dígito 3 recibido (**9 bits completos en el bus**).

### B. Ecuaciones del Contador y Bandera `Listo`:
Utiliza la misma lógica síncrona compacta de 1 NOR y 1 AND:
- $D_2 = \overline{Q_3 + Q_2}$
- $D_3 = Q_2 \cdot \overline{Q_3}$

Al transitar al tercer dígito, una compuerta combinatoria de salida detecta la culminación de la trama y conmuta:
$$\mathbf{Listo = 1}$$

Esta señal notifica al **Decodificador de Anuncios** que la lectura concluyó, habilitando en el display de 7 segmentos la evaluación del estado: apertura exitosa (`A`), cierre exitoso (`C`) o contraseña inválida (`E`).