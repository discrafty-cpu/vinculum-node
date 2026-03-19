# Deployment Plan: Android (Google Play Store)
## VINCULUM Hub — Native Android App via Capacitor
**Status: Requires PWA layer first (DEPLOYMENT-PLAN-PWA.md)**

---

## Prerequisites
- [x] Mac mini (you have this)
- [ ] PWA layer complete (manifest + service worker)
- [ ] Google Play Developer account ($25 one-time fee)
- [ ] App icon (512x512 minimum, 1024x1024 recommended)
- [ ] Privacy policy URL (even without student PII — Play Store requires it)
- [ ] Node.js + npm installed
- [ ] Android Studio installed (free)

---

## Task 1: Install Capacitor

```bash
cd /Users/jondrummond/lesson-digester
npm init -y                           # if no package.json exists
npm install @capacitor/core @capacitor/cli
npx cap init "VINCULUM Math" "com.vinculummath.hub" --web-dir webapp
```

This creates:
- `capacitor.config.ts` — points to `webapp/` as the web root
- Capacitor CLI ready for platform adds

---

## Task 2: Add Android Platform

```bash
npx cap add android
```

This creates an `android/` directory with a full Android Studio project.

---

## Task 3: Configure Android Project

**File:** `capacitor.config.ts`
```typescript
const config = {
  appId: 'com.vinculummath.hub',
  appName: 'VINCULUM Math',
  webDir: 'webapp',
  server: {
    // No server needed — all local files
  },
  android: {
    buildOptions: {
      keystorePath: 'release.keystore',
      keystoreAlias: 'vinculum'
    }
  }
};
```

**File:** `android/app/src/main/res/values/strings.xml`
- Set app name to "VINCULUM Math"

**Icons:**
- Replace `android/app/src/main/res/mipmap-*/` with your icons
- Capacitor provides `npx capacitor-assets generate` to auto-generate all sizes from a source image

---

## Task 4: Build and Test Locally

```bash
npx cap sync android              # copies webapp/ into Android project
npx cap open android               # opens in Android Studio
```

In Android Studio:
1. Click Run → Select emulator or connected device
2. Test all Socratic flows, visual engine, theme switching
3. Test offline mode (airplane mode)
4. Test localStorage persistence (close app, reopen, progress saved)

---

## Task 5: Generate Signed APK/AAB

```bash
# Generate a signing key (one time, keep this safe!)
keytool -genkey -v -keystore release.keystore -alias vinculum \
  -keyalg RSA -keysize 2048 -validity 10000

# Build release AAB (Android App Bundle — required by Play Store)
cd android
./gradlew bundleRelease
```

Output: `android/app/build/outputs/bundle/release/app-release.aab`

---

## Task 6: Create Google Play Developer Account

1. Go to https://play.google.com/console/
2. Pay $25 one-time registration fee
3. Complete identity verification (takes 1-3 days)

---

## Task 7: Create Play Store Listing

**Required assets:**
| Asset | Spec |
|---|---|
| App icon | 512x512 PNG |
| Feature graphic | 1024x500 PNG |
| Screenshots | Min 2, phone (1080x1920) + tablet (1200x1920) |
| Short description | Max 80 chars |
| Full description | Max 4000 chars |
| Privacy policy URL | Required (even for no-PII apps) |
| Content rating | Complete IARC questionnaire (Education, no violence, no ads) |
| Target audience | Teacher tool + student ages 5-18 |

**Suggested short description:**
"K-PreCalc math tools: Socratic flows, visual models, and differentiated worksheets."

**Content rating notes:**
- Category: Education
- No user-generated content
- No ads
- No in-app purchases (initially)
- No personal data collection
- Target: Teachers and students ages 5+

---

## Task 8: Upload and Submit for Review

1. Upload `.aab` file to Play Console
2. Fill in all listing details
3. Complete content rating questionnaire
4. Set pricing: Free
5. Select countries: United States (expand later)
6. Submit for review (typically 1-3 days for new apps)

---

## Task 9: Set Up CI/CD for Android Builds

**File:** `vinculum-node/.github/workflows/android-build.yml` (or in lesson-digester)

```yaml
name: Build Android Release
on:
  workflow_dispatch:
  release:
    types: [published]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { distribution: 'temurin', java-version: '17' }
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npx cap sync android
      - run: cd android && ./gradlew bundleRelease
      - uses: actions/upload-artifact@v4
        with:
          name: vinculum-android
          path: android/app/build/outputs/bundle/release/app-release.aab
```

---

## Estimated Timeline
| Step | Time |
|---|---|
| Capacitor setup + Android add | 30 min |
| Local testing | 1 hour |
| Signing + build | 30 min |
| Play Store listing | 1 hour |
| Review approval | 1-3 days |

---

## Cost Summary
| Item | Cost |
|---|---|
| Google Play Developer | $25 (one-time) |
| Capacitor | Free (open source) |
| Android Studio | Free |
| CI/CD (GitHub Actions) | Free (2000 min/month on free tier) |
