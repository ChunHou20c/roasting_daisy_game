# roasting daisy game

A digital take on "she loves me, she loves me not" — except the flower also roasts you for needing a flower to make your decisions.

**Try it:** [chunhou20c.github.io/roasting_daisy_game](https://chunhou20c.github.io/roasting_daisy_game/)

## what it's for

You're stuck on a choice: lunch, career, Saturday plans, whether to text them back. Your brain has locked up and "flip a coin" isn't dramatic enough.

Type in your options, pluck petals, get answers. Every now and then, get told you're a grown adult consulting flora.

That's the whole thing. No accounts, no backend, no AI — just a yellow flower with opinions, and a share link you can send to an indecisive friend so the flower can judge *them* for a change.

## how to share it

Configure your options in the settings bar, then copy the share link. The recipient sees only the flower — the settings panel is hidden on their end via a `?play=1` flag so they don't know what the choices were or how they were weighted.

All state lives in URL query params:

| param       | meaning                                                             |
| ----------- | ------------------------------------------------------------------- |
| `choices`   | comma-separated list of phrases, one shown per plucked petal        |
| `mode`      | `sequence` (cycle the list) or `random`                             |
| `pmin`      | minimum petal count (re-randomized on every new flower)             |
| `pmax`      | maximum petal count                                                 |
| `double`    | `0`–`100`: chance that a click drops two petals at once             |
| `roasts`    | roast pool filename (`roasts.json`, `roasts.zh.json`, etc.)         |
| `play`      | `1` hides the settings bar — recipient-only view                    |

Browser language is auto-detected on first visit (`zh*` → Chinese, everything else → English). The language radio in settings can override it; the choice round-trips through the share URL.

## customizing roasts

Roasts are plain string arrays in `public/roasts.json` (English) and `public/roasts.zh.json` (Chinese). Edit freely — the site fetches them at runtime, no rebuild needed.

To add a new language:

1. Drop `public/roasts.<code>.json` into place (e.g. `roasts.fr.json`).
2. Add a matching `UI_STRINGS.<code>` entry and a `<input type="radio" name="lang">` option in `index.html`.

## running locally

No build step — everything is in `public/`. Serve it with any static file server:

```sh
python3 -m http.server --directory public
```

Opening `index.html` directly over `file://` mostly works, but `fetch('roasts.json')` will fail and the game silently falls back to a built-in roast array.

## deploying

`.github/workflows/pages.yml` deploys `public/` to GitHub Pages on every push to `main`. In repo **Settings → Pages**, set the source to **GitHub Actions**. The live site is at [chunhou20c.github.io/roasting_daisy_game](https://chunhou20c.github.io/roasting_daisy_game/).
