# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

**G Consulting.ltd** is a single-page marketing website for a Bulgarian–Greek translation, document legalization, and university enrollment consultancy. The entire site lives in one self-contained file: `gconsulting_5.html` (~1,400 lines, ~570 KB). There is no build system, no package manager, and no framework — just vanilla HTML, CSS, and JavaScript.

The `_5` suffix in the filename indicates this is version 5 of the site. Do not rename or split the file without explicit instruction.

## Development workflow

Open `gconsulting_5.html` directly in a browser (`file://` protocol works). All changes are live after a hard-refresh. There is no dev server, no compilation step, and no hot reload.

To preview a specific section quickly, append the section anchor in the browser address bar (e.g. `#education`, `#order`, `#roadmap`).

## Architecture

### Single-file structure (top-to-bottom)

| Block | Location | Purpose |
|---|---|---|
| CSS custom properties | `:root` / `[data-theme="dark"]` | Design tokens — colors, shadows, easing. Edit here first for visual changes. |
| Dark-mode overrides | `[data-theme="dark"] …` selectors | Scoped overrides applied when `<html data-theme="dark">`. |
| Global styles + section CSS | Lines ~55–350 | Styled entirely with short utility-like class names (`.bp`, `.bo`, `.stitle`, etc.). No external stylesheet. |
| Responsive / media queries | Near line 326 | Single breakpoint around 768 px for mobile layout. |
| HTML sections | Lines ~351–685 | Markup only. Sections: `#hero`, `#why`, `#about`, `#education`, `#reviews`, `#order`, `#roadmap`, `#faq`, footer. `#services` is `display:none` — prices were moved to a modal. |
| `<script>` block | Lines ~686–1411 | All JavaScript. Three logical chunks separated by banner comments. |

### JavaScript internals

**Internationalization (`T` object, `setLang`, `data-i18n`)**
- All user-visible strings live in the `T` object: `T.en`, `T.bg`, `T.gr`.
- DOM elements carry `data-i18n="key"` attributes. `setLang(lang)` iterates them and sets `innerHTML`.
- `currentLang` (global) tracks the active language; it is updated by an override wrapper around `setLang` so that `selectCity()` can re-render university cards in the correct language after a language switch.
- Language buttons use class `.lb` and `.active`. Add a new language by adding a top-level key to `T` and a matching `.lb` button.

**University data (`UNIS` object, `selectCity`)**
- `UNIS` is a JS object keyed by city slug (`sofia`, `plovdiv`, `varna`, `pleven`, `burgas`, `ruse`, `stara_zagora`, `blagoevgrad`).
- Each entry is an array of university objects: `{ name:{en,bg,gr}, desc:{en,bg,gr}, specs:{en:[],bg:[],gr:[]} }`.
- `selectCity(city)` builds the `.uni-grid` HTML, injects it into `#uniContainer`, then stagger-animates `.uni-card` elements via `setTimeout`. It also toggles the Google Maps iframe in `#mapContainer` using `CITY_MAPS[city]`.
- `CITY_MAPS` maps city slugs to Google Maps embed URLs.

**Dark mode**
- Toggled by `toggleDarkMode()` — flips `data-theme` on `<html>` and persists to `localStorage` under the key `gc_theme`.
- IIFE at the bottom of the script block restores the saved theme on load.

**Prices modal**
- `openPricesModal()` / `closePricesModal()` control `#pricesModal`. The overlay closes on backdrop click and `Escape` key.
- The old `#services` section is kept in the HTML but hidden (`display:none`).

**Scroll behaviors**
- `IntersectionObserver` (`obs`) adds `.visible` to `.reveal` elements as they enter the viewport (staggered by 90 ms per element).
- A `scroll` listener updates the reading progress bar (`#progressBar`) width and toggles `.scrolled` on `<nav>` and `.visible` on `#scrollTop`.

**Form**
- `validateAndSubmit()` is purely client-side. On success it hides form fields and shows `#formSuccess`. No data is sent anywhere — add a backend/email integration if real submission is needed.
- Real-time `.error` class removal is wired to `input` events on form fields.

### Design system conventions

- **Color palette**: all colors are CSS custom properties on `:root`. Never hard-code hex values outside `:root` / dark-mode override blocks.
- **Typography**: `Cormorant Garamond` (serif) for headings (`.stitle`, `.ht`, `.oft`, etc.); `Inter` (sans-serif) for body text. Both load from Google Fonts.
- **Short class names**: the stylesheet uses terse abbreviations (`bp` = button primary, `bo` = button outline, `fq` = FAQ question, `fi` = FAQ item, etc.). Follow the same naming convention for new elements.
- **Animations**: use the `--ease-premium` variable (`cubic-bezier(0.16,1,0.3,1)`) for all transitions that should feel polished.

## Adding or editing content

- **New FAQ entry**: add a `.fi` block in `#faq`, mirror its question/answer keys in all three language objects inside `T`.
- **New university**: add an object to the correct city array in `UNIS`. All three language fields (`name`, `desc`, `specs`) are required.
- **New city tab**: add a `.city-btn` in `#cityGrid`, add a key to `CITY_MAPS`, add a city array in `UNIS`, and add the city name strings to all three `T` language objects.
- **Price changes**: prices are inside `#pricesModal` directly in the HTML (`.prv` elements); there is no separate data structure for them.
- **Contact links**: WhatsApp number and Viber/email links are hardcoded in the `#order` section HTML. Update them there directly.
