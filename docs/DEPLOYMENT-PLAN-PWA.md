# Deployment Plan: Progressive Web App (PWA)
## VINCULUM Hub — Installable Offline Math Platform
**Status: Ready to build**

---

## What This Gives You
- Students tap "Add to Home Screen" on any device — instant app icon
- Works offline (all 26 engines + data cached locally)
- No app store approval needed
- Works on Chromebooks, iPads, Android, phones, desktops
- Foundation for native app wrappers (Capacitor)

---

## Task 1: Create Web App Manifest

**File:** `webapp/manifest.json`

```json
{
  "name": "VINCULUM Math Solutions",
  "short_name": "VINCULUM",
  "description": "K-PreCalc interactive math tools — Socratic flows, visual models, differentiated worksheets",
  "start_url": "/index.html",
  "display": "standalone",
  "background_color": "#0B1120",
  "theme_color": "#06B6D4",
  "orientation": "any",
  "categories": ["education", "kids"],
  "lang": "en",
  "icons": [
    { "src": "icons/icon-72.png",  "sizes": "72x72",   "type": "image/png" },
    { "src": "icons/icon-96.png",  "sizes": "96x96",   "type": "image/png" },
    { "src": "icons/icon-128.png", "sizes": "128x128", "type": "image/png" },
    { "src": "icons/icon-144.png", "sizes": "144x144", "type": "image/png" },
    { "src": "icons/icon-152.png", "sizes": "152x152", "type": "image/png" },
    { "src": "icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "icons/icon-384.png", "sizes": "384x384", "type": "image/png" },
    { "src": "icons/icon-512.png", "sizes": "512x512", "type": "image/png" },
    { "src": "icons/icon-512-maskable.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ]
}
```

**Action items:**
1. Create `webapp/manifest.json` with the above content
2. Add `<link rel="manifest" href="manifest.json">` to every HTML entry point
3. Add `<meta name="theme-color" content="#06B6D4">` to all HTML heads
4. Add Apple-specific meta tags for iOS home screen support

---

## Task 2: Create Service Worker

**File:** `webapp/sw.js`

The service worker caches all engines, data files, CSS, and HTML for offline use.

**Strategy:** Cache-first for engines/data (they change rarely), network-first for HTML (picks up updates).

**Cache list (auto-generated from file inventory):**
- All 26 JS engines from `js/`
- All CSS files from `css/`
- All HTML entry points (index.html, dialogue.html, socratic-flow-demo.html, etc.)
- App icons

**Versioning:** Use a `CACHE_VERSION` constant. Bump it in the service worker when engines update. Old caches auto-purge on activation.

**Action items:**
1. Create `webapp/sw.js` with install/activate/fetch handlers
2. Register the service worker from each HTML entry point
3. Add an update notification ("New version available — tap to refresh")

---

## Task 3: Generate App Icons

**Required sizes:** 72, 96, 128, 144, 152, 192, 384, 512 (plus 512 maskable)

**Options:**
- A. Design a VINCULUM logo icon (mathematical bridge/vinculum bar motif)
- B. Use a placeholder and replace later
- C. Generate from existing branding

**Action items:**
1. Create or obtain a 1024x1024 source icon
2. Generate all required sizes
3. Place in `webapp/icons/`

---

## Task 4: Add PWA Meta Tags to All HTML Entry Points

**Files to update:**
- `webapp/index.html`
- `webapp/dialogue.html`
- `webapp/socratic-flow-demo.html`
- Any other student-facing HTML pages

**Tags to add to each `<head>`:**
```html
<link rel="manifest" href="manifest.json">
<meta name="theme-color" content="#06B6D4">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="VINCULUM">
<link rel="apple-touch-icon" href="icons/icon-152.png">
```

---

## Task 5: Test PWA

**Checklist:**
- [ ] Chrome DevTools → Application → Manifest shows all fields
- [ ] Service worker registers and caches files
- [ ] "Install" prompt appears in Chrome
- [ ] App works offline (disable network in DevTools)
- [ ] Lighthouse PWA audit scores 100
- [ ] Works on iOS Safari (Add to Home Screen)
- [ ] Works on Android Chrome (Add to Home Screen)
- [ ] Works on Chromebook

---

## Verification
Run Lighthouse audit: Chrome DevTools → Lighthouse → PWA category → should score 100.
