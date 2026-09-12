# 🎲 Cubo Mágico — Solucionador de Rubik 3D

### 👉 **[ABRIR LA APP: mr-d0nut.github.io/cubo-magico](https://mr-d0nut.github.io/cubo-magico/)**

No hay que instalar nada: funciona en el navegador del móvil y del ordenador.

---

Aplicación web que resuelve un cubo de Rubik y te enseña la solución **animada en 3D**,
paso a paso, para que puedas seguirla con tu cubo real. Incluye además un **modo juego**
para intentar resolverlo tú, con cronómetro y pistas.

Todo en **un único archivo `index.html`**: sin build, sin dependencias que instalar.

## Qué hace

### 🎨 Modo edición
- Pinta el estado de tu cubo real sobre un **mapa 2D desplegado** o tocando directamente
  las pegatinas del **cubo 3D**.
- **Validación en vivo**: recuento por color y detección de estados físicamente imposibles
  (10 pegatinas rojas, una esquina girada, una arista volteada, dos piezas intercambiadas…).
- Botón de mezcla aleatoria.

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

## El motor de resolución

Implementación propia del **algoritmo de dos fases de Kociemba**, sin librerías externas:

| Fase | Objetivo | Coordenadas |
|------|----------|-------------|
| 1 | Llevar el cubo al subgrupo `G1 = <U, D, R2, L2, F2, B2>` | orientación de esquinas (2187) × orientación de aristas (2048) × capa media (495) |
| 2 | Resolver dentro de `G1` con sólo 10 movimientos | permutación de esquinas (40320) × aristas (40320) × capa media (24) |

Ambas fases usan **IDA\*** con tablas de poda generadas por BFS al arrancar
(~4 MB, unos 300 ms).

**Resultados medidos sobre 1000 cubos aleatorios**, verificando en cada caso que la
solución deja el cubo efectivamente resuelto:

```
fallos            : 0
longitud media    : 20,78 movimientos
longitud máxima   : 22
tiempo medio      : 46,7 ms
tiempo máximo     : 1501 ms
```

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
