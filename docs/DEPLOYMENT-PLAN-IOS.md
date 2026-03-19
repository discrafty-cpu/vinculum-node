# Deployment Plan: iOS (Apple App Store)
## VINCULUM Hub — Native iOS App via Capacitor
**Status: Requires PWA layer + Capacitor setup first**

---

## Prerequisites
- [x] Mac mini (required — iOS builds ONLY work on macOS)
- [ ] PWA layer complete (DEPLOYMENT-PLAN-PWA.md)
- [ ] Capacitor installed (DEPLOYMENT-PLAN-ANDROID.md Task 1)
- [ ] Apple Developer account ($99/year)
- [ ] Xcode installed (free from Mac App Store, ~12 GB)
- [ ] App icon (1024x1024)
- [ ] Privacy policy URL

---

## Important: Apple's Web Wrapper Policy

Apple sometimes rejects apps that are "just a web view." To pass review:
1. **Offline functionality is critical** — the service worker must work perfectly
2. **Native-feeling interactions** — smooth scrolling, no browser chrome
3. **Unique value** — not just a website in a wrapper
4. **Content justifies an app** — 291+ tools, Socratic flows, visual engine = strong case

VINCULUM Hub has a strong case because:
- 26 engines running client-side
- Full offline capability
- Interactive Socratic dialogues
- SVG visual generation
- Student progress persistence
- This is more like a native app than a website

---

## Task 1: Add iOS Platform

```bash
cd /Users/jondrummond/lesson-digester
npx cap add ios
```

Creates `ios/` directory with a full Xcode project.

---

## Task 2: Configure iOS Project

**In Xcode (ios/App/App.xcworkspace):**
1. Set Bundle Identifier: `com.vinculummath.hub`
2. Set Display Name: `VINCULUM Math`
3. Set Deployment Target: iOS 15.0 (covers ~95% of devices)
4. Set Device Family: Universal (iPhone + iPad)
5. Configure App Icons in Asset Catalog

**Capacitor iOS config in `capacitor.config.ts`:**
```typescript
ios: {
  contentInset: 'always',
  scheme: 'VINCULUM Math'
}
```

---

## Task 3: Generate iOS Icons

iOS requires these exact sizes in the Asset Catalog:

| Size | Usage |
|---|---|
| 20x20 @2x, @3x | Notification |
| 29x29 @2x, @3x | Settings |
| 40x40 @2x, @3x | Spotlight |
| 60x60 @2x, @3x | App icon (iPhone) |
| 76x76 @1x, @2x | App icon (iPad) |
| 83.5x83.5 @2x | App icon (iPad Pro) |
| 1024x1024 | App Store listing |

Use `npx capacitor-assets generate` with a 1024x1024 source to auto-generate all.

---

## Task 4: Build and Test Locally

```bash
npx cap sync ios
npx cap open ios                    # opens Xcode
```

In Xcode:
1. Select a simulator (iPhone 15, iPad Pro)
2. Click Run (Cmd+R)
3. Test all Socratic flows, visuals, themes
4. Test offline (enable airplane mode in simulator)
5. Test orientation changes (portrait + landscape)
6. Test on iPad (split view, slide over)

---

## Task 5: Create Apple Developer Account

1. Go to https://developer.apple.com/
2. Enroll as Individual or Organization ($99/year)
3. If Organization: need a D-U-N-S number (free, takes ~1 week)
4. Individual enrollment is faster (same day)

---

## Task 6: Create App Store Connect Listing

1. Go to https://appstoreconnect.apple.com/
2. Create new app → Bundle ID: `com.vinculummath.hub`

**Required assets:**
| Asset | Spec |
|---|---|
| App icon | 1024x1024 PNG (no alpha/transparency) |
| iPhone screenshots | 6.7" (1290x2796), 6.5" (1284x2778), 5.5" (1242x2208) |
| iPad screenshots | 12.9" (2048x2732) |
| App preview video | Optional but recommended (15-30 sec) |
| Description | What the app does, key features |
| Keywords | math, education, K-12, Socratic, CRA, visual |
| Support URL | GitHub repo or dedicated page |
| Privacy policy URL | Required |
| Age rating | 4+ (Education, no objectionable content) |

---

## Task 7: Privacy & Data Practices

Apple requires you to declare what data your app collects. For VINCULUM Hub:

**Data NOT collected:**
- No account creation
- No personal information
- No analytics/tracking
- No third-party SDKs
- No ads
- No location data

**Data stored on device only:**
- Quiz scores (localStorage)
- Flow progress (localStorage)
- Theme preferences (localStorage)

**App Privacy label:** "Data Not Collected"
This is the simplest and most favorable privacy classification.

---

## Task 8: Archive and Upload

In Xcode:
1. Set signing team to your Apple Developer account
2. Product → Archive
3. Distribute App → App Store Connect
4. Upload

Or via command line:
```bash
cd ios/App
xcodebuild -workspace App.xcworkspace -scheme App -configuration Release archive -archivePath build/VINCULUM.xcarchive
xcodebuild -exportArchive -archivePath build/VINCULUM.xcarchive -exportOptionsPlist ExportOptions.plist -exportPath build/
```

---

## Task 9: Submit for Review

1. In App Store Connect, select the uploaded build
2. Complete all metadata
3. Submit for review
4. Typical review time: 1-3 days (can be up to 7)

**Common rejection reasons and mitigations:**
| Reason | Mitigation |
|---|---|
| "Minimal functionality" | 291 tools + Socratic engine = NOT minimal |
| "Web wrapper" | Full offline + interactive content + localStorage |
| "Missing privacy policy" | Create one before submitting |
| "Crashes" | Test on real devices, not just simulator |

---

## Task 10: CI/CD for iOS Builds (Optional)

iOS CI requires macOS runners. Options:
- **GitHub Actions macOS runners** — included in free tier (but limited minutes)
- **Manual builds from your Mac mini** — simplest for now

```yaml
# .github/workflows/ios-build.yml
name: Build iOS Release
on:
  workflow_dispatch:

jobs:
  build:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npx cap sync ios
      - run: cd ios/App && xcodebuild -workspace App.xcworkspace -scheme App -sdk iphoneos -configuration Release build
```

---

## Estimated Timeline
| Step | Time |
|---|---|
| Xcode install | 1-2 hours (download) |
| Capacitor iOS setup | 30 min |
| Local testing | 1-2 hours |
| Apple Developer enrollment | Same day (individual) or 1 week (org) |
| Build + upload | 1 hour |
| Review approval | 1-7 days |

---

## Cost Summary
| Item | Cost |
|---|---|
| Apple Developer Program | $99/year |
| Xcode | Free |
| Capacitor | Free (open source) |
| macOS runner (CI) | Free tier: 25 min/month (or build locally) |
