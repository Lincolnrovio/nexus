# Nexus AI File Intelligence v2 — Test Report

## Checks performed

- `node --check` passed for every inline JavaScript block in `index.html`.
- `node --check` passed for `vault.js`, `file-analysis.js`, `sw.js`, and all three Netlify Functions.
- Local extraction harness passed for TXT, Markdown, CSV, JSON, and JavaScript files.
- File workspace manifest and context assembly passed.
- Resource-limit constants and PWA cache inclusion for `file-analysis.js` were inspected.
- Existing project files, vault module, provider functions, icons, manifest, and Netlify configuration were retained.

## Not verified in this environment

- Actual browser extraction of PDF, DOCX, XLSX/XLS, PPTX, and ZIP files. These paths use browser parser libraries loaded from CDNs and require a live browser session.
- Live image understanding with Groq/Gemini provider keys.
- Full model request behavior with a large real-world project archive. A per-request extracted-context cap is intentionally applied to protect mobile memory and provider context limits.
- Android Chrome/PWA visual and touch testing.

## Known constraints

- Parsing runs locally in the browser; extracted text and images are sent to the selected AI provider only after the user submits a question.
- ZIP extraction skips common dependency/build folders and limits archive size, entry count, expansion size, nested ZIP depth, extracted characters, PDF pages, and image count.
- The file workspace is retained in page memory for follow-up questions until cleared or the page is closed/refreshed. It is not saved in localStorage or the vault.
- Parser libraries are loaded from public CDNs, so first-time parsing of supported binary document formats requires internet access.


## ZIP completeness and ZIP download follow-up

- Increased ZIP inspection capacity to 1,500 entries, 200 MB compressed upload, 300 MB declared expanded data, and two nested ZIP levels.
- Removed automatic exclusion of dependency/build folders so their files are included in the manifest and available for extraction.
- Preserves original Blob bytes for extracted, unsupported, and failed-to-parse files where available.
- Added a **Download workspace ZIP** action that packages the workspace using JSZip and triggers a browser download.
- ZIP contents and archive entry progress are reported. Safety limits remain intentional; archives exceeding them must be split, and the UI reports that extraction is incomplete.
- Extracted text is split into numbered ~85,000-character model-request parts, with Analyze/Next/Previous controls so the user can send each part separately. The model cannot receive arbitrarily large archives in one request; processing all parts requires multiple provider calls.
- Static JavaScript syntax checks pass. Real browser extraction/download tests and Android memory testing require deployment in Chrome Android.

- Verified that file-analysis.js and all inline JavaScript blocks pass Node syntax checks after the ZIP-part controls were added.
