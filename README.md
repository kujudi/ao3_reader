# Archive Style Reader

A local, offline fan fiction reader that mimics the look and feel of [Archive of Our Own (AO3)](https://archiveofourown.org). Drop in a file, fill in some tags, and pretend you're reading on AO3.

---

## What it does

Upload a fic file → fill in metadata (fandom, tags, rating, etc.) → read it in a pixel-faithful AO3 layout, complete with the red nav bar, the metadata box, chapter navigation, kudos button, and a comment section that goes absolutely nowhere.

Everything runs locally in your browser. No server, no account, no data leaves your machine.

---

## Supported formats

| Format | Notes |
|--------|-------|
| `.txt` | Auto-detects GBK / UTF-8 / UTF-16 encoding. Chapters auto-detected from common heading patterns. |
| `.md`  | Same as above, with Markdown rendering (headings, bold, italic, dividers). |
| `.epub` | Chapters parsed from OPF spine order. Requires an internet connection to load JSZip on first use. |
| `.docx` | Fully offline via bundled Mammoth.js. |

---

## Features

- **Auto chapter detection** — recognises 第X章, Chapter X, （一）, 【title】 and more; prompts you to confirm before splitting

---

## How to use

1. Download `index.html`
2. Open it in any modern browser (Chrome, Firefox, Safari)
3. Drop in your fic file
4. Fill in whatever tags you feel like
5. Hit **渲染** and enjoy

---

## Limitations

- Reading progress is not saved — refreshing the page resets everything
- The Kudos button and comment section are decorative only
- Images inside epub files are ignored (text only)
- epub parsing requires JSZip from a CDN; offline epub reading is not supported
