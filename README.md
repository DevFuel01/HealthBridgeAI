# HealthBridgeAI
Offline-first AI healthcare triage system with voice input, risk scoring and resource-aware recommendations.

# HealthBridge AI

**An offline-first PWA healthcare triage and guidance system** — built to serve underserved communities with safe, accessible, AI-assisted health guidance.

---

## ✨ Features

### 🩺 AI Triage System
- Text + voice input, 3-step form, multi-language support
- Rules-first safety engine (deterministic) — detects 15+ red-flag patterns; AI cannot override
- Gemini 1.5 Flash — empathetic patient instructions, clinical summaries, multilingual
- Risk scoring (0–100), urgency classification (EMERGENCY / URGENT / ROUTINE / SELF-CARE)
- Patient Guidance Generator — next steps, home care, when to see a doctor, red flags

### ♿ Accessibility
- Voice input (Web Speech API) and text-to-speech result reading
- High contrast mode, large text toggle, simple language mode
- ARIA labels, keyboard navigation, skip links, WCAG AA compliant
- Screen reader optimised HTML, minimal cognitive load UI

### 📡 Offline-First (PWA)
- Works without internet after first load (service worker app shell cache)
- Saves cases offline in IndexedDB sync queue
- Automatic sync when connection returns + manual sync button
- Real-time connection status banner (🔴 offline / 🟢 back online)
- Dedicated offline fallback page

### 🌿 Sustainability
- Low bandwidth mode — skips AI call entirely, serves static fallback guidance
- Auto-detection of slow networks (`slow-2g` / `2g`)
- Resource-aware recommendations based on local healthcare access level
- Low-resource treatment alternatives when hospitalisation isn't available
- Efficient API usage, minimal data transfer

### 🏥 Clinic / Staff Dashboard
- Secure PHP session login (bcrypt + HttpOnly cookies)
- Case list with pagination, urgency badges, risk scores
- Priority sorting and multi-filter (urgency, status, date range)
- Full case detail view — symptoms, AI summary, risk circle, red flags, comorbidities
- Case resolution marking (NEW → REVIEWING → RESOLVED) with live badge update
- **Case History Log** — chronological timeline of all status changes with staff name + timestamp

### 📦 Resource Management
- Inventory manager — enter medicine stock, supply levels, equipment availability
- Threshold-based low-stock alerts shown on case detail views
- AI recommendations automatically adjust based on inventory shortage
- Alternative suggestions (e.g. oral rehydration instead of IV fluids)
- Full audit trail of every stock update

### 📊 Analytics Dashboard
- Case count stats — Total, Today, This Week, New, Reviewing, Resolved
- Urgency distribution — colour-coded horizontal bar chart with % breakdown
- 14-day trend sparkline — daily case volume, CSS-only, no libraries
- Average resolution time — from case created → RESOLVED audit log
- Top symptom keyword frequency — ranked bars across last 200 cases

### � Security
- PDO prepared statements everywhere (zero string-concatenated queries)
- Input sanitisation (`strip_tags`, whitelist validation, `mb_strlen`)
- Session protection — HttpOnly, SameSite=Lax, Secure flag, `session_regenerate_id()`
- IP-based rate limiting with atomic MySQL upsert (triage: 10/min, auth: 10/min)
- Gemini API key server-side only — never exposed to the browser
- `display_errors` forced off in code regardless of `php.ini`
- `error_log` used for all internal errors; JSON-only public error responses

---

## 🏗 Architecture

```
Browser (PWA)
  │  Service Worker → cache-first for app shell, network-first for API
  │  IndexedDB → offline case queue
  │
  ▼
PHP API Layer  (/api/*.php)
  ├── RateLimiter    → IP+endpoint window limiting (MySQL)
  ├── Auth           → PHP session guard
  ├── RulesEngine    → deterministic urgency (ALWAYS runs first)
  ├── GeminiClient   → AI guidance (advisory only, cannot override rules)
  └── PDO/MySQL      → cases, users, inventory, audit_logs, rate_limits
```

### Request flow — `POST /api/triage`

```
Browser POST { symptoms, severity, duration, age_band, comorbidities,
               red_flags, healthcare_access, low_bandwidth, lang }
  → RateLimiter::check('triage')        10 req/min per IP
  → Input validation + strip_tags()
  → RulesEngine::evaluate()             ← ALWAYS runs, cannot be overridden
  → if low_bandwidth → GeminiClient::getFallbackOnly()
    else             → GeminiClient::generateGuidance()
  → INSERT INTO cases … (PDO prepared)
  → Return JSON { urgency, risk_score, ai_summary, ai_guidance, … }
```

---

## 📁 Folder Structure

```
HealthBridgeAI/
├── api/
│   ├── analytics.php     # GET — 5 analytics metrics
│   ├── auth.php          # POST login / logout / check
│   ├── cases.php         # GET list/single/history, PATCH status
│   ├── inventory.php     # GET list, PATCH stock level
│   ├── sync.php          # POST offline batch sync
│   └── triage.php        # POST main triage endpoint
├── config/
│   ├── database.php      # PDO singleton
│   └── env.php           # .env loader + error_reporting hardening
├── css/
│   ├── accessibility.css # High-contrast + large-text overrides
│   └── main.css          # Full design system
├── database/
│   ├── schema.sql        # MySQL 8 schema — 5 tables + indexes
│   └── seed.sql          # Demo user + inventory items
├── icons/
│   ├── icon.svg
│   ├── icon-192.png
│   └── icon-512.png
├── js/
│   ├── accessibility.js      # All a11y toggles + BBC-style voice input
│   ├── analytics.js          # Analytics dashboard renderer
│   ├── app.js                # Core init, toast, logout
│   ├── case-detail.js        # Case detail + history timeline
│   ├── connection-status.js  # Real-time online/offline banner
│   ├── dashboard.js          # Staff case list + filters
│   ├── inventory.js          # Inventory viewer + stock update
│   ├── results.js            # Triage results + TTS
│   ├── sync.js               # IndexedDB offline queue + sync
│   └── triage.js             # Triage form + voice + low-bandwidth
├── modules/
│   ├── Auth.php          # Session management (bcrypt + HttpOnly)
│   ├── GeminiClient.php  # Gemini API wrapper (server-side only)
│   ├── RateLimiter.php   # IP rate limiting (MySQL-backed, atomic)
│   └── RulesEngine.php   # Deterministic triage logic ⭐
├── analytics.html
├── case-detail.html
├── dashboard.html
├── generate-icons.php    # Run once to generate PWA icons
├── index.html
├── inventory.html
├── manifest.json
├── offline.html
├── results.html
├── staff-login.html
├── sw.js                 # Service worker (cache-first + background sync)
├── triage.html
├── .env.example
└── README.md
```

---

## 🛠 Tech Stack

| Layer       | Technology                                          |
|-------------|-----------------------------------------------------|
| Frontend    | HTML5, CSS3 (vanilla), JS ES2021                    |
| Backend     | PHP 8.x, strict_types                               |
| Database    | MySQL 8 (InnoDB, JSON columns, FK constraints)      |
| AI          | Google Gemini 1.5 Flash (server-side only)          |
| Offline     | Service Worker, IndexedDB, Background Sync API      |
| Voice       | Web Speech API (SpeechRecognition + SpeechSynthesis)|
| Auth        | PHP sessions, bcrypt password_hash                  |
| Server      | XAMPP / Laragon (Apache + PHP)                      |

---

## ⚙️ Setup (XAMPP)

### 1. Prerequisites
- XAMPP with PHP 8.1+ and MySQL 8
- Gemini API key (free at [aistudio.google.com](https://aistudio.google.com))

### 2. Database

```bash
mysql -u root < database/schema.sql
mysql -u root healthbridge_ai < database/seed.sql
```

Or via **phpMyAdmin → SQL tab**:
```sql
SOURCE c:/xampp/htdocs/HealthBridgeAI/database/schema.sql;
SOURCE c:/xampp/htdocs/HealthBridgeAI/database/seed.sql;
```

### 3. Environment

```bash
cp .env.example .env
```

Edit `.env`:
```env
GEMINI_API_KEY=AIza...yourkey...
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=healthbridge_ai
DB_USER=root
DB_PASS=
APP_SECRET=any-random-32-char-string
APP_DEBUG=false
```

### 4. Generate PWA icons

```bash
php generate-icons.php
```

### 5. Open the app

Start **Apache** and **MySQL** in XAMPP, then visit:
```
http://localhost/HealthBridgeAI/
```

---

## 🔑 Default Staff Login

| Email | Password |
|-------|----------|
| `staff@healthbridge.ai` | `Demo1234!` |

> Created by `database/seed.sql`. Change immediately in production.

---

## 📱 Install as PWA

1. Open `http://localhost/HealthBridgeAI/` in Chrome or Edge
2. Click the install icon in the address bar
3. The app launches standalone and works offline after first visit

---

## 🎬 Demo Script

### Scene 1: Landing + Accessibility (30s)
1. Show landing page — headline, features grid, urgency badge examples
2. Click ♿ → enable **High Contrast** → show colour shift
3. Enable **Large Text** → show font scaling
4. Enable **Simple Language** → AI guidance simplifies terminology

### Scene 2: Emergency Triage + Voice (60s)
1. Click **Start Triage**
2. Click 🎤 → speak: *"I have severe chest pain radiating to my left arm and I can't breathe"*
3. Severity = 5, Duration = Less than 24 hours
4. Step 2: Age adult, no pregnancy
5. Step 3: Check **Chest pain** + **Difficulty breathing** → Submit
6. 🚨 EMERGENCY overlay → "Call 999/911/112" button displayed

### Scene 3: SELF-CARE case + TTS (30s)
1. New triage → *"Sore throat, mild fever, runny nose for 2 days"*
2. Severity 2, duration 1-3 days → submit
3. SELF-CARE result appears → click 🔊 Play → guidance read aloud

### Scene 4: Offline Mode (30s)
1. DevTools → Network → **Offline**
2. Submit symptoms → 🔴 offline banner appears → case queued toast
3. Network → **Online** → sync triggered → "Synced 1 case" toast

### Scene 5: Staff Portal (60s)
1. Navigate to **Staff Login** → `staff@healthbridge.ai` / `Demo1234!`
2. Dashboard → case list with urgency badges + risk scores
3. Filter by **EMERGENCY** → filtered list
4. Click a case → full detail view with AI summary, risk score ring
5. Update status: NEW → REVIEWING → RESOLVED → badge updates live
6. Scroll to **🕓 Case History Log** → timeline of status changes shown

### Scene 6: Analytics (30s)
1. Click **📊 Analytics** in nav
2. Show case summary stat cards — Total, Today, Resolved
3. Scroll to **Urgency Distribution** bars — colour-coded
4. Show **14-Day Trend** sparkline
5. Show **Top Symptom Keywords** frequency bars

### Scene 7: Inventory + Resource Awareness (30s)
1. Navigate to **Inventory**
2. Show low-stock items flagged in red (⚠ Low badge)
3. Update a stock level → Save → reload shows updated count
4. Return to a case detail → show Low Inventory Alert banner

---

## 🔒 Security Summary

| Feature | Implementation |
|---------|----------------|
| SQL injection | PDO prepared statements on every query |
| XSS output | `htmlspecialchars()`, `escHtml()` in JS |
| Input validation | `strip_tags()`, whitelist enums, `mb_strlen` |
| Passwords | `password_hash()` / `password_verify()` (bcrypt) |
| Sessions | HttpOnly, SameSite=Lax, Secure flag, `session_regenerate_id()` |
| Rate limiting | IP-based, atomic MySQL upsert, 10 req/min on triage |
| API key | Gemini key server-side only — never reaches browser |
| Error display | `display_errors=0` forced in code; logs go to PHP error log |
| Audit trail | Every status change + stock update written to `audit_logs` |

---

## ⚠️ Known Limitations

- **Not a medical device** — hackathon demo, not CE/FDA certified
- Voice input requires Chrome or Edge (Web Speech API)
- Gemini AI requires internet; graceful fallback to static guidance in low-bandwidth mode
- No password reset flow (demo scope)
- PWA background sync requires `SyncManager` browser support
- Rate limiting resets per-minute window; not persistent across server restarts

---

## 🚀 Post-Hackathon Roadmap

1. HTTPS + production hardening (HSTS, CSP headers, disable directory listing)
2. Integrate ICD-10 / SNOMED CT ontologies for structured symptom classification
3. Multi-user role management (admin dashboard to manage staff)
4. Audit trail export to CSV/PDF for compliance
5. Photo / image symptom input via camera API
6. Integration with OpenMRS or DHIS2 for national health data sync
7. WCAG AAA compliance audit + independent accessibility review
8. Clinical validation by licenced medical professionals

---

## 📜 Disclaimer

> HealthBridge AI is a **technology demonstration**. It is NOT a certified medical device and NOT a substitute for professional medical advice, diagnosis, or treatment. Always consult a qualified healthcare provider. In life-threatening emergencies, call your local emergency number (999 / 911 / 112) immediately.

---

*Built for healthcare hackathon 2026.*
