# 📡 Comunicación Óptica Inalámbrica mediante Lámparas

## 📖 Descripción

Este proyecto desarrolla un sistema de **comunicación óptica inalámbrica** capaz de transmitir y reconstruir una matriz de información entre dos puntos separados aproximadamente **60 metros**.

El sistema utiliza dos lámparas, identificadas como **A** y **B**, que son controladas mediante Arduino. La información de la matriz se transforma en una secuencia de señales luminosas que posteriormente son observadas y decodificadas por el sistema receptor.

La matriz está compuesta por diferentes celdas y cada una de ellas puede presentar dos características independientes:

- **Color:** blanco o negro.
- **Contenido:** con letra o sin letra.

Por lo tanto, una celda puede ser blanca con letra, blanca sin letra, negra con letra o negra sin letra.

Un aspecto fundamental del protocolo es que **el color de una celda no determina si contiene una letra**. Ambas características se transmiten de manera independiente.

---

## 🎯 Objetivo

Diseñar e implementar un protocolo de comunicación óptica que permita transmitir una matriz utilizando únicamente las señales generadas por las lámparas A y B.

El receptor deberá ser capaz de identificar correctamente:

1. El inicio de la transmisión.
2. El color de cada celda.
3. La presencia o ausencia de una letra.
4. La letra contenida en cada celda mediante código Morse.
5. El cambio entre filas.
6. El final de la transmisión.

A partir de esta información, el receptor deberá reconstruir la matriz original.

---

# 💡 Principio de funcionamiento

El sistema utiliza dos fuentes luminosas con funciones diferentes.

La **lámpara A** representa un punto (`·`) del código Morse, mientras que la **lámpara B** representa una raya (`—`).

Cuando ambas lámparas se encienden simultáneamente, la señal se interpreta como una señal de control. De esta manera, A y B pueden trabajar tanto de forma individual como conjunta para transmitir diferentes tipos de información.

| Señal | Interpretación |
|---|---|
| A | Punto Morse `·` |
| B | Raya Morse `—` |
| A + B | Señal de control / color |
| A → B → A → B | La celda contiene una letra |
| A + B × 6 | Cambio de fila |
| A + B durante 5 s | Inicio o final de trama |

---

# 📡 Estructura del protocolo

La transmisión se realiza de forma secuencial, recorriendo la matriz **fila por fila y celda por celda**.

Antes de comenzar se envía una señal de inicio de trama. Posteriormente, cada celda se transmite siguiendo siempre el mismo procedimiento:

1. Se realiza una espera inicial de 3 segundos.
2. Se transmite el color de la celda.
3. Si la celda contiene una letra, se transmite el indicador `A-B-A-B`.
4. Se espera 2 segundos.
5. Se transmite la letra utilizando código Morse.
6. Se espera 2 segundos antes de continuar con la siguiente celda.

Cuando termina una fila, se transmite una señal especial de seis destellos simultáneos de A y B para indicar el cambio a la siguiente fila.

Una vez transmitida la última celda de la matriz, se envía la señal de finalización.

---

# ⏱️ Temporización

Los tiempos del protocolo fueron definidos buscando que las señales puedan ser diferenciadas correctamente durante la transmisión óptica.

| Evento | Señal | Temporización |
|---|---|---|
| Inicio de trama | A + B encendidas | 5 s |
| Espera antes de cada celda | A y B apagadas | 3 s |
| Celda blanca | A + B, 1 destello | 0,7 s ON / 0,5 s OFF |
| Celda negra | A + B, 2 destellos | 0,7 s ON / 0,5 s OFF |
| Indicador de letra | A → B → A → B | 0,4 s ON / 0,4 s OFF |
| Separación color-letra | A y B apagadas | 2 s |
| Punto Morse | A | 0,7 s ON |
| Raya Morse | B | 0,7 s ON |
| Separación Morse | A y B apagadas | 0,5 s |
| Separación entre celdas | A y B apagadas | 2 s |
| Salto de fila | A + B, 6 destellos | 0,4 s ON / 0,4 s OFF |
| Fin de trama | A + B encendidas | 5 s |

Estos tiempos pueden ser ajustados durante las pruebas experimentales con el objetivo de mejorar la confiabilidad de la detección a la distancia de operación.

---

# ⬜⬛ Transmisión del color

El color se transmite utilizando ambas lámparas simultáneamente.

Una **celda blanca** se representa mediante un único destello de A y B:

> **A + B → 1 destello = BLANCO**

Una **celda negra** se representa mediante dos destellos consecutivos:

> **A + B → 2 destellos = NEGRO**

Cada destello tiene una duración de 0,7 segundos encendido y 0,5 segundos apagado.

Es importante destacar que esta señal únicamente representa el **color**. Después de ella todavía debe determinarse si la celda contiene o no una letra.

---

# 🔤 Detección de letras

La presencia de una letra se transmite independientemente del color.

Cuando una celda contiene una letra, después de transmitir su color se genera una secuencia rápida de cuatro señales:

> **A → B → A → B**

Cada pulso tiene una duración de 0,4 segundos encendido y 0,4 segundos apagado.

Esta secuencia funciona como un indicador que informa al receptor:

> **"Esta celda contiene una letra."**

Si la secuencia `A-B-A-B` no aparece, el receptor interpreta que la celda no contiene ninguna letra y no debe intentar decodificar código Morse.

---

# 🔠 Codificación mediante código Morse

Cuando una celda contiene una letra, se realiza una espera de 2 segundos y posteriormente se transmite la letra utilizando las lámparas A y B.

La correspondencia utilizada es:

- **A = punto (`·`)**
- **B = raya (`—`)**

Cada símbolo tiene una duración de 0,7 segundos y los símbolos consecutivos de una misma letra se separan mediante 0,5 segundos.

### Ejemplos

**Letra A**

Código Morse:

`· —`

Transmisión:

`A → B`

---

**Letra D**

Código Morse:

`— · ·`

Transmisión:

`B → A → A`

---

**Letra G**

Código Morse:

`— — ·`

Transmisión:

`B → B → A`

---

**Letra S**

Código Morse:

`· · ·`

Transmisión:

`A → A → A`

---

# 🧩 Ejemplos de transmisión

## Celda blanca sin letra

La secuencia correspondiente es:

**Espera 3 s → A+B durante un destello → espera 2 s → siguiente celda.**

En este caso no aparece la secuencia `A-B-A-B`, por lo que el receptor sabe que la celda es blanca y no contiene ninguna letra.

---

## Celda negra con letra G

Para una celda negra que contiene la letra **G**, la transmisión sería:

**Espera 3 s → A+B, dos destellos → A-B-A-B → espera 2 s → B-B-A → espera 2 s.**

El receptor interpreta esta secuencia como:

> **Celda negra + contiene una letra + letra G.**

---

# ↩️ Cambio de fila

Al finalizar todas las celdas de una fila, se transmite una señal especial para indicar que la siguiente señal corresponde a una nueva fila.

La señal está formada por **seis destellos simultáneos de A y B**:

`A+B → A+B → A+B → A+B → A+B → A+B`

Cada destello tiene una duración de 0,4 segundos encendido y 0,4 segundos apagado.

El receptor reconoce esta secuencia como:

> **Fin de fila / inicio de la siguiente fila.**

Esta señal no se transmite después de la última fila.

---

# 🚦 Inicio de la transmisión

Antes de comenzar a transmitir la matriz, ambas lámparas permanecen encendidas simultáneamente durante **5 segundos**.

Esta señal permite al receptor identificar el comienzo de una nueva trama.

La misma señal se utiliza al final de la transmisión. La diferencia se determina mediante el estado del receptor:

- Si el sistema todavía no está transmitiendo una matriz, A+B durante 5 segundos significa **inicio de trama**.
- Si ya se está transmitiendo una matriz, A+B durante 5 segundos significa **fin de trama**.

---

# 🛑 Final de la transmisión

Después de transmitir la última celda de la última fila, ambas lámparas permanecen encendidas durante 5 segundos.

Esta señal indica que la matriz ha sido transmitida completamente.

El receptor utiliza esta señal para finalizar el proceso de adquisición y mostrar o almacenar la matriz reconstruida.

---

# 🔄 Proceso de reconstrucción

El receptor debe analizar las señales luminosas en el mismo orden en que fueron transmitidas.

Primero detecta la señal de inicio y comienza a recibir la matriz. Para cada celda, identifica inicialmente su color mediante el número de destellos simultáneos de A y B.

Después verifica si aparece la secuencia `A-B-A-B`.

Si aparece, la celda contiene una letra y el receptor espera 2 segundos para recibir el código Morse correspondiente.

Si no aparece, la celda se registra simplemente como una celda sin letra.

Una vez procesada la celda, el receptor continúa con la siguiente. Cuando detecta seis destellos simultáneos de A y B, incrementa el número de fila y continúa con la siguiente fila.

Finalmente, cuando detecta A+B encendidas durante 5 segundos estando dentro de una transmisión, considera que la matriz ha sido recibida completamente.

---

# 🧠 Lógica de interpretación

La información de cada celda puede representarse conceptualmente como:

| Color | Indicador | Morse | Resultado |
|---|---|---|---|
| Blanco | No | — | ⬜ Sin letra |
| Blanco | Sí | Código Morse | ⬜ Con letra |
| Negro | No | — | ⬛ Sin letra |
| Negro | Sí | Código Morse | ⬛ Con letra |

Por lo tanto, el receptor no debe asumir que un determinado color implica necesariamente la existencia de una letra.

La información de **color** y **contenido** se obtiene mediante dos etapas diferentes del protocolo.

---

# 📐 Secuencia general

La transmisión completa sigue la siguiente estructura:

**Inicio de trama**

↓  

**Fila 1**

↓  

**Celda 1**

↓  

**Celda 2**

↓  

**...**

↓  

**Última celda de la fila**

↓  

**Señal de salto de fila**

↓  

**Fila siguiente**

↓  

**...**

↓  

**Última fila**

↓  

**Última celda**

↓  

**Fin de trama**

---

# ⚙️ Implementación

El sistema puede implementarse mediante una lógica secuencial o una máquina de estados que permita controlar cada etapa del protocolo.

Una posible organización de estados es:

- `IDLE`
- `START`
- `WAIT_CELL`
- `SEND_COLOR`
- `CHECK_LETTER`
- `LETTER_FLAG`
- `WAIT_MORSE`
- `SEND_MORSE`
- `CELL_SEPARATOR`
- `ROW_SEPARATOR`
- `END`

La implementación deberá garantizar que cada señal se genere con los tiempos establecidos y que el receptor pueda diferenciar entre las diferentes secuencias.

---

# 🧪 Pruebas experimentales

Antes de realizar la transmisión a la distancia máxima, se recomienda verificar cada parte del protocolo de forma independiente.

Las pruebas pueden realizarse progresivamente:

1. Verificar el funcionamiento individual de las lámparas A y B.
2. Comprobar la transmisión simultánea A+B.
3. Verificar la detección de uno y dos destellos.
4. Comprobar la secuencia A-B-A-B.
5. Verificar la transmisión de diferentes letras en Morse.
6. Comprobar los seis destellos correspondientes al cambio de fila.
7. Verificar correctamente el inicio y el final de la trama.
8. Realizar pruebas con matrices pequeñas.
9. Incrementar progresivamente la distancia.
10. Realizar la transmisión completa a aproximadamente 60 metros.

Los tiempos establecidos inicialmente podrán modificarse si las condiciones de iluminación, distancia o características de los sensores afectan la detección.

---

# 📊 Ejemplo conceptual

Para una matriz de ejemplo:

| | | | |
|---|---|---|---|
| ⬜ A | ⬛ | ⬜ | ⬛ D |
| ⬛ | ⬜ G | ⬛ | ⬜ |
| ⬜ S | ⬛ | ⬛ A | ⬜ |

el sistema no transmite directamente la imagen de la matriz.

En su lugar, transforma cada celda en una secuencia de señales luminosas.

Por ejemplo:

> **⬜ A**

se transforma en:

`A+B × 1 → A-B-A-B → A-B`

Mientras que:

> **⬛**

se transforma en:

`A+B × 2`

De esta manera, el receptor puede reconstruir tanto el color como el contenido de cada posición.

---

## Uso del transmisor

El sistema utiliza un **Arduino Mega 2560** para controlar dos lámparas:

- **Lámpara A → Pin D8:** representa el punto Morse (`·`).
- **Lámpara B → Pin D9:** representa la raya Morse (`—`).
- **A+B:** se utilizan para indicar el color y las señales de control.

### Ingreso de la matriz

La matriz se introduce fácilmente desde el **Monitor Serial**. Primero se indican el número de filas y columnas y después se escribe cada fila.

Cada celda se representa como:

| Código | Significado |
|---|---|
| `B` | Blanca sin letra |
| `N` | Negra sin letra |
| `BA` | Blanca con letra A |
| `NG` | Negra con letra G |

Por ejemplo, una matriz de 3×4:
```text
BA N B ND
N BG N B
BS N NA B



Después de ingresar la matriz, el Arduino la muestra en el Monitor Serial para verificarla. Al escribir S, comienza la transmisión.

Funcionamiento del código

El programa almacena la matriz separando color y letra. Luego recorre las celdas de izquierda a derecha y de arriba hacia abajo.

Para cada celda:

Espera el tiempo establecido.
Transmite el color mediante destellos simultáneos de A+B.
Si contiene una letra, envía el indicador A-B-A-B.
Transmite la letra utilizando código Morse:
A = punto (·)
B = raya (—)
Continúa con la siguiente celda.

Al terminar cada fila se envían 6 destellos A+B como señal de cambio de fila. Finalmente, se envía la señal de fin de trama.

De esta manera, el Arduino convierte automáticamente la matriz introducida por el usuario en una secuencia de señales ópticas que puede ser interpretada por el receptor.
