# Negeri Sembilan 农友天气 · WeatherNext

A single-file Progressive Web App (PWA) delivering farm weather forecasts and a
**microclimate disease-risk dashboard** for plantation, orchard and field-crop
locations around Mantin and Spg. Durian, Negeri Sembilan. Multilingual interface
(中文 / English / Bahasa Melayu / தமிழ் / မြန်မာ) with optional AI-generated
farming briefings.

Part of the WeatherNext family of per-region agricultural weather builds. This
build runs the full WeatherNext microclimate architecture (disease-risk engine +
fog engine), recalibrated for hot lowland conditions — the same engine as the
Raub build.

---

## What's new in this build (migrated to lowland microclimate architecture)

This release migrates Negeri Sembilan from the older forecast-only build onto
the fuller WeatherNext microclimate architecture, matching the Raub reference
build:

- **Microclimate disease-risk dashboard** — per-farm 0–100 risk scores for six
  fungal diseases (Botrytis / gray mould, downy mildew, late blight, powdery
  mildew, early blight, anthracnose), driven by leaf-wetness hours, humidity,
  rain, temperature suitability and a 3-day infection-pressure buildup.
- **Phase-2 lowland disease models** — crop-gated: Phytophthora (durian/pepper/
  citrus, rain + root-zone soil moisture + drainage), rice blast and Sigatoka
  (leaf-wetness driven) as daily scores; Ganoderma (oil palm) and Fusarium /
  Panama wilt (banana) as standing soil advisories. Field-calibration estimates,
  not field-validated — surfaced via the AI agronomist with a "tell us if wrong"
  framing.
- **29-crop master list** — tree/plantation and field crops plus vegetables;
  each farm picks its own crop, default durian.
- **Fog engine** — morning mist detection and a leaf-wetness contribution to
  disease pressure; lowland-weighted.
- **Coordinate-aware terrain note** — the storm card auto-detects the nearest
  mountain range and side from each farm's lat/lng.

### Broadcast & correctness improvements (shared with the Raub build)

- **GPS broadcast sort** — the WhatsApp broadcast lists farms NORTH→SOUTH then
  WEST→EAST, across all three broadcast modes.
- **Storm-confidence wording** — the storm line carries a single bracketed
  confidence tag (`（较确定）` when models agree, `（不确定）` otherwise); the
  old "models agree / uncertain" contradiction is gone.
- **AI greeting crop-owner fix** — the AI briefing addresses each farm by its
  actual crop in the user's language (29-crop salutation table, language-aware
  fallback).
- **Real model-run freshness** — the broadcast header reports the true ECMWF
  run time from live Open-Meteo metadata and warns only when data is genuinely
  stale (no false "data stale" warning at the morning broadcast windows).
- **Open-Meteo rate-limit handling** — multi-farm broadcasts fetch through a
  throttled pool with retry/backoff, avoiding HTTP 429.

**Calibration note:** the per-crop susceptibility values and Phase-2 thresholds
are agronomic estimates calibrated for lowland disease pressure; they are **not
field-validated**. They ship as sensible working defaults and should be reviewed
with a qualified agronomist and the growers' own field observation before being
relied on as absolute numbers.

---

## Live app

The app is served at:

```
https://stanleywoosweeleong.github.io/negerisembilan/
```

Open that link on a phone and use **"Add to Home Screen"** to install it as an
app. It works offline after the first visit (service-worker cached).

---

## Seeded locations

On first launch the app seeds the six farms below. They are auto-favourited and
can be renamed, edited, or deleted freely afterwards. Add as many more farms as
you like from inside the app. Each seeded farm carries a default crop
(**durian**) and a terrain zone, both editable in the app.

| English | 中文 | Coordinates | Zone |
|---|---|---|---|
| Mantin Tmn Permai Jaya | 文丁 中华华文小学 | 2.81778, 101.89250 | open plain |
| Mantin Andalas GIO GIO | 文丁 湖滨花园 | 2.82639, 101.88028 | open plain |
| MANTIN DALAM | 文丁达南 | 2.806111, 101.887500 | sheltered basin |
| Spg. Durian (Kg. Mentaus) 1 | 新邦榴梿 Pitt-猫王 | 3.12000, 102.21972 | open plain |
| Spg. Durian (Kg. Mentaus) 2 | 新邦榴梿 SAM | 3.11778, 102.21944 | open plain |
| Titi | 知知港 | 2.81670, 102.06670 | sheltered basin |

The terrain zone (riverine / sheltered basin / open plain / coastal) only
adjusts the disease-risk weighting. The zones above are sensible defaults from
the farm locations and can be refined per farm in the app.

The app also seeds a default user display name (**森美兰**, ASCII slug **NS9**),
which stays editable via **Edit Name** in the app.

This build carries the seed version **`ns-arch1`** (the lowland-architecture
seed), which re-applies the six farms with their crop/zone tags. Existing
installs pick this up automatically on their next visit — no need to clear data.
Any farm a user renamed, moved, or customised themselves is left untouched.

---

## API key — bring your own (important)

This app **does not ship with an embedded API key.** AI features (the farming
briefings) are powered by Google's Gemini API, and each user supplies their own
free key.

To enable the AI briefing:

1. Visit https://aistudio.google.com/app/apikey
2. Click **"Create API key"** — it's free.
3. In the app, open the **API Key** modal and paste the key (starts with `AIzaSy...`).

The key is stored only in that device's browser (`localStorage`) and is never
uploaded anywhere or committed to this repo. The core weather forecast and the
disease-risk dashboard both work without a key — only the AI briefing needs one.

**Recommended for users:** restrict your key in Google Cloud Console
(Application restrictions → Websites) to `stanleywoosweeleong.github.io/*`,
and limit it to the Generative Language API.

---

## Deploying

All 7 files live in the **repository root** — the service worker and manifest
use relative `./` paths, so a root deploy works with no changes.

```
index.html            — the app (single file: HTML + inlined CSS + JS + disease engine)
manifest.json         — PWA metadata
sw.js                 — service worker (offline cache)
icon-512.png          — app icon 512×512
icon-192.png          — app icon 192×192
apple-touch-icon.png  — iOS home-screen icon 180×180
favicon-32.png        — browser tab icon 32×32
```

To enable GitHub Pages: **Settings → Pages → Source: Deploy from branch →
`main` / `root`.** Pages serves over HTTPS automatically.

### Updating the app

The service worker caches the app shell. When you push changes, bump the
`CACHE_VERSION` string at the top of `sw.js` so users receive the update on
their next visit. The current value is:

```
wnext-negerisembilan-202606032020
```

---

## Tech notes

- **Weather data:** Open-Meteo API (no key required, network-first with cache fallback).
- **Disease-risk engine:** rule-based fungal-risk model (6 diseases + Phase-2
  tiers) using Open-Meteo leaf-wetness probability where available, otherwise
  derived from RH / dew-point spread / rain / cloud; adjusted by a per-crop
  susceptibility table, a terrain-zone multiplier and a 3-day infection-pressure
  buildup. Disease temperature bands are tuned for lowland pathogens.
- **AI model:** `gemini-2.5-flash` via the Generative Language API.
- **Storage namespace:** `negerisembilan__*` keys in `localStorage`, isolated
  from other WeatherNext regional builds so data never collides.
- **Cloud sync:** Firebase, namespaced under `appId: wnext-ag-v41-negerisembilan`.
- **Offline:** full app shell + last-fetched weather cached by the service worker.

---

## Disclaimer

The disease-risk scores are a **decision-support heuristic** based on weather
conditions and published infection thresholds — not a guarantee. They do not
replace field scouting or a qualified agronomist's judgment. Always confirm with
on-the-ground inspection before acting.
