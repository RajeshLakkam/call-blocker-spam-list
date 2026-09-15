# How to use Call Blocker

A guide for two audiences: the person running the app on a phone, and the developer building it.
Start at the section that applies to you.

---

# Part 1 — Using the app

## What it does

Every incoming call is checked before your phone rings. Depending on what matches, the call either
rings normally, rings silently (you still see it), or is disconnected before you hear anything.

Emergency numbers always ring. No setting, rule or list can change that.

## First-time setup

1. Open **Call Blocker**. It asks for notifications, then for the caller ID & spam role, straight
   away — those two are what everything else depends on.
2. Confirm **Set as caller ID & spam app** in the system dialog. Android only lets one app hold
   this role at a time — if another blocker has it, you will be asked to replace it.
3. Work down the setup card on the home screen until it disappears. It lists everything still
   outstanding, explains what each one costs you, and takes you to the right settings screen.
   [Keeping it working when the app is closed](#keeping-it-working-when-the-app-is-closed) explains
   the background ones.
4. Tap **Allow contacts** so calls from people you know always ring. Contacts are read on the phone
   only; nothing is uploaded.

The top card tells you the truth about your state at any moment. If it says **Not protected**,
calls are *not* being screened — whatever else the app shows.

## Keeping it working when the app is closed

### Does closing the app stop it screening calls?

**No.** Android itself hands every incoming call to whichever app holds the caller ID & spam role,
and starts that app for the purpose. Call Blocker does not need to be open, or in recents, or
running in the background. Swiping it away changes nothing.

There is one exception, and it is worth knowing: **Force stop**, in Android's own app settings.
After a force stop, Android will not start the app for anything — including a call — until you open
it yourself again. That is deliberate on Android's part, no app can work around it, and any app
that says it can is not telling you the truth. If you ever force stop Call Blocker, open it once
afterwards.

### Then why does the app mention batteries at all?

It does not ask for a battery permission — screening does not need one. But some phone
manufacturers do not follow the rule above. Xiaomi, Redmi, Poco, Oppo, Realme,
OnePlus, Vivo, iQOO, Huawei, Honor and Samsung all ship extra battery software that can treat
"swiped out of recents" as closer to a force stop — unless the app is on an allow list that only
you can put it on.

The app asks you for three things, and only three:

| Asked for | What happens without it |
|---|---|
| **Caller ID & spam app** | Nothing is screened at all. This is the only one that matters. |
| **Contacts** | Calls from people you know are screened like strangers' calls. |
| **Notifications** | You are not told when a call is blocked, and lose the one-tap undo. |

On the manufacturers listed above — and only there — the home screen also shows one dismissible
card headed *If calls stop being screened on this phone*, pointing at the autostart and battery
screens. It is advice, not a requirement: tap **Got it** and it does not come back.

The card takes you straight to your manufacturer's own settings page where it can find it, and to
the app's settings page otherwise — Android gives no standard way to reach those screens, and no
way to check afterwards whether you switched anything on. That is why the card is dismissed rather
than ticked off: there is nothing for the app to verify.

### Manufacturer-specific advice

- **Xiaomi, Redmi, Poco** — turn on Autostart, then in Recents press and hold the app and tap the
  padlock so clearing recents leaves it alone.
- **Oppo, Realme, OnePlus** — allow Auto-startup, and set battery usage to "Allow background
  activity" or "Don't optimise".
- **Vivo, iQOO** — turn on "Auto-start" and "Allow high background power consumption".
- **Huawei, Honor** — Battery → App launch → Manage manually, then enable Auto-launch and Run in
  background.
- **Samsung** — Battery → Background usage limits: make sure Call Blocker is not in Sleeping apps
  or Deep sleeping apps.

The home screen shows the advice for your phone automatically.

### What the status card means

| It says | What that means |
|---|---|
| **Protection is on**, with a green tick | Calls are being screened before your phone rings. |
| **Not protected** | Calls are not being screened. Another app holds the role, or it was never granted. |

The card never says you are protected when you are not — it re-checks every time you open the app,
because the role and those permissions can be taken away from outside the app at any moment.

## The four tabs

| Tab | What it is for |
|---|---|
| **Protection** | Whether screening is active, what is missing, how many calls were blocked this week, and the last few blocks with an **Allow** button on each. |
| **History** | The last 8 days of screened calls, with the reason for each. Block, allow or report any of them. |
| **Lists** | Your block list, your allow list, and the numbers you have reported — which are blocked too. |
| **Settings** | Every switch, plus the rules. |

## Blocking a number

- **From History** — find the call, tap **Block**. Fastest way, and the usual one.
- **From Lists → Blocked** — type the number and tap **Block this number**.
- **A whole range** — type the first digits followed by `*`, for example `140*`, to block every
  number that starts with them. This is how you stop a range of telemarketing numbers that keeps
  changing its last four digits.

Numbers on your block list are always disconnected, never merely silenced.

## Allowing a number

Lists → **Allowed** → type the number → **Always allow**. An allow-listed number rings even if it
matches every rule you have switched on, and even if others have reported it as spam.

You can also tap **Allow** on any blocked call in History or on the blocked-call notification,
which removes it from the block list and adds it to the allow list in one step.

---

## Whitelist-only mode

**Only the numbers you have allowed can reach you. Everything else is blocked.**

Use it when you want silence with exceptions: a deadline, a hospital stay, a trip, or a phone that
has simply been found by too many call centres.

### Turning it on

1. Go to **Lists → Allowed**.
2. Add every number you still want to hear from. Take your time — this is the whole point.
3. Turn on **Only allow these numbers** at the top of the same screen.

The switch lives next to the list on purpose, with the count in view, so the mode cannot be turned
on without seeing what will still get through. The same switch is in **Settings → Whitelist-only
mode**.

### What still gets through

| Caller | Gets through? |
|---|---|
| On your allow list | Yes |
| Emergency numbers | Yes, always |
| In your contacts | Only while **Trust my contacts** is on (Settings → Who always gets through) |
| Everyone else — unknown numbers, new numbers, people who have never called before | No |

If you want a phone that rings **only** for your explicit list and not for your whole address book,
turn **Trust my contacts** off as well.

### While it is on

- The home screen shows a card: *Whitelist-only mode is on*, with the number of allowed callers.
  You will not forget it is running.
- Blocked calls still appear in **History** with the reason *Whitelist-only mode is on*, so you can
  see exactly who tried and tap **Allow** to let them through in future.
- Nothing else is consulted — your rules, the community list and reputation scores are all bypassed
  because the mode already answers the question.

### A gentler version

In **Settings → How calls are blocked**, turn on **Silence instead of disconnect**. Non-allowed
callers then ring silently and show up as missed calls instead of being disconnected. Good for the
first day or two while you find out who you forgot to add. Your block list still disconnects.

### Turning it off

Flip the same switch. Screening returns to your block list and rules; your allow list stays exactly
as you left it.

---

## The shared spam list

The app ships with a list of known nuisance numbers — call centres, robocallers, scam lines — that
anyone can add to. It lives in the project repository as a file called `spamlist.json`, and every
installed copy of the app reads the same list.

**A call from a number on the list is silenced.** Your phone does not ring; the call appears in
your call log as a missed call, and in the app's **History** tab with the caller's name and a line
saying what the call actually is:

> **Loan offer robocall** · 14000 00001
> Automated call offering a pre-approved personal loan, transfers to an agent asking for your date
> of birth.

It is silenced rather than disconnected on purpose. Strangers can add to this list, so a wrong
entry should cost you a call you can still see and return — not one you never knew about. If the
list gets one wrong, tap **Allow** on it in History: your own allow list beats the shared list from
then on. Emergency numbers are never affected.

### When it updates

Once a day, at roughly the time of day you first opened the app. You can also update it whenever
you like: **Settings → Shared spam list → Update now**. The same screen shows how many numbers are
on the list and when it last refreshed.

A fresh install is protected immediately — the list is built into the app — and if an update fails
because the phone is offline, the copy already on the phone keeps working.

To turn the whole thing off: **Settings → Shared spam list → Use the shared spam list**.

### Reporting a number yourself

Blocking a number stops it calling **you**. Reporting it does that *and* puts it forward for the
shared list, so it stops calling everyone.

You can report from either screen:

- **Lists → Reported** — the form is the tab. Fill it in and tap **Report**; everything you have
  reported is listed underneath it.
- **History** — tap **Report spam** under any call that got through, or any call that was silenced.
  The number is filled in for you.
- **Lists → Blocked** — tap **Report** next to a number you have already blocked, which also fills
  the number in.

Two things are required, and the rest are worth filling in but optional:

| Field | What to write | |
|---|---|---|
| **Mobile number** | With the country code, e.g. `+91 140 000 0001`. Reporting from History fills this in for you. | Required |
| **Who does the caller claim to be?** | `Loan offer robocall`, `Fake bank KYC call`. This is the name strangers will see on their screen. | Required |
| **What happens on the call?** | Enough for someone else to recognise it: `Recorded pitch for a pre-approved loan, then transfers to an agent asking for your date of birth.` | Optional |
| **Category** | Telemarketing, promotional, fraud, survey, debt collection or other. Tap the chosen one again to clear it. | Optional |

Tap **Report**, and three things happen at once:

1. The number is blocked on your phone straight away. It appears under **Lists → Reported** — not
   under Blocked, because the report is what is doing the blocking. Delete the report and the
   number can call you again.
2. The app says **reported successfully**, and posts a notification saying the same.
3. Your mail app opens with the report filled in, addressed to the person who maintains the list,
   under the subject `call_blocker_reported_spam`. **Press send.** Until you do, the number is
   blocked on your phone but nobody else has heard about it.

The **Block** and **Report spam** buttons for that number then stay greyed out — including after
you close and reopen the app — so there is no guessing whether the first tap worked.

### If your mail app did not open

The app never sends the mail itself: that would mean every copy of the app carrying a mailbox
password, which would let anyone who downloaded it send as that address. So it hands the report to
whatever mail app you have, and you press send.

If you have no mail app set up, the app tells you so — the number is still blocked, and the report
is still saved. Set up mail, then go to **Lists → Reported** and tap **Email** next to the report
to send it. You can delete a report from the same screen if you change your mind, which also
unblocks the number.

### Before you report

- **Only numbers that call strangers in bulk** — call centres, robocallers, scam and phishing lines.
- **Never a number belonging to someone you know.** A reported number is silenced on thousands of
  phones. This is not a way to deal with a person you would rather not hear from, and reports that
  look personal are rejected.
- Report what you were actually called by. If you only want to stop a range of numbers calling
  *you*, block a prefix like `140*` instead — that is a rule on your phone and affects nobody else.

### Adding a number to it directly

If you are comfortable with GitHub you can skip the report queue: open `spamlist.json` in the
project repository, add the number with a name and a description, and open a pull request. It
reaches everyone else's phone within about a day of being merged.

The rules and the format are in [CONTRIBUTING.md](../CONTRIBUTING.md#adding-a-number-to-the-shared-spam-list).
One of them matters more than the rest: **never add a real person's number**. This list silences
calls on thousands of phones, and it is not a way to deal with someone you would rather not hear
from.

## The other settings

**Who always gets through**

- *Trust my contacts* — people in your address book always ring. On by default.

**What gets blocked**

- *Hidden and withheld numbers* — callers who hide their caller ID.
- *International calls* — anything outside your country code.
- *Strict mode* — silences every caller who is not a contact. Milder than whitelist-only: it
  silences rather than disconnects, and it trusts your whole address book rather than a list you
  curate.
- *Use the shared spam list* — silence numbers on the community-maintained list described above,
  which you can contribute to by reporting a number as spam. On by default.
- *Community spam list* — numbers other users have reported.
- *Caller reputation lookup* — off by default, because a lookup sends the calling number to a
  third-party service.

**How calls are blocked**

- *Silence instead of disconnect* — applies to the rule-based layers; your block list always
  disconnects.
- *Notify me when a call is blocked* — with a one-tap undo.

**Rules** — the shipped patterns, each with its own switch. `140` (India's telemarketing series) is
on by default. Turn any of them off if they catch calls you want.

**About** — at the very bottom: what the app is, the version it is, and why it exists. It ends with
a **Support me with a coffee** button, which shows a UPI QR code and the ID to pay to. It is a tip
jar and nothing else — every feature works the same whether you use it or not, and the app never
asks again.

## If a call you wanted was blocked

Open **History**, find it, tap **Allow**. It goes on the allow list, and from then on it rings
whatever else matches. If it keeps happening to a whole range of numbers, the reason line in
History names the rule that caught it — switch that rule off in Settings.

## What the app keeps

Screened calls are kept for 8 days and then deleted, so History is a rolling week and a bit rather
than a record that grows for as long as the app is installed — and the app's storage settles at
about that size instead of climbing. Nothing else expires: your block list, allow list and reported
numbers stay until you remove them. **Clear** on the History tab empties history immediately if you
would rather not wait.

## What leaves your phone

Your contacts, your call history and your lists stay on the device. The only thing uploaded is a
number you explicitly report as spam, and it is sent as a salted hash with a category — no device
or account identifier attached. Reputation lookups are the one exception, and they are off unless
you turn them on.

The daily spam-list update is a download, not an upload: the app fetches one public file over
HTTPS and sends nothing about you or your calls.

Numbers you report are the one thing meant to leave the phone, and even they do not go on their
own: the app writes the report into a mail your own mail app opens, and nothing is sent until you
press send. What you wrote becomes public if the number is added to the list, so keep it about the
call. GitHub, which serves the file, sees that some IP address asked for it — the same thing it
would see if you opened the file in a browser.

---

# Part 2 — Building and running it

## Prerequisites

- **Android Studio** Ladybug or newer
- **JDK 17 or newer** — JDK 21 is what this project is built and tested with
- **Android SDK** with platform 35 and build-tools 35 (Android Studio's SDK Manager installs both)
- A device or emulator on **API 29+** for the full feature set. The app installs from API 24, but
  below 29 Android cannot grant the call-screening role and cannot silence calls.

The Gradle wrapper is committed, so you do not need Gradle installed — `gradlew` downloads the
right version (9.6) on first run.

## Getting the code

```bash
git clone https://github.com/RajeshLakkam/call-blocker.git
cd call-blocker
```

Android Studio writes `local.properties` pointing at your SDK on first sync. If you build from the
command line without opening Android Studio first, create it yourself:

```
sdk.dir=C:\\Users\\<you>\\AppData\\Local\\Android\\Sdk   # Windows
sdk.dir=/home/<you>/Android/Sdk                          # Linux
sdk.dir=/Users/<you>/Library/Android/sdk                 # macOS
```

That file is gitignored — it is specific to your machine and must never be committed.

## Steps

1. Open this folder in Android Studio and let the first Gradle sync finish.
2. Run the `app` configuration.
3. On the device, complete the three onboarding steps in Part 1.
4. Run the tests with `./gradlew test`. They cover the decision engine and number normalisation and
   need no device.

## Building an APK

### In Android Studio (simplest)

1. Open this folder and let the first Gradle sync finish.
2. **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
3. The notification that appears has a **locate** link, or find the file at
   `app\build\outputs\apk\debug\app-debug.apk`.

### From the command line

```
gradlew.bat assembleDebug      # Windows
./gradlew assembleDebug        # macOS / Linux
```

| Build | Command | Output | Size |
|---|---|---|---|
| Debug | `assembleDebug` | `app/build/outputs/apk/debug/app-debug.apk` | ~18 MB |
| Release | `assembleRelease` | `app/build/outputs/apk/release/app-release.apk` | ~1.8 MB |

The debug APK is signed with the automatic debug key, installs on any device, and is the right
choice for trying the app or testing a change. The release APK is minified by R8 — much smaller,
and what you would actually distribute — but it needs a signing key (see below).

You can also grab a debug APK without building anything: every push and pull request runs the
**Build** workflow, and the resulting `app-debug-apk` is attached to that run under
[Actions](https://github.com/RajeshLakkam/call-blocker/actions). Open a run, scroll to **Artifacts**,
download, unzip.

---

## Installing it on your phone

Android will not install an app from outside the Play Store until you allow it, and it will warn
you. That is expected for a self-built APK. Two ways in:

### Option A — copy the file to the phone (no cable, no tools)

1. Get `app-debug.apk` onto the phone: USB file transfer, Google Drive, email it to yourself, or
   any messaging app that allows file attachments.
2. On the phone, open **Files** (or your browser's Downloads) and tap the APK.
3. Android asks to allow installs from that app: **Settings → Install unknown apps → Files →
   Allow from this source**. You do this once per app you install from.
4. Tap **Install**. If Play Protect warns about an unrecognised app, choose **Install anyway** — it
   says that about every app not distributed through the Play Store.
5. Open **Call Blocker** and follow first-time setup in Part 1. Screening does nothing until you
   tap **Set as caller ID & spam app**.

### Option B — over USB with adb (best while developing)

1. On the phone: **Settings → About phone → tap Build number seven times** to unlock Developer
   options, then **Settings → System → Developer options → USB debugging → on**.
2. Plug the phone into the computer and accept the **Allow USB debugging** prompt on the phone.
3. Check the phone is visible, then install:

```
adb devices                                                  # should list your device
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

`adb` lives in `<Android SDK>/platform-tools/`. On Windows that is usually
`C:\Users\<you>\AppData\Local\Android\Sdk\platform-tools\adb.exe`. Add that folder to your PATH, or
call it by its full path.

`-r` reinstalls over an existing copy, keeping its data. Use `adb install -r -d ...` if you are
going back to a lower version code.

Running straight from Android Studio (the green Run button) does all of this for you and is the
fastest loop while you are changing code.

### Things that go wrong

| What you see | What it means |
|---|---|
| `INSTALL_FAILED_UPDATE_INCOMPATIBLE` | A copy signed with a different key is already installed — a debug build over a release build, or the other way round. Uninstall the old one first: `adb uninstall com.rlakkam.callblocker`. |
| `INSTALL_FAILED_ALREADY_EXISTS` | You left off `-r`. |
| `adb devices` shows nothing | USB debugging is off, the cable is charge-only, or the driver is missing. Try another cable first — it is usually the cable. |
| `adb devices` shows `unauthorized` | The **Allow USB debugging** prompt on the phone has not been accepted. Unplug, replug, look at the phone. |
| **App not installed**, no reason given | Usually a partially copied APK, or an existing install signed with a different key. |
| Installed, but calls are not screened | The app does not hold the caller ID and spam role. The Protection tab's top card tells you the truth — tap **Set as caller ID & spam app**. Below Android 10 (API 29) the role does not exist and screening cannot work at all. |

### Uninstalling

Long-press the app icon and choose **Uninstall**, or run `adb uninstall com.rlakkam.callblocker`.
Your lists and history live in the app's own storage and go with it.

---

## A release APK (for sharing beyond your own device)

Release builds are signed with a key you create and keep. **The same key must sign every future
update** — Android refuses an update signed by a different one, and there is no recovery if you
lose it. Back up the keystore file and its password somewhere durable.

This project reads signing details from an optional `keystore.properties` in the project root. It
is gitignored and absent from a fresh clone: without it the build still works, it just produces an
unsigned APK that Android will not install.

1. Create a keystore, once:

```
keytool -genkeypair -v -keystore keystore/release.jks -storetype PKCS12 \
  -keyalg RSA -keysize 2048 -validity 10000 -alias callblocker
```

2. Create `keystore.properties` in the project root:

```
storeFile=keystore/release.jks
storePassword=<the password you chose>
keyAlias=callblocker
keyPassword=<the same password>
```

3. Build it, and check the signature took:

```
./gradlew assembleRelease
<Android SDK>/build-tools/35.0.0/apksigner verify --print-certs -v app/build/outputs/apk/release/app-release.apk
```

Android Studio's **Build → Generate Signed App Bundle / APK** does the same through a wizard, and
can create the keystore for you.

For the Play Store, run `./gradlew bundleRelease` instead — an `.aab` at
`app/build/outputs/bundle/release/app-release.aab`, signed by the same config.

Release builds have `isMinifyEnabled = true`, so if something works in debug and breaks in release,
suspect R8 stripping something and check `app/proguard-rules.pro`.

## Testing whitelist-only without waiting for a spam call

The engine is pure Kotlin, so the quickest check is a unit test — see
`ScreeningEngineTest.kt`, the `whitelist-only …` cases. On a device: add one number to the allow
list, turn the mode on, and call the phone from any other number. It should not ring, and the call
should appear in History with the reason *Whitelist-only mode is on*.

## Where things live

```
app/src/main/java/com/rlakkam/callblocker/
  domain/     models, repository interfaces, the screening engine and its layers (pure Kotlin)
  data/       Room database, settings, repository implementations, sync worker
  service/    CallScreeningService, notifications, the call-screening role helper
  ui/         Compose screens and ViewModels
  util/       number normalisation, emergency-number checks
```

## Changing what gets blocked

Add a `ScreeningLayer` in `domain/engine/Layers.kt` and one line in `AppContainer.screeningEngine`.
Position in that list is precedence — it is the table in REQUIREMENTS.md section 6, and the two
must be changed together.

For a new pattern rather than a new kind of check, add a `PatternRule` to
`domain/engine/BuiltInRules.kt`; it is seeded on first run and appears in Settings with its own
switch.

## See also

- [REQUIREMENTS.md](REQUIREMENTS.md) — what the app must do, and the platform constraints behind it
- [ARCHITECTURE.md](ARCHITECTURE.md) — how it is put together and why
