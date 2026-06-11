# 📦 Warehouse Routing Intelligence

A single-file, zero-dependency web app that predicts which warehouse a parcel belongs to — from a barcode scan, a tracking code, or a camera photo of a shipping label. Runs entirely in the browser; no backend, no installation.

---

## ✨ Features

### Prediction engine
- **Exact code lookup** — instant match against a trained dataset of known tracking IDs
- **Prefix / pattern matching** — rule-based engine that reads code structure (e.g. `BE…KESS` → Kess Berlin → POZ2)
- **Merchant keyword extraction** — scans the tracking code itself for embedded merchant tokens (e.g. `KESS`, `JACKS`) and maps them to the correct warehouse
- **Merchant hint lookup** — type or scan a merchant name for direct routing
- **Base-rate fallback** — when no signal is found, uses historical warehouse distribution as a prior
- All layers are combined with confidence scoring; the highest-confidence signal wins

### Input methods
| Method | Desktop | Mobile |
|--------|---------|--------|
| USB / Bluetooth HID barcode scanner | ✅ | — |
| Manual keyboard entry | ✅ | ✅ |
| Camera barcode scan (BarcodeDetector API) | ✅ | ✅ |
| Camera OCR label scan (Tesseract.js) | ✅ | ✅ |

### Multi-device sync
- Create a **session** on the desktop to generate a 6-character code
- Scan the **QR code** shown in the sidebar to connect a phone instantly
- Both devices share the same prediction results in real time via **PeerJS** (WebRTC)

### Dataset upload *(Settings panel)*
Upload two CSV files to replace the built-in dataset at runtime — no page reload needed:

| File | Required columns |
|------|-----------------|
| Merchant lookup | `Merchant, Warehouse` |
| Tracking ID dataset | `TrackingID, Merchant, Warehouse` |

Click **🔄 Train new model** after uploading to rebuild the prediction engine with your data.

### Dashboard
- Live **warehouse split** bars (BER3 / POZ1 / POZ2 / other)
- **Merchant breakdown** table — predicted merchant name, count, and percentage
- **Recent scans** log with warehouse badge, confidence, and timestamp
- Session stats counter strip (mobile-friendly)

### UX
- 🌙 / ☀️ Dark / light theme toggle
- ⚙️ Settings panel (desktop only) for dataset management
- Fully responsive — switches to a focused mobile scan view automatically
- **QWERTZ → QWERTY remapping** for HID scanners on German keyboard layouts — scans `(Y)#13796` correctly even when the OS layout produces `)Z=§13796`
- Strips leading `(Y)` prefix automatically before prediction

---

## 🚀 Getting started

No build step. Just open the file.

```bash
# Clone and open
git clone https://github.com/YOUR_USERNAME/warehouse-routing-intelligence.git
open warehouse_predictor_app.html
```

Or host it anywhere static — GitHub Pages, Netlify, a local file server.

### With your own data

1. Open the app and click ⚙️ (bottom-left of the sidebar)
2. Upload your **Merchant lookup** CSV (`Merchant, Warehouse`)
3. Upload your **Tracking ID dataset** CSV (`TrackingID, Merchant, Warehouse`)
4. Click **🔄 Train new model**
5. Start scanning

Both comma (`,`) and semicolon (`;`) delimiters are supported. Column headers are case-insensitive.

---

## 📁 File structure

```
warehouse_predictor_app.html   ← entire app (HTML + CSS + JS, single file)
README.md
```

---

## 🧠 Prediction logic

```
Input code
    │
    ├─ Exact match in EXACT{}?           → warehouse (100% confidence)
    │
    ├─ Prefix / pattern rule match?      → warehouse + confidence score
    │
    ├─ Merchant hint provided?
    │     └─ MERCHANT_PURE / MIXED lookup → warehouse + confidence
    │
    ├─ Merchant token found in code?
    │     └─ Substring scan of code body  → warehouse + confidence (85%)
    │
    └─ None of the above?                → base-rate warehouse (BER3 ~73%)

Multiple signals → highest confidence wins
Ties → merchant signal preferred over base rate
```

---

## 📦 Dependencies (CDN, no install)

| Library | Purpose |
|---------|---------|
| [PeerJS](https://peerjs.com) | WebRTC multi-device sync |
| [QRCode.js](https://github.com/davidshimjs/qrcodejs) | QR code generation for session joining |
| [Tesseract.js](https://tesseract.projectnaptha.com) | OCR for camera label scanning |
| [Syne + JetBrains Mono](https://fonts.google.com) | Typography |

All loaded from CDN. The app works offline after first load (fonts and Tesseract model cached by the browser).

---

## 🔒 Privacy

All processing happens **in your browser**. No tracking codes, merchant names, or session data are sent to any server. PeerJS uses a public signalling server only to establish the WebRTC connection — the scan data itself flows peer-to-peer.

---

## 🛠 Development notes

- Single HTML file — edit CSS in `<style>`, logic in `<script>`
- Prediction data lives in `const EXACT`, `const MERCHANT_PURE`, `const MERCHANT_MIXED` near the top of the script — replace with your own or load via CSV upload
- To add a new warehouse, add entries to those objects and update the warehouse stat tiles in the HTML

---

## 📄 License

MIT — use freely, attribution appreciated.
