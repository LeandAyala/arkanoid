# SPEC 01 — MVP jugable de Arkanoid

> **Status:** Draft
> **Depends on:** Ninguna
> **Date:** 2026-11-06
> **Objective:** Crear un Arkanoid de un solo nivel, jugable en el navegador con ratón y teclado, con 3 vidas, 10 puntos por bloque, sonido, explosiones y un overlay de inicio/victoria/derrota.

## Por qué existe esta spec

El repositorio solo tiene los assets (spritesheet y sonidos) y ningún código del juego. Esta spec define el mínimo que lo convierte en algo que una persona pueda abrir y jugar de principio a fin. Todo lo que no sea imprescindible para eso queda fuera.

## Scope

**In:**

- Una página `index.html` con un `<canvas>` de 800x600, sin dependencias, que se abre directamente en el navegador.
- Pala controlada **a la vez** con ratón (sigue el cursor en X) y teclado (flechas izquierda/derecha y A/D).
- Pelota que nace pegada a la pala y se lanza con Espacio o clic. Se repite tras perder una vida.
- Rebote de la pelota en paredes, techo, pala y bloques.
- Rebote en la pala con ángulo según el punto de impacto (centro = casi vertical, bordes = oblicuo).
- Cuadrícula de 5 filas x 10 columnas de bloques, una fila por color. Cada bloque se rompe con un solo golpe.
- 3 vidas. La pelota que cae por debajo de la pala resta una vida.
- Puntuación: +10 por bloque roto. Se muestran puntos y vidas en pantalla.
- Un único overlay sobre el canvas que sirve de pantalla de inicio, victoria y derrota, con texto distinto en cada caso. Clic o Espacio inicia o reinicia.
- Sonido: `ball-bounce.mp3` en rebotes y `break-sound.mp3` al romper un bloque.
- Animación de explosión del bloque roto con `EXPLOSION_FRAMES` y `EXPLOSION_DURATION`.
- Aplanar `assets/assets/` a `assets/` para que `loadSpritesheet` resuelva la ruta sin tocar `spritesheet.js`.

**Out of scope (para futuras specs):**

- Varios niveles o patrones de bloques distintos.
- Power-ups, bloques con varios golpes o indestructibles.
- Pantallas separadas de menú, victoria o game over (aquí es un overlay).
- Puntuación máxima persistente (`localStorage`).
- Pausa, control táctil, versión móvil, responsive.
- Aceleración progresiva de la pelota.
- Música de fondo.

## Data model

Este es el primer código del proyecto. Estructuras nuevas, todas en `game.js`:

```js
const CONFIG = {
  width: 800, height: 600,
  paddle: { w: 120, h: 14, y: 560, speed: 8 },   // speed: px/frame con teclado
  ball:   { size: 16, speed: 6 },                 // px/frame
  blocks: { rows: 5, cols: 10, w: 72, h: 24, offsetX: 40, offsetY: 60 },
  rowColors: ['red', 'yellow', 'cyan', 'magenta', 'green'],
  lives: 3, pointsPerBlock: 10,
  maxBounceAngle: 60, // grados desde la vertical, en los bordes de la pala
};

const state = {
  phase: 'start',      // 'start' | 'ready' | 'playing' | 'won' | 'lost'
  score: 0,
  lives: 3,
  paddle: { x: 0 },
  ball: { x: 0, y: 0, vx: 0, vy: 0 },
  blocks: [/* { x, y, w, h, color, alive } */],
  explosions: [/* { x, y, color, startTime } */],
};
```

Convenciones:

- Origen de coordenadas arriba a la izquierda.
- `x`, `y` de pala, pelota y bloques son la esquina superior izquierda.
- Velocidades en píxeles por frame.
- Fases: `start` (overlay inicial), `ready` (pelota pegada a la pala), `playing`, `won` y `lost` (overlay final).

## Implementation plan

1. Aplanar `assets/assets/*` a `assets/*` y borrar la carpeta `assets/assets/` vacía. Verificación: `assets/spritesheet-breakout.png`, `assets/spritesheet.js` y `assets/sounds/*.mp3` existen.
2. Crear `index.html`, `style.css` y `game.js` mínimos. El HTML carga `assets/spritesheet.js` y `game.js`. `game.js` llama a `loadSpritesheet` y, en su callback, dibuja el fondo y la pala con `drawSprite`. Verificación: la pala se ve y la consola no muestra errores.
3. Añadir el bucle con `requestAnimationFrame` y el movimiento de la pala con teclado y ratón, limitada a los bordes del canvas. Verificación: la pala se mueve con ambos controles.
4. Añadir la pelota en estado `ready` pegada a la pala y su lanzamiento con Espacio o clic. Rebote en paredes y techo. Verificación: la pelota sale y rebota.
5. Añadir el rebote en la pala con ángulo según el punto de impacto, y la pérdida de pelota por debajo. Verificación: el ángulo cambia según dónde golpea.
6. Generar la cuadrícula de bloques, dibujarlos y detectar colisión pelota-bloque (rebote según el eje de menor solape). Los bloques se marcan como `alive = false`. Verificación: los bloques desaparecen al golpearlos.
7. Añadir puntuación, vidas y HUD (texto en el canvas). Perder la última vida pasa a `lost`, romper todos los bloques pasa a `won`. Verificación: los contadores cambian bien.
8. Añadir el overlay de inicio/victoria/derrota con reinicio completo del estado. Verificación: se puede jugar varias partidas seguidas sin recargar.
9. Añadir sonidos de rebote y de rotura. Verificación: suenan en cada evento sin errores de autoplay (el primer sonido ocurre tras un clic o tecla del usuario).
10. Añadir la animación de explosión al romper un bloque, con una duración de `EXPLOSION_DURATION` ms. Verificación: se ve la animación y desaparece sola.

## Acceptance criteria

- [ ] Abrir `index.html` en el navegador muestra el overlay de inicio sin errores en la consola.
- [ ] `assets/assets/` ya no existe y el spritesheet carga desde `assets/spritesheet-breakout.png`.
- [ ] Clic o Espacio en el overlay de inicio comienza la partida con la pelota pegada a la pala.
- [ ] Mover el ratón mueve la pala y las flechas/A/D también, sin tener que cambiar de modo.
- [ ] La pala no sale del canvas por ningún lado.
- [ ] Espacio o clic lanza la pelota estando en `ready`; no hace nada mientras está en `playing`.
- [ ] La pelota rebota en las paredes laterales y el techo.
- [ ] Golpear la pala en el centro envía la pelota casi vertical y en el borde con un ángulo cercano a 60° desde la vertical.
- [ ] Hay 50 bloques (5 filas x 10 columnas) con los colores `red`, `yellow`, `cyan`, `magenta`, `green`, uno por fila.
- [ ] Un bloque golpeado desaparece, la pelota rebota, suma exactamente 10 puntos y suena `break-sound.mp3`.
- [ ] Se ve la animación de explosión en el lugar del bloque durante unos 150 ms.
- [ ] Rebotar en pared, techo o pala reproduce `ball-bounce.mp3`.
- [ ] Dejar caer la pelota resta una vida y vuelve a `ready`; el HUD muestra las vidas restantes.
- [ ] Perder la tercera vida muestra el overlay de derrota con la puntuación final.
- [ ] Romper los 50 bloques muestra el overlay de victoria con 500 puntos.
- [ ] Desde el overlay de victoria o derrota, clic o Espacio reinicia con 0 puntos, 3 vidas y los 50 bloques.

## Decisions

- **Sí:** un solo overlay para inicio, victoria y derrota. Lo pidió el usuario y evita crear pantallas aparte.
- **Sí:** ratón y teclado activos a la vez, sin selector. Lo pidió el usuario. Gana la última entrada recibida.
- **Sí:** aplanar `assets/assets/` en vez de editar la ruta en `spritesheet.js`. Deja rutas limpias y el archivo provisto intacto.
- **Sí:** pelota pegada a la pala hasta lanzar. Es lo clásico y da control al jugador.
- **Sí:** rebote con ángulo según el punto de impacto. Sin esto el juego se vuelve repetitivo.
- **Sí:** canvas 800x600 con 5 filas x 10 columnas. Elección del usuario.
- **Sí:** tres archivos propios (`index.html`, `style.css`, `game.js`), sin módulos ES. `spritesheet.js` es un script clásico con globales, así que se mantiene el mismo estilo.
- **Sí:** 5 de los 7 colores del spritesheet. `gray` y `hotpink` quedan libres para niveles futuros.
- **No:** persistir la puntuación máxima. Es otra spec.
- **No:** pausa. No es necesaria para considerar el MVP jugable.
- **No:** librerías ni frameworks. Es requisito del proyecto (cero dependencias).

## Risks

| Riesgo                                                          | Mitigación                                                                                                 |
| --------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Autoplay de audio bloqueado por el navegador                    | Los sonidos solo se reproducen tras una interacción del usuario (clic o Espacio de inicio).               |
| La pelota atraviesa bloques o la pala a alta velocidad          | Velocidad moderada (6 px/frame, menor que el alto de un bloque). Sin aceleración en este MVP.             |
| Velocidad dependiente de la frecuencia de refresco del monitor  | Aceptado en el MVP (px/frame). Se puede pasar a delta time en otra spec.                                  |
| Ángulo muy horizontal que alarga la partida                     | `maxBounceAngle` limitado a 60° desde la vertical.                                                         |
| `spritesheet.js` no dibuja hasta cargar la imagen               | El bucle del juego arranca solo dentro del callback de `loadSpritesheet`.                                  |

## What is **not** in this spec

- Más de un nivel ni power-ups.
- Pantallas de menú, victoria o game over separadas del overlay.
- Puntuación máxima persistente.
- Pausa, control táctil o versión móvil.
- Música de fondo y aceleración de la pelota.

Cada uno de estos puntos, si se hace, va en su propia spec.
