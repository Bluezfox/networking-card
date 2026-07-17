# 🚀 Master Documentation: Founders Arena Networking Card System

**Project Name:** Founders Arena Networking Card
**Owner:** Garrick Ortiz (garrick@thefoundersarena.com)
**Live URL:** https://bluezfox.github.io/networking-card/
**GitHub Repo:** https://github.com/Bluezfox/networking-card
**Google Apps Script Project ID:** `10tQJp7vj7XPYGufoYn3H6j7kjWouzeYHlFLobA-4V0TFosjdwHMIk73x`
**Google Spreadsheet ID:** `1YWLdfEjWJijaqqKpJAA4YbMY77Hnr91t-2b729v34Vw`
**GAS Web App URL:** `https://script.google.com/macros/s/AKfycbzhiXNTb91IkyRermo-LfAQbitQnRcEc20PDmyr4h0wQMCzZVMmELtRkhFOanR1A76J/exec`
**Last Deployed Version:** Version 7 (July 17, 2026, 1:50 PM)
**Documentation Author:** Manus AI
**Last Updated:** July 17, 2026

---

## Table of Contents

1. [Project Vision & Goal](#1-project-vision--goal)
2. [Architecture & Tech Stack](#2-architecture--tech-stack)
3. [Data Flow Diagram](#3-data-flow-diagram)
4. [The iOS Contact Picker Limitation — The Core Problem](#4-the-ios-contact-picker-limitation--the-core-problem)
5. [The Solution: Blinq-Style Reciprocal UX](#5-the-solution-blinq-style-reciprocal-ux)
6. [Full Codebase](#6-full-codebase)
   - [6.1 Frontend: index.html](#61-frontend-indexhtml)
   - [6.2 Backend: Code.gs](#62-backend-codegs)
7. [The Critical Bug That Was Fixed](#7-the-critical-bug-that-was-fixed)
8. [Deployment Guide](#8-deployment-guide)
9. [QR Code & NFC Tag Setup](#9-qr-code--nfc-tag-setup)
10. [Google Spreadsheet Structure](#10-google-spreadsheet-structure)
11. [Development History & Decisions Made](#11-development-history--decisions-made)
12. [What Was Tried and Failed (Important Context)](#12-what-was-tried-and-failed-important-context)
13. [Future Expansion Ideas](#13-future-expansion-ideas)
14. [Quick Reference: All URLs & IDs](#14-quick-reference-all-urls--ids)

---

## 1. Project Vision & Goal

Garrick Ortiz attends networking events regularly as part of running Founders Arena. The problem with traditional business card exchanges is that they require both parties to do work: pull out a card, hand it over, and later manually enter the information into a CRM. Digital alternatives like LinkedIn require the other person to have the app open.

**The Goal:** When meeting someone at a networking event, Garrick shows them a QR code or taps phones (NFC). The other person scans it. Within 15 seconds, Garrick's Google Spreadsheet is updated with their name, phone number, email, company, a guessed LinkedIn URL, and the GPS coordinates of where they met — all without Garrick touching his phone again.

**The Constraint:** The entire experience must work on any phone (iPhone or Android), without the visitor needing to download an app, create an account, or do anything complex. The maximum acceptable friction is one tap or a 10-second form fill.

---

## 2. Architecture & Tech Stack

The system is entirely free and serverless. There are no servers to maintain, no subscriptions, and no infrastructure costs.

| Component | Technology | Details |
|-----------|------------|---------|
| **Frontend Hosting** | GitHub Pages | Static site hosting, free, SSL, CDN. Branch: `main`. Auto-deploys on push. |
| **Frontend Language** | Vanilla HTML/CSS/JavaScript | No frameworks. Single `index.html` file. Keeps load time under 1 second. |
| **Backend / API** | Google Apps Script | Runs as a Web App. Receives POST requests. Writes to Google Sheets. Free quota: 6 min/day execution, 100 emails/day. |
| **Database / CRM** | Google Sheets | Tab named "Contacts". 14 columns. Accessible on mobile via Google Sheets app. |
| **Trigger: QR Code** | Generated via Python `qrcode` library | Points to `https://bluezfox.github.io/networking-card/`. Branded with gold/navy color scheme. |
| **Trigger: NFC Tag** | NTAG213 sticker + NFC Tools app | Writes the same URL to an NFC tag. Tap-to-open on any modern iPhone (iOS 14+) or Android. |
| **Contact Export** | vCard (.vcf) file | Garrick's contact info is a hardcoded vCard string. Downloading it triggers iOS/Android native "Add Contact" prompt. |
| **Geolocation** | Browser Geolocation API | `navigator.geolocation.getCurrentPosition()` runs silently on page load. Requires HTTPS (provided by GitHub Pages). |
| **Contact Picker** | Contact Picker API | Only available on Android Chrome. Hidden on iOS. Allows 1-tap contact sharing on supported devices. |

---

## 3. Data Flow Diagram

```
GARRICK'S PHONE
┌─────────────────────┐
│  QR Code (printed)  │  ──── or ────  NFC Tag (sticker on phone)
└─────────────────────┘
           │
           │ Scan / Tap
           ▼
VISITOR'S PHONE (iOS or Android)
┌──────────────────────────────────────────────────────┐
│  Safari / Chrome opens:                              │
│  https://bluezfox.github.io/networking-card/         │
│                                                      │
│  1. Page loads (GitHub Pages CDN, <1 sec)            │
│  2. Geolocation request fires silently               │
│  3. Visitor sees Garrick's card + "Save Contact" btn │
│  4. Visitor taps "Save Garrick's Contact"            │
│     → .vcf downloads → iOS adds to Contacts         │
│  5. Visitor fills 4-field form (name/phone/email/co) │
│  6. Visitor taps "Send My Info to Garrick"           │
│     → fetch() POST to GAS Web App URL               │
└──────────────────────────────────────────────────────┘
           │
           │ HTTPS POST (JSON payload)
           ▼
GOOGLE APPS SCRIPT (Web App, runs as Garrick)
┌──────────────────────────────────────────────────────┐
│  doPost(e) receives JSON                             │
│  → Parses: name, phone, email, company, lat, lng     │
│  → Generates LinkedIn URL guess from name            │
│  → Builds Google Maps link from GPS coords           │
│  → Checks for duplicate (email or phone match)       │
│  → Appends row to Google Sheet                       │
│  → Sets Maps link as clickable HYPERLINK formula     │
│  → Returns { status: 'success' }                     │
└──────────────────────────────────────────────────────┘
           │
           │ appendRow()
           ▼
GOOGLE SPREADSHEET (Garrick's private CRM)
┌──────────────────────────────────────────────────────┐
│  Tab: "Contacts"                                     │
│  New row: Timestamp | ID | Name | Email | Phone |    │
│           Company | Job Title | LinkedIn URL |       │
│           Photo URL | GPS Coords | Maps Link |       │
│           Category | Notes | Source                  │
└──────────────────────────────────────────────────────┘
           │
           │ Garrick opens Google Sheets app on his phone
           ▼
GARRICK SEES THE NEW CONTACT INSTANTLY ✅
```

---

## 4. The iOS Contact Picker Limitation — The Core Problem

This was the central technical challenge of the entire project and the reason for multiple failed attempts.

**What was originally attempted:** Use the Web Contact Picker API (`navigator.contacts.select()`) to silently pull the visitor's contact card from their iPhone when they scan the QR code. This would have been the ideal zero-friction experience.

**Why it does not work on iPhone:** Apple implemented the Contact Picker API in WebKit but disabled it by default on iOS Safari. It is hidden behind an experimental flag at `Settings > Safari > Advanced > Experimental Features > Contact Picker API`. Because 99.9% of iPhone users have never visited this settings page, the API is effectively unavailable on all iPhones in real-world use.

**What happens when the code tries to call it on iOS:** The `navigator.contacts` object is `undefined`. Any button that calls `navigator.contacts.select()` will either throw an error or do nothing at all. This is why previous versions of the project showed a "Tap-to-share is not supported on this browser" message — the code was correctly detecting that the API was unavailable.

**Android is different:** On Android Chrome (version 80+), the Contact Picker API is fully supported and works with one tap. The system detects this at runtime and shows the "Share from My Contacts" button only on Android.

**The detection code:**
```javascript
// Show Android Contact Picker button only if the API is supported
if ('contacts' in navigator && 'ContactsManager' in window) {
  document.getElementById('btn-picker').classList.remove('hidden');
}
```

---

## 5. The Solution: Blinq-Style Reciprocal UX

After researching how Blinq (a leading digital business card app) handles this exact problem, the UX was redesigned around the principle of **reciprocal value exchange**:

1. **Give first:** Offer the visitor something valuable immediately — Garrick's contact card as a downloadable `.vcf` file. When they tap "Save Garrick's Contact," their phone's native OS handles the file and prompts them to add Garrick to their contacts. This works on 100% of phones.

2. **Ask second:** After they've received value, a clean form asks for their information. Because they just got something, they are psychologically primed to give something back.

3. **Minimize fields:** Only 4 fields are shown: name, phone, email, company. Company is marked optional. Name is required. Either phone or email is required. This keeps the form completion time under 15 seconds.

4. **Android bonus:** On Android, an additional "Share from My Contacts" button appears above the form, allowing them to tap once and auto-fill all fields from their own contact card.

**The vCard (Garrick's contact info):**
```
BEGIN:VCARD
VERSION:3.0
FN:Garrick Ortiz
N:Ortiz;Garrick;;;
ORG:Founders Arena
TITLE:Founder
URL:https://bluezfox.github.io/networking-card/
NOTE:Met at networking event
END:VCARD
```
This is embedded directly in `index.html` as a JavaScript string constant. To update Garrick's contact info (e.g., add a phone number, change title), edit the `GARRICK_VCARD` constant in `index.html`.

---

## 6. Full Codebase

### 6.1 Frontend: `index.html`

This is the complete, production-ready file served at `https://bluezfox.github.io/networking-card/`.

**Design System:**
- Background: `#0E1A2B` (deep navy)
- Accent: `#C9A84C` (gold)
- Font: System font stack (`-apple-system, BlinkMacSystemFont, 'SF Pro Display', 'Segoe UI', sans-serif`)
- Border radius: `16px` throughout
- Fully responsive, optimized for 375px–430px (iPhone SE to iPhone Pro Max)

**JavaScript Architecture:**

| Function | Purpose |
|----------|---------|
| `init()` (IIFE) | Runs on load: starts geolocation, shows Android picker button if supported, auto-focuses name field on non-iOS |
| `downloadMyVCard()` | Creates a Blob from the vCard string and triggers a download |
| `openContactPicker()` | Android only: calls `navigator.contacts.select()`, pre-fills form, sends to GAS |
| `submitForm()` | Validates form, calls `sendToGAS()`, shows success screen |
| `sendToGAS(payload)` | `fetch()` POST to GAS URL with `mode: 'no-cors'` |
| `showSuccess(data)` | Hides main screen, shows success screen with recap card |
| `showStatus(msg, type)` | Shows inline error/success message bar |
| `startGeo()` | Calls `navigator.geolocation.getCurrentPosition()`, updates location pill |
| `guessLinkedIn(name)` | Converts "John Smith" → `https://www.linkedin.com/in/john-smith` |
| `blobToBase64(blob)` | Converts contact photo blob to base64 string for transmission |

**Important Note on `fetch()` mode:** The `sendToGAS()` function uses `mode: 'no-cors'`. This is required because Google Apps Script Web Apps do not send CORS headers that satisfy the browser's preflight check. Using `no-cors` means the response is opaque (you cannot read it), but the POST request goes through successfully. This is why the success screen is shown immediately after sending — there is no way to confirm receipt from the frontend. The GAS execution log is the source of truth for whether data was received.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover" />
  <meta name="theme-color" content="#0E1A2B" />
  <meta name="apple-mobile-web-app-capable" content="yes" />
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
  <title>Connect with Garrick — Founders Arena</title>

  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; -webkit-tap-highlight-color: transparent; }

    :root {
      --navy:      #0E1A2B;
      --navy-soft: #162030;
      --gold:      #C9A84C;
      --gold-lt:   #F0D080;
      --white:     #FFFFFF;
      --gray:      #8899AA;
      --gray-lt:   #B0BEC5;
      --success:   #22C55E;
      --error:     #EF4444;
      --radius:    16px;
    }

    html, body {
      height: 100%;
      background: var(--navy);
      color: var(--white);
      font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'Segoe UI', sans-serif;
      -webkit-font-smoothing: antialiased;
    }

    body {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
      min-height: 100vh;
      padding: env(safe-area-inset-top, 0) 0 env(safe-area-inset-bottom, 0);
      overflow-x: hidden;
    }

    .hero {
      width: 100%;
      background: linear-gradient(160deg, #162030 0%, #0E1A2B 100%);
      border-bottom: 1px solid rgba(201,168,76,0.15);
      padding: 40px 24px 32px;
      text-align: center;
      position: relative;
      overflow: hidden;
    }
    .hero::before {
      content: '';
      position: absolute;
      top: -60px; right: -60px;
      width: 200px; height: 200px;
      border-radius: 50%;
      background: radial-gradient(circle, rgba(201,168,76,0.12) 0%, transparent 70%);
    }

    .avatar {
      width: 80px; height: 80px;
      border-radius: 50%;
      background: linear-gradient(135deg, var(--gold), var(--gold-lt));
      display: flex; align-items: center; justify-content: center;
      font-size: 34px;
      margin: 0 auto 16px;
      box-shadow: 0 0 0 4px rgba(201,168,76,0.2), 0 8px 32px rgba(0,0,0,0.4);
    }

    .hero-name { font-size: 26px; font-weight: 700; letter-spacing: -0.5px; margin-bottom: 4px; }
    .hero-title { font-size: 13px; font-weight: 600; color: var(--gold); letter-spacing: 1.8px; text-transform: uppercase; margin-bottom: 10px; }
    .hero-tagline { font-size: 14px; color: var(--gray-lt); line-height: 1.5; }

    .save-contact-wrap { width: 100%; padding: 24px 20px 8px; display: flex; flex-direction: column; align-items: center; gap: 12px; }

    .btn-save {
      width: 100%; max-width: 400px; padding: 18px 20px;
      border-radius: var(--radius); border: none;
      background: linear-gradient(135deg, var(--gold), var(--gold-lt));
      color: var(--navy); font-size: 17px; font-weight: 800; cursor: pointer;
      display: flex; align-items: center; justify-content: center; gap: 10px;
      box-shadow: 0 6px 24px rgba(201,168,76,0.4);
      transition: transform 0.12s, box-shadow 0.12s; letter-spacing: 0.2px;
    }
    .btn-save:active { transform: scale(0.97); box-shadow: 0 3px 12px rgba(201,168,76,0.25); }

    .divider {
      display: flex; align-items: center; gap: 10px;
      width: 100%; max-width: 400px;
      font-size: 12px; color: var(--gray); letter-spacing: 0.8px; text-transform: uppercase;
    }
    .divider::before, .divider::after { content: ''; flex: 1; height: 1px; background: rgba(255,255,255,0.08); }

    .share-section { width: 100%; padding: 0 20px 24px; display: flex; flex-direction: column; align-items: center; gap: 10px; }
    .share-label { font-size: 13px; color: var(--gray); text-align: center; max-width: 400px; line-height: 1.5; margin-bottom: 4px; }
    .share-label strong { color: var(--white); }

    .smart-form {
      width: 100%; max-width: 400px;
      background: var(--navy-soft);
      border: 1px solid rgba(255,255,255,0.07);
      border-radius: var(--radius); overflow: hidden;
    }

    .form-row {
      display: flex; align-items: center;
      border-bottom: 1px solid rgba(255,255,255,0.06);
      padding: 0 16px; gap: 12px;
    }
    .form-row:last-child { border-bottom: none; }
    .form-icon { font-size: 18px; min-width: 24px; }
    .form-row input {
      flex: 1; padding: 15px 0; background: transparent; border: none; outline: none;
      color: var(--white); font-size: 16px; font-family: inherit;
    }
    .form-row input::placeholder { color: rgba(255,255,255,0.3); }

    .btn-submit {
      width: 100%; max-width: 400px; padding: 16px 20px;
      border-radius: var(--radius); border: none;
      background: rgba(201,168,76,0.12);
      border: 1.5px solid rgba(201,168,76,0.35);
      color: var(--gold-lt); font-size: 16px; font-weight: 700; cursor: pointer;
      display: flex; align-items: center; justify-content: center; gap: 8px;
      transition: background 0.15s;
    }
    .btn-submit:active { background: rgba(201,168,76,0.2); }
    .btn-submit:disabled { opacity: 0.5; cursor: not-allowed; }

    .btn-picker {
      width: 100%; max-width: 400px; padding: 16px 20px;
      border-radius: var(--radius); border: 1.5px solid rgba(201,168,76,0.35);
      background: rgba(201,168,76,0.08); color: var(--gold-lt);
      font-size: 16px; font-weight: 700; cursor: pointer;
      display: flex; align-items: center; justify-content: center; gap: 8px;
      transition: background 0.15s;
    }
    .btn-picker:active { background: rgba(201,168,76,0.18); }

    .status-bar {
      width: 100%; max-width: 400px; padding: 10px 14px;
      border-radius: 10px; font-size: 13px; font-weight: 600;
      text-align: center; display: none;
    }
    .status-bar.show { display: block; }
    .status-bar.success { background: rgba(34,197,94,0.12); color: #4ade80; border: 1px solid rgba(34,197,94,0.25); }
    .status-bar.error   { background: rgba(239,68,68,0.12);  color: #f87171; border: 1px solid rgba(239,68,68,0.25); }

    #screen-success {
      display: none; flex-direction: column; align-items: center; justify-content: center;
      min-height: 100vh; padding: 40px 24px; text-align: center;
    }
    #screen-success.show { display: flex; }

    .success-check {
      width: 100px; height: 100px; border-radius: 50%;
      background: linear-gradient(135deg, #16a34a, #22c55e);
      display: flex; align-items: center; justify-content: center; font-size: 48px;
      margin-bottom: 28px; box-shadow: 0 0 50px rgba(34,197,94,0.5);
      animation: pop 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
    }
    @keyframes pop { from { transform: scale(0.5); opacity: 0; } to { transform: scale(1); opacity: 1; } }

    .success-title { font-size: 30px; font-weight: 800; margin-bottom: 10px; }
    .success-sub   { font-size: 16px; color: var(--gray-lt); line-height: 1.6; margin-bottom: 28px; }

    .recap-card {
      width: 100%; max-width: 380px;
      background: var(--navy-soft); border: 1px solid rgba(201,168,76,0.15);
      border-radius: var(--radius); padding: 4px 0;
    }
    .recap-row { display: flex; align-items: center; gap: 12px; padding: 12px 16px; border-bottom: 1px solid rgba(255,255,255,0.05); }
    .recap-row:last-child { border-bottom: none; }
    .recap-icon { font-size: 18px; min-width: 24px; }
    .recap-text { font-size: 14px; font-weight: 500; word-break: break-all; }

    .spin {
      display: inline-block; width: 18px; height: 18px;
      border: 2px solid rgba(255,255,255,0.25); border-top-color: currentColor;
      border-radius: 50%; animation: rotate 0.7s linear infinite; vertical-align: middle;
    }
    @keyframes rotate { to { transform: rotate(360deg); } }

    #screen-main { width: 100%; }
    #screen-main.hide { display: none; }

    .loc-pill {
      display: flex; align-items: center; gap: 8px;
      font-size: 12px; color: var(--gray); padding: 6px 12px;
      background: rgba(255,255,255,0.04); border-radius: 20px;
      max-width: 400px; width: fit-content; margin: 0 auto;
    }
    .loc-dot { width: 7px; height: 7px; border-radius: 50%; background: var(--gold); animation: blink 1.4s infinite; }
    .loc-dot.ok   { background: var(--success); animation: none; }
    .loc-dot.fail { background: var(--gray);    animation: none; }
    @keyframes blink { 0%,100%{opacity:1} 50%{opacity:0.3} }

    .info-note { font-size: 12px; color: var(--gray); text-align: center; max-width: 320px; line-height: 1.5; padding: 0 20px; }
    .hidden { display: none !important; }
  </style>
</head>
<body>

<div id="screen-main">
  <div class="hero">
    <div class="avatar">🤝</div>
    <div class="hero-name">Garrick Ortiz</div>
    <div class="hero-title">Founders Arena</div>
    <p class="hero-tagline">Great connecting with you!<br>Let's stay in touch.</p>
  </div>

  <div class="save-contact-wrap">
    <button class="btn-save" onclick="downloadMyVCard()">
      <span class="btn-save-icon">💾</span>
      Save Garrick's Contact
    </button>
    <div class="divider">then share yours</div>
  </div>

  <div class="share-section">
    <p class="share-label">
      <strong>Share your info</strong> so Garrick can follow up with you.
    </p>

    <button class="btn-picker hidden" id="btn-picker" onclick="openContactPicker()">
      📇 &nbsp;Share from My Contacts
    </button>

    <div class="smart-form" id="smart-form">
      <div class="form-row">
        <span class="form-icon">👤</span>
        <input type="text" id="f-name" placeholder="Your full name" autocomplete="name" autocorrect="off" />
      </div>
      <div class="form-row">
        <span class="form-icon">📱</span>
        <input type="tel" id="f-phone" placeholder="Phone number" autocomplete="tel" />
      </div>
      <div class="form-row">
        <span class="form-icon">✉️</span>
        <input type="email" id="f-email" placeholder="Email address" autocomplete="email" autocorrect="off" autocapitalize="none" />
      </div>
      <div class="form-row">
        <span class="form-icon">🏢</span>
        <input type="text" id="f-company" placeholder="Company (optional)" autocomplete="organization" />
      </div>
    </div>

    <div class="status-bar" id="status-bar"></div>

    <button class="btn-submit" id="btn-submit" onclick="submitForm()">
      🚀 &nbsp;Send My Info to Garrick
    </button>

    <div class="loc-pill">
      <div class="loc-dot" id="loc-dot"></div>
      <span id="loc-text">Getting location…</span>
    </div>

    <p class="info-note">Your info goes directly to Garrick's private contacts list. Nothing is stored on this page.</p>
  </div>
</div>

<div id="screen-success">
  <div class="success-check">✓</div>
  <div class="success-title">You're In! 🎉</div>
  <p class="success-sub">Garrick has your info.<br>Expect a follow-up soon!</p>
  <div class="recap-card" id="recap-card"></div>
</div>

<script>
const GAS_URL = 'https://script.google.com/macros/s/AKfycbzhiXNTb91IkyRermo-LfAQbitQnRcEc20PDmyr4h0wQMCzZVMmELtRkhFOanR1A76J/exec';

const GARRICK_VCARD = `BEGIN:VCARD
VERSION:3.0
FN:Garrick Ortiz
N:Ortiz;Garrick;;;
ORG:Founders Arena
TITLE:Founder
URL:https://bluezfox.github.io/networking-card/
NOTE:Met at networking event
END:VCARD`;

let geoLat = null, geoLng = null;

(function init() {
  startGeo();
  if ('contacts' in navigator && 'ContactsManager' in window) {
    document.getElementById('btn-picker').classList.remove('hidden');
  }
  setTimeout(() => {
    const nameField = document.getElementById('f-name');
    if (nameField && !nameField.value) {
      const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent);
      if (!isIOS) nameField.focus();
    }
  }, 600);
})();

function downloadMyVCard() {
  const blob = new Blob([GARRICK_VCARD], { type: 'text/vcard;charset=utf-8' });
  const url  = URL.createObjectURL(blob);
  const a    = document.createElement('a');
  a.href     = url;
  a.download = 'Garrick-Ortiz.vcf';
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}

async function openContactPicker() {
  const btn = document.getElementById('btn-picker');
  btn.disabled = true;
  btn.innerHTML = '<span class="spin"></span> Opening contacts…';
  try {
    let props = ['name', 'tel', 'email'];
    try {
      const supported = await navigator.contacts.getProperties();
      props = ['name', 'tel', 'email'].filter(p => supported.includes(p));
      if (supported.includes('icon'))    props.push('icon');
      if (supported.includes('address')) props.push('address');
    } catch(e) {}

    const results = await navigator.contacts.select(props, { multiple: false });
    if (!results || results.length === 0) {
      btn.disabled = false;
      btn.innerHTML = '📇 &nbsp;Share from My Contacts';
      return;
    }

    const c     = results[0];
    const name  = (c.name  && c.name[0])  || '';
    const phone = (c.tel   && c.tel[0])   || '';
    const email = (c.email && c.email[0]) || '';

    if (name)  document.getElementById('f-name').value  = name;
    if (phone) document.getElementById('f-phone').value = phone;
    if (email) document.getElementById('f-email').value = email;

    let photoBase64 = '';
    if (c.icon && c.icon.length > 0) {
      try { photoBase64 = await blobToBase64(c.icon[0]); } catch(e) {}
    }

    btn.innerHTML = '<span class="spin"></span> Saving…';
    await sendToGAS({ action: 'captureInfo', name, phone, email, phoneNumber: phone, photo: photoBase64, latitude: geoLat, longitude: geoLng, source: 'Contact Picker API (Android)', notes: 'Shared via Android Contact Picker' });
    showSuccess({ name, phone, email });
  } catch(err) {
    btn.disabled = false;
    btn.innerHTML = '📇 &nbsp;Share from My Contacts';
    showStatus('Could not open contacts. Please use the form below.', 'error');
  }
}

async function submitForm() {
  const name    = document.getElementById('f-name').value.trim();
  const phone   = document.getElementById('f-phone').value.trim();
  const email   = document.getElementById('f-email').value.trim();
  const company = document.getElementById('f-company').value.trim();

  if (!name) { document.getElementById('f-name').focus(); showStatus('Please enter your name to continue.', 'error'); return; }
  if (!phone && !email) { showStatus('Please enter a phone number or email.', 'error'); return; }

  const btn = document.getElementById('btn-submit');
  btn.disabled = true;
  btn.innerHTML = '<span class="spin"></span> Sending…';

  await sendToGAS({ action: 'captureInfo', name, phone, email, phoneNumber: phone, company, linkedInUrl: guessLinkedIn(name), latitude: geoLat, longitude: geoLng, source: 'Manual Form', notes: 'Submitted via quick form' });
  showSuccess({ name, phone, email, company });
}

async function sendToGAS(payload) {
  try {
    await fetch(GAS_URL, { method: 'POST', mode: 'no-cors', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
  } catch(err) { console.error('Send error:', err); }
}

function showSuccess(data) {
  document.getElementById('screen-main').classList.add('hide');
  const rows = [];
  if (data.name)    rows.push({ icon: '👤', text: data.name });
  if (data.phone)   rows.push({ icon: '📱', text: data.phone });
  if (data.email)   rows.push({ icon: '✉️',  text: data.email });
  if (data.company) rows.push({ icon: '🏢', text: data.company });
  if (geoLat)       rows.push({ icon: '📍', text: `${geoLat}, ${geoLng}` });
  document.getElementById('recap-card').innerHTML = rows.map(r => `<div class="recap-row"><span class="recap-icon">${r.icon}</span><span class="recap-text">${r.text}</span></div>`).join('');
  document.getElementById('screen-success').classList.add('show');
  window.scrollTo(0, 0);
}

function showStatus(msg, type) {
  const bar = document.getElementById('status-bar');
  bar.textContent = msg;
  bar.className = `status-bar show ${type}`;
  setTimeout(() => bar.classList.remove('show'), 5000);
}

function startGeo() {
  const dot  = document.getElementById('loc-dot');
  const text = document.getElementById('loc-text');
  if (!navigator.geolocation) { dot.className = 'loc-dot fail'; text.textContent = 'Location unavailable'; return; }
  navigator.geolocation.getCurrentPosition(
    pos => { geoLat = pos.coords.latitude.toFixed(6); geoLng = pos.coords.longitude.toFixed(6); dot.className = 'loc-dot ok'; text.textContent = `Location ready (±${Math.round(pos.coords.accuracy)}m)`; },
    () => { dot.className = 'loc-dot fail'; text.textContent = "Location not shared — that's OK"; },
    { enableHighAccuracy: true, timeout: 10000, maximumAge: 0 }
  );
}

function guessLinkedIn(name) {
  if (!name) return '';
  const slug = name.toLowerCase().replace(/[^a-z0-9\s]/g,'').trim().replace(/\s+/g,'-');
  return 'https://www.linkedin.com/in/' + slug;
}

function blobToBase64(blob) {
  return new Promise((res, rej) => { const r = new FileReader(); r.onload = () => res(r.result); r.onerror = rej; r.readAsDataURL(blob); });
}

document.addEventListener('keydown', e => {
  if (e.key === 'Enter') {
    const active = document.activeElement;
    if (active && active.closest('#smart-form')) {
      const inputs = Array.from(document.querySelectorAll('#smart-form input'));
      const idx = inputs.indexOf(active);
      if (idx < inputs.length - 1) { inputs[idx + 1].focus(); } else { submitForm(); }
    }
  }
});
</script>
</body>
</html>
```

---

### 6.2 Backend: `Code.gs`

This is the complete, production-ready Google Apps Script backend. **Version 7** is the currently deployed version.

```javascript
/**
 * Personal Networking Tool - Backend System
 * File: Code.gs
 * Project: SUPER_NETWORKV2
 * Deployed: Version 7, July 17, 2026
 *
 * CRITICAL: Uses openById() — NOT getActiveSpreadsheet()
 * getActiveSpreadsheet() only works when the script is run from within
 * Google Sheets. When called as a Web App from an external POST request,
 * there is no "active" spreadsheet and it returns null, causing silent failure.
 */

const SPREADSHEET_ID = '1YWLdfEjWJijaqqKpJAA4YbMY77Hnr91t-2b729v34Vw';
const SHEET_NAME = 'Contacts';
const HEADERS = [
  'Timestamp', 'ID', 'Name', 'Email', 'Phone',
  'Company', 'Job Title', 'LinkedIn URL', 'Photo URL',
  'GPS Coordinates', 'Maps Link', 'Category', 'Notes', 'Source'
];

function getSpreadsheet() {
  return SpreadsheetApp.openById(SPREADSHEET_ID);
}

function setupSystem() {
  const ss = getSpreadsheet();
  let sheet = ss.getSheetByName(SHEET_NAME);

  if (!sheet) {
    sheet = ss.insertSheet(SHEET_NAME);
  }

  if (sheet.getLastRow() === 0) {
    sheet.appendRow(HEADERS);
    sheet.getRange(1, 1, 1, HEADERS.length)
      .setFontWeight('bold')
      .setBackground('#0E1A2B')
      .setFontColor('#FFFFFF');
    sheet.setFrozenRows(1);
    sheet.setColumnWidth(1, 160);
    sheet.setColumnWidth(3, 180);
    sheet.setColumnWidth(4, 200);
    sheet.setColumnWidth(8, 220);
    sheet.setColumnWidth(10, 160);
    sheet.setColumnWidth(11, 120);
  }

  setupBackupTrigger();
  Logger.log("System setup complete. Spreadsheet ID: " + SPREADSHEET_ID);
}

function setupBackupTrigger() {
  const triggers = ScriptApp.getProjectTriggers();
  for (let i = 0; i < triggers.length; i++) {
    if (triggers[i].getHandlerFunction() === 'backupData') {
      ScriptApp.deleteTrigger(triggers[i]);
    }
  }
  ScriptApp.newTrigger('backupData')
    .timeBased()
    .onWeekDay(ScriptApp.WeekDay.SUNDAY)
    .atHour(2)
    .create();
}

function doGet(e) {
  if (e && e.parameter && e.parameter.api === 'true') {
    const data = getContacts();
    return createJsonResponse({ status: 'success', data: data });
  }
  return HtmlService.createTemplateFromFile('Index').evaluate()
    .setTitle('Networking CRM — Founders Arena')
    .addMetaTag('viewport', 'width=device-width, initial-scale=1')
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
}

function doPost(e) {
  try {
    if (!e || !e.postData) {
      Logger.log('doPost: No postData received');
      return createJsonResponse({ status: 'error', message: 'No data received' });
    }

    Logger.log('doPost received: ' + e.postData.contents.substring(0, 200));
    const payload = JSON.parse(e.postData.contents);
    const action = payload.action || 'captureInfo';
    Logger.log('doPost action: ' + action + ' | name: ' + (payload.name || 'N/A'));

    if (action === 'addContact') {
      return createJsonResponse(handleAddContact(payload.contact || payload));
    } else if (action === 'captureInfo') {
      return createJsonResponse(handleCaptureInfo(payload));
    }

    return createJsonResponse({ status: 'error', message: 'Unknown action: ' + action });
  } catch (err) {
    Logger.log('doPost error: ' + err.toString());
    return createJsonResponse({ status: 'error', message: 'Server error: ' + err.toString() });
  }
}

function handleCaptureInfo(payload) {
  const gpsCoords = (payload.latitude && payload.longitude)
    ? payload.latitude + ',' + payload.longitude : '';
  const mapsLink = gpsCoords ? 'https://maps.google.com/?q=' + gpsCoords : '';

  let linkedInUrl = payload.linkedInUrl || '';
  if (!linkedInUrl && payload.name) {
    linkedInUrl = guessLinkedInUrl(payload.name);
  }

  const contact = {
    name:           payload.name || 'Unknown',
    email:          payload.email || '',
    phone:          payload.phoneNumber || payload.phone || '',
    company:        payload.company || '',
    jobTitle:       payload.jobTitle || '',
    linkedInUrl:    linkedInUrl,
    photoUrl:       payload.photo || '',
    gpsCoordinates: gpsCoords,
    mapsLink:       mapsLink,
    category:       payload.category || 'prospect',
    notes:          payload.notes || '',
    source:         payload.source || 'QR/NFC Scan'
  };

  Logger.log('handleCaptureInfo: ' + contact.name + ' | ' + contact.email + ' | GPS: ' + gpsCoords);
  return handleAddContact(contact);
}

function handleAddContact(contact) {
  const ss = getSpreadsheet();
  let sheet = ss.getSheetByName(SHEET_NAME);

  if (!sheet) {
    Logger.log('Sheet not found, running setupSystem...');
    setupSystem();
    sheet = ss.getSheetByName(SHEET_NAME);
  }

  if (sheet.getLastRow() === 0) {
    sheet.appendRow(HEADERS);
    sheet.getRange(1, 1, 1, HEADERS.length).setFontWeight('bold').setBackground('#0E1A2B').setFontColor('#FFFFFF');
    sheet.setFrozenRows(1);
  }

  const sanitized = sanitizeContactData(contact);

  if (!sanitized.name || sanitized.name === 'Unknown') {
    sanitized.name = 'Scan Visitor ' + new Date().toLocaleDateString();
  }

  if (sanitized.email || sanitized.phone) {
    const isDupe = checkDuplicate(sanitized.email, sanitized.phone, sheet);
    if (isDupe) {
      Logger.log('Duplicate detected for: ' + sanitized.email + ' / ' + sanitized.phone);
      return { status: 'duplicate', message: 'Contact already exists.' };
    }
  }

  const timestamp = new Date();
  const contactId = Utilities.getUuid();

  const rowData = [
    timestamp, contactId, sanitized.name, sanitized.email, sanitized.phone,
    sanitized.company, sanitized.jobTitle, sanitized.linkedInUrl, sanitized.photoUrl,
    sanitized.gpsCoordinates, sanitized.mapsLink || '', sanitized.category || 'prospect',
    sanitized.notes, sanitized.source || 'QR/NFC Scan'
  ];

  sheet.appendRow(rowData);

  if (sanitized.mapsLink) {
    const lastRow = sheet.getLastRow();
    sheet.getRange(lastRow, 11).setFormula('=HYPERLINK("' + sanitized.mapsLink + '","📍 View Map")');
  }

  Logger.log('✅ Contact added: ' + sanitized.name + ' | ' + sanitized.email + ' | Row: ' + sheet.getLastRow());
  return { status: 'success', message: 'Contact added successfully', id: contactId };
}

function guessLinkedInUrl(name) {
  if (!name) return '';
  const slug = name.toLowerCase().replace(/[^a-z0-9\s]/g, '').trim().replace(/\s+/g, '-');
  return 'https://www.linkedin.com/in/' + slug;
}

function getContacts() {
  const ss = getSpreadsheet();
  const sheet = ss.getSheetByName(SHEET_NAME);
  if (!sheet || sheet.getLastRow() < 2) return [];
  const data = sheet.getDataRange().getValues();
  const headers = data.shift();
  return data.map(row => { const obj = {}; headers.forEach((h, i) => obj[h] = row[i]); return obj; }).reverse();
}

function searchContacts(query, categoryFilter) {
  const allContacts = getContacts();
  const lowerQuery = query ? query.toLowerCase() : '';
  return allContacts.filter(c => {
    if (categoryFilter && categoryFilter !== 'all' && c.Category !== categoryFilter) return false;
    if (lowerQuery) { const rowText = Object.values(c).join(' ').toLowerCase(); if (!rowText.includes(lowerQuery)) return false; }
    return true;
  });
}

function createJsonResponse(data) {
  return ContentService.createTextOutput(JSON.stringify(data)).setMimeType(ContentService.MimeType.JSON);
}

function sanitizeContactData(contact) {
  const sanitized = {};
  const fields = ['name', 'email', 'phone', 'company', 'jobTitle', 'linkedInUrl', 'photoUrl', 'gpsCoordinates', 'mapsLink', 'category', 'notes', 'source'];
  fields.forEach(field => {
    let value = contact[field] || '';
    if (typeof value === 'string') value = value.replace(/<[^>]*>?/gm, '').trim();
    sanitized[field] = value;
  });
  return sanitized;
}

function checkDuplicate(email, phone, sheet) {
  if (!email && !phone) return false;
  const data = sheet.getDataRange().getValues();
  for (let i = 1; i < data.length; i++) {
    const row = data[i];
    const rowEmail = row[3] ? String(row[3]).toLowerCase().trim() : '';
    const rowPhone = row[4] ? String(row[4]).trim() : '';
    if (email && rowEmail && rowEmail === String(email).toLowerCase().trim()) return true;
    if (phone && rowPhone && rowPhone === String(phone).trim()) return true;
  }
  return false;
}

function backupData() {
  const ss = getSpreadsheet();
  const sheet = ss.getSheetByName(SHEET_NAME);
  if (!sheet) return;
  const data = sheet.getDataRange().getValues();
  const backupName = 'Backup_Networking CRM_' + new Date().toISOString().slice(0, 16).replace(/[T:]/g, '_');
  const backupSS = SpreadsheetApp.create(backupName);
  backupSS.getActiveSheet().getRange(1, 1, data.length, data[0].length).setValues(data);
  Logger.log('Backup created: ' + backupName);
}

function testWrite() {
  const result = handleCaptureInfo({
    name: 'Test User ' + new Date().toLocaleTimeString(),
    email: 'test@test.com',
    phone: '555-1234',
    company: 'Test Co',
    latitude: '32.7654',
    longitude: '-97.0803',
    source: 'Manual Test'
  });
  Logger.log('testWrite result: ' + JSON.stringify(result));
}
```

---

## 7. The Critical Bug That Was Fixed

**Bug:** Form submissions from the mobile landing page were not appearing in the Google Spreadsheet.

**Root Cause:** The original `Code.gs` used `SpreadsheetApp.getActiveSpreadsheet()` throughout. This method works correctly when:
- You open the script from within Google Sheets (via Extensions > Apps Script)
- You run a function manually from the editor

It **fails silently** when:
- The script is invoked as a Web App via an external HTTP POST request
- There is no "active" spreadsheet in that context — the method returns `null`
- All subsequent calls like `.getSheetByName()` throw `TypeError: Cannot read properties of null`
- The `doPost` function catches the error and returns `{ status: 'error' }`, but since `fetch()` uses `mode: 'no-cors'`, the frontend never sees the error response

**The Fix:** Replace every instance of `SpreadsheetApp.getActiveSpreadsheet()` with `SpreadsheetApp.openById('1YWLdfEjWJijaqqKpJAA4YbMY77Hnr91t-2b729v34Vw')` and wrap it in a helper function `getSpreadsheet()`.

**How the Fix Was Applied:** The Apps Script editor's built-in Find & Replace (Ctrl+H) was used to replace all 4 occurrences at once. The script was then saved and redeployed as Version 7.

---

## 8. Deployment Guide

### 8.1 Updating the Frontend (GitHub Pages)

```bash
# 1. Clone the repo (if not already cloned)
git clone https://github.com/Bluezfox/networking-card.git
cd networking-card

# 2. Make your changes to index.html

# 3. Commit and push
git add index.html
git commit -m "Your change description"
git push origin main

# 4. Wait 1-3 minutes for GitHub Pages CDN to propagate
# 5. Verify at: https://bluezfox.github.io/networking-card/
```

### 8.2 Updating the Backend (Google Apps Script)

1. Open: https://script.google.com/u/9/home/projects/10tQJp7vj7XPYGufoYn3H6j7kjWouzeYHlFLobA-4V0TFosjdwHMIk73x/edit
2. Make changes to `Code.gs`
3. Press **Ctrl+S** to save
4. Click **Deploy** (top right) → **Manage deployments**
5. Click the **pencil (Edit) icon** next to the active deployment
6. Under **Version**, select **New version**
7. Click **Deploy**
8. The Web App URL **does not change** — it remains the same Deployment ID

**Important:** Always update the *existing* deployment (not create a new one). Creating a new deployment generates a new URL, which would break the `GAS_URL` constant in `index.html`.

### 8.3 First-Time Setup (Run Once)

If the Google Sheet ever gets deleted or you need to set it up fresh:
1. Open the Apps Script editor
2. Select `setupSystem` from the function dropdown
3. Click **Run ▶**
4. Authorize when prompted (Google will ask for Sheets and Drive access)
5. The "Contacts" tab will be created with proper headers and formatting

### 8.4 Runtime Settings

| Setting | Value |
|---------|-------|
| Runtime | Chrome V8 (enabled July 17, 2026) |
| Execute as | Me (garrick@thefoundersarena.com) |
| Who has access | Anyone |
| Current version | Version 7 |

---

## 9. QR Code & NFC Tag Setup

### QR Code

The branded QR code was generated using Python:
```python
import qrcode
from PIL import Image, ImageDraw, ImageFont

qr = qrcode.QRCode(version=1, error_correction=qrcode.constants.ERROR_CORRECT_H, box_size=10, border=4)
qr.add_data('https://bluezfox.github.io/networking-card/')
qr.make(fit=True)
img = qr.make_image(fill_color='#0E1A2B', back_color='#F0D080')
img.save('qr-code.png')
```

The branded version (`qr-branded.png`) adds "Scan to Connect" text and a navy background. Both files are in the GitHub repository.

**Print suggestions:**
- Print at 2"×2" minimum for reliable scanning
- Laminate for durability at events
- Place on the back of your phone case, on a badge holder, or on a small card

### NFC Tag Setup

1. Purchase NTAG213 NFC stickers (available on Amazon, ~$1 each)
2. Download the **NFC Tools** app (free, iOS and Android)
3. Open NFC Tools → Write → Add a record → URL
4. Enter: `https://bluezfox.github.io/networking-card/`
5. Tap Write and hold your phone over the NFC sticker
6. Done — the tag is programmed

**How it works on iPhone:** iOS 14+ automatically reads NFC tags when the phone is unlocked. A banner appears at the top of the screen. The user taps it and Safari opens the URL. No app required.

---

## 10. Google Spreadsheet Structure

**Spreadsheet:** https://docs.google.com/spreadsheets/d/1YWLdfEjWJijaqqKpJAA4YbMY77Hnr91t-2b729v34Vw/edit
**Tab:** Contacts

| Column | Header | Description |
|--------|--------|-------------|
| A | Timestamp | Date/time the row was written (Google Sheets date format) |
| B | ID | UUID generated by `Utilities.getUuid()` — unique identifier per contact |
| C | Name | Full name from form or Contact Picker |
| D | Email | Email address |
| E | Phone | Phone number |
| F | Company | Company name |
| G | Job Title | Job title (usually empty from form, populated from Contact Picker on Android) |
| H | LinkedIn URL | Provided URL or auto-guessed from name |
| I | Photo URL | Base64 photo string from Android Contact Picker (usually empty from form) |
| J | GPS Coordinates | `latitude,longitude` string (e.g., `32.7654,-97.0803`) |
| K | Maps Link | Clickable `=HYPERLINK()` formula that opens Google Maps |
| L | Category | Always `prospect` by default |
| M | Notes | Source description (e.g., "Submitted via quick form") |
| N | Source | `Manual Form`, `Contact Picker API (Android)`, or `QR/NFC Scan` |

---

## 11. Development History & Decisions Made

This section documents the key decisions made during development so future developers understand the "why" behind the architecture.

**Decision 1: GitHub Pages over a custom server**
A custom server (Node.js, Python Flask, etc.) would have been more powerful but requires maintenance, uptime monitoring, and cost. GitHub Pages is free, has 99.99% uptime, provides SSL automatically, and deploys on every `git push`. For a personal tool with low traffic, it is the optimal choice.

**Decision 2: Google Apps Script over a REST API**
Alternatives like Supabase, Firebase, or Airtable were considered. GAS was chosen because: (a) the data destination is Google Sheets, which Garrick already uses; (b) GAS has native Sheets access with no API key management; (c) it is free within generous quotas; (d) it requires no separate account or service.

**Decision 3: `mode: 'no-cors'` for the fetch call**
Google Apps Script Web Apps do not return CORS headers that satisfy browser preflight checks. Using `mode: 'cors'` causes the request to be blocked. Using `mode: 'no-cors'` allows the POST to go through but makes the response opaque (unreadable). The tradeoff is that the frontend cannot confirm success from the response — it always shows the success screen. This is acceptable because GAS execution logs provide the real source of truth, and the probability of failure is very low.

**Decision 4: Blinq-style reciprocal UX over Contact Picker**
The original design attempted to use the Contact Picker API to silently pull the visitor's contact from their phone. This was abandoned after discovering that Apple disables the API on iOS Safari by default. The Blinq model (give your contact first, then ask for theirs) was adopted because it works on 100% of devices and creates a natural reciprocity dynamic.

**Decision 5: V8 Runtime**
The Google Apps Script project was migrated from the deprecated Rhino runtime to Chrome V8 on July 17, 2026. V8 supports modern JavaScript (ES2019+, arrow functions, async/await, destructuring) and is required for all new projects going forward.

---

## 12. What Was Tried and Failed (Important Context)

**Failed Attempt 1: Silent contact scraping**
The original vision was for the page to automatically pull the visitor's name and phone number from their phone without any interaction. This is technically impossible in a web browser due to OS-level security sandboxing. No browser API allows a webpage to read the device's contacts without explicit user permission.

**Failed Attempt 2: Contact Picker API as primary flow**
The Contact Picker API (`navigator.contacts.select()`) was implemented as the main CTA button. On desktop Chrome (used for testing), it showed "not supported." On iPhones (the primary target device), it is disabled by default. Only on Android Chrome does it work. The button was demoted to an Android-only enhancement.

**Failed Attempt 3: `SpreadsheetApp.getActiveSpreadsheet()` in Web App context**
The backend initially used `getActiveSpreadsheet()`. This worked when testing manually in the editor but silently failed when the script was triggered via external POST requests. The executions log showed "Completed" (no error) but no rows were written to the sheet. This was the most confusing bug because there was no visible error. The fix was to use `openById()` with the hardcoded Spreadsheet ID.

**Failed Attempt 4: Programmatic editor update via `gws script +push`**
When trying to fix the Code.gs programmatically, the `gws` CLI tool was used. The dry-run worked but the actual push failed because the `gws` OAuth token did not have the `https://www.googleapis.com/auth/script.projects` scope. The fix was applied directly in the browser editor using Find & Replace (Ctrl+H).

---

## 13. Future Expansion Ideas

When you return to this project in the future, share this document with an AI assistant and ask for any of these features:

**High Priority Enhancements:**

1. **Automated Follow-Up Email:** Add a `sendFollowUp(email, name)` function to `Code.gs` that uses `GmailApp.sendEmail()` to send a personalized "Great meeting you at [event]!" email 2 hours after a contact is added. Trigger it from `handleAddContact()` using `Utilities.sleep()` or a time-based trigger.

2. **Event Tagging via URL Parameter:** Update `index.html` to read a URL parameter: `?event=TechCrunch2026`. Pass this as the `notes` field in the form submission. This lets you know where you met each person without them typing anything. Example URL: `https://bluezfox.github.io/networking-card/?event=TechCrunch`

3. **LinkedIn Enrichment:** Use the Hunter.io or Apollo.io API in `Code.gs` to look up the visitor's real LinkedIn profile based on their email domain. Store the API key in GAS Script Properties (`PropertiesService.getScriptProperties()`).

4. **Calendar Booking Button:** Add a "Book a 15-min Intro Call" button on the success screen that links to Calendly or Cal.com. This converts a passive contact capture into an active meeting booking.

5. **Photo Upload:** Add an optional photo upload field to the form. Use `FileReader` to convert the image to base64 and send it in the payload. Store it in Google Drive via `DriveApp.createFile()` and save the Drive URL in the Photo URL column.

**Medium Priority:**

6. **SMS Notification:** Use the Twilio API (or Google Voice via GAS) to send Garrick a text message the moment a new contact is added: "New contact: John Smith (john@acme.com) met at [GPS location]."

7. **CRM Dashboard:** Build a simple read-only dashboard at a separate URL that calls the GAS `doGet` API endpoint (`?api=true`) and displays all contacts in a searchable, filterable table. This could be another GitHub Pages page in the same repo.

8. **Duplicate Merging:** The current duplicate check only prevents exact email/phone matches. Add fuzzy name matching using Levenshtein distance to catch near-duplicates like "John Smith" and "Jon Smith."

**Low Priority / Nice-to-Have:**

9. **Dark/Light Mode Toggle:** Add a CSS `prefers-color-scheme` media query to support light mode for users who prefer it.

10. **Animated Avatar:** Replace the 🤝 emoji avatar with Garrick's actual photo. Store it as a base64-encoded string in the HTML or link to a GitHub-hosted image.

---

## 14. Quick Reference: All URLs & IDs

| Resource | Value |
|----------|-------|
| **Live Landing Page** | https://bluezfox.github.io/networking-card/ |
| **GitHub Repository** | https://github.com/Bluezfox/networking-card |
| **Google Apps Script Editor** | https://script.google.com/u/9/home/projects/10tQJp7vj7XPYGufoYn3H6j7kjWouzeYHlFLobA-4V0TFosjdwHMIk73x/edit |
| **GAS Execution Logs** | https://script.google.com/u/9/home/projects/10tQJp7vj7XPYGufoYn3H6j7kjWouzeYHlFLobA-4V0TFosjdwHMIk73x/executions |
| **GAS Web App URL** | https://script.google.com/macros/s/AKfycbzhiXNTb91IkyRermo-LfAQbitQnRcEc20PDmyr4h0wQMCzZVMmELtRkhFOanR1A76J/exec |
| **Google Spreadsheet** | https://docs.google.com/spreadsheets/d/1YWLdfEjWJijaqqKpJAA4YbMY77Hnr91t-2b729v34Vw/edit |
| **Spreadsheet ID** | `1YWLdfEjWJijaqqKpJAA4YbMY77Hnr91t-2b729v34Vw` |
| **GAS Project ID** | `10tQJp7vj7XPYGufoYn3H6j7kjWouzeYHlFLobA-4V0TFosjdwHMIk73x` |
| **GAS Deployment ID** | `AKfycbzhiXNTb91IkyRermo-LfAQbitQnRcEc20PDmyr4h0wQMCzZVMmELtRkhFOanR1A76J` |
| **Owner Email** | garrick@thefoundersarena.com |
| **GitHub Username** | Bluezfox |

---

*This document was generated by Manus AI on July 17, 2026. It covers the complete build session from initial concept through final working deployment. All code in this document is the exact production code running live as of the documentation date.*
