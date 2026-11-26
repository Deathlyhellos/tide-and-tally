# Tide & Tally — Google Play Starter Bundle

This folder contains your complete **Tide & Tally (v17)** game packaged for either **Capacitor + Android Studio** (offline native `.aab`) or **PWABuilder** (web-hosted `.aab`).

## Option A: Build an Offline Android App with Capacitor
1. Install **Node.js** and **Android Studio** on your computer.
2. Open a terminal in this folder and run:
   ```bash
   npm install
   npx cap add android
   npx cap sync android
   npx cap open android
   ```
3. In Android Studio, go to **Build > Generate Signed Bundle / APK**, choose **Android App Bundle (.aab)**, and create a new upload keystore.

## Option B: Package via PWABuilder (No Android Studio)
1. Upload the contents of the `www/` folder (`index.html`, `manifest.json`, `sw.js`) to **GitHub Pages** or **Netlify** (and add a `512x512` PNG icon in `manifest.json`).
2. Paste your HTTPS URL into https://www.pwabuilder.com/ and click **Package for Stores > Android** to download your signed `.aab`.

## Google Play Store Checklist
- Register a Google Play Console developer account ($25 one-time fee).
- Prepare a `512x512` icon, `1024x500` feature graphic, and at least 2 `1080x1920` phone screenshots.
- Add a public Privacy Policy URL.
- Run a 14-day closed test with 12–20 testers if using a new personal developer account.
