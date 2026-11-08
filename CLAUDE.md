# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Idioma

Responde siempre en español en este proyecto.

## Project state

An Arkanoid/Breakout game in plain HTML, CSS and JS with **zero dependencies** (per `README.md`, written in Spanish). The game itself is **not implemented yet**: there is no `index.html`, no build system, no package manager, no tests and no linter. The repo is not a git repository (yet). Run the game by opening `index.html` in a browser once it exists (or any static file server).

## What exists

- `assets/assets/` (note the doubled `assets` folder) holds the provided art and audio:
  - `spritesheet-breakout.png` — the single spritesheet.
  - `spritesheet.js` — a plain (non-module) script defining globals: `SPRITES` (paddle, ball, per-colour blocks), `EXPLOSION_FRAMES` (4 frames per colour) and `EXPLOSION_DURATION` (150 ms), plus `loadSpritesheet(cb)`, `drawFrame(ctx, frame, x, y, w, h)` and `drawSprite(ctx, name, x, y, w, h)`. Block sprites are addressed as `'block_<colour>'` (gray, red, yellow, cyan, magenta, hotpink, green); `paddle` and `ball` by name.
  - `sounds/ball-bounce.mp3`, `sounds/break-sound.mp3`.

### Gotchas in `spritesheet.js`

- `loadSpritesheet` loads `'assets/spritesheet-breakout.png'` **relative to the HTML page**, not to the script. The page must therefore sit in a directory where `assets/spritesheet-breakout.png` resolves, which does not match the current `assets/assets/` layout. Either move/flatten the folder or change that path when wiring up the game.
- The sheet is copied to an offscreen canvas on load; drawing functions silently no-op until `ssLoaded` is true, so start the game loop inside the `loadSpritesheet` callback.
- `EXPLOSION_FRAMES.gray` reuses the red frames (same coordinates).

## Spec-driven workflow

Features are developed with two project skills in `.agents/skills/` (both `disable-model-invocation`, user-invoked only):

- `/spec <description>` — asks clarifying questions, then writes `specs/NN-slug.md` (state `Draft`) from `.agents/skills/spec/template.md`, and seeds `specs/.spec-config.yml`. It never writes code.
- `/spec-impl <NN-slug>` — only proceeds if the spec's state means "Approved" (the human changes it from Draft). Creates/switches to git branch `spec-NN-slug` (controlled by `AutoCreateBranch` in `specs/.spec-config.yml`), then implements the plan one step at a time, pausing for diff review. It never commits automatically and does not implement things outside the spec's scope.

Specs are written in the same language as the prompt/existing specs. `skills-lock.json` pins installed skills; `.claude/skills/neko` is an unrelated persona skill.
