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

La matriz se transmite **fila por fila**, recorriendo las celdas de izquierda a derecha. Cada celda transmite primero su **color** y, si corresponde, la **presencia y contenido de una letra**.

### 🔄 Secuencia general

**Inicio**  
`A+B → 4 s`

→ **Celda**  
`Espera 1 s` → `Color` → `Letra (si existe)` → `Espera 500 ms`

→ **Siguiente celda**

→ **Fin de fila**  
`Espera 1 s` → `A+B × 6`

→ **Siguiente fila**

→ **Fin**  
`A+B → 4 s`

### 💡 Codificación

| Señal | Significado |
|:---:|---|
| `A` | Punto Morse `·` |
| `B` | Raya Morse `—` |
| `A+B × 1` | Blanco |
| `A+B × 2` | Negro |
| `A-B-A-B` | Presencia de letra |
| `A+B × 6` | Cambio de fila |
| `A+B durante 4 s` | Inicio / fin de trama |

### 🧩 Funcionamiento general

El transmisor recibe la matriz mediante el **Monitor Serial** y almacena de forma independiente el **color** y la **letra** de cada celda.

Después, recorre la matriz **de izquierda a derecha y de arriba hacia abajo**, convirtiendo cada celda en una secuencia de señales luminosas. Si la celda contiene una letra, esta se codifica mediante **Morse**, utilizando:

- `A` → punto `·`
- `B` → raya `—`

De esta manera, el receptor puede reconstruir tanto el **color** como el **contenido** de cada celda.

## 🚀 Estado actual

El transmisor está implementado en un **Arduino Mega 2560** y permite ingresar matrices dinámicamente desde el Monitor Serial.

El protocolo permite representar independientemente:

- ⚪ Blanca sin letra
- ⚪ Blanca con letra
- ⚫ Negra sin letra
- ⚫ Negra con letra

- ## 🔤 Tabla de código Morse

El protocolo utiliza código Morse para transmitir las letras. La **Lámpara A representa un punto (`·`)** y la **Lámpara B representa una raya (`—`)**.

| Letra | Morse | Secuencia de lámparas |
|:---:|:---:|:---:|
| A | `· —` | A B |
| B | `— · · ·` | B A A A |
| C | `— · — ·` | B A B A |
| D | `— · ·` | B A A |
| E | `·` | A |
| F | `· · — ·` | A A B A |
| G | `— — ·` | B B A |
| H | `· · · ·` | A A A A |
| I | `· ·` | A A |
| J | `· — — —` | A B B B |
| K | `— · —` | B A B |
| L | `· — · ·` | A B A A |
| M | `— —` | B B |
| N | `— ·` | B A |
| O | `— — —` | B B B |
| P | `· — — ·` | A B B A |
| Q | `— — · —` | B B A B |
| R | `· — ·` | A B A |
| S | `· · ·` | A A A |
| T | `—` | B |
| U | `· · —` | A A B |
| V | `· · · —` | A A A B |
| W | `· — —` | A B B |
| X | `— · · —` | B A A B |
| Y | `— · — —` | B A B B |
| Z | `— — · ·` | B B A A |

### Ejemplo

Para transmitir la letra **G**:

`G → — — · → B B A`

Por lo tanto, el transmisor enciende:

**Lámpara B → Lámpara B → Lámpara A**

Cada símbolo permanece encendido durante **300 ms**, con una separación de **200 ms** entre símbolos.
