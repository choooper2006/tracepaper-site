# Verify Tracepaper's Claims

Last updated: 22 September 2026 · Applies to version 0.11.2

Every privacy claim on this site is meant to be checked, not believed. This page is
how. None of it requires contacting us, and none of it requires a tool you do not
already have.

If you are evaluating Tracepaper for a school district, a clinic or an IT department,
this is the page to work through.

## Start inside the extension

Tracepaper has a **Privacy review** screen built into it. Click the Tracepaper icon in
the Chrome toolbar, then "Privacy review" at the bottom of the panel.

It shows what is stored on that computer right now, which websites the extension can
currently read, every permission the build asks for, and the connection rules Chrome
is enforcing on it. Those figures are read out of the browser as the page opens — they
are not copied from this website, so they cannot disagree with what the extension is
actually doing.

The same screen has a **Delete everything** button, which erases every guide, every
screenshot and every setting. There is no copy anywhere else, so that is the end of it.

## Check that nothing leaves the browser

This is the claim everything else rests on, and it is the easiest one to test.

1. Open `chrome://net-export` in a tab.
2. Press **Start Logging to Disk** and choose somewhere to save the file.
3. In another tab, use Tracepaper normally: record a guide, edit it, blur something,
   export it as a PDF and as a web page.
4. Come back and press **Stop Logging**.
5. Open the saved `.json` file in a text editor and search it for `tracepaper`.

Nothing belonging to the extension should appear, because there is nowhere for it to
connect to. Tracepaper has no server for your content. We did not build one and then
promise not to look at it.

A quicker version of the same test: open the Privacy review screen, press F12, choose
the Network tab, and export a guide. Exports are assembled in memory and handed
straight to your downloads folder.

## Check what is stored, and where

Press F12 on any page, choose **Application**, then **IndexedDB**, then `tracepaper`.

You will find two stores: `guides` and `images`. That is everything. Guides and
screenshots are never written to `chrome.storage.sync`, which would relay them through
Google's servers to your other devices.

## Check which websites it can read

Go to `chrome://extensions`, find Tracepaper, click **Details**, and look at **Site
access**.

Between recordings, Tracepaper holds access to nothing. Access is requested for one
site when you press Record and released when the recording stops.

**One thing this page will not tell you cleanly**, so we will: since Chrome 130, Chrome
keeps its own permanent record of every site you have ever approved for an extension,
and releasing access does not remove a site from that record. No extension can clear
it. A site you approved once can be re-acquired later without a fresh prompt. We
mention this because a reviewer who finds it on their own should not have to wonder
what else we left out.

## Read the code

There is no build step and nothing is minified, so the files inside the extension are
the files we wrote. These are the ones that matter:

| File | What to look for |
|---|---|
| `manifest.json` | Everything the extension is permitted to do, including the `connect-src` rule that stops it connecting anywhere. |
| `src/lib/capture.js` | The screenshot is scaled, field regions are destroyed, and only then is anything encoded. The full-size original is never written to disk. |
| `src/lib/fields.js` | What counts as a field, and therefore what gets blurred. |
| `src/content/recorder.js` | What a click and a keystroke actually record. Field *labels*, never field contents. |
| `src/background/service-worker.js` | Where screenshots are taken and site access is released. |

To read any of them, install the extension and open
`chrome-extension://lhflcpbdlckocgneldfbafillhakbdch/` followed by the path above — for
example `.../manifest.json` or `.../src/lib/capture.js`. They open as plain text.

We deliberately do not link these from inside the extension. A link there would break
silently the day a file is renamed, and a privacy page that quietly 404s is worse than
one that asks you to type a path.

## What we cannot prove to you

Being straight about the edges is part of the point.

- **We cannot prove a future version will behave the same way.** Every release is
  checked by `scripts/check-network.sh`, which fails the build if networking code
  appears anywhere outside the payments module, and the version number on each of
  these pages is stamped from the extension's manifest rather than typed by hand. But
  the check you run today applies to the build you ran it on.
- **Blurring is only as good as what was on screen.** Tracepaper blurs every text
  field automatically and gives you manual blur for anything else, but it cannot know
  that a name in a page heading is a patient's. Review your screenshots before you
  share them.
- **A closed shadow root is invisible to us too.** A small number of sites build form
  fields in a way that no extension can inspect. Those fields cannot be blurred
  automatically. Use manual blur, or hide the area before recording.
- **Once you export a file, it is an ordinary file.** A PDF you email is subject to
  whatever your email system does with it. Our promise ends at your downloads folder.

## What we would say under oath

Tracepaper supports FERPA and HIPAA compliance because we never receive your data.
There is no account, no server for your content, and no analytics of any kind — not
even anonymous counts.

We are not a certifying body, Tracepaper is not "HIPAA certified", and no software can
be. What we can tell you is that there is nothing on our side to breach, and this page
is how you confirm that for yourself.

See [Privacy](privacy) for the plain-language policy,
[Where your data goes](data-flow) for a diagram of where data goes, and
[Permissions](permissions) for why each permission is requested.

## Contact

support@gettracepaper.com
