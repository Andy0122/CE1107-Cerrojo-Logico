# Bitácora de Diseño: Finales de Carrera NC, Filtrado Antirrebote y Pruebas del Actuador

**Curso:** CE 1107 — Fundamentos de Arquitectura de Computadores  
**Fecha:** 25 de septiembre de 2026  
**Autores:** Juan Pablo Araya Rodríguez, Dylan Elizondo Picado  
**Módulo:** Integración Electromecánica, Supresión de Rebotes y Frenado de Puerta  

---

## 1. Conexión de Finales de Carrera mediante Contactos `NC`

Para evitar la complejidad de incorporar memorias biestables adicionales en la etapa de potencia, el corte de recorrido de la puerta se implementó mediante la interrupción directa del circuito de excitación de los optoacopladores:
- **Switch $FC_A$ (Tope de Apertura):** Pines `COM` y `NC` intercalados en serie en el cable de la señal de apertura hacia el Optoacoplador 1.
- **Switch $FC_C$ (Tope de Cierre):** Pines `COM` y `NC` intercalados en serie en el cable de la señal de cierre hacia el Optoacoplador 2.

### Dinámica de Operación Experimental:
1. Durante el desplazamiento, el resorte del switch mantiene el contacto `NC` firmemente cerrado, permitiendo el flujo de corriente hacia el LED infrarrojo del `PC817`.
2. Al impactar el tope físico de la maqueta, la leva de la puerta presiona la palanca del microswitch.
3. El contacto mecánico se abre, desconectando el optoacoplador en menos de $5\text{ ms}$.
4. La entrada correspondiente del puente H `L9110S` regresa a nivel alto ($9\text{ V}$) por su pull-up interno, apagando el motor de inmediato.
5. **Comprobación de Rebote Mecánico:** Gracias a la alta fricción estática del tren de engranajes del motorreductor, la puerta no retrocede por inercia, asegurando que el switch permanezca accionado o en reposo estable sin oscilaciones cíclicas.

---

## 2. Supresión de Rebotes Eléctricos mediante Filtro RC con Capacitores de $10\,\mu\text{F}$

Los pulsadores manuales de avance de reloj (`Clock`) y reinicio maestro (`Reset`) presentan rebotes mecánicos microscópicos (*contact bounce*) que introducen trenes de pulsos falsos en los flip-flops `74LS74`.

Para asegurar pulsos limpios de transición única:
- Se implementó un filtro pasa-bajas RC conectando una resistencia de pull-up de $10\text{ k}\Omega$ a $5\text{ V}$ y un **capacitor de $10\,\mu\text{F}$** en paralelo con los contactos del pulsador a tierra.
- **Constante de Tiempo:**
  $$\tau = R \cdot C = 10\text{ k}\Omega \times 10\,\mu\text{F} = 100\text{ ms}$$
- Esta constante amortigua completamente las transiciones de conmutación de alta frecuencia ($<20\text{ ms}$), garantizando que el reloj de bits y el reloj de dígitos avancen exactamente un estado por cada pulsación física del usuario.