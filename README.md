# Sistema de Cerradura Digital con Validación en Base 5

**CE 1107 — Fundamentos de Arquitectura de Computadores**  
*Escuela de Ingeniería en Computadores*  
*Instituto Tecnológico de Costa Rica (TEC)*  

---

### Integrantes
- **Juan Pablo Araya Rodríguez**
- **Dylan Elizondo Picado**

**Profesor:** Luis Chavarría Zamora  
**Semestre:** II Semestre 2026  

---

## 1. Descripción del Proyecto

El objetivo de este proyecto es diseñar e implementar un sistema de cerrojo electrónico de seguridad accionado mediante combinaciones numéricas representadas por los dedos de una mano (código de 3 dígitos en base 5).

El sistema se compone de una arquitectura digital combinatoria y secuencial alimentada bajo la disciplina estática de 0 V a 5 V, encargada de validar la secuencia ingresada, gobernar una memoria de estado, informar el estatus actual al usuario mediante un display de siete segmentos y excitar un mecanismo de apertura/cierre físico impulsado por un motor de corriente directa con desacople eléctrico.

---

## 2. Diagrama de Bloques General

El flujo funcional del sistema se divide en las etapas descritas por la especificación:

```text
+----------+   bus (1)   +-----------------------+    (2)     +--------------+   (7)   +--------------+
| Sensores | ----------> | Circuito Combinatorio | ---------> | Decodificador| ------> | Visualizador |
+----------+             |  (Lógica y Memoria)   |            |     BCD      |         |  7 Segmentos |
     | (n)               +-----------------------+            +--------------+         +--------------+
     v                              | (1)
+--------------+                    v
| Visualizador |             +--------------+   (1)   +--------------+
|   con LEDs   |             |   Desacople  | ------> |  Accionador  |
+--------------+             |  Eléctrico   |         |  (Motor CD)  |
                             +--------------+         +--------------+
```

---

## 3. Especificación Preliminar de Estados y Claves

### 3.1. Claves en Base 5
Cada número representado por los dedos de la mano toma valores del rango $[0 \dots 4]$, requiriendo un esquema de codificación binaria de 3 bits por dígito (9 bits totales para el vector de contraseña):
- **Clave de Apertura:** Secuencia de 3 dígitos en base 5.
- **Clave de Cierre:** Secuencia de 3 dígitos en base 5 (diferente a la de apertura).

### 3.2. Estados del Visualizador de 7 Segmentos
El decodificador de anuncios traducirá las condiciones del sistema en cuatro símbolos:
- **`L`:** Leyendo la contraseña.
- **`A`:** Abriendo puerta (clave de apertura validada).
- **`C`:** Cerrando puerta (clave de cierre validada).
- **`E`:** Contraseña inválida / Error de validación.

---

## 4. Estructura del Repositorio

```text
├── docs/                   # Documentación técnica, bitácoras y paper LaTeX
├── sim/                    # Archivos de simulación (Logisim Evolution)
├── hardware/               # Esquemáticos y diagramas de conexión
└── README.md
```