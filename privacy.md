# Tracepaper Privacy

Last updated: 19 September 2026 · Applies to version 0.5.0

## The short version

Tracepaper has no server. There is no account to create, no sign-in, and nowhere for
your guides to go. Everything you record stays in your own browser, on your own
computer, until you export it yourself.

We cannot see your guides. Not because we promise not to look — because there is no
copy anywhere we could look at.

## What Tracepaper records

Only while you are recording, and only in the tab you chose:

- Screenshot pixels of the visible page at each click
- Where on the page you clicked
- The visible label of the thing you clicked, such as the word on a button, used to
  write the step — `Click "Save"`. Labels are cut off at 60 characters.
- The page address and title, if you turn that option on (it is off by default)

## What Tracepaper never records

- **Anything you type.** Tracepaper never reads the value of a form field. Step text
  is built from the field's *label*, never its contents.
- **Password fields.** These are never photographed and never become a step.
- **Your other tabs, your desktop, your camera, your microphone, your browser history,
  or your cookies.**

Because a screenshot photographs whatever is on the screen, text you typed earlier
could otherwise appear in the picture. So Tracepaper blurs it automatically, before the
picture is saved, and you decide which ones to un-blur later.

**Blurred automatically:** every text box, number, email, phone, date and search
field, every large text area, every password field, and any area of a page you can
type into directly.

**Hidden before anything is captured:** any area you marked yourself. Open the
Tracepaper popup on a page, choose "Hide areas on this site", and click the things that
should never appear — a patient banner, a student name column, a sidebar. Those regions
are destroyed on the way out of the camera, so unlike a blur you add afterwards, the
pixels never reach your disk at all. The choice is remembered for that site.

**Not blurred automatically:** what a dropdown is currently showing. A dropdown choice
is usually the instruction itself — "select United States" — so hiding it would empty
out the guide. If a dropdown on your system shows a person's name or anything else
private, blur it by hand in the editor before you share the guide.

The blur is permanent and it happens first. The screenshot is scaled down, every field
region is averaged into blocks, and only then is anything written to disk. The
full-size original exists for a few milliseconds inside the extension and is discarded.
There is no pristine copy underneath for anyone to recover, and no setting that brings
one back.

Screenshots are saved at 1280 pixels wide, which keeps a long guide to a few megabytes
instead of sixty.

## Which websites Tracepaper can see

- Between recordings, none at all.
- The first time you record a site, Chrome asks you, naming that site. Decline and
  nothing happens.
- A site you have already approved can be used again without a new prompt. Chrome
  keeps a permanent record of every permission you have ever granted an extension, and
  releasing a permission does not remove it from that record. No extension can change
  this. [Permissions](permissions) explains it in full.

## Where your guides are stored

In your browser's own storage (IndexedDB), on your computer. That means:

- Nobody else can see them, including us.
- They do not sync to your other computers.
- **If you clear your browsing data, they are deleted.** There is no backup and we
  cannot recover them. Export anything you want to keep.

Tracepaper asks Chrome to treat its storage as persistent, which asks the browser not
to discard it when disk space runs low. That is a request, not a guarantee, and it does
nothing to protect against clearing browsing data on purpose. Every guide that has
never been exported is labelled as such, in the guide list and on the guide itself.

Stopping a recording does not save it. You are asked whether to keep it, and a guide
you discard is deleted along with its screenshots straight away.

## What leaves your computer

Nothing, unless you ask for it:

- **Exporting** a guide writes a file to your Downloads folder. It does not pass
  through us.
- **Saving to Google Drive** sends the file from your browser to your own Google
  Drive account, using a permission that only lets Tracepaper see files it created.
  Google is your provider in that transaction, not ours.
- **Paying for Pro** sends your email address to ExtensionPay, which runs on Stripe,
  so it can answer one question: has this person paid? It never receives guide
  content, screenshots, or anything you recorded.

That is the complete list.

## Analytics

There are none. No usage counts, no crash reports, no "anonymous" statistics, no
advertising identifiers, no cookies. We do not know how many guides you have made or
whether you have opened the extension today.

## FERPA and HIPAA

Tracepaper supports FERPA and HIPAA compliance because we never receive your data.
Since no student records or health information ever reach us, there is no vendor
agreement to sign, no Business Associate Agreement needed, and nothing for your
privacy office to review about us.

Tracepaper is not "certified" and is not "HIPAA compliant" — those words describe
organizations and programs, not tools. Your own handling of what you record is still
your responsibility.

## Children

Tracepaper is a workplace tool and is not directed to children under 13. It collects
nothing from anyone, including children.

## How to check any of this yourself

See [Where your data goes](data-flow) for a diagram, and [Permissions](permissions)
for why the extension asks for each thing it asks for. The code is structured so that
a reviewer can verify these claims without taking our word for it:

- The manifest restricts network connections to the extension itself.
- `scripts/check-network.sh` fails the build if any networking code appears outside
  the payments module.
- You can open the service worker's developer tools, record a guide, and watch the
  Network tab stay empty.

## Contact

support@gettracepaper.com
