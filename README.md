# Sistema de Comunicación Óptica mediante Lámparas

## 📡 Descripción del proyecto

Este proyecto consiste en el diseño e implementación de un sistema de **comunicación óptica inalámbrica** entre dos puntos separados aproximadamente **60 metros**.

El sistema utiliza **dos lámparas**, denominadas **A** y **B**, controladas mediante Arduino para transmitir la información de una matriz de celdas.

La matriz entregada por el profesor tendrá un tamaño determinado y estará compuesta por celdas que pueden ser:

- ⬜ Blancas
- ⬛ Negras
- Con letra
- Sin letra

Es importante aclarar que **el color de una celda es independiente de si esta contiene una letra**. Una celda blanca puede contener una letra, al igual que una celda negra puede contener una letra.

El receptor debe reconstruir la matriz utilizando **únicamente la información transmitida mediante las lámparas**.

---

# 👥 Arquitectura del sistema

El sistema estará compuesto principalmente por:

```text
┌──────────────────────┐
│      EMISOR          │
│                      │
│       Arduino        │
│          │           │
│     ┌────┴────┐      │
│     │         │      │
│  Lámpara A  Lámpara B│
└─────┬─────────┬──────┘
      │         │
      │  ~60 m  │
      │         │
      ▼         ▼
┌──────────────────────┐
│      RECEPTOR        │
│                      │
│  Sensor / recepción  │
│        óptica        │
│          │           │
│       Arduino        │
│          │           │
│          ▼           │
│  Reconstrucción      │
│      de matriz       │
└──────────────────────┘

💡 Lámparas

Se utilizarán dos lámparas:
Lámpara	Función
A	Representa un punto · en código Morse
B	Representa una raya — en código Morse
A + B	Se utiliza para señales de control

Las lámparas A y B no representan directamente el color de una celda cuando se transmite una letra.

El color se determina mediante el número de destellos simultáneos de ambas lámparas.
📦 Protocolo de comunicación

El protocolo está diseñado para que el receptor pueda identificar:

    El inicio de la matriz.

    El inicio de cada celda.

    El color de cada celda.

    Si una celda contiene una letra.

    La letra mediante código Morse.

    El final de cada fila.

    El final de la matriz.

La estructura general de la transmisión será:

INICIO DE TRAMA
      │
      ▼
   FILA 1
      │
      ├── CELDA 1
      ├── CELDA 2
      ├── CELDA 3
      ├── ...
      └── CELDA N
      │
      ▼
 SALTO DE FILA
      │
      ▼
   FILA 2
      │
      ├── CELDA 1
      ├── CELDA 2
      ├── ...
      └── CELDA N
      │
      ▼
     ...
      │
      ▼
   ÚLTIMA FILA
      │
      ▼
 FIN DE TRAMA

⏱️ Tiempos del protocolo

Los tiempos definidos inicialmente para la transmisión son los siguientes:
Evento	Señal	Tiempo
Inicio de trama	A + B encendidas continuamente	5 s
Inicio de celda	Ambas lámparas apagadas	3 s
Celda blanca	A + B → 1 destello	0,7 s ON / 0,5 s OFF
Celda negra	A + B → 2 destellos	0,7 s ON / 0,5 s OFF
Indicador de letra	A → B → A → B	0,4 s ON / 0,4 s OFF
Separación color-letra	Ambas apagadas	2 s
Punto Morse ·	Lámpara A	0,7 s ON
Raya Morse —	Lámpara B	0,7 s ON
Separación entre símbolos Morse	Ambas apagadas	0,5 s
Separación entre celdas	Ambas apagadas	2 s
Salto de fila	A + B → 6 destellos	0,4 s ON / 0,4 s OFF
Fin de trama	A + B encendidas continuamente	5 s

    Nota: Los tiempos podrán ajustarse experimentalmente durante las pruebas para garantizar una correcta detección a la distancia de aproximadamente 60 metros.

🟦⬛ Codificación del color

El color de cada celda se transmite mediante ambas lámparas encendidas simultáneamente.
Celda blanca

Una celda blanca se representa mediante un destello simultáneo:

A: ────████────
B: ────████────

       1
    DESTELLO

Celda negra

Una celda negra se representa mediante dos destellos simultáneos:

A: ──████──████──
B: ──████──████──

      1     2
   DESTELLO DESTELLO

Por lo tanto:

1 destello A+B → CELDA BLANCA

2 destellos A+B → CELDA NEGRA

🔤 Indicador de celda con letra

Después de transmitir el color, el sistema debe indicar si la celda contiene una letra.

Para ello se utiliza una secuencia rápida:

A → B → A → B

Cada cambio tiene una duración de:

0,4 s ON
0,4 s OFF

Esta secuencia significa:

A-B-A-B → LA CELDA CONTIENE UNA LETRA

Si esta secuencia no aparece, significa que la celda no contiene una letra.
📖 Código Morse

Cuando una celda contiene una letra, después del indicador:

A-B-A-B

se espera:

2 segundos

Posteriormente se transmite la letra utilizando código Morse.

La correspondencia utilizada es:
Lámpara	Símbolo Morse
A	Punto ·
B	Raya —

Cada símbolo tiene una duración de:

0,7 segundos

La separación entre símbolos Morse será:

0,5 segundos

🔠 Ejemplos de letras
Letra A

Código Morse:

· —

Transmisión:

A → B

Letra D

Código Morse:

— · ·

Transmisión:

B → A → A

Letra G

Código Morse:

— — ·

Transmisión:

B → B → A

Letra S

Código Morse:

· · ·

Transmisión:

A → A → A

🧩 Transmisión de una celda

Cada celda seguirá la siguiente lógica:

                 ┌──────────────┐
                 │ ESPERA 3 s   │
                 └──────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ TRANSMITIR    │
                │    COLOR      │
                └───────┬───────┘
                        │
                        ▼
                ¿TIENE LETRA?
                   /       \
                 NO         SÍ
                 │           │
                 │           ▼
                 │       A-B-A-B
                 │       rápidamente
                 │           │
                 │           ▼
                 │       ESPERA 2 s
                 │           │
                 │           ▼
                 │       CÓDIGO MORSE
                 │           │
                 └─────┬─────┘
                       │
                       ▼
                  ESPERA 2 s
                       │
                       ▼
                SIGUIENTE CELDA

⬜ Ejemplo: celda blanca sin letra

ESPERA 3 s
    ↓
A+B → 1 destello
    ↓
NO hay A-B-A-B
    ↓
NO se transmite Morse
    ↓
ESPERA 2 s
    ↓
SIGUIENTE CELDA

⬛ Ejemplo: celda negra con letra G

La letra G corresponde a:

— — ·

Por lo tanto:

ESPERA 3 s
    ↓
A+B → A+B
    ↓
CELDA NEGRA
    ↓
A → B → A → B
    ↓
TIENE LETRA
    ↓
ESPERA 2 s
    ↓
B → B → A
    ↓
— — ·
    ↓
LETRA G
    ↓
ESPERA 2 s
    ↓
SIGUIENTE CELDA

↩️ Salto de fila

Cuando se hayan transmitido todas las celdas de una fila, se transmitirá una señal especial para indicar el cambio de fila.

La señal será:

A+B → A+B → A+B → A+B → A+B → A+B

Es decir:

6 destellos simultáneos

Cada destello tendrá:

0,4 s ON
0,4 s OFF

Por lo tanto:

6 destellos A+B → SALTO DE FILA

Esta señal permite diferenciar el final de una fila de las señales utilizadas para representar los colores de las celdas.

    Importante: después de la última fila no se transmite un salto de fila. Después de la última celda se procede directamente a la señal de fin de trama.

🚦 Inicio de trama

Antes de comenzar a transmitir la matriz, ambas lámparas se mantienen encendidas simultáneamente durante:

5 segundos

Esto representa:

A + B
████████████████████
       5 segundos

→ INICIO DE TRAMA

El receptor utiliza esta señal para comenzar una nueva reconstrucción de la matriz.
🛑 Fin de trama

Una vez transmitida la última celda de la matriz, ambas lámparas se mantienen encendidas simultáneamente durante:

5 segundos

Esto representa:

A + B
████████████████████
       5 segundos

→ FIN DE TRAMA

El receptor interpreta esta señal como el final de la transmisión y procede a finalizar la reconstrucción de la matriz.
📋 Estructura completa de una transmisión

Una transmisión completa tendrá la siguiente estructura:

┌────────────────────────────┐
│      INICIO DE TRAMA       │
│          A+B / 5 s         │
└──────────────┬─────────────┘
               │
               ▼
       ┌───────────────┐
       │    FILA 1     │
       └───────┬───────┘
               │
       ┌───────▼───────┐
       │    CELDA 1    │
       └───────┬───────┘
               │
             2 s
               │
               ▼
       ┌───────────────┐
       │    CELDA 2    │
       └───────┬───────┘
               │
              ...
               │
               ▼
       ┌───────────────┐
       │    CELDA N    │
       └───────┬───────┘
               │
               ▼
       A+B × 6 DESTELLOS
               │
               ▼
       ┌───────────────┐
       │    FILA 2     │
       └───────────────┘
               │
              ...
               │
               ▼
       ┌───────────────┐
       │  ÚLTIMA FILA   │
       └───────┬───────┘
               │
               ▼
       ÚLTIMA CELDA
               │
               ▼
       A+B / 5 segundos
               │
               ▼
       ┌───────────────┐
       │ FIN DE TRAMA  │
       └───────────────┘

🔄 Proceso de reconstrucción

El receptor observará las señales ópticas y seguirá el siguiente proceso:

1. Detectar A+B durante 5 s
             ↓
2. Iniciar recepción
             ↓
3. Esperar 3 s
             ↓
4. Determinar color
             ↓
5. Determinar si existe letra
             ↓
6. Si existe letra:
       └── recibir Morse
             ↓
7. Guardar información de la celda
             ↓
8. Esperar 2 s
             ↓
9. ¿Se detectaron 6 destellos A+B?
       │
       ├── NO → siguiente celda
       │
       └── SÍ → siguiente fila
             ↓
10. ¿Se detectaron A+B durante 5 s?
       │
       └── SÍ → FIN DE TRAMA
             ↓
11. Mostrar matriz reconstruida

🧠 Principio de funcionamiento

El sistema utiliza diferentes patrones de iluminación para representar diferentes tipos de información.

A+B durante 5 s
       ↓
INICIO / FIN DE TRAMA

A+B × 1
       ↓
CELDA BLANCA

A+B × 2
       ↓
CELDA NEGRA

A-B-A-B rápido
       ↓
CELDA CON LETRA

A
       ↓
PUNTO (·)

B
       ↓
RAYA (—)

A+B × 6 rápido
       ↓
SALTO DE FILA

De esta manera, el receptor puede determinar la información de cada celda sin tener acceso a la matriz original.
👁️ Consideraciones sobre la velocidad de transmisión

Debido a que la transmisión se realiza mediante señales ópticas y se busca que el receptor pueda identificar visualmente los patrones de las lámparas, se utilizarán tiempos suficientemente largos para permitir una correcta percepción de los pulsos.

Los tiempos iniciales establecidos son:

    0,4 s para pulsos rápidos de control.

    0,5 s para separación entre símbolos Morse.

    0,7 s para los símbolos y pulsos principales.

    1 s para separar determinadas etapas de la transmisión.

    2 s para separación entre celdas.

    3 s para preparación antes de cada celda.

    5 s para inicio y fin de trama.

Estos valores podrán ser modificados durante las pruebas experimentales para encontrar el mejor equilibrio entre:

Velocidad de transmisión ↔ facilidad de detección ↔ confiabilidad
🎯 Objetivo final

El objetivo del sistema es lograr que una matriz proporcionada al Arduino emisor pueda ser transmitida mediante señales luminosas a través de las lámparas y posteriormente reconstruida por el receptor.

El receptor deberá obtener exclusivamente a partir de las señales ópticas:

    La posición de cada celda.

    El color de cada celda.

    La presencia o ausencia de una letra.

    La letra correspondiente cuando exista.

    El cambio entre filas.

    El inicio y final de la transmisión.

Finalmente, la matriz reconstruida deberá corresponder a la matriz original proporcionada al emisor.
📌 Resumen del protocolo

INICIO
A+B durante 5 s

PARA CADA CELDA:

    Esperar 3 s

    Si BLANCA:
        A+B × 1

    Si NEGRA:
        A+B × 2

    Si tiene LETRA:
        A-B-A-B rápido
        Esperar 2 s
        Transmitir Morse
            A = ·
            B = —

    Esperar 2 s

AL TERMINAR CADA FILA:
    A+B × 6 rápido

EXCEPTO DESPUÉS DE LA ÚLTIMA FILA

FIN
A+B durante 5 s

🛠️ Tecnologías previstas

    Arduino

    Lámpara A

    Lámpara B

    Sistema de recepción óptica

    Comunicación inalámbrica mediante luz

    Código Morse

    Procesamiento y reconstrucción de matrices
