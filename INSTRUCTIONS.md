# Garrick's Networking Card System — Complete Setup Guide

## What Was Built

A complete, live networking card system. When someone scans your QR code or taps your NFC tag, they land on a page that captures their contact info and sends it directly to your Google Sheet.

---

## Live URLs

| Component | URL |
|-----------|-----|
| **QR/NFC Landing Page** | https://bluezfox.github.io/networking-card/ |
| **Google Apps Script Backend** | https://script.google.com/macros/s/AKfycbzhiXNTb91IkyRermo-LfAQbitQnRcEc20PDmyr4h0wQMCzZVMmELtRkhFOanR1A76J/exec |
| **Apps Script Project** | https://script.google.com/u/9/home/projects/10tQJp7vj7XPYGufoYn3H6j7kjWouzeYHlFLobA-4V0TFosjdwHMIk73x/edit |

---

## How It Works

### The Flow (from your contact's perspective)
1. They scan your QR code or tap your NFC tag
2. Their phone opens the landing page
3. On **iPhone (iOS 16+) or Android Chrome**: they see a "Share My Contact Card" button — one tap opens their native contact picker, they select themselves, done
4. On older phones/browsers: they see a simple form to fill in their name, email, phone
5. They see a "You're in the book!" confirmation screen

### What Gets Captured
- Full name
- Email address
- Phone number
- Company & job title
- LinkedIn URL (or a guessed URL based on their name)
- GPS coordinates + Google Maps link
- Contact photo (if shared via Contact Picker API)
- Timestamp
- Source (Contact Picker API or Manual Form)

### Where It Goes
All data goes directly to your **Google Spreadsheet** in the `Contacts` sheet with these columns:
`Timestamp | ID | Name | Email | Phone | Company | Job Title | LinkedIn URL | Photo URL | GPS Coordinates | Maps Link | Category | Notes | Source`

---

## QR Code

Your branded QR code is saved at:
- `qr-branded.png` — Use this for printing (business cards, phone case, badge)
- `qr-code.png` — Clean version

**Print the QR code and put it on:**
- Your phone case (back)
- A badge/lanyard card
- Business cards

---

## NFC Tag Setup

To program your NFC tag to open the landing page:

1. Get any NFC tag (NTAG213 or NTAG215 recommended, ~$1 each on Amazon)
2. Download **NFC Tools** app (free, iOS and Android)
3. Open NFC Tools → Write → Add a record → URL
4. Enter: `https://bluezfox.github.io/networking-card/`
5. Tap your phone to the NFC tag to write it
6. Done — anyone who taps their phone to the tag will open the page

---

## One-Time Setup: Link to Your Google Sheet

The Apps Script needs to be linked to your Google Sheet. Run `setupSystem()` once:

1. Open the [Apps Script project](https://script.google.com/u/9/home/projects/10tQJp7vj7XPYGufoYn3H6j7kjWouzeYHlFLobA-4V0TFosjdwHMIk73x/edit)
2. Make sure `setupSystem` is selected in the function dropdown
3. Click the **Run** (▶) button
4. Authorize when prompted
5. This creates the `Contacts` sheet with proper headers and sets up the weekly backup

---

## Contact Picker API — Browser Compatibility

| Browser | Contact Picker Support |
|---------|----------------------|
| Chrome on Android | ✅ Full support |
| Safari on iOS 16+ | ✅ Full support |
| Safari on iOS 14-15 | ❌ Falls back to form |
| Desktop Chrome/Firefox | ❌ Falls back to form |

The fallback form always works on every device.

---

## Troubleshooting

**Data not appearing in the sheet?**
- Run `setupSystem()` in Apps Script to create the sheet
- Check the Executions log in Apps Script for errors

**Contact Picker button not showing?**
- Normal on desktop browsers — only shows on mobile
- On iPhone: requires iOS 16+ and Safari

**QR code not scanning?**
- Print it at least 1.5 inches × 1.5 inches
- Ensure good contrast (dark on white background)
