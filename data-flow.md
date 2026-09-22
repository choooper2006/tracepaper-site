# Tracepaper Data Flow

Last updated: 22 September 2026 · Applies to version 0.11.2

This document exists so a school or clinic reviewer can see exactly where recorded
content goes. If a change to Tracepaper would alter this diagram, the change does not
ship until this file is updated.

## Everything, in one picture

```
╔═══════════════════════════════════════════════════════════════════╗
║  YOUR COMPUTER                                                    ║
║                                                                   ║
║   The web page you are recording                                  ║
║            │                                                      ║
║            │  you click                                           ║
║            ▼                                                      ║
║   ┌──────────────────┐        ┌────────────────────────┐          ║
║   │  content script  │        │    service worker      │          ║
║   │  click position  │───────▶│  screenshot of the     │          ║
║   │  visible label   │        │  visible tab, masked   │          ║
║   └──────────────────┘        └───────────┬────────────┘          ║
║                                           │                       ║
║                                           ▼                       ║
║                               ┌────────────────────────┐          ║
║                               │      IndexedDB         │          ║
║                               │  guides + screenshots  │          ║
║                               └───────────┬────────────┘          ║
║                                           │                       ║
║                                           ▼                       ║
║                               ┌────────────────────────┐          ║
║                               │        editor          │          ║
║                               │  reorder, rewrite,     │          ║
║                               │  blur (destructive)    │          ║
║                               └───────────┬────────────┘          ║
║                                           │                       ║
║                                           ▼                       ║
║                               ┌────────────────────────┐          ║
║                               │       exporter         │          ║
║                               │     PDF · MD · HTML    │          ║
║                               └───────────┬────────────┘          ║
║                                           │                       ║
╚═══════════════════════════════════════════╪═══════════════════════╝
                                            │
                     ┌──────────────────────┴──────────────────────┐
                     ▼                                             ▼
          your Downloads folder                       your own Google Drive
          (stays on this computer)                    (drive.file scope only,
                                                       only if you choose it)
```

No arrow crosses that boundary toward us, because there is no "us" to cross to.
Tracepaper has no content server.

## The only two outbound connections that will ever exist

| Destination | What is sent | What is never sent | When |
|---|---|---|---|
| ExtensionPay (on Stripe) | Your email address, your extension install ID | Guide content, screenshots, page addresses, anything recorded | Only when you subscribe or when the extension checks whether you have paid |
| Google Drive API | The single exported file you chose to save | Anything else in your Drive, anything you did not export | Only when you click "Save to Drive" |

Both are opt-in. Neither exists yet — payments arrive at Step 15 and Drive export at
Step 13. In the version named at the top of this page, and in every version before it,
Tracepaper makes **zero** outbound connections.

## Where each kind of data lives

| Data | Storage | Syncs to Google? | Survives "Clear browsing data"? |
|---|---|---|---|
| Guides and screenshots | IndexedDB (`tracepaper`) | No | No |
| Settings (blur defaults, URL toggle) | `chrome.storage.local` | No | No |
| Recording state while recording | `chrome.storage.session` | No | Cleared when Chrome closes |
| Step sentences during a recording | `chrome.storage.session` | No | Cleared when Chrome closes |
| Free-tier guide counter (arrives at Step 15) | `chrome.storage.local` | No | No |
| Areas you chose to hide, per site | `chrome.storage.local` | No | No |
| Fields you chose to leave visible, per site | `chrome.storage.local` | No | No |

`chrome.storage.sync` is never used anywhere in this codebase. It would relay data
through Google's servers, which would break the claim this document makes.
`scripts/check-network.sh` fails the build if it appears.

## How the boundary is enforced in code

Three layers, each checkable by a stranger:

1. **The manifest.** `content_security_policy.extension_pages` sets
   `connect-src 'self'`. The browser itself refuses any outbound request from an
   extension page. Widening this list is a one-line diff in `manifest.json` and must
   be justified in this file.
2. **No host permissions at install, and none kept afterwards.** The extension is
   granted access to a website only at the moment you press Record on it, one site at
   a time, and the permission is released when the recording ends — always, with no
   opt-out. What releasing does not do: since Chrome 130, Chrome keeps its own
   permanent record of every site you have ever approved for an extension, which no
   extension can clear.
3. **`scripts/check-network.sh`.** Fails if `fetch`, `XMLHttpRequest`, `sendBeacon`,
   `WebSocket`, `EventSource` or `importScripts` appear anywhere outside
   `src/payments/`, or if `chrome.storage.sync` appears at all.

## How to verify it yourself in two minutes

1. Open `chrome://extensions`, enable Developer mode.
2. Find Tracepaper and click **service worker** to open its developer tools.
3. Open the **Network** tab and leave it open.
4. Record a guide, edit it, export it.
5. The Network tab stays empty.

## Change log for this document

Every version in which something about the diagram above changed — a new place data
is written, a new way it leaves, or a claim withdrawn. Newest first. Releases that
only fixed a bug without moving data are not listed; the full history is in
`CHANGELOG.md`.

**Outbound connections have been zero in every version listed here.** That is the
column that has never changed, and the one worth checking against
`chrome://net-export` rather than against this table.

| Date | Version | Change |
|---|---|---|
| 22 Sep 2026 | 0.11.0 | A **Privacy review** screen inside the extension reads the live manifest, permission list, settings and database and reports them, so this document can be checked against the running code. It adds **Delete everything**, which clears both IndexedDB stores and `chrome.storage.local`. Nothing new is written or sent. |
| 22 Sep 2026 | 0.10.1 | Stored screenshots go from 1280 to 1920 pixels wide, so roughly 44 KB a step instead of 25 KB. The mosaic block became a fraction of the image width, so the amount destroyed does not change with resolution. |
| 22 Sep 2026 | 0.10.0 | **Markdown export**: a `.zip` holding `guide.md` and an `images/` folder, built in memory by our own zip writer and handed to the downloads folder. The click marker is drawn into the *exported copy* of each image, because Markdown cannot overlay one image on another; the stored screenshot is still unmarked. |
| 22 Sep 2026 | 0.9.0 | **The first time content leaves the extension.** Export to a self-contained HTML file, and to PDF through Chrome's own print engine. Both go to the user's downloads folder through an anchor with `download`, which needs no permission. Nothing is uploaded, and no export service is contacted. |
| 22 Sep 2026 | 0.8.2 | Field exemptions are stored as **keys** (a field's name or label) rather than CSS selectors, in `chrome.storage.local`, per site. A selector could not reach inside a shadow root, so exemptions never matched. |
| 22 Sep 2026 | 0.8.0 | Manual blur in the editor, sharing the same destructive mosaic as automatic masking: the blurred image replaces the original in IndexedDB and the old pixels are gone. Per-site lists of fields to leave readable are stored in `chrome.storage.local`. |
| 21 Sep 2026 | 0.7.0 | The click marker is stored as a **fraction of the viewport beside the image**, never drawn into it, so it can be moved or removed later. |
| 21 Sep 2026 | 0.6.0 | The editor reads and writes guides in IndexedDB. Deleting a step does not delete its screenshot, so undo is honest; unreferenced images are swept at browser start. |
| 19 Sep 2026 | 0.5.0 | **Guides and screenshots become persistent**, in one IndexedDB database named `tracepaper` with stores `guides` and `images`. Never `chrome.storage.sync`, which would relay them through Google's servers. |
| 19 Sep 2026 | 0.4.4 | Areas can be marked to hide **before** recording starts, so those pixels are destroyed on the way out of the camera and never written to disk at all. Selectors are stored per site in `chrome.storage.local`. |
| 19 Sep 2026 | 0.4.0 | **Screenshots begin.** A tab capture is scaled down, every text-field region is destroyed by mosaic, and only then is anything encoded. The full-resolution original exists as a bitmap for a few milliseconds and is discarded; there is no pristine copy underneath. |
| 19 Sep 2026 | 0.3.3 | **A claim withdrawn.** The opt-in per-site "keep access" list added in 0.2.1 was removed. It existed to skip a permission prompt that Chrome never shows, because since Chrome 130 Chrome keeps a permanent record of every site ever approved. Site access is now always released when a recording ends, with no opt-out. |
| 19 Sep 2026 | 0.3.0 | Click capture. The text of a step is generated from a control's **label**, never its value, and held in `chrome.storage.session` until the guide is saved. Password fields are never read and never become a step. |
| 19 Sep 2026 | 0.2.2 | Per-tab badge, and a paused state when the recorded tab reaches a host that was not granted. All hosts granted during one recording are released when it ends. |
| 19 Sep 2026 | 0.2.1 | Site access is released when a recording ends, so the granted-site list stops growing. (This version also added an opt-in "keep access" list, withdrawn in 0.3.3 — see above.) |
| 19 Sep 2026 | 0.2.0 | Recording state machine added. Clicks are counted in `chrome.storage.session` as a number only; no page content is read or stored. |
| 19 Sep 2026 | 0.1.0 | First version. No outbound connections exist. |
