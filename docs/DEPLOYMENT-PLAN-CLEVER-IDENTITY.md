# Deployment Plan: Clever SSO & Student Identity
## VINCULUM Hub — School-Level Authentication Without Storing PII
**Status: Research phase — no student data stored currently**

---

## Current State

VINCULUM Hub stores **zero** personally identifiable information (PII):
- No accounts, no logins, no names, no emails
- Quiz scores and flow progress saved in `localStorage` on the student's device
- Data stays on the device — never sent to a server

This is the simplest and safest privacy posture. The question is: when do you need more?

---

## When You'd Need Clever

You'd integrate Clever when you want:
1. **Teacher dashboards** — "Show me how each student did on the ratios flow"
2. **Cross-device continuity** — student starts on Chromebook, continues on iPad
3. **School deployment** — IT admin pushes VINCULUM to all students via Clever Library
4. **Rostering** — auto-import class lists instead of manual setup
5. **District adoption** — districts require Clever/ClassLink for procurement

You do NOT need Clever if students use the app independently without teacher tracking.

---

## What is Clever?

Clever (clever.com) is an SSO (Single Sign-On) and rostering platform used by 65%+ of U.S. K-12 schools. It provides:

- **Single Sign-On**: Students click one button to log into all their school apps
- **Rostering**: Automatically syncs class rosters from the SIS (Student Information System)
- **Data privacy**: Clever handles FERPA/COPPA compliance — the ed-tech app never sees raw PII
- **Clever Library**: School app store where IT admins discover and deploy apps

### How Clever Handles Student Data

**The key insight: Clever gives your app a pseudonymous ID, not the student's real name.**

When a student logs in via Clever:
1. Clever authenticates the student against the school's SIS
2. Clever sends your app a **Clever ID** (opaque string) + limited data you request
3. Your app stores progress against that Clever ID
4. You NEVER need to store: real name, email, address, birthdate, etc.

**What you can request (scopes):**
| Scope | Data | Do you need it? |
|---|---|---|
| `read:user_id` | Clever ID only | Yes (minimum) |
| `read:sis` | Grade level, school | Useful (auto-set grade) |
| `read:user` | First name (display only) | Optional |
| `read:contacts` | Parent email | No |

**VINCULUM would only need:** `read:user_id` + `read:sis` (grade level)

---

## Integration Architecture

```
Student clicks "Log in with Clever"
        │
        ▼
Clever authenticates against school SIS
        │
        ▼
Clever redirects to your app with auth code
        │
        ▼
Your app exchanges code for Clever ID + grade
        │
        ▼
App stores progress as: { cleverID: "abc123", scores: {...} }
        │
        ▼
Teacher dashboard queries by class (Clever provides roster)
```

### What You'd Need to Build

1. **Backend endpoint** — Receives Clever OAuth callback, exchanges code for token
2. **Simple database** — Maps Clever IDs to progress data (no PII)
3. **Teacher dashboard** — Shows class-level aggregated data

### What You Would NOT Need to Build

- User registration / password management (Clever handles it)
- FERPA/COPPA compliance paperwork (Clever's Data Privacy Agreement covers it)
- Roster management (Clever auto-syncs from SIS)

---

## Clever Partnership Process

### Step 1: Apply as a Clever Developer (Free)
- Go to https://dev.clever.com/
- Create a developer account
- Get sandbox credentials for testing

### Step 2: Build the Integration
- Implement Clever OAuth 2.0 (standard flow)
- Handle Clever Instant Login (SSO from Clever portal)
- Handle roster sync via Clever API (GET /sections, GET /students)

### Step 3: Test in Clever Sandbox
- Clever provides a sandbox district with fake students
- Test SSO flow, roster import, data exchange
- Verify no PII leaks

### Step 4: Submit for Clever Review
- Clever reviews your integration for:
  - Correct OAuth implementation
  - Proper data handling
  - Privacy compliance
  - User experience quality
- Review takes ~2-4 weeks

### Step 5: List on Clever Library
- Once approved, your app appears in Clever Library
- Schools can discover and deploy with one click
- Districts can bulk-deploy to all schools

---

## Privacy Policy Requirements

Even without Clever, app stores require a privacy policy. With Clever, you also need a Clever Data Privacy Addendum.

**For VINCULUM Hub's current state (no server, no PII):**

```
VINCULUM Math Solutions — Privacy Policy

What we collect: Nothing. VINCULUM Hub is a client-side application.
All data (quiz scores, flow progress, theme preferences) is stored
locally on your device using your browser's localStorage.

No data is transmitted to any server.
No accounts are created.
No personal information is collected, stored, or shared.

For school deployments using Clever SSO: We receive only a
pseudonymous Clever ID and grade level. We do not receive or
store student names, emails, addresses, or any other personally
identifiable information.

Contact: [your email]
Last updated: [date]
```

---

## Recommended Phased Approach

| Phase | What | When |
|---|---|---|
| **Now** | No accounts, localStorage only, zero PII | Current |
| **Phase 2** | PWA + app store deployment (still no accounts) | Next |
| **Phase 3** | Optional Clever SSO for school deployments | When a district asks |
| **Phase 4** | Teacher dashboard showing class progress | After Clever integration |

**The key principle: Don't add identity infrastructure until a school district asks for it.** The current zero-PII approach is your strongest privacy story. Clever integration is straightforward when you need it, but premature if you don't.

---

## Alternative: ClassLink

Some districts use ClassLink instead of Clever. The integration is similar (OAuth 2.0 + roster API). Consider supporting both if districts request it. The app architecture would be the same — just different OAuth endpoints.

---

## Cost Summary
| Item | Cost |
|---|---|
| Clever Developer account | Free |
| Clever Library listing | Free (Clever charges schools, not developers) |
| Backend hosting (when needed) | ~$5-20/month (Render, Railway, or Fly.io) |
| Database (when needed) | Free tier (Supabase, PlanetScale, or Turso) |
