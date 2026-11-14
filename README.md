# Nexus AI — Private Vault Upgrade

This ZIP is based on the project you supplied. Existing chat UI, conversation history, local long-term memory, research mode, image analysis, image generation, voice input/output, themes, settings, Netlify Functions, manifest, icons, and PWA service worker are retained. File Intelligence has been upgraded with a persistent in-session analysis workspace and broader document/project extraction.

## New: Nexus AI Private Vault

- Browser-only API-key storage in IndexedDB.
- AES-256-GCM authenticated encryption for the complete vault payload.
- PBKDF2-SHA-256 key derivation with a unique 16-byte salt and 600,000 iterations.
- Master password is never saved. The derived non-extractable CryptoKey and decrypted keys exist in page memory only while unlocked.
- 30 logical key slots: `GROQ_API_KEY_1`–`GROQ_API_KEY_10`, `GEMINI_API_KEY_1`–`GEMINI_API_KEY_10`, and `TAVILY_API_KEY_1`–`TAVILY_API_KEY_10`.
- Add, edit, delete, enable/disable, test, mask, and inspect key status/usage/error history.
- Encrypted backup export/import with a separate backup password.
- Round-robin key selection, temporary suspension after transient failures, and exponential backoff.
- HTTP 401/403 disables a key until manually re-enabled. HTTP 429 activates a provider-wide cooldown. Delayed alternate-key retry after HTTP 429 is **off by default** and can only be enabled when you confirm that keys have independently permitted quotas. Do not use multiple keys to bypass shared account quotas, billing limits, or provider terms.
- Direct browser calls to Groq, Gemini, and Tavily when the vault is unlocked and contains enabled keys.
- Model selection for Groq and Gemini, plus Gemini image generation model selection.


## New: File Intelligence v2

- Supports plain text, Markdown, CSV/TSV, JSON/JSONL, common programming languages, HTML/CSS, XML, YAML, TOML, SQL, shell scripts, logs, SVG, PDF, DOCX, XLSX/XLSM/XLS, PPTX, ZIP archives, and common image formats.
- ZIP inspection walks all archive folders, including dependency/build folders, lists unsupported entries, preserves original file bytes, and expands nested ZIP files up to two nested levels.
- Spreadsheet extraction converts workbook sheets to CSV-like text; presentation extraction reads slide text; PDF extraction labels pages; extracted content retains file paths/names.
- Images are passed to the selected provider's vision-capable model when File Intelligence is used.
- A file workspace stays available across follow-up questions in the same open page session, so users can ask several questions without uploading the files again. Use **Download workspace ZIP** to create a new downloadable ZIP containing the original bytes of retained uploads/archive entries, including binary files, plus extracted text for entries whose bytes are unavailable. Use **Clear file workspace** to remove it from the current page session.
- Extraction progress, file manifest, per-file status, warnings, and truncation notices are shown in the interface.
- Browser-side safety limits: 100 MB per individual file, 200 MB per ZIP upload, 1,500 archive entries, 300 MB declared expanded archive data, 200,000 extracted characters per file, 3,000,000 characters per analysis workspace, 500 PDF pages, and 12 images up to 20 MB each. These limits protect mobile browser memory. Oversized archives must be split into smaller ZIP parts; the UI reports skipped or unextracted entries rather than claiming complete extraction. A single AI request is still capped at approximately 105,000 characters because model context windows are finite.
- File parsing happens locally in the browser. Extracted text and selected images are sent to the configured AI provider only when you submit a question; they are not sent to Nexus Netlify Functions when using the Private Vault direct-provider mode.
- Parser libraries (PDF.js, Mammoth, JSZip, SheetJS) are loaded from public CDNs. An internet connection is required for those libraries unless they are already cached by the browser. Do not upload confidential documents unless you trust the browser, device, provider, and provider data-handling terms.

## Critical security model

This is browser-side key storage, not a way to make browser API keys secret from the running application.

1. Encryption protects stored data while the vault is locked.
2. When unlocked, the page must decrypt keys to use them. Malicious JavaScript, a compromised dependency, browser extension, or script running on the same origin may be able to access keys while unlocked.
3. API keys are sent directly from the browser to the selected provider. They are not sent to the Nexus Netlify Functions by the Private Vault integration.
4. Use HTTPS. Web Crypto and service-worker/PWA features require a secure context (HTTPS or localhost).
5. Use a unique master password of at least 12 characters. Do not reuse your email, device, or other account password.
6. Export an encrypted backup and store it somewhere private. If browser data is cleared, the vault can be lost. If you forget the master password and have no usable backup, the vault cannot be recovered.
7. Lock the vault when finished. It auto-locks after 10 minutes without vault activity.
8. Browser storage is not guaranteed to be permanent and can be cleared by the user, browser, or device.
9. The app does not validate whether a provider permits browser-origin requests for your account or key. If a provider blocks CORS, use a provider-approved architecture rather than weakening browser security.
10. Do not expose the deployed site to untrusted editors or inject third-party scripts you do not trust.

The old Netlify Functions remain in the project for compatibility. The legacy Netlify Functions remain available but are **not used by default**. The Settings panel contains an explicit opt-in checkbox for the legacy backend; enabling it allows the original functions to use their own Netlify environment variables. Leave it off for browser-only operation. Unlock the vault, add the required keys, and select the provider in the Vault dashboard.

## Install / deploy from Android

### GitHub + Netlify

1. Extract this ZIP on your phone.
2. Upload the **contents** of the extracted folder to the root of your GitHub repository. Keep the `netlify/` folder and `.github/` folder structure.
3. Confirm the repository contains `index.html`, `vault.js`, `sw.js`, `manifest.json`, `icon-192.png`, `icon-512.png`, `netlify.toml`, `README.md`, and `netlify/functions/`.
4. In Netlify, import the GitHub repository as a site. `netlify.toml` configures the existing functions directory.
5. Wait for deployment, then open the HTTPS site.
6. Open **Nexus AI Private Vault** → **Create new vault**. Create and confirm a strong master password.
7. Add API keys under the correct provider. The key labels are assigned the logical identifiers shown in the dashboard.
8. Choose the chat provider and models, then save provider settings.
9. Test each key from its row. Start a chat or research request while the vault is unlocked.
10. Export an encrypted backup. Keep its backup password separate from the backup file.

### GitHub Pages / static hosting

GitHub Pages can serve the static PWA and the Private Vault front end. It cannot run the included Netlify Functions. In static-only hosting, browser-vault requests work directly with providers if the provider permits browser CORS; the legacy backend fallback will not work without a separately deployed compatible backend.

### PWABuilder / Android

1. Deploy the PWA to an HTTPS URL first.
2. Confirm `manifest.json`, `sw.js`, and both PNG icons load successfully.
3. Open PWABuilder and enter the deployed HTTPS site URL.
4. Run its PWA validation. Resolve manifest, icon, and service-worker errors before packaging.
5. Choose the Android packaging option and follow PWABuilder's current signing/package instructions.
6. Install the generated test package on your own device and test vault unlock, key entry, chat, research, attachments, voice, and offline shell behavior.

PWA packaging does not hide browser-side keys. The same-origin JavaScript security warning applies to the installed PWA.

## Provider API details

- **Groq:** OpenAI-compatible chat completions endpoint; current model availability can change. The app offers a selection of model IDs and lets provider responses determine whether a selected model is available.
- **Gemini:** `generateContent` endpoint for text and image understanding. Gemini image generation uses the selected Gemini image model. Google may change model names, availability, or API behavior.
- **Tavily:** Search endpoint with bearer-token authentication; returns source titles, snippets, and URLs.

API providers may impose account-specific rate limits, usage charges, age/eligibility requirements, and terms. Check those requirements and obtain appropriate adult assistance before using any paid account or enabling billable usage. Adding more keys does not increase a shared account quota.

## Testing and known limitations

- JavaScript syntax checks are run on the main inline application scripts and `vault.js`.
- Vault encryption/decryption, unlock/lock, and encrypted backup flows are tested with a local browser-API test harness.
- Real provider integration tests require valid provider keys and network access from the deployed browser. No real API keys were included in the supplied ZIP, so live Groq/Gemini/Tavily calls cannot be verified in this build environment.
- Mobile layout is responsive by CSS and should still be tested on the target Android device/browser.
- The original Netlify Functions were retained rather than removed. They remain a server-side fallback only; vault-based calls are direct from the browser.
- Image generation and model availability depend on provider access and current API support.

## Project files

- `index.html` — existing Nexus AI interface plus Vault dashboard, file workspace UI, and direct-provider routing.
- `file-analysis.js` — local document, spreadsheet, presentation, image and recursive ZIP extraction; preserves original bytes and can generate downloadable ZIP archives from the workspace.
- `vault.js` — IndexedDB, PBKDF2, AES-GCM encryption, key management, rotation, provider clients, encrypted backup.
- `sw.js` — updated PWA cache including `vault.js` and `file-analysis.js`.
- `manifest.json`, `icon-192.png`, `icon-512.png` — retained.
- `netlify/functions/chat.js`, `research.js`, `image.js` — retained legacy backend endpoints.
- `netlify.toml`, `.github/workflows/build.yml` — retained deployment/build configuration.

## Nexus AI app icon refresh

The app branding now uses a custom AI-inspired interwoven-knot symbol (not the ChatGPT trademark/logo). The new artwork is used for the sidebar brand mark, welcome-screen emblem, browser favicon, Android home-screen icon, and PWA install icons.

Icon assets:
- `favicon.png` — 64×64 browser favicon
- `apple-touch-icon.png` — 180×180 touch icon
- `icon-192.png` — 192×192 PWA icon
- `icon-512.png` — 512×512 PWA icon, also declared as maskable

The service-worker cache version was incremented so deployed clients can fetch the updated icon assets. If an installed PWA keeps showing the old icon, remove the old home-screen shortcut and install it again after the new deployment is live.
