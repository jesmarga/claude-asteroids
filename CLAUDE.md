# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Clon de Asteroids en HTML5 Canvas + JavaScript ES6 vanilla. Sin dependencias, sin bundler, sin build, sin tests ni linter. Toda la lógica vive en `game.js`; `index.html` solo monta un `<canvas id="canvas" width="800" height="600">` y carga el script. Textos de UI y comentarios en español.

## Cómo correr

- Abrir `index.html` directamente en el navegador, o
- `npx serve .` y visitar `http://localhost:3000`

No hay comandos de build/lint/test. La verificación es manual en el navegador.

## Arquitectura de `game.js`

- **Loop**: `requestAnimationFrame(loop)` → `update(dt)` + `draw()`. `dt` en segundos, limitado a 0.05 para evitar saltos. Todas las velocidades están en px/s y se multiplican por `dt`.
- **Estado global** (variables `let` de módulo): `ship, bullets, asteroids, particles, score, lives, level, state, deadTimer`. `initGame()` reinicia todo; `nextLevel()` se dispara cuando `asteroids` queda vacío y genera `3 + level` asteroides grandes.
- **Máquina de estados** `state`: `'playing'` | `'dead'` (cuenta regresiva `deadTimer` de 2 s antes de `ship.reset()`) | `'gameover'` (Espacio reinicia). `update()` hace early-return por estado.
- **Entidades**: clases `Ship`, `Bullet`, `Asteroid`, `Particle`, cada una con `update(dt)` y `draw()`, y un flag `dead`. Se eliminan filtrando arrays (`arr.filter(x => !x.dead)`) tras actualizar/colisionar.
- **Espacio toroidal**: posiciones envueltas con `wrap(v, max)` usando las constantes `W`/`H` (deben coincidir con el tamaño del canvas en `index.html`). `Particle` no envuelve.
- **Colisiones**: circulares con `dist()`. Bala vs asteroide usa `a.radius`; nave vs asteroide usa `ship.radius + a.radius * 0.82` y se omite mientras `ship.invincible > 0`.
- **Asteroides**: tamaño 3/2/1 indexa los arrays paralelos `RADII`, `SPEEDS`, `POINTS` (índice 0 sin uso). `split()` devuelve dos de tamaño `size - 1`. Los grandes usan actualmente siempre la forma fija `LARGE_SHAPE` (la condición `Math.random() < 1` es siempre verdadera); el resto genera polígonos aleatorios.
- **Input**: `keys[code]` para teclas mantenidas; `pressed(code)` consume un flanco de `justPressed` (usado para disparar y reiniciar). Usa `e.code` (`ArrowLeft`, `Space`, etc.).
- **Render**: todo en vectores (stroke blanco) sobre fondo negro; HUD con `drawHUD()` y overlay con `drawOverlay()`.

## Notas

- El README todavía menciona power-ups y la "estrella fugaz", pero fueron eliminados del código (commit `13e713f`).
- Demo publicada en GitHub Pages: https://klerith.github.io/claude-asteroids/
