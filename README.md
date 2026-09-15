# 📡 Sistema de Comunicación Óptica mediante Lámparas

## 📋 Descripción del proyecto

El proyecto consiste en diseñar e implementar un sistema de **comunicación óptica inalámbrica** entre dos puntos separados aproximadamente **60 metros**.

Para la transmisión se utilizarán dos lámparas, denominadas **A** y **B**, controladas mediante Arduino. El emisor recibirá una matriz de celdas proporcionada por el profesor y deberá transmitir toda la información mediante las lámparas.

El receptor observará las señales luminosas y, utilizando únicamente la información obtenida mediante las lámparas, deberá **decodificar y reconstruir la matriz original**.

Cada celda de la matriz puede ser:

- ⬜ Blanca con letra
- ⬜ Blanca sin letra
- ⬛ Negra con letra
- ⬛ Negra sin letra

> **Importante:** el color de la celda es independiente de la presencia de una letra.

---

## 👥 Arquitectura del sistema

```text
┌─────────────────────────┐
│         EMISOR          │
│                         │
│         Arduino         │
│            │            │
│       ┌────┴────┐       │
│       │         │       │
│   Lámpara A  Lámpara B │
└───────┬─────────┬───────┘
        │         │
        │  ~60 m  │
        │         │
        ▼         ▼
┌─────────────────────────┐
│        RECEPTOR         │
│                         │
│     Recepción óptica    │
│            │            │
│         Arduino         │
│            │            │
│            ▼            │
│  Reconstrucción de      │
│        la matriz        │
└─────────────────────────┘
💡 Función de las lámparas
Lámpara	Función
A	Representa un punto · en código Morse
B	Representa una raya — en código Morse
A + B	Señales de control y transmisión del color

Las lámparas A y B pueden encenderse de forma individual, simultánea o intercalada dependiendo de la información que se quiera transmitir.

📦 Protocolo de comunicación

La comunicación se realizará mediante una secuencia de señales luminosas que permitirá al receptor identificar cada elemento de la matriz.

La transmisión tendrá la siguiente estructura:

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
    CELDA N
       ↓
  SALTO DE FILA
       ↓
     FILA 2
       ↓
      ...
       ↓
 ÚLTIMA FILA
       ↓
 ÚLTIMA CELDA
       ↓
 FIN DE TRAMA
⏱️ Tiempos del protocolo
Evento	Señal	Tiempo
Inicio de trama	A + B encendidas continuamente	5 s
Espera antes de cada celda	Ambas lámparas apagadas	3 s
Celda blanca	A + B → 1 destello	0,7 s ON / 0,5 s OFF
Celda negra	A + B → 2 destellos	0,7 s ON / 0,5 s OFF
Indicador de letra	A → B → A → B	0,4 s ON / 0,4 s OFF
Separación color-letra	Ambas lámparas apagadas	2 s
Punto Morse ·	Lámpara A	0,7 s ON
Raya Morse —	Lámpara B	0,7 s ON
Separación entre símbolos Morse	Ambas lámparas apagadas	0,5 s
Separación entre celdas	Ambas lámparas apagadas	2 s
Salto de fila	A + B → 6 destellos	0,4 s ON / 0,4 s OFF
Fin de trama	A + B encendidas continuamente	5 s

Los tiempos definidos podrán ajustarse experimentalmente durante las pruebas para garantizar una correcta detección de las señales a una distancia aproximada de 60 metros.

🚦 1. Inicio de trama

Para indicar el comienzo de una nueva matriz, las lámparas A y B se encenderán simultáneamente durante 5 segundos.

A: ████████████████████
B: ████████████████████
          5 s

       INICIO DE TRAMA

El receptor utilizará esta señal para identificar que comienza una nueva transmisión.

🧩 2. Transmisión de cada celda

La información se transmitirá celda por celda y fila por fila.

Antes de transmitir cada celda se realizará una espera de:

3 segundos

Posteriormente se transmitirá el color de la celda.

⬜ Celda blanca

Una celda blanca se representa mediante un destello simultáneo de A y B.

A: ███
B: ███

1 destello → BLANCA
⬛ Celda negra

Una celda negra se representa mediante dos destellos simultáneos de A y B.

A: ███   ███
B: ███   ███

2 destellos → NEGRA

Cada destello tendrá una duración de:

0,7 s ON
0,5 s OFF
🔤 3. Identificación de una celda con letra

El color de la celda no determina si existe una letra.

Una celda blanca o negra puede contener una letra.

Cuando la celda contiene una letra, después de transmitir el color se enviará la secuencia:

A → B → A → B

Esta secuencia se realizará rápidamente:

0,4 s ON
0,4 s OFF

La secuencia:

A-B-A-B

significa:

La celda contiene una letra.

Si esta secuencia no aparece, significa que la celda no contiene una letra y no se transmitirá código Morse.

📖 4. Código Morse

Cuando una celda contiene una letra, después de la secuencia:

A-B-A-B

se realizará una espera de:

2 segundos

Posteriormente se transmitirá la letra utilizando código Morse.

La correspondencia será:

Lámpara	Símbolo
A	Punto ·
B	Raya —

Cada símbolo Morse tendrá una duración de:

0,7 segundos

Entre símbolos consecutivos habrá una separación de:

0,5 segundos
Ejemplo: letra A

La letra A corresponde a:

· —

Por lo tanto:

A → B
Ejemplo: letra D
— · ·

Se transmite como:

B → A → A
Ejemplo: letra G
— — ·

Se transmite como:

B → B → A
Ejemplo: letra S
· · ·

Se transmite como:

A → A → A
📝 Ejemplo de celda blanca sin letra
Espera 3 s
     ↓
A+B → 1 destello
     ↓
CELDA BLANCA
     ↓
No aparece A-B-A-B
     ↓
No se transmite Morse
     ↓
Espera 2 s
     ↓
Siguiente celda
📝 Ejemplo de celda negra con letra G

La letra G corresponde al código:

— — ·

La transmisión será:

Espera 3 s
     ↓
A+B → A+B
     ↓
CELDA NEGRA
     ↓
A → B → A → B
     ↓
LA CELDA TIENE LETRA
     ↓
Espera 2 s
     ↓
B → B → A
     ↓
— — ·
     ↓
LETRA G
     ↓
Espera 2 s
     ↓
Siguiente celda
↩️ 5. Salto de fila

Cuando se hayan transmitido todas las celdas de una fila, se enviará una señal especial para indicar el cambio de fila.

La señal estará formada por:

A+B → A+B → A+B → A+B → A+B → A+B

Es decir:

6 destellos simultáneos de A y B.

Cada destello tendrá:

0,4 s ON
0,4 s OFF

Por lo tanto:

A+B × 6 → SALTO DE FILA

El receptor interpretará esta secuencia como el final de la fila actual y continuará con la siguiente fila.

Importante: después de la última fila no se enviará el salto de fila. Después de la última celda se enviará directamente la señal de fin de trama.

🛑 6. Fin de trama

Una vez transmitida la última celda de la matriz, ambas lámparas se mantendrán encendidas simultáneamente durante:

5 segundos
A: ████████████████████
B: ████████████████████
          5 s

        FIN DE TRAMA

Esta señal indica al receptor que la transmisión ha finalizado.

🔄 Secuencia completa de transmisión
┌──────────────────────────┐
│      INICIO DE TRAMA     │
│          A+B / 5 s       │
└─────────────┬────────────┘
              ↓
        ┌───────────┐
        │   FILA    │
        └─────┬─────┘
              ↓
        ESPERA 3 s
              ↓
       TRANSMITIR COLOR
              ↓
       ¿TIENE LETRA?
          ↙        ↘
        NO          SÍ
        ↓            ↓
    Sin Morse     A-B-A-B
                     ↓
                 Espera 2 s
                     ↓
                Código Morse
                     ↓
                 Espera 2 s
          ↘        ↙
              ↓
         SIGUIENTE CELDA
              ↓
             ...
              ↓
      A+B × 6 DESTELLOS
              ↓
       SIGUIENTE FILA
              ↓
             ...
              ↓
       ÚLTIMA CELDA
              ↓
        A+B durante 5 s
              ↓
         FIN DE TRAMA
👁️ Reconstrucción en el receptor

El receptor deberá utilizar únicamente las señales luminosas recibidas para reconstruir la matriz.

El proceso será:

Detectar A+B durante 5 s
           ↓
     INICIAR TRAMA
           ↓
       Esperar 3 s
           ↓
    Detectar el color
           ↓
¿Aparece A-B-A-B rápido?
       ↙           ↘
     NO             SÍ
      ↓              ↓
 Sin letra       Tiene letra
      ↓              ↓
      │          Esperar 2 s
      │              ↓
      │         Recibir Morse
      │              ↓
      └───────┬──────┘
              ↓
        Guardar celda
              ↓
         Esperar 2 s
              ↓
¿Se detectan 6 destellos A+B?
       ↙             ↘
     NO               SÍ
      ↓                ↓
Siguiente celda    Siguiente fila
                       ↓
                      ...
                       ↓
          Detectar A+B durante 5 s
                       ↓
                 FIN DE TRAMA
                       ↓
            Mostrar matriz final
👁️ Consideraciones sobre la velocidad

Debido a que la comunicación se realiza mediante señales ópticas y el receptor debe poder identificar los patrones de iluminación, los tiempos fueron seleccionados buscando un equilibrio entre la velocidad de transmisión y la facilidad de detección visual.

Los tiempos principales son:

Tiempo	Función
0,4 s	Pulsos rápidos de control y salto de fila
0,5 s	Separación entre símbolos Morse
0,7 s	Pulsos principales y símbolos Morse
2 s	Separación entre celdas y separación color-letra
3 s	Espera antes de cada celda
5 s	Inicio y fin de trama

Estos valores podrán ajustarse experimentalmente de acuerdo con los resultados obtenidos durante las pruebas a diferentes distancias.

El objetivo es encontrar un equilibrio entre:

Velocidad de transmisión ↔ Facilidad de observación ↔ Confiabilidad

🎯 Objetivo final

El objetivo del proyecto es transmitir una matriz mediante comunicación óptica utilizando dos lámparas controladas por Arduino, de manera que el receptor pueda reconstruir la matriz utilizando exclusivamente la información obtenida mediante las señales luminosas.

El receptor deberá identificar:

El inicio de la transmisión.
El color de cada celda.
La presencia o ausencia de una letra.
La letra correspondiente mediante código Morse.
El cambio entre filas.
El final de la transmisión.

Finalmente, la matriz reconstruida por el receptor deberá corresponder con la matriz original proporcionada al emisor.

🛠️ Tecnologías
Arduino
Comunicación óptica inalámbrica
Lámparas LED
Sistema de recepción óptica
Código Morse
Procesamiento de matrices
Reconstrucción de información
