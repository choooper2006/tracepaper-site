# Tracepaper Data Flow

Last updated: 23 September 2026 · Applies to version 0.12.0

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
          your Downloads folder                       your own Google Drive (planned)
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

Both are opt-in, and **neither is built yet**. In the version named at the top of this
page, and in every version before it, Tracepaper makes **zero** outbound connections.
When either one ships, it will appear in the change log at the foot of this page.

## Where each kind of data lives

| Data | Storage | Syncs to Google? | Survives "Clear browsing data"? |
|---|---|---|---|
| Guides and screenshots | IndexedDB (`tracepaper`) | No | No |
| Recording state while recording | `chrome.storage.session` | No | Cleared when Chrome closes |
| Step sentences during a recording | `chrome.storage.session` | No | Cleared when Chrome closes |
| Free-tier guide counter (not built yet) | `chrome.storage.local` | No | No |
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

1. Open `chrome://extensions` and enable Developer mode.
2. Under Tracepaper, click **service worker** to open its developer tools, and open
   the **Network** tab. Recording and screenshots happen here.
3. Record a guide. The Network tab stays empty.
4. Open the guide in the editor, press F12 there, and open that **Network** tab too.
   Editing and exporting happen in the editor page, not the service worker.
5. Edit the guide and export it in every format. That tab stays empty as well.

[Verify our claims](verify) has a fuller test using Chrome's own network log.

## Change log for this document

One row per version in which the picture above changed: a new place data is written, a
new way it leaves, or a claim withdrawn. Newest first.

Interface changes, bug fixes and work that moved nothing are deliberately not here —
a list long enough to skim past is not a disclosure. **Outbound connections have been
zero in every version ever released**, which is the line this table exists to let you
check, and `chrome://net-export` checks it better than any table can.

| Date | Version | Change |
|---|---|---|
| 23 Sep 2026 | 0.12.0 | **A stopped recording is written to IndexedDB at once**, as an unnamed draft, instead of waiting in session memory until you chose to keep it. A guide's name no longer defaults to the site's address, so the address reaches an export only if you tick **Include the site address in exports**. A screenshot is kept only if the tab is still showing the page that was clicked. Nothing new is sent. |
| 22 Sep 2026 | 0.11.0 | **Delete everything** added: one action clears both IndexedDB stores and `chrome.storage.local`. Nothing new is written or sent. |
| 22 Sep 2026 | 0.10.0 | Markdown export, to the same downloads folder: a `.zip` holding `guide.md` and an `images` folder, assembled in memory. |
| 22 Sep 2026 | 0.9.0 | **The first time content leaves the extension.** Guides can be exported as a self-contained HTML file and as PDF, both handed straight to your downloads folder. Nothing is uploaded and no export service is contacted. |
| 22 Sep 2026 | 0.8.0 | Manual blur. The blurred image **replaces** the original in IndexedDB and the old pixels are gone. Fields you choose to leave readable are stored per site in `chrome.storage.local`. |
| 19 Sep 2026 | 0.5.0 | **Guides and screenshots become persistent**, in one IndexedDB database named `tracepaper`. Never `chrome.storage.sync`, which would relay them through Google's servers. |
| 19 Sep 2026 | 0.4.4 | Areas can be marked to hide **before** recording starts, so those pixels are destroyed on the way out of the camera and never written to disk at all. |
| 19 Sep 2026 | 0.4.0 | **Screenshots begin.** A capture is scaled down, every text-field region is destroyed, and only then is anything encoded. The full-resolution original is never written anywhere. |
| 19 Sep 2026 | 0.3.3 | **A claim withdrawn.** The opt-in per-site "keep access" list added in 0.2.1 was removed; it existed to skip a prompt Chrome never shows. Site access is now always released when a recording ends, with no opt-out. |
| 19 Sep 2026 | 0.3.0 | Click capture. A step's text comes from a control's **label**, never its value, and is held in `chrome.storage.session` until the guide is saved. Password fields are never read. |
| 19 Sep 2026 | 0.2.1 | Site access is released when a recording ends, so the granted-site list stops growing. |
| 19 Sep 2026 | 0.2.0 | Recording state machine. Clicks are counted in `chrome.storage.session` as a number only; no page content is read or stored. |
| 19 Sep 2026 | 0.1.0 | First version. No outbound connections exist. |
