# Tide & Tally — Commercial Fishing Tycoon

*Catch the tide. Count the haul.*

Play it live: https://deathlyhellos.github.io/tide-and-tally/

A mobile-first fishing tycoon game (v17). Deploy gear on fishing grounds, haul catch, manage fuel and ice, sell to seafood processors, complete contracts, upgrade your boat, hire deckhands, and send expeditions offshore.

## Play on the web (GitHub Pages)

GitHub Pages serves this repo from the root: `index.html`, `manifest.json`, `sw.js`, plus `icon-192.png` / `icon-512.png`.

## Repository layout

- `index.html`, `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png` — the live GitHub Pages site (root)
- `capacitor.config.json`, `package.json` — Capacitor Android starter (see `README_PLAY_STORE.md`)
- `README_PLAY_STORE.md` — Play Store packaging notes (Capacitor / PWABuilder)

## Build an Android app

See `README_PLAY_STORE.md` — Capacitor + Android Studio, or PWABuilder from the live Pages URL.

GitHub Actions builds it in the cloud instead: `.github/workflows/android-release.yml`
— run it from the repo's Actions tab. It produces a debug APK and a release AAB.
Signing needs the four `ANDROID_*` secrets (see `WIRING-PLAY-BILLING-AND-ADMOB.md` §5).

## Monetization (Play Billing + AdMob)

See `WIRING-PLAY-BILLING-AND-ADMOB.md`. The shop is wired to RevenueCat
(Starter Pack $1.99 one-time, Captain's Pass $4.99/mo) and AdMob rewarded
ads, native only — the web version has no store. Code is wired; the native
build and a real test purchase still need Android Studio + Play Console
product setup.
