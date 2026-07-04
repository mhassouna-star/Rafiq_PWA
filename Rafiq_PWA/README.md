# رفيق — Rafiq PWA (v1.0.0)

Arabic-first caregiver companion for parents of children with autism / developmental delay.
Zero dependencies. Same deployment pattern as Ibadah Index (static → GitHub Pages).

## Files
| File | Purpose |
|---|---|
| `index.html` | Entire app: UI, store, insight engine, coach, routine, report |
| `sw.js` | Service worker — offline app shell (bump `CACHE` version each deploy) |
| `manifest.webmanifest` | Install-to-home-screen metadata (RTL, Arabic) |
| `icon-192.png`, `icon-512.png` | PWA icons |

## Deploy (GitHub Pages)
1. New repo → upload all 5 files to root.
2. Settings → Pages → Deploy from branch → `main` / root.
3. Open the URL on a phone → browser menu → **Add to Home Screen**.
   (Service worker + install require HTTPS, which Pages provides. On `file://` the app still runs; SW is skipped automatically.)

## Architecture (why it won't hang or freeze)
- **No framework, no bundler** — one delegated click listener for the whole app; no per-element handlers to leak.
- **Versioned store with a migration gate** (`SCHEMA_VERSION`): corrupt or legacy data is salvaged or quarantined, never crashes boot. *(Direct fix for the Ibadah Index legacy-schema bug.)*
- **Single Timer controller** — one interval at any moment, always cleared on exit. *(Direct fix for the stale-closure bug.)*
- **Debounced persistence** — max one `localStorage` write / 400 ms, wrapped in try/catch (quota-safe).
- **Render caps** — timeline renders ≤60 entries; log store hard-capped at 1,000 records.
- **Lazy daily reset** — routine "done" state resets on first read of a new day; no midnight timers.
- **All user text passes `Fmt.esc()`** — XSS-safe even with pasted content.
- `prefers-reduced-motion` respected; safe-area insets handled for notched phones.

## Feature map
- **Onboarding** → child name/age (stored locally only — privacy line shown to user).
- **Home** → greeting, 14-day stats, AI-style insight card (rule-based sleep↔meltdown correlation over 28 days, min-evidence threshold before claiming a pattern).
- **Log** → six 3-tap flows (meltdown, sleep, meal, new word, skill, toilet), timeline with delete.
- **Coach** → offline knowledge base (6 topics, keyword-matched), answers injected with the child's real logged data; safe fallback; "not a doctor" disclaimer throughout.
- **Routine** → editable steps, daily check-off, star rewards, full-screen child mode with visual countdown.
- **Report** → 30-day clinic-ready summary + SVG chart (meltdowns colored by prior-night sleep), print-to-PDF via `@media print`.

## Tested
12 headless unit tests pass: schema migration, log CRUD + caps, pattern-detection math,
coach matching/fallback/data-injection, lazy daily reset, numeral formatting, HTML escaping.

## Next steps (when ready)
- Wire the Coach seam to the Claude API (replace `Coach.answer()` — the chat UI already supports async typing state).
- Export report as shareable PDF file (currently via system print dialog).
- Multi-child profiles: lift `child` to `children[]` in schema **v2** — the migration gate is already in place for this.
