# LinkVault

**Your bookmarks. Your vault. Your privacy.**

LinkVault is a privacy-first bookmark manager that runs entirely in your browser as a **single HTML file**. Your bookmarks stay local by default, are encrypted end to end when you sync, and travel between devices through any [xBrowserSync](#xbrowsersync-compatibility)-compatible service — so your vault can move between browsers and between apps, not just between tabs.

![Version](https://img.shields.io/badge/version-3.0.0-7c3aed)
![Single file](https://img.shields.io/badge/build-none%20required-2563eb)
![Privacy](https://img.shields.io/badge/sync-end--to--end%20encrypted-0f9d6b)

---

## Table of contents

- [Features](#features)
- [Quick start](#quick-start)
- [Setting up sync](#setting-up-sync)
- [Privacy and security](#privacy-and-security)
- [xBrowserSync compatibility](#xbrowsersync-compatibility)
- [Keyboard shortcuts](#keyboard-shortcuts)
- [Import and export](#import-and-export)
- [Browser support](#browser-support)
- [Architecture](#architecture)
- [Development](#development)
- [Contributing](#contributing)
- [Credits and attribution](#credits-and-attribution)
- [License](#license)
- [Disclaimer](#disclaimer)

---

## Features

### Organize

- **Bookmarks** with title, URL, description, tags, and an optional favorite flag.
- **Nested folders** with a collapsible sidebar tree.
- **Tags** for cross-cutting collections.
- **Drag and drop** to move bookmarks and folders between folders.
- **Recent views** for recently added, recently opened, and recently updated links.
- **Grid and list** layouts, plus a compact mode.

### Find

- **Instant search** across titles, URLs, descriptions, tags, and folder names, with match highlighting and a `/` shortcut.
- **Command palette** (`Ctrl`/`⌘ + K`) with fuzzy search over commands, folders, and bookmarks.
- **Duplicate finder** that groups links pointing at the same page, ignoring the scheme, a leading `www.`, a trailing slash, the `#fragment`, and tracking parameters — then lets you keep the oldest copy or remove all duplicates.

### Sync and privacy

- **End-to-end encrypted sync** — bookmarks are made unreadable on your device before they ever leave it.
- **Zero-knowledge** by design: your password is never transmitted to the sync service.
- **Conflict handling** with a three-way merge and a clear prompt when the same item changed on two devices; nothing is overwritten by surprise.
- **Automatic sync** roughly every 5 minutes while online, with exponential backoff (up to 30 minutes) when a service is unavailable.
- **Offline-first** — every change is saved locally and synced later.
- **Device pairing by QR code** — scan to connect a new device; the code contains only the service address and sync ID, never your password.

### Data portability

- **Import** from Netscape bookmark HTML (standard browser exports), LinkVault JSON backups, and xBrowserSync trees.
- **Export** a LinkVault JSON backup or a Netscape HTML file for another browser.
- **One-click backup** and restore, plus a downloadable diagnostic report.

### Experience

- **Automatic page metadata** — fetches real titles, descriptions, and favicons, with optional online fallbacks when a site blocks direct reads.
- **Light, dark, and system themes**, remembered per device.
- **Responsive layout** with a mobile bottom navigation bar and a slide-out drawer.
- **Accessible** — skip link, ARIA roles, visible focus states, reduced-motion support, and higher-contrast borders.
- **Graceful degradation** — works offline; icons fall back to built-in glyphs if the icon font cannot load.
- **App lock** to require a password when LinkVault opens.

---

## Quick start

LinkVault has **no build step and no runtime dependencies**. It is one self-contained file.

### Option 1 — Open it locally

1. Download or clone this repository.
2. Open `index.html` in a modern browser.

That is the whole installation. Your bookmarks are stored in your browser's IndexedDB and remain on your device.

> **Tip:** To use encrypted sync, serve the app over **HTTPS or `localhost`**, because the Web Crypto API requires a secure context. Opening the file directly works for local, offline use in most browsers.

### Option 2 — Self-host it

Because LinkVault is a static file, it deploys anywhere that can serve HTML:

- **GitHub Pages** — push the repository and enable Pages for the branch that contains `index.html`.
- **Netlify / Vercel / Cloudflare Pages** — point the project at the repository; no build command is needed.
- **Any web server or reverse proxy** — copy `index.html` into your document root.

Always serve over HTTPS in production so the Web Crypto API and the camera-based QR scanner are available.

### First run

On first launch you choose how to start:

| Choice | What it does |
| --- | --- |
| **Create New Sync** | Generates a new sync ID and lets you set a password. Recommended for keeping devices in step. |
| **Connect Existing Sync** | Connects to a sync ID and password you already have, or scans a QR code from another device. |
| **Use Offline** | Keeps everything local on this device. You can enable sync later from **Sync**. |

---

## Setting up sync

Sync uses the xBrowserSync-compatible REST API. By default LinkVault talks to `https://api.xbrowsersync.org`, and you can point it at any compatible service you prefer.

1. Open **Sync** from the sidebar.
2. Choose **Set Up Sync** (or **Connect Another Device** on a device that is already set up).
3. LinkVault generates a **sync ID** and asks for a **password**.
4. On your other devices, either enter the same sync ID and password, or scan the QR code / paste the share link.

**Keep your password safe.** Because only you can decrypt your data, there is no password reset — not even LinkVault can recover your bookmarks if the password is lost.

---

## Privacy and security

LinkVault is designed so that the sync service stores only data it cannot read.

- **Encrypted before upload.** Bookmark data is compressed and encrypted on your device; the service only ever sees ciphertext.
- **Your password stays with you.** It is used locally to derive the encryption key and is never sent to the service.
- **Local first.** Bookmarks, folders, and settings live in your browser. No analytics or tracking are sent anywhere.
- **Safe sharing.** QR codes and share links contain only the sync address and sync ID — never your password or key.
- **URL safety.** Dangerous URL schemes (`javascript:`, `data:`, `vbscript:`, `blob:`) are rejected.

### Cryptography

LinkVault uses the browser's native **Web Crypto API** and follows the xBrowserSync key-derivation format:

| Property | Value |
| --- | --- |
| Key derivation | PBKDF2-SHA256 |
| Iterations (sync) | 250,000 |
| Salt | The sync ID |
| Cipher | AES-GCM, 256-bit |
| Initialization vector | 16 random bytes per message |
| Payload encoding | `base64(IV ‖ ciphertext)` |
| Advertised format version | `1.5.0` (PBKDF2 key format, requires ≥ 1.4.0) |

The optional **app lock** is a separate convenience feature that derives a local key with PBKDF2 (150,000 iterations) and protects the interface only. It is not a substitute for your sync password, and your sync password is never stored.

---

## xBrowserSync compatibility

LinkVault implements the [xBrowserSync](https://www.xbrowsersync.org/)-compatible API and data format, which means it can interoperate with compatible sync services and clients:

- It advertises **format version `1.5.0`**, so compatible clients derive the encryption key with PBKDF2 (the post-1.4.0 format) rather than treating the password as a legacy raw key.
- It validates the service's reported API version against a minimum of **`1.1.9`**.
- Its tree model maps local bookmarks and folders to and from the xBrowserSync tree shape, preserving favourites and timestamps.

**LinkVault is an independent project.** It is not an official xBrowserSync client and is not affiliated with or endorsed by the xBrowserSync project. No xBrowserSync logos or artwork are used.

---

## Keyboard shortcuts

| Shortcut | Action |
| --- | --- |
| `Ctrl` / `⌘ + K` | Open the command palette |
| `/` | Focus the search box |
| `N` | Add a new bookmark |
| `F` | Create a new folder |
| `S` | Sync now |
| `D` | Find duplicate bookmarks |
| `T` | Cycle light / dark / system theme |
| `G` then `D` | Go to the dashboard |
| `?` | Show the shortcut help |
| `Esc` | Close dialogs and the palette |

---

## Import and export

| Direction | Format | Notes |
| --- | --- | --- |
| Import | Netscape bookmark HTML | Standard browser exports |
| Import | LinkVault JSON backup | Includes bookmarks, folders, and settings |
| Import | xBrowserSync tree | Compatible tree payloads |
| Export | LinkVault JSON backup | Full, restorable backup |
| Export | Netscape HTML | Move your bookmarks to another browser |

Backups can be created and restored from **Settings → Data**, and a diagnostic report can be downloaded for troubleshooting.

---

## Browser support

LinkVault targets modern, evergreen browsers on desktop and mobile (Chrome, Edge, Firefox, and Safari).

- **Storage:** IndexedDB.
- **Encryption:** Web Crypto API (requires a secure context — HTTPS or `localhost`).
- **QR scanning:** the `BarcodeDetector` API where available (Chromium-based browsers); other browsers fall back to pasting a share link.
- **QR generation:** built in, with no external dependency.

---

## Architecture

LinkVault is deliberately a single file. Everything — markup, styles, and logic — lives in `index.html`, organized into clearly separated modules in a documented order:

```
CONFIG → UTILITIES → COMPRESSION → CRYPTO → INDEXEDDB → API CLIENT
       → TREE / MERGE → SYNC ENGINE → DATA SERVICE → IMPORT / EXPORT
       → UI COMPONENTS → ROUTER / VIEWS → EVENT HANDLERS → INIT
```

Highlights:

- **CONFIG** centralizes versioning, the default service URL, crypto parameters, and the database name.
- **COMPRESSION** is a small LZUTF8 codec used before encryption.
- **CRYPTO** implements the xBrowserSync-compatible key derivation and AES-GCM payload format.
- **INDEXEDDB** is the local storage layer.
- **API CLIENT** wraps the xBrowserSync-compatible REST endpoints.
- **TREE / MERGE** converts between local records and the sync tree, and performs three-way merges.
- **SYNC ENGINE** orchestrates pull, push, conflict detection, and retries.
- **DATA SERVICE** holds in-memory state and bookmark, folder, and tag operations.
- **IMPORT / EXPORT** parses Netscape HTML, LinkVault backups, and xBrowserSync trees.
- **UI / ROUTER / EVENTS** render the interface and wire up interaction.

---

## Development

There is no build pipeline, package manager, or bundler.

1. Edit `index.html` directly.
2. Serve the folder locally so the app runs in a secure context:

   ```bash
   python3 -m http.server 8000
   ```

   Then open `http://localhost:8000/`.

3. Reload the page to see your changes.

The pure logic modules — **Compression** and **XBrowserSyncCrypto** — expose themselves via `module.exports` when loaded under Node, so they can be unit-tested without a browser:

```js
// Example: exercise the crypto module in Node
const { XBrowserSyncCrypto } = require('./index.html'); // via a test harness that extracts the script
```

> When adding features, keep the file self-contained, avoid new runtime dependencies, and preserve the existing module order and offline/graceful-degradation behaviour.

---

## Contributing

Contributions are welcome. To keep LinkVault dependable:

1. **Open an issue first** for substantial changes so the approach can be discussed.
2. **Fork the repository** and create a focused branch (`feature/short-description` or `fix/short-description`).
3. **Keep it a single file.** LinkVault's zero-build, zero-dependency design is a feature, not an accident.
4. **Preserve privacy guarantees.** Never send the user's password or plaintext bookmarks off-device.
5. **Maintain graceful degradation.** The app must keep working offline and when the icon CDN is unreachable.
6. **Follow the existing style.** Match the surrounding code, keep the module order, and document non-obvious decisions with comments.
7. **Test manually** across light/dark themes and at mobile widths, and verify sync against a compatible service.
8. **Write a clear pull request** describing the problem, the approach, and how you tested it.

By participating, you agree to keep discussion respectful and constructive.

---

## Credits and attribution

LinkVault stands on the work of others:

- **[xBrowserSync](https://www.xbrowsersync.org/)** — the open-source project whose compatible API and encrypted sync format LinkVault implements.
- **[lzutf8.js](https://github.com/rotemdan/lzutf8.js)** by Rotem Dan (MIT) — the LZUTF8 compression codec ported into LinkVault.
- **[Font Awesome](https://fontawesome.com/)** — icons, loaded from a CDN with a built-in text-glyph fallback so the interface never loses its icons.

---

## License

LinkVault is released under the **MIT License**. See the [LICENSE](LICENSE) file for the full text.

Copyright (c) 2026 LinkVault contributors.

---

## Disclaimer

LinkVault is an independent, community-oriented project. It is **not** affiliated with, endorsed by, or an official client of the xBrowserSync project. Because encryption is end-to-end and no password recovery exists, **you are responsible for keeping your sync ID and password safe** — lost credentials mean unrecoverable data.