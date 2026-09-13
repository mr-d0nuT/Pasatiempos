# 🎲 Cubo Mágico — Solucionador de Rubik 3D

### 👉 **[ABRIR LA APP: mr-d0nut.github.io/cubo-magico](https://mr-d0nut.github.io/cubo-magico/)**

No hay que instalar nada: funciona en el navegador del móvil y del ordenador.

---

**Tres juegos en una sola página.** Se cambia entre ellos pulsando el título de la cabecera.

## 🎲 Cubo Mágico

Resuelve un cubo de Rubik y te enseña la solución **animada en 3D**, paso a paso, para que
puedas seguirla con tu cubo real. Incluye además un **modo juego** para intentar resolverlo
tú, con cronómetro y pistas.

## 🔴 Simon

El clásico juego de memoria: cuatro teclas se encienden formando una secuencia que crece una
tecla por ronda, y hay que repetirla sin fallar.

Disco en 3D reproduciendo el aparato real: carcasa gris torneada con suelo hundido, pared y
reborde que sobresale por encima de las teclas. Las teclas se elevan y se iluminan al sonar,
y una luz de su color derrama sobre la carcasa.

Un detalle que importa más de lo que parece: las teclas **no** se recortan por radios sino por
rectas paralelas a las diagonales. Con radios, la separación entre teclas sería una cuña —
estrecha junto al centro y ancha en el borde—; con rectas es un canal de anchura constante,
como en el original.

**Sonido sintetizado** con los tonos clásicos (mi4, la4, do4 y sol3) generados con la Web Audio
API, dos osciladores ligeramente desafinados por tono: ni un solo archivo de audio. Cuatro
dificultades, modo estricto y récord guardado, todo dentro del engranaje de ajustes.

## 🟦 Tetris

Tablero 3D de 10×20 con **sombra de caída** (marca en gris dónde aterrizará la pieza), bolsa
de 7 (cada pieza sale una vez por tanda), empujes contra la pared al girar, vista previa de la
siguiente, y nivel que sube solo cada 10 líneas.

El tablero se encuadra por **caja, no por esfera envolvente**: es un rectángulo alto y estrecho,
y ajustarlo como esfera desperdiciaba casi la mitad del alto disponible. En el teléfono el panel
ocupa lo justo (una tira de datos y los controles) y el tablero se queda con el **70 % de la
pantalla**; empezar y pausar viven como iconos en la columna derecha.

**Controles en móvil**, que es donde se juega de verdad: arrastrar a los lados mueve, arrastrar
abajo baja, tocar gira y un golpe seco hacia abajo suelta la pieza. El golpe exige que el gesto
sea **claramente vertical**; sin esa condición, un arrastre lateral rápido soltaba la pieza sin
querer. Con teclado: flechas, `Espacio` suelta y `P` pausa.

Aquí el arrastre **no** gira la cámara, al contrario que en las otras dos apps: el arrastre es
el control del juego.

---

Las tres comparten lienzo, renderizador y controles — tres contextos WebGL serían el triple de
memoria de vídeo para nada. Sólo cambia la escena y la cámara que se dibujan.

Todo en **un único archivo `index.html`**: sin build, sin dependencias que instalar.

## Qué hace

### 🎨 Modo edición
- Pintas **tocando directamente las pegatinas del cubo 3D**. Para llegar a las caras de
  atrás, el botón de desplegar abre el cubo en cruz y ahí también se pinta.
- **Validación en vivo**: recuento por color y detección de estados físicamente imposibles
  (10 pegatinas rojas, una esquina girada, una arista volteada, dos piezas intercambiadas…).
- Botón de mezcla aleatoria.

### ⚙️ Ajustes (icono de engranaje, abajo a la izquierda)
- **Tipo de cubo**: 3×3×3 clásico o **2×2×2 pocket**, ambos con solucionador propio.
- **El color rojo**: liso o con el logotipo de CCOO, dibujado por código sin archivos
  externos. Un solo ajuste que afecta a las pegatinas rojas del cubo y a la tecla roja de
  Simon, en vez de duplicar la opción en cada app.
- Velocidad de la animación y ayudas visuales (flecha de giro, recolocación automática
  de la cámara, notación clásica).
- Botón aparte de **modo día / noche**.

### 🎮 Modo juego
- Gira las capas **arrastrando directamente sobre el cubo 3D**, con la botonera, o con el
  teclado (`U` `R` `F` `D` `L` `B`, con `Mayús` para el sentido contrario).
- Cronómetro, contador de movimientos y **deshacer**.
- Botón de **pista**: calcula el siguiente movimiento óptimo desde donde estés.
- ¿Atascado? «Resolver desde aquí» pasa tu estado actual al solucionador.

### ▶️ Modo resolución
- Animación fluida (`ease-in-out`) de cada giro.
- Controles de reproducción: play/pausa, paso a paso adelante y atrás, barra de progreso
  e ir a cualquier movimiento pulsando sobre él.
- Cuatro velocidades, de «lenta» a «turbo».

## Los motores de resolución

Dos solucionadores propios, sin librerías externas.

### 3×3×3 — algoritmo de dos fases de Kociemba

| Fase | Objetivo | Coordenadas |
|------|----------|-------------|
| 1 | Llevar el cubo al subgrupo `G1 = <U, D, R2, L2, F2, B2>` | orientación de esquinas (2187) × orientación de aristas (2048) × capa media (495) |
| 2 | Resolver dentro de `G1` con sólo 10 movimientos | permutación de esquinas (40320) × aristas (40320) × capa media (24) |

Ambas fases usan **IDA\*** con tablas de poda generadas por BFS al arrancar
(~4 MB, unos 300 ms).

**Medido sobre 1000 cubos aleatorios**, verificando en cada caso que la solución deja
el cubo efectivamente resuelto:

```
fallos            : 0
longitud media    : 20,78 movimientos
longitud máxima   : 22
tiempo medio      : 46,7 ms
tiempo máximo     : 1501 ms
```

### 2×2×2 — búsqueda exhaustiva, solución demostrablemente óptima

El 2×2 sólo tiene esquinas y, fijando una como referencia, su espacio de estados es de
**3.674.160** posiciones: cabe entero en memoria. Así que aquí no se busca nada. Se hace
un BFS completo desde el cubo resuelto, se guarda la distancia exacta de *cada* estado, y
resolver consiste en bajar por el gradiente. La solución es siempre la más corta posible.

Las dos coordenadas (permutación y orientación) evolucionan de forma independiente porque
ambas se guardan *por posición* y no *por pieza*, lo que permite tablas de movimiento
minúsculas (5040×9 y 729×9) y un BFS de ~100 ms.

Como el 2×2 no tiene centros, el cubo puede estar girado de cualquiera de las 24 formas
posibles; antes de resolver se prueban todas hasta encontrar la que deja la esquina de
referencia en su sitio, y después se traducen los giros de vuelta al marco del usuario.

Verificación del espacio de estados completo:

```
estados totales      : 3.674.160
inalcanzables        : 0
diámetro             : 11   (el número de Dios del 2×2)
por distancia        : 1, 9, 54, 321, 1847, 9992, 50136, 227536,
                       870072, 1887748, 623800, 2644
```

Esa distribución coincide exactamente con los valores publicados para el 2×2×2.

### Qué NO está soportado

**4×4×4 y superiores, y los mods de forma** (Pyraminx, Megaminx, Skewb). No es una
limitación de tiempo: necesitan otra familia de algoritmos por completo —método de
reducción, más los casos de paridad que sólo aparecen en cubos de lado par— y cada uno
es un proyecto en sí mismo. Preferí dos motores verificados a cinco a medias.

### Otros detalles
- **Desplegar**: el cubo se abre en cruz para ver y pintar las 6 caras a la vez, y se
  vuelve a plegar. Se animan copias de las pegatinas, no las originales, para no tocar la
  jerarquía real del cubo.
- **Pantalla completa**: oculta el panel y pide pantalla completa al navegador, para jugar
  sólo con arrastres sobre el cubo. El cronómetro y el contador se mudan al chip de la
  cabecera, y las dos columnas de iconos siguen a mano (pista, deshacer, mezclar).
- **No pierdes el trabajo**: el cubo pintado y la partida en curso se guardan solos.
- **Vibración** en cada giro y al resolver, donde el dispositivo la soporte.

## Uso

Abre `index.html` en el navegador. No hace falta nada más.

Para servirlo en local:

```bash
python3 -m http.server 8000
```

## Tecnología

- **Three.js r128** (vía CDN) para el 3D. Cubies con aristas redondeadas generadas por
  código y pegatinas como formas con esquinas redondeadas.
- Cámara orbital propia, con amortiguación, inercia, zoom por rueda y pellizco.
- Cero dependencias para el solucionador: está todo en el propio archivo.
- Diseño responsive real: en móvil el mapa 2D pasa de cruz a «T» para que las pegatinas
  se puedan tocar con el dedo, con zonas seguras para pantallas con muesca.

## Notas de implementación

- El estado del cubo vive en un array de **54 facelets** en el orden estándar de Kociemba
  (`U R F D L B`). La escena 3D se reconstruye desde ese array tras cada giro, de modo que
  no se acumula deriva numérica por mucho que gires.
- El sentido de giro de cada cara está verificado: los 18 movimientos animados coinciden
  exactamente con lo que predice el modelo lógico del cubo.
- `window.__cubo` expone el estado interno para trastear desde la consola del navegador.

## Licencia

MIT
