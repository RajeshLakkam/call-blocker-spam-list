# Call Blocker

[![Build](https://github.com/RajeshLakkam/call-blocker/actions/workflows/build.yml/badge.svg)](https://github.com/RajeshLakkam/call-blocker/actions/workflows/build.yml)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)

Android app that rejects telemarketing, spam and promotional calls before the phone rings.

All documentation lives in the **`how to use/`** folder:

- [HOW_TO_USE.md](how%20to%20use/HOW_TO_USE.md) — using the app, and building it
- [REQUIREMENTS.md](how%20to%20use/REQUIREMENTS.md) — what it must do, and the platform constraints behind it
- [ARCHITECTURE.md](how%20to%20use/ARCHITECTURE.md) — how it is put together and why

Contributors: start with [CONTRIBUTING.md](CONTRIBUTING.md). Release notes: [CHANGELOG.md](CHANGELOG.md).

## Status

**v0.0.2** — a layered screening engine with whitelist-only mode, the shared spam list with in-app
reporting, background-reliability setup, Room storage, a Compose UI, and 85 unit tests covering the
engine, number normalisation, the submission rules, the spam-list parser and the list files
themselves.

The community backend and the reputation provider are still behind interfaces with local stand-ins
(`LocalCommunityApi`, `NoOpReputationProvider`) — see [Known gaps](#known-gaps-before-this-ships).

## How blocking works

`CallBlockerScreeningService` (a platform `CallScreeningService`) hands each incoming call to
`ScreeningEngine`, which runs an ordered list of layers and takes the first decision:

| Order | Layer | Outcome |
|---|---|---|
| 0 | Emergency number | always allow |
| 1 | User allow list | allow |
| 2 | Contact | allow |
| 3 | **Whitelist-only mode**, when on | reject everything else |
| 4 | User block list | reject |
| 5 | **Numbers this user reported** as spam | reject |
| 6 | Hidden / withheld caller ID | per setting |
| 7 | Pattern rules (140-series, custom regex) | per rule |
| 8 | International | per setting |
| 9 | **Shared spam list** (spamlist.json) | silence |
| 10 | Community reports over threshold | reject or silence |
| 11 | Cached reputation score | silence |
| 12 | Strict mode (not a contact) | silence |
| — | default | allow |

Adding a detection strategy means adding one `ScreeningLayer` and one line in
`AppContainer.screeningEngine`.

### Whitelist-only mode

Turn it on and the allow list becomes the only way through: every other caller is blocked, whatever
the rules say. Contacts still ring while **Trust my contacts** is on; emergency numbers always ring.
The switch sits on the allow-list screen next to the list it depends on, and again in Settings.
It is one layer — `AllowlistOnlyLayer` — placed directly under the two "always allow" layers, which
is what gives it the guarantee.

## Staying alive in the background, on four permissions

The app asks for **four** permissions — contacts, notifications, internet, network state — and
**screening needs none of them**. The only thing it needs is the caller ID & spam role.

Every incoming call is handed to the app by Android itself: a `CallScreeningService` is bound by
the Telecom framework when a call arrives. That is a system-initiated bind, so it is not governed
by battery optimisation, by Doze, or by whether the app has run since boot. **The app does not need
to be running, and swiping it out of recents does not stop screening.** There is nothing to keep
alive: no foreground service, no battery exemption, no `READ_PHONE_STATE`.

Two things genuinely break that, and the app handles both as far as anything can:

- **Force stop.** Android will not start a force-stopped app for anything, including a call, until
  the user opens it again. Nothing works around this, by design.
- **Manufacturer battery software.** Xiaomi, Oppo, Vivo, Realme, OnePlus, Huawei, Honor and Samsung
  can treat a swipe from recents like a force stop unless the app is on an autostart allow list.

So the app asks for exactly three things — the role, and optionally contacts and notifications —
and asks for no battery exemption at all. The manufacturer advice is a separate, dismissible card
shown only on the phones it applies to, phrased as "if calls stop being screened on this phone"
rather than as a demand. Android exposes no way to read an OEM autostart list, so there is nothing
to verify and nothing to nag about. See `SetupChecklist` and `BackgroundReliability`.

## The shared spam list

[`spamlist.json`](spamlist.json) is a list of known nuisance numbers. Every install reads it, so a
number added to it protects everyone who updates after.

**It is served from its own public repository:**
[RajeshLakkam/call-blocker-spam-list](https://github.com/RajeshLakkam/call-blocker-spam-list), which
is where pull requests adding numbers go. This repository is private, and every install fetches the
list unauthenticated — a private repository answers 404 to all of them, and the only way round that
would be a token inside the APK that anyone could unzip and read. So the code is private and the
data it serves is not.

The copy here is the one bundled into the APK and the one `mergeNewAdditions` writes to. Publishing
means copying it to the list repository — see [Publishing the list](#publishing-the-list).

```json
{
  "number": "+911400000001",
  "name": "Loan offer robocall",
  "details": "Automated call offering a pre-approved personal loan.",
  "category": "TELEMARKETING"
}
```

**In the app**, a number on the list is **silenced** — it goes to your call log as a missed call
with the name and details attached, never disconnected outright. That asymmetry is deliberate: the
list is open to strangers, so a wrong entry costs you a silent ring you can see and undo, not a
call you never knew about. Anything on your own allow list overrides it, and emergency numbers are
untouchable.

**Updating** happens once a day, anchored to the time you first ran the app, and there is an
**Update now** button in Settings. The copy bundled in the APK means a fresh install is protected
before its first sync, and a failed sync keeps the copy already on the phone.

### Reporting a number from the app

**Lists → Reported** is the form — it is the tab, not something behind a button, with everything
already reported listed underneath it. **History → Report spam** does the same for a call that is
already on screen, whether it got through or was silenced, and so does **Report** on a row of the
block list. The form requires two things — the number and who the caller claims to be — and offers
a description and a category, which are optional. The report already protects the person making it,
so a thin one is worth more than an abandoned one; the review is where thin reports are caught.

Reporting does three things. The number is blocked on that phone immediately, by
`ReportedNumberLayer` reading the reports rather than by an entry on the block list — so it is
listed once, under **Lists → Reported**, and deleting the report is the one action that undoes it.
The app confirms it on screen and in a notification. And the report is composed as an email to the
list's maintainer under a fixed subject, carrying the entry as prose and as pasteable JSON.

**The app sends neither the commit nor the mail itself** — either would mean shipping a credential
inside the APK, which anyone could pull out and use to vandalise the list or send as that address.
So the phone's own mail app opens with the report filled in and the user presses send. A phone with
no mail app still gets the block, is told the mail did not go, and can retry from **Email** on the
report's row under Reported. A number can also be raised as an
[issue](https://github.com/RajeshLakkam/call-blocker-spam-list/issues/new) on the list repository.

### `new_additions.json` — the inbox

Reports that have arrived but are not yet on the list go in [`new_additions.json`](new_additions.json)
at the repository root, in the same shape as `spamlist.json`. It is the review queue: a maintainer
reads what is in it, deletes anything that should not ship, and merges the rest.

```bash
./gradlew mergeNewAdditions
```

That folds every entry into `spamlist.json`, skips numbers already on it, fills in `addedOn`, and
empties the inbox back to `[]`. Bad entries fail the task with a reason and nothing is merged.
**Release builds run it automatically** — `assembleRelease` and `bundleRelease` depend on it — so a
shipped APK always carries everything reviewed up to that point. Debug builds and CI deliberately
do not, since rewriting two tracked files on every build would leave contributors with a dirty
working tree.

### Publishing the list

The copy in this repository is the source of truth: `mergeNewAdditions` writes to it and
`bundleSpamList` copies it into the APK's assets. Installs read the copy in the **list repository**,
so the two have to be kept in step — a merge that is never published reaches only people who install
a new APK.

After merging, publish it:

```bash
./gradlew mergeNewAdditions
cp spamlist.json ../call-blocker-spam-list/spamlist.json
cd ../call-blocker-spam-list && git commit -am "Update spam list" && git push
```

Installs pick it up within about 24 hours, anchored to when each one first ran.

### Adding a number

1. Edit [`spamlist.json`](https://github.com/RajeshLakkam/call-blocker-spam-list/blob/HEAD/spamlist.json)
   in the list repository — on GitHub you can do this in the browser, which forks the repo and opens
   a pull request for you.
2. Add an object with `number` (international form, e.g. `+911400000001`), `name` (what the caller
   claims to be) and `details` (what actually happens on the call). `category` is optional.
3. Open the pull request against the list repository. A maintainer folds accepted numbers back into
   this repository's copy, where `SpamListFileTest` gates them on the next build: a bad number, a
   duplicate, an unknown category or anything the app would silently drop fails CI.

**Never add a number belonging to a real person.** This list silences calls on other people's
phones — it is for call centres, robocallers and scam lines, not for someone you are in a dispute
with. Entries that look personal will be rejected, and anyone can open a PR removing an entry that
turned out to be wrong.

## Install it on your phone

Requires **Android 10 (API 29) or newer** for screening to work. The app installs from Android 7,
but below 10 the platform will not grant it the caller ID role.

**If you just want to run it**, download a debug APK from the
[Actions](https://github.com/RajeshLakkam/call-blocker/actions) tab — open the most recent green **Build**
run, scroll to **Artifacts**, and download `app-debug-apk`. Unzip it, copy the `.apk` to your phone,
and tap it. Android will ask you to allow installs from whichever app you opened it with
(**Settings → Install unknown apps**) and Play Protect will warn you about an unrecognised app —
both are normal for an APK that did not come from the Play Store.

**If you would rather build it yourself:**

```bash
git clone https://github.com/RajeshLakkam/call-blocker.git
cd call-blocker
./gradlew assembleDebug        # gradlew.bat on Windows
```

The APK lands at `app/build/outputs/apk/debug/app-debug.apk`. With USB debugging on:

```bash
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

**Then, on the phone**, open Call Blocker and tap **Set as caller ID & spam app** — nothing is
screened until you do. Grant contacts so people you know always ring, and notifications so you are
told when a call is blocked.

Full instructions, including what to do when an install fails and how to build a signed release
APK: [HOW_TO_USE.md → Part 2](how%20to%20use/HOW_TO_USE.md#part-2--building-and-running-it).

## Building

```bash
./gradlew test             # engine and normalisation tests, no device needed
./gradlew assembleDebug    # debug APK, ~18 MB
./gradlew assembleRelease  # release APK, ~1.8 MB, needs keystore.properties
```

Needs **JDK 17+** and the Android SDK with platform 35. The Gradle wrapper is committed, so Gradle
itself does not need to be installed. Release signing reads an optional, gitignored
`keystore.properties`; without it `assembleRelease` produces an unsigned APK and everything else
works normally.

## Contributing

Pull requests are welcome, against `main`. Fork it, branch, run `./gradlew test`, open the PR — CI
runs the tests and builds an APK on every one.

The smallest useful contribution is a number: add one to [`spamlist.json`](spamlist.json) and every
install picks it up within a day. See [The shared spam list](#the-shared-spam-list) above.

Read [CONTRIBUTING.md](CONTRIBUTING.md) first if you are changing how calls are screened: the layer
order *is* the precedence rule, and two invariants can never break — emergency numbers always ring,
and an allow-listed number always rings.

Bugs and ideas go in [Issues](https://github.com/RajeshLakkam/call-blocker/issues). Please mask phone
numbers in anything you post (`+91 140xxxxxxx`), and never post someone else's number. Security
problems go to rajesh.lakkam327@gmail.com by email, not to a public issue.

## Layout

```
app/src/main/java/com/rlakkam/callblocker/
  domain/     pure Kotlin: models, repository interfaces, screening engine + layers
  data/       Room database, DataStore settings, repository implementations, sync worker
  service/    CallScreeningService, notifications, call-screening role helper
  ui/         Compose screens and ViewModels
  util/       number normalisation, emergency-number checks
app/src/test/  engine and normalisation unit tests
```

## Known gaps before this ships

- Community backend does not exist yet; the sync pulls from a local stub. The shared spam list is
  the shipping answer in the meantime: curated by pull request, with no server to run.
- The shared list has no moderation beyond pull request review, and no way to report a wrong entry
  from inside the app — you have to open an issue or a PR.
- Reports made in the app do not reach the repository on their own: the user has to send them, and
  a maintainer has to paste them into `new_additions.json`. A small submission endpoint would close
  that gap, at the cost of having a server to run and abuse to handle.
- No reputation vendor chosen — `ReputationProvider` is a no-op.
- Play Store needs a sensitive-permissions declaration for `READ_CONTACTS`.
- Export/import of lists (FR-9) is specified but not implemented.
- No instrumented tests for the screening service itself.

## Licence

[Apache License 2.0](LICENSE). Copyright 2026 rlakkam.
