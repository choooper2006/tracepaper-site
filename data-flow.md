# Tracepaper Data Flow

Last updated: 19 September 2026 · Applies to version 0.5.0

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
║                               │  PDF · DOCX · MD · HTML│          ║
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
Step 13. As of version 0.5.0, Tracepaper makes **zero** outbound connections.

## Where each kind of data lives

| Data | Storage | Syncs to Google? | Survives "Clear browsing data"? |
|---|---|---|---|
| Guides and screenshots | IndexedDB (`tracepaper`) | No | No |
| Settings (blur defaults, URL toggle) | `chrome.storage.local` | No | No |
| Recording state while recording | `chrome.storage.session` | No | Cleared when Chrome closes |
| Step sentences during a recording | `chrome.storage.session` | No | Cleared when Chrome closes |
| Free-tier guide counter | `chrome.storage.local` | No | No |
| Areas you chose to hide, per site | `chrome.storage.local` | No | No |

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
   a time, and the permission is released when the recording ends unless you asked to
   keep that site.
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

| Date | Version | Change |
|---|---|---|
| 19 Sep 2026 | 0.1.0 | First version. No outbound connections exist. |
| 19 Sep 2026 | 0.2.2 | Per-tab badge, and a paused state when the recorded tab reaches a host that was not granted. All hosts granted during one recording are released when it ends. No change to outbound connections: still none. |
| 19 Sep 2026 | 0.2.1 | Site access is released when a recording ends, so the granted-site list stops growing. An opt-in per-site "keep access" list is stored in `chrome.storage.local`. No change to outbound connections: still none. |
| 19 Sep 2026 | 0.2.0 | Recording state machine added. Clicks are counted in `chrome.storage.session` as a number only; no page content is read or stored. No change to outbound connections: still none. |
