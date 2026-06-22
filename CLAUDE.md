# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ODO Logger (v1.10.0) is a **single-file HTML application** (`index.html`) with no build system, no dependencies, and no framework. All CSS, HTML, and JavaScript live inline in one file. Open it directly in a browser — no server required.

## Development

There is no build step, no package manager, and no test suite. To develop:

- Edit `index.html` directly.
- Open the file in a browser to test (`file:///path/to/index.html`).
- The app can also self-download itself via the "Download App" button, which serializes `document.documentElement.outerHTML`.

**CORS constraint:** Several AI provider APIs block cross-origin requests from cloud-hosted origins (e.g., claude.ai). Claude's API requires the `anthropic-dangerous-direct-browser-access: true` header. Gemini blocks requests from claude.ai entirely — users must run the file locally to use Gemini. Always test AI API changes locally.

## Architecture

### State and Storage

All runtime state lives in the `entries` array (module-level `let`). API keys, selected provider, and models are persisted to `localStorage` with these keys:

- `odo_provider` — currently active provider ID
- `odo_key_<provider>` — API key for each provider
- `odo_model_<provider>` — selected model for each provider
- `odo_endpoint_9arm` — custom base URL for the 9ARM provider

### Data Flow

1. User drops/selects image files → `processFiles(files)`
2. Per file: `getFileDate()` extracts EXIF date (manual JPEG byte parser in `getExifDate()`), falling back to `file.lastModified`
3. `extractODO(file)` sends the image as base64 to the active AI provider and parses the JSON response `{odo, found}`
4. Entry objects `{name, date, odo, found, preview}` are pushed to `entries`, sorted by date, then `render()` redraws the table

### AI Provider Dispatch (`extractODO`)

All four providers are handled with conditional branches in a single function. The prompt is fixed:
> "Extract the ODO odometer reading km from this dashboard. Return ONLY valid JSON: `{\"odo\":123456,\"found\":true}` or `{\"odo\":null,\"found\":false}`"

Provider-specific API shapes:
- **Claude** — `POST /v1/messages`, multipart content with `image` + `text` blocks
- **OpenAI** — `POST /v1/chat/completions`, `image_url` with inline base64 data URL
- **Gemini** — `POST /v1beta/models/{model}:generateContent`, `inline_data` part
- **9ARM** — OpenAI-compatible, same shape as OpenAI, using a configurable base URL

### XLSX Export (`exportXLSX`)

No library is used. The export is built entirely in-memory:

1. `resizeImage()` — uses a `<canvas>` chain to step-downscale images to ≤320×200px (progressive halving for quality) and re-encodes as JPEG base64
2. The XLSX (`.xlsx`) file is an OOXML ZIP. A custom `zipFiles()` function builds the ZIP binary including CRC32 computation from scratch
3. The ZIP contains the standard OOXML parts: `workbook.xml`, `sheet1.xml`, `sharedStrings.xml`, `styles.xml`, `drawing1.xml`, and image media files
4. Images are embedded using Excel's `<cellImages>` extension (not the standard drawing anchor), which enables the "Place in Cell" feature in modern Excel

### Render Cycle

`render()` is the single re-render function — it rebuilds the entire `<tbody>` on every call. It also updates the four stat cards (photo count, total distance, first/last ODO). Inline ODO editing (`startEdit` / `commitEdit`) mutates `entries[idx].odo` then calls `render()`.
