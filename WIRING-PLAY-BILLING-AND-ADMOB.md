# Wiring Tide & Tally for real — Play Billing (RevenueCat) + AdMob

Date: 2026-11-26. Status: **code wired, not yet tested end-to-end.**
The game code now calls real RevenueCat / AdMob plugins on native Android,
and does nothing on the web. The native build, the Play Console products,
and a real test purchase have NOT been done yet — this machine has no Java /
Android SDK, so those steps need Android Studio (see §5).

## 1. What changed in the code (2026-11-26)

- `package.json`: Capacitor 6 → **8.6.3** (`@capacitor/core`, `android`, `cli`).
  Verified by a clean `npm install` in a scratch folder on 2026-11-26:
  `@capacitor/core 8.6.3`, `@capacitor/android 8.6.3`, `@capacitor/cli 8.6.3`,
  `@revenuecat/purchases-capacitor 15.0.0`, `@capacitor-community/admob 8.4.2`
  all resolve with no peer conflicts.
- New deps: `@revenuecat/purchases-capacitor ^15.0.0` (peer `@capacitor/core >=8`),
  `@capacitor-community/admob ^8.4.2` (peer `@capacitor/core ^8`).
  RevenueCat's Android SDK in this version (`purchases-hybrid-common 21.0.0`)
  bundles Billing Library 9, which meets Play's Billing Library 8+ rule for
  new apps.
- `index.html` — the old fake shop is gone:
  - `buyStarterPack()` used to grant +25 pearls free on tap. It now calls
    `Purchases.purchasePackage()` for the `starter_pack` package and grants
    the pearls **only** when RevenueCat confirms the purchase.
  - `toggleCaptainsPass()` used to flip the pass on/off free. It is now
    `manageCaptainsPass()`: subscribe via `purchasePackage()` for the
    `captains_pass` package; if already active, tapping tells the player to
    cancel in Play Store > Payments & subscriptions > Subscriptions. The
    game never toggles a subscription itself.
  - `watchRewardedAd()` used to show a fake 3-second modal and grant the
    reward. It now calls `AdMob.prepareRewardVideoAd()` /
    `showRewardVideoAd()` and grants the reward **only** if the AdMob
    `onRewardedVideoAdReward` event fires (or `showRewardVideoAd()` resolves
    with a reward). The 4-per-day cap is kept.
  - A **Restore purchases** button calls `Purchases.restorePurchases()`.
  - Entitlements are synced on launch (`getCustomerInfo()`): the pass turns
    on/off with the real subscription, and `starterPackOwned` is restored
    without re-granting the +25 pearls (a reinstall must not pay out twice).
  - Web (GitHub Pages): `window.Capacitor` is absent, so the store shows
    "Play app only", the ads show "only in the Google Play app", and no
    price is charged and nothing is granted. The "Play Billing Ready" badge
    is replaced by a live status ("Web preview" / "Play Billing" /
    "Store unavailable").
- Plugins are reached through `window.Capacitor.Plugins.Purchases` /
  `.AdMob` — no bundler was added, the game is still a single file.
  `npx cap sync` registers the native plugins after `npm install`.

Smoke tests run 2026-11-26 (Node, mocked DOM / mocked plugins — not a
real Android build):
- Web mode: tapping all three shop buttons grants nothing; buttons read
  "Play app only". PASSED.
- Native mock: purchase grants +25 pearls exactly once, a later entitlement
  sync does not re-grant, subscribing activates the pass, a rewarded ad
  grants only after the reward event. PASSED.

## 2. Play Console — products to create (do this first)

Same developer account as House Guest Planner (Majo.brown, Canada).
Create the app entry first: package `com.tideandtally.game`, name
"Tide & Tally". Then, under Monetize:

| Product ID | Type | Price | What it unlocks in-game |
|---|---|---|---|
| `starter_pack` | In-app product, one-time, non-consumable | $1.99 CAD | +25 pearls, +2 hold slots, once |
| `captains_pass` | Subscription, monthly base plan, no free trial to start | $4.99 CAD / month | Auto-Ice, +20% contract bonus, +1 fleet slot, while active |

Notes:
- Set the prices in CAD in Play Console; Google converts for other countries.
- For the subscription, create one base plan (monthly, auto-renewing). Leave
  free trial / intro pricing off for launch — add later if you want.
- Do not create the products as consumables: the Starter Pack must be
  buy-once (non-consumable), and the Pass is a subscription, not a consumable.

## 3. RevenueCat — same pattern as House Guest Planner

- Create a **new RevenueCat project/app** for Tide & Tally
  (`com.tideandtally.game`). Do not reuse the "Guest Planner" project.
- Link it to Google Play the same way as House Guest Planner (service
  account credentials in Play Console > Users and permissions).
- In RevenueCat, create two entitlements: `starter_pack` and
  `captains_pass`. Import the two Play products above and attach each to
  its entitlement, then attach both to the **default offering** as packages
  (the code finds a package by its product identifier, so the offering
  must contain a package whose product is `starter_pack`, and one whose
  product is `captains_pass`).
- Copy the RevenueCat **public Google API key** for this app and paste it
  into `index.html`, in the `MONETIZATION` block:
  `revenueCatGoogleApiKey: 'goog_…'` (public key — safe to put in the app).
  Until that key is pasted, the native store shows "Store not configured yet".

## 4. AdMob — separate from RevenueCat

- Create an AdMob account (if none), then an app for Tide & Tally, then a
  **rewarded** ad unit. Note the App ID (`ca-app-pub-…`) and the ad unit ID.
- The code currently uses Google's test rewarded unit
  `ca-app-pub-3940256099942544/5224354917` with `admobTestMode: true`.
  Before release: replace it with the production unit ID in `index.html`,
  set `admobTestMode: false`, and put the production App ID in
  `android/app/src/main/AndroidManifest.xml` as
  `<meta-data android:name="com.google.android.gms.ads.APPLICATION_ID"
  android:value="ca-app-pub-…"/>` (added by `cap sync` / the AdMob plugin
  setup — verify it is there in Android Studio).
- Never tap your own production ads, and never use production ad units
  while testing — that can get the AdMob account suspended.

## 5. Native build — GitHub Actions (no Android Studio needed)

`.github/workflows/android-release.yml` builds the app in the cloud.
Run it manually: repo > Actions > "Android release build" > Run workflow.
It produces two artifacts: a debug APK (install on a phone to test) and a
release AAB (upload to Play Console).

What the workflow does on every run: check out the repo, set up Node 22
and Java 21, stage the web files into `www/`, `npm ci`, `npx cap add
android` + `npx cap sync android` (this scaffolds the native project and
registers the AdMob + RevenueCat plugins), set `versionName` to 1.0.0,
inject the AdMob App ID into `AndroidManifest.xml`, then run Gradle
(`assembleDebug` + `bundleRelease`). The `android/` folder is **not**
committed to the repo — the workflow regenerates it each run.

Verified 2026-11-26 by simulating every step locally up to the Gradle
compile: `npm ci` works from the committed `package-lock.json`, `cap add`
+ `cap sync` find both plugins, the version bump applies, and the AdMob
App ID lands in the manifest. The Gradle compile itself is the one step
not yet run anywhere — it needs the Android SDK, which the GitHub runner
has preinstalled. The first workflow run is the real test of that step;
if Gradle fails, its log will say which dependency or SDK piece is missing.

### Signing the release AAB (one-time setup, still manual)

The workflow signs the AAB only when these four repository secrets exist
(Settings > Secrets and variables > Actions). Without them it builds an
unsigned AAB and says so in the run summary — Play Console will reject
an unsigned AAB.

- `ANDROID_KEYSTORE_BASE64` — base64 of the upload keystore file
- `ANDROID_KEY_ALIAS` — key alias inside the keystore
- `ANDROID_KEYSTORE_PASSWORD` — keystore password
- `ANDROID_KEY_PASSWORD` — key password (often the same)

Create the keystore once, on any computer with Java (JDK 21) installed:

```bash
keytool -genkeypair -v -keystore tide-and-tally-upload.jks \
  -alias tide-and-tally -keyalg RSA -keysize 2048 -validity 10000
base64 -w0 tide-and-tally-upload.jks   # paste the output as ANDROID_KEYSTORE_BASE64
```

Keep the `.jks` file and its passwords somewhere safe — you need the same
upload key for every future release. If it is lost, Google Play support can
reset the upload key for an app using Play App Signing, but that is a
support request, not something to rely on.

AdMob App ID: set the repository **variable** `ADMOB_APP_ID` to the
production App ID before a real release. Without it the workflow uses
Google's test App ID (`ca-app-pub-3940256099942544~3347511713`), which is
for testing only and must never ship.

### Alternative: Android Studio

If you would rather build on your own computer, Android Studio does the
same Gradle build locally: `npm install`, `npx cap add android`,
`npx cap sync android`, open `android/` in Android Studio, build a signed
AAB. Same result, more setup on your machine. Replit alone cannot do this
step — it has no Android SDK, Gradle, or signing (verified Nov 26, 2026);
Capgo Build from Replit is a paid alternative ($14/mo after trial).

## 6. Test plan (before closed testing / production)

Use a Play license tester account and Google's test payment methods.

1. Install from internal testing. Store shows "Play Billing", prices $1.99 / $4.99.
2. Buy the Starter Pack with a test card: +25 pearls and +2 hold slots land.
3. Kill and reopen the app: Starter Pack still owned, pearls NOT granted again.
4. Uninstall and reinstall, tap Restore purchases: Starter Pack returns, no double pearls.
5. Subscribe to Captain's Pass: pass benefits on (Auto-Ice, +20% contracts, +1 fleet slot).
6. Cancel in Play Store > Subscriptions: benefits stay until the period ends, then drop.
7. Watch a rewarded ad with test units: reward lands only after the ad completes;
   closing early grants nothing; 4th ad in a day is refused with the cap message.
8. Web version (GitHub Pages): shop buttons read "Play app only", no prices
   are charged, nothing is granted.

## 7. Still outstanding (not part of this wiring)

- Privacy policy URL (must disclose the Tailwind / Google Fonts CDNs) and the
  Data Safety form; IARC content rating.
- Trader status declaration — deadline **Feb 1, 2027**, account-level,
  affects House Guest Planner too.
- Possible 14-day closed test with 12+ testers before production access.
- The fictional-regulations disclaimer for in-game seasons / quotas / licences.
