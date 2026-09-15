# 📡 Comunicación Óptica Inalámbrica mediante Lámparas

## 📖 Descripción

Este proyecto desarrolla un sistema de **comunicación óptica inalámbrica** capaz de transmitir y reconstruir una matriz de información entre dos puntos separados aproximadamente **60 metros**.

El sistema utiliza dos lámparas, identificadas como **A** y **B**, controladas mediante un **Arduino Mega 2560**. La información de la matriz se convierte en una secuencia de señales luminosas que posteriormente son observadas y decodificadas por el receptor.

Cada celda de la matriz posee dos características independientes:

- **Color:** blanco o negro.
- **Contenido:** con letra o sin letra.

Por lo tanto, una celda puede ser:

- Blanca sin letra.
- Blanca con letra.
- Negra sin letra.
- Negra con letra.

El color y la presencia de una letra se transmiten de forma independiente.

---

## 🎯 Objetivo

Diseñar e implementar un protocolo de comunicación óptica que permita transmitir una matriz utilizando únicamente las señales generadas por las lámparas A y B.

El receptor deberá identificar:

1. El inicio de la transmisión.
2. El color de cada celda.
3. La presencia o ausencia de una letra.
4. La letra mediante código Morse.
5. El cambio entre filas.
6. El final de la transmisión.

Con esta información, el receptor podrá reconstruir la matriz original.

---

# 💡 Principio de funcionamiento

Las dos lámparas tienen funciones diferentes:

- **Lámpara A:** representa un punto Morse (`·`).
- **Lámpara B:** representa una raya Morse (`—`).
- **A+B:** representa señales de control y color.

| Señal | Interpretación |
|---|---|
| A | Punto Morse `·` |
| B | Raya Morse `—` |
| A+B | Color / señal de control |
| A → B → A → B | La celda contiene una letra |
| A+B × 6 | Cambio de fila |
| A+B durante 4 s | Inicio o final de trama |

---

# 📡 Estructura del protocolo

La matriz se transmite de forma secuencial, recorriendo las celdas **de izquierda a derecha y de arriba hacia abajo**.

La estructura general es:

```text
INICIO DE TRAMA
       ↓
   FILA 1
       ↓
   CELDA 1
       ↓
   CELDA 2
       ↓
      ...
       ↓
CAMBIO DE FILA
       ↓
   FILA 2
       ↓
      ...
       ↓
ÚLTIMA FILA
       ↓
FIN DE TRAMA
INICIO
A+B durante 4 s

        ↓

POR CADA CELDA

Espera 1 s
        ↓
Color
A+B × 1 → Blanco
A+B × 2 → Negro
        ↓
Si tiene letra:
A-B-A-B
        ↓
Espera 500 ms
        ↓
Morse
A = ·
B = —
        ↓
Espera 500 ms
        ↓
Siguiente celda

        ↓

FIN DE FILA
Espera 1 s
A+B × 6

        ↓

SIGUIENTE FILA

        ↓

ÚLTIMA CELDA

        ↓

FIN
A+B durante 4 s
🚀 Estado actual

El transmisor se encuentra implementado sobre Arduino Mega 2560 y permite introducir una matriz mediante el Monitor Serial para posteriormente convertirla automáticamente en una secuencia de señales ópticas.

El protocolo está diseñado para mantener separadas las dos variables principales de cada celda: color y contenido, permitiendo reconstruir la matriz a partir de las señales luminosas recibidas.
