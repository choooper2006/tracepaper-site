# Tracepaper Permissions

Last updated: 19 September 2026 · Applies to version 0.5.0

Every permission Tracepaper requests is listed here, with the reason and the moment it
is asked for. If a permission is not on this list, Tracepaper does not have it.

## What you are asked for at install time

**Nothing.**

Install Tracepaper and Chrome shows no permission warning, because the extension
cannot see any website until you tell it to. This is unusual for a screen-recording
extension and it is deliberate.

## Requested at install (no user-visible warning)

| Permission | Why | What it does not do |
|---|---|---|
| `storage` | Saves your settings and the recording state. Chrome requires it for both. | Does not grant access to any website. Chrome shows no warning for it. |
| `scripting` | Lets Tracepaper run its recorder, and its area picker, in the tab you are working on. On its own it grants access to nothing: it is useless without permission for a site. | Cannot reach any website by itself. Chrome shows no warning for it. |
| `activeTab` | Lets Tracepaper see the address of the tab you are looking at, and lets you pick areas to hide on it, but **only while the popup is open**. Without it, Tracepaper could not tell you which site it is about to ask permission for. | Does not give any access while the popup is closed, and none at all to your other tabs. Chrome shows no warning for it. |

## Requested later, only when you act

| Permission | Asked when | Why |
|---|---|---|
| A single site, e.g. `https://example.com/*` | You press Record on that site, or the page moves to a host you have not approved and you choose to continue | Lets Tracepaper watch and photograph that site, and lets the recording survive clicking a link to another page on it. **This is the one prompt you will see**, and it names the site. Granted one site at a time, revocable at any time. |

You will see a Chrome prompt naming the exact site. Tracepaper never asks for "all
websites."

## Access is handed back when you stop

When a recording ends — you pressed Stop, you closed the tab, or the tab left the site
you approved — Tracepaper **gives the site permission back to Chrome**. While no
recording is running, Tracepaper holds access to nothing.

Chrome does not do this on its own. Extensions normally keep a granted site forever, so
a permission list quietly grows for years. Tracepaper releases it so that "this
extension has no access to any website right now" is true between recordings, and you
can check it yourself at any time.

### What releasing does not do

You are asked about a site **the first time** you record it. After that, Chrome
remembers your answer permanently, and Tracepaper can take the permission back without
asking you again.

That is Chrome's behaviour and no extension can change it. Since Chrome 130 the browser
keeps a *granted set* — every permission you have ever approved for an extension —
and `chrome.permissions.remove()` does not remove anything from it. A later request for
a site already in that set is granted silently.

So the accurate statement is:

- **Between recordings, Tracepaper holds no site access.** Verifiable, and true.
- **Tracepaper can regain access to a site you have already approved, without a new
  prompt.** Also true, and worth knowing.
- **A site you have never approved always prompts.**

If you want a site out of the granted set entirely, remove and reinstall the extension;
that is the only thing that clears it.

Choosing areas to hide asks for nothing at all: it runs on `activeTab`, so no site is
added to Chrome's list just because you marked something on it.

Access is released at the end of every recording, and any leftover is released when
Chrome starts, so "no recording running means no site access" holds without you having
to do anything. The **Site access** line in the popup reads the live answer straight
from Chrome.

### Chrome keeps a permanent list of sites you have approved

This one is confusing and worth understanding before you conclude something is wrong.

After Tracepaper releases a site, `chrome://extensions` still **lists** that site under
"Automatically allow access on the following sites", with its toggle **off**.

That is deliberate on Chrome's part, and it is not something an extension can change.
Since Chrome 130, that page shows the *granted set*: every permission the extension has
ever been granted, rather than the permissions it currently holds. Chrome's own
developer relations team states it plainly:

> "Permissions remain in the 'granted' set even if they are removed in an update, or by
> using the permissions.remove() API."
> — Oliver Dunk, Chrome Extensions, [chromium-extensions, October 2024](https://groups.google.com/a/chromium.org/g/chromium-extensions/c/tqbVLwgVh58)

No API lets an extension clear those rows. A `{revoke: true}` option for
`chrome.permissions.remove()` has been proposed and does not exist yet. Only removing
and reinstalling the extension clears the list.

Chrome does this to protect you: a sticky list means an extension cannot quietly
re-acquire a site in some later update without that site being visible to you. So the
list is best read as **a permanent record of every site you have ever approved**, and
the toggle beside each row is the live answer.

Three ways to confirm access really is gone:

1. **The toggle next to the site is off.** Off means no access.
2. **The Tracepaper popup says "Site access: none granted."** That line is read live
   from Chrome's permissions API and is the authoritative answer.
3. **Press Record on that site again.** If Chrome prompts you, the permission was
   released. An extension that still had access would start recording silently.

## Planned, not yet present

These are listed now so there are no surprises later. Neither exists in version 0.5.0.

| Permission | Arrives at | Why | Scope |
|---|---|---|---|
| `identity` plus Google's `drive.file` scope | Step 13 | Saving an exported guide to your own Google Drive. | `drive.file` lets Tracepaper see only files it created itself. It cannot read anything else in your Drive. |
| A network connection to ExtensionPay | Step 15 | Checking whether you have paid for Pro. | Sends your email address and install ID. Never guide content. |

## Permissions Tracepaper will never request

| Permission | Why we avoid it |
|---|---|
| `tabs` | Would let us read the address and title of every tab you have open. We only need the one tab you are recording, which `activeTab` already covers. |
| `<all_urls>` at install | Would grant access to every website you visit, forever, from the moment you install. We ask per site instead. |
| `history`, `bookmarks`, `cookies`, `downloads.open` | No feature needs them. |
| `desktopCapture` | Screen and window recording is out of scope. Tracepaper photographs one browser tab. |
| Broader Google Drive scopes | `drive.file` is enough to save a file. Anything wider would let us read your existing documents. |

## How to check what Tracepaper currently has

1. Open `chrome://extensions`.
2. Find Tracepaper and click **Details**.
3. Look at **Site access**. On a fresh install it says "On specific sites" with an
   empty list, or nothing at all.

The extension popup shows the same count, so you can see it without leaving the page
you are on.

## Revoking site access

`chrome://extensions` → Tracepaper → **Details** → **Site access** → remove any site.
Guides you already recorded stay where they are, on your computer.
