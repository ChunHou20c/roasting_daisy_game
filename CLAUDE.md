## CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-file browser game: user-configurable "he loves me / he loves me not"-style daisy petal picker. Users click petals to reveal phrases from a customizable choice list; settings are shareable via URL query params.

## Structure

The entire app lives in **`public/index.html`** — HTML, CSS, and JavaScript in one file. There is no build step, no package manager, no tests, and no dependencies beyond the browser. Alongside it, **`public/roasts.json`** is an editable array of roast strings the site owner can customize without touching the HTML.

To run locally, **serve** the `public/` directory (e.g. `python3 -m http.server --directory public`) — opening `index.html` via `file://` will work, but the `fetch()` of `roasts.json` fails under `file://` and the game silently falls back to the inline `DEFAULT_ROASTS` array.

## Architecture

The script section in `index.html` is organized around a few globals and a handful of functions — worth understanding before editing:

- **State globals** (top of `<script>`): `choices`, `mode` (`sequence` | `random`), `petalMin`/`petalMax`/`doubleChance`, `N`, `remaining`, `clickCount`, `petalEls`. All mutation is direct — no framework.
- **URL sync**: `loadFromUrl()` hydrates state from query params on page load. Two URL builders exist: `buildCurrentUrl()` produces the "author view" URL (used by `apply-btn` for `history.replaceState` — settings stay visible on refresh), while `buildShareUrl()` wraps it and appends `&play=1` for the share-box value. The `play=1` param is consumed by `loadFromUrl` to hide the entire `#settings` bar on load — so a recipient opening a shared link sees only the flower. When adding new settings, wire them through `loadFromUrl` and `buildCurrentUrl`.
- **Flower rendering**: `buildFlower()` lays petals out on a Fermat/Vogel spiral using the golden angle (`PHI = π·(3−√5)`). Petal count is re-randomized in `[petalMin, petalMax]` on every build/reset. Petals are sorted inner-to-outer so outer petals render on top. Each petal's geometric/color params are stashed on `dataset` for reuse by the falling-petal animation.
- **Petal click flow** (`onPetalClick`): with `doubleChance` probability, a second random visible petal is also removed; otherwise just the clicked one. Each removed petal emits a phrase via `getPhrase(clickCount)` — sequential or random depending on `mode` — and spawns a `launchFallingPetal` animation via `requestAnimationFrame` (projectile motion with gravity, spin, wobble, fade). After the phrase(s) are composed, a single roast roll (per click event, not per petal) has `ROAST_CHANCE` probability of appending a random entry from `roasts` using the same ` · ` separator. The center disc scales with `N` via `centerR = 10 + N * 0.35`.
- **Roast loading** (`loadRoasts`): the `roastsFile` global is set by `loadFromUrl` from the `?roasts=<name>.json` query param (sanitized by `^[a-z0-9_.-]+\.json$` — slashes still blocked, so no path traversal; dots allowed for i18n-style names like `roasts.zh.json`). Defaults to `roasts.json`. `loadRoasts` fetches it and replaces `DEFAULT_ROASTS` on success; any failure — 404, network, `file://` protocol, malformed JSON — silently keeps the defaults. Call is fire-and-forget after `buildFlower()`; the fallback ensures the first click never misses due to async. `buildShareUrl` echoes `roastsFile` back into the URL only when it differs from the default, so a shared link preserves the language choice. A Chinese roast set ships as `public/roasts.zh.json` — reach it via `?roasts=roasts.zh.json`.
- **UI i18n** (`UI_STRINGS` / `currentLang` / `t`): a minimal lookup for UI strings that need to match the roast language. `currentLang()` derives the code from `roastsFile` via the `/^roasts\.([a-z]+)\.json$/i` pattern — `roasts.zh.json` → `zh`, anything else falls back to `en`. To add a language, drop a `roasts.<code>.json` into `public/`, add an entry to `UI_STRINGS`, and add a `<input type="radio" name="lang" value="<code>">` option. Initial `roastsFile` is set by: `?roasts=` URL param > `navigator.language` prefix (`zh*` → Chinese) > default `roasts.json`. The language radio handler updates `roastsFile`, re-fetches roasts, refreshes the pick-another button text, and updates the share box URL — it does **not** rebuild the flower, so mid-game language swaps are non-destructive.
- **End-of-game CTA** (`#pickAnotherBtn`): a pill button anchored at the bottom of `#stage`, hidden by default (`opacity: 0; pointer-events: none`). When `remaining === 0`, `onPetalClick` adds the `.show` class after a 600 ms delay so it fades/slides in after the final message; `buildFlower()` clears the class and re-sets the text via `t('pickAnother')` so language changes (or reset-after-URL-edit) are picked up.
- **DOM coupling**: element IDs referenced from JS are defined inline in the HTML (`flowerSvg`, `petalsGroup`, `message`, `counter`, `stage`, choice/slider inputs). Renaming an ID requires updating both.

## Conventions

- Lowercase UI copy throughout (`hide`, `apply & reset`, `copy link`).
- CSS uses hairline borders (`0.5px solid rgba(0,0,0,0.xx)`) and a warm off-white palette (`#fafaf8`, `#f0efe8`). Keep the minimalist aesthetic when adding UI.
- Petal colors are HSL-parameterized off the petal's radial position `t`; adjust the `hue`/`light` formulas rather than hardcoding colors.
