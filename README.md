# ODO Logger v1.10.0

A single-file HTML app that reads odometer values from dashboard photos using AI vision, extracts dates from EXIF metadata, and exports the results to a formatted XLSX spreadsheet with embedded images.

No installation, no server, no dependencies — just open `index.html` in a browser.

---

## How to Use

### 1. Get the App

Download `index.html` and open it directly in your browser (double-click, or drag it into a browser tab). No internet connection is required to run the app itself — only AI API calls need the network.

### 2. Choose an AI Provider and Enter Your API Key

On first launch, a settings modal appears. Pick one of the four supported providers:

| Provider | Cost | Notes |
|----------|------|-------|
| **Claude** (Anthropic) | Paid | Best accuracy; requires `sk-ant-...` key |
| **OpenAI** | Paid | Requires `sk-proj-...` key |
| **Gemini** (Google) | Free tier | 1,500 req/day on `gemini-2.0-flash`; must run the file **locally** (see CORS note below) |
| **9ARM-AI** | Varies | OpenAI-compatible; configure your own endpoint |

To get a key:
- Claude: https://console.anthropic.com
- OpenAI: https://platform.openai.com/api-keys
- Gemini: https://aistudio.google.com/app/apikey
- 9ARM: https://gateway.9arm.co

Click **Save & Start** to continue. You can change the provider at any time by clicking the model pill in the top-right corner of the header.

### 3. Upload Dashboard Photos

Drag and drop image files onto the drop zone, or click it to browse. Multiple files are supported. Accepted formats: JPG, PNG (any image/* type).

For each photo the app will:
1. Extract the photo date from EXIF metadata (falls back to file modification date).
2. Send the image to the selected AI provider and parse the odometer reading.
3. Add a row to the table, sorted by date.

### 4. Review and Correct Entries

- Click the pencil icon next to any ODO value to edit it manually.
- Click **x** on a row to remove it.
- The stat cards at the top show total distance, first ODO, and last ODO automatically.

### 5. Export to XLSX

Click **Export XLSX with Images** to download a spreadsheet that includes:
- Thumbnail images embedded directly in cells (Excel "Place in Cell" format).
- Date, filename, ODO reading, and distance between consecutive entries.

### 6. Download the App for Local Use

Click **Download App** to save the current page as `ODO_Logger.html`. Your API keys are **not** saved into the downloaded file — the recipient must enter their own key.

---

## CORS Note (Important for Gemini and Cloud Use)

When the app is opened from a cloud-hosted URL (e.g., claude.ai/code), some AI providers reject the request:

- **Gemini** blocks all cross-origin requests from cloud origins entirely.
- **Claude** requires the `anthropic-dangerous-direct-browser-access: true` header, which the app sends automatically.
- **OpenAI** and **9ARM** generally work from any origin.

**Fix:** Download the app and open the `ODO_Logger.html` file locally in your browser. All providers work when the page origin is `file://`.

---

## Security and Safe Usage

### What Leaves Your Browser

| Data | Destination | When |
|------|-------------|------|
| Dashboard image (base64) | AI provider API | On every file upload |
| Odometer prompt text | AI provider API | On every file upload |
| Nothing else | — | All other data stays local |

Your images are sent directly from your browser to the AI provider you chose. They pass through no intermediate server operated by this app.

### API Keys

- **Storage:** API keys are saved in your browser's `localStorage` under the keys `odo_key_claude`, `odo_key_openai`, `odo_key_gemini`, and `odo_key_9arm`.
- **Not exported:** Clicking "Download App" exports the page HTML only — `localStorage` is never included. The downloaded file contains no credentials.
- **Not shared:** Keys are scoped to your browser profile. Other users on different browser profiles cannot read them.
- **Visibility risk:** Anyone with physical or remote access to your browser (e.g., via DevTools → Application → Local Storage) can read stored keys. Keep your browser session private and log out of shared computers.
- **Recommendation:** If you suspect a key has been exposed, revoke and regenerate it at the provider's console.

### What the App Cannot Do

- It has no backend, no analytics, and no telemetry.
- It does not make any network request other than the AI provider API call you explicitly trigger by uploading a photo.
- It does not read any files on your system beyond the images you explicitly select or drop.

### Sharing the App File Safely

The `index.html` or downloaded `ODO_Logger.html` file is safe to share with others:

- It contains no embedded API keys.
- It contains no user data (entries, photos, or ODO readings).
- Each recipient enters their own API key on first launch.

Do not share your browser's `localStorage` export or any screenshot that shows your API key in the settings modal.

### Third-Party AI Provider Privacy

When you upload a photo, the image is sent to the AI provider's API. Each provider has its own data retention and privacy policy:

- Anthropic (Claude): https://www.anthropic.com/privacy
- OpenAI: https://openai.com/policies/privacy-policy
- Google (Gemini): https://policies.google.com/privacy
- 9ARM: refer to their own privacy policy

If your dashboard photos contain sensitive information (personal plates, location data, etc.), review the provider's data usage terms before use. EXIF GPS data embedded in images is stripped during base64 encoding only if the provider ignores it — the raw image bytes are sent as-is.

### 9ARM Custom Endpoint

If you use the 9ARM provider with a custom endpoint URL, ensure you trust the operator of that endpoint. The full image and prompt are sent to the configured URL as a standard OpenAI-compatible request.

---

## Architecture Summary

| Concern | Approach |
|---------|----------|
| No build step | Single `index.html`; open directly |
| State | In-memory `entries` array; nothing persisted to disk |
| API keys | `localStorage` only |
| EXIF parsing | Custom JPEG byte parser; no library |
| ZIP/XLSX generation | Built from scratch in-memory; no library |
| Image resizing | `<canvas>` step-downscale chain |

---

## Supported Models

**Claude:** `claude-sonnet-4-6` (recommended), `claude-opus-4-6`, `claude-haiku-4-5`

**OpenAI:** `gpt-4o` (recommended), `gpt-4o-mini`

**Gemini:** `gemini-2.0-flash` (recommended, free), `gemini-2.0-flash-lite`, `gemini-2.5-flash-preview-05-20`, `gemini-1.5-flash-latest`

**9ARM:** Any OpenAI-compatible model name (configurable)
