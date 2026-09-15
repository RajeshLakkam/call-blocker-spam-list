# Call Blocker — Requirements Specification

**Project:** Call Blocker (Android)
**Owner:** rlakkam
**Status:** v0.0.2
**Date:** 2026-09-14

---

## 1. Purpose

An Android application that automatically rejects or silences incoming calls from
telemarketers, spam callers and promotional/robocall sources, while guaranteeing that
legitimate calls (contacts, allowlisted numbers, emergency callbacks) always ring.

## 2. Scope

### In scope (v1)
- Real-time screening of incoming calls before the phone rings.
- Layered decision engine: allowlist → blocklist → pattern rules → shared spam list → community
  reports → reputation lookup.
- User-managed block and allow lists.
- Configurable pattern rules (number series, prefixes, unknown/private/international callers).
- History of screened calls with the reason for each decision, and one-tap block, report or unblock.
- User reporting of a number as spam, feeding the community database.
- Offline-first: all core blocking works with no network.

### Out of scope (v1)
- iOS. (iOS cannot reject calls in real time; only a CallKit *Call Directory* extension with a
  pre-loaded static list is possible. Tracked as a future item.)
- SMS spam filtering.
- Call recording.
- Voicemail transcription / AI answering.

## 3. Platform constraints (important)

| Constraint | Impact on design |
|---|---|
| `CallScreeningService` requires **API 24+**, and silent rejection requires the app to hold `ROLE_CALL_SCREENING` (API 29+) | minSdk 24; on API 24–28 the app can only screen when it is the default dialer. The onboarding flow must request the role and degrade gracefully. |
| Only **one** app can hold the call-screening role at a time | Onboarding must detect that another blocker holds the role and tell the user. |
| Google Play policy: `READ_CALL_LOG` / `READ_CONTACTS` are sensitive permissions | Contacts are read only to build the allowlist, with an in-app rationale; call-log access is optional and only used for the "recently seen" enrichment. A Play Store declaration is required. |
| Screening callback has a **hard time budget** (a few hundred ms) | The engine must return a decision from local storage synchronously. Any network reputation lookup is fire-and-forget and used only to enrich *future* calls. |
| Emergency numbers must never be blocked | Hard-coded bypass, checked first, not user-overridable. |
| The platform binds `CallScreeningService` per call | Screening needs no resident process: the app does not have to be running, and a foreground service would add a permanent notification for nothing. |
| A force-stopped app is never started by the system | Unfixable in code. The app says so plainly rather than implying it can survive it. |
| Xiaomi, Oppo, Vivo, Realme, OnePlus, Huawei, Honor and Samsung kill swiped-away apps unless allow-listed, and expose no API for it | `BackgroundReliability` resolves the undocumented OEM autostart activities against the package manager and falls back to app settings; the step is confirmed by the user because it cannot be read back. |

## 4. Functional requirements

### FR-1 Call screening
- FR-1.1 The app SHALL evaluate every incoming call through the screening engine before the device rings.
- FR-1.2 The app SHALL support three outcomes: **Allow**, **Silence** (ring suppressed, call still shown), **Reject** (call disconnected).
- FR-1.3 Rejected calls SHALL optionally be kept out of the notification tray (user setting). They
  SHALL NOT be kept out of the system call log: `setSkipCallLog` is not honoured by the dialers
  this was tested against, which write their own log, and a switch that promises to hide a call and
  does not is worse than not offering one.
- FR-1.4 Every decision SHALL be recorded with: number, timestamp, outcome, matched rule, confidence.
- FR-1.5 Emergency numbers SHALL always be allowed, regardless of any rule.

### FR-2 Allowlist
- FR-2.1 Numbers in the user's contacts SHALL be allowed by default (setting, on by default).
- FR-2.2 The user SHALL be able to add/remove numbers on an explicit allowlist.
- FR-2.3 An allowlist match SHALL short-circuit all further evaluation.
- FR-2.4 **Whitelist-only mode.** The user SHALL be able to enable a mode in which the allow list is
  the only way through: every caller that is not allow-listed SHALL be blocked, regardless of block
  list, rules, community data or reputation.
  - FR-2.4.1 Contacts SHALL continue to ring in this mode only while FR-2.1 ("trust contacts") is on.
  - FR-2.4.2 Emergency numbers SHALL be unaffected (FR-1.5 outranks this mode).
  - FR-2.4.3 The mode SHALL honour the "silence instead of disconnect" setting, so a user can run it
    in a reversible way while they are still building the list.
  - FR-2.4.4 The mode SHALL be reachable from the allow-list screen itself, showing how many numbers
    are on the list, so it cannot be switched on without seeing what will still get through.
  - FR-2.4.5 While the mode is on, the app SHALL say so unmistakably on the home screen.

### FR-3 Blocklist
- FR-3.1 The user SHALL be able to block a number, a prefix, or a wildcard pattern.
- FR-3.2 The user SHALL be able to block a number directly from the call history screen.
- FR-3.3 Blocklist entries SHALL be editable, with an optional label and note.

### FR-4 Pattern rules
- FR-4.1 The app SHALL ship with built-in, toggleable rules:
  - Indian telemarketing series: `140xxxxxxx` (TRAI-assigned telemarketing series).
  - Robocall/promotional prefixes maintained in a bundled ruleset.
  - Numbers not in contacts (opt-in, "strict mode").
  - Private/withheld/unknown caller ID.
  - International calls when the user has no international contacts (opt-in).
  - Numbers shorter/longer than the valid national length.
- FR-4.2 The user SHALL be able to add custom regex/prefix rules.
- FR-4.3 Each rule SHALL be individually enabled/disabled and carry its own action (silence vs reject).

### FR-4b Shared spam list

- FR-4b.1 The app SHALL ship with a curated list of known nuisance numbers — `spamlist.json` — and
  SHALL refresh it from the published file once a day.
- FR-4b.1a The published file SHALL be served from a **public** repository separate from the app's.
  Installs fetch it unauthenticated, so a private repository would answer 404 to every one of them,
  and the only alternative — a token inside the APK — is readable by anyone who unzips it.
- FR-4b.1b The published URL SHALL address the default branch as `HEAD` rather than by name. Every
  APK shipped hardcodes it, so a branch rename must not be able to strand installs that can no
  longer be reached.
- FR-4b.2 A number on that list SHALL be silenced, never disconnected: the list is open to pull
  requests from strangers, so a wrong entry must cost a silent ring, not an unseen call.
- FR-4b.3 The name and details from the entry SHALL be shown wherever the app reports the call —
  the blocked-call notification and the history screen.
- FR-4b.4 The list SHALL be usable offline: the copy bundled in the APK seeds the database on
  first launch, and the last synced copy is kept when a refresh fails.
- FR-4b.5 The user SHALL be able to turn the list off, and to refresh it on demand.
- FR-4b.6 A number on the user allow list SHALL override the shared list.

### FR-4c Reporting from the app

- FR-4c.1 The Reported tab SHALL present the report form itself, with the numbers already reported
  listed below it. Reporting SHALL also be startable from a row of the block list and from any call
  in history that was allowed or silenced, with the number filled in.
- FR-4c.2 A report SHALL be refused unless it carries a number in international form and a caller
  name. A description and a category SHALL be offered and SHALL be optional: the report already
  blocks the caller on the reporter's own phone, so the cost of a thin report falls on the review,
  not on the reporter.
- FR-4c.3 An accepted report SHALL be confirmed in the app by a dialog saying the number was
  reported successfully, and by a notification saying it is blocked on this phone and sent for
  review.
- FR-4c.4 Reports SHALL be held on the device in `new_additions.json`, listed in the app under
  Reported, and removable by the user.
- FR-4c.5 A reported number SHALL be blocked on the reporter's own phone from the moment the report
  is accepted, SHALL appear under Reported and SHALL NOT be added to the block list. Removing the
  report SHALL be the one action that lets the caller through again.
- FR-4c.6 Once a number is blocked or reported from the history screen, the button that did so
  SHALL be disabled, and SHALL stay disabled across restarts for as long as the block or report
  stands. A call the app silenced SHALL offer block and report alongside allow: something other
  than the user decided to let it past quietly, and this is where that is overruled. A call that
  was rejected outright SHALL offer allow only.
- FR-4c.7 An accepted report SHALL be composed as an email to the address maintaining the shared
  list, with a fixed subject the mailbox can filter on, carrying the report as prose and as the
  JSON `new_additions.json` expects.
- FR-4c.8 The app SHALL NOT send that email by itself. It cannot hold a mailbox credential — every
  install would carry it — so the phone's own mail app composes it and the user presses send. A
  phone with no mail app SHALL still get the block, SHALL be told the report was not sent, and
  SHALL be able to send any report again from its row under Reported.
- FR-4c.9 Merging `new_additions.json` into `spamlist.json` SHALL validate every entry, skip
  numbers already listed, and empty the queue. Release builds SHALL perform the merge.

### FR-4d Surviving in the background

- FR-4d.1 Screening SHALL NOT depend on the app being open, in recents, or otherwise running. The
  platform binds the `CallScreeningService` per call; there is nothing to keep alive.
- FR-4d.2 The app SHALL request the caller ID & spam role and notification permission on first
  launch, before the user has to go looking for either.
- FR-4d.3 The app SHALL request only the caller ID & spam role, and optionally contacts and
  notifications. It SHALL NOT request a battery-optimisation exemption: screening does not depend
  on one, and asking for a permission a feature does not need is a cost paid by every user.
- FR-4d.4 On manufacturers known to force-stop swiped-away apps, the app SHALL offer that
  manufacturer's autostart screen, and SHALL surface background restriction where the system
  reports it. This advice SHALL be dismissible, and SHALL NOT appear on devices it does not apply
  to.
- FR-4d.5 Because Android exposes no API for an OEM autostart list, the advice SHALL be dismissed
  by the user rather than marked as verified.
- FR-4d.6 The status card SHALL report whether calls are being screened — the role alone — and
  SHALL never report protection that is not in place.
- FR-4d.7 The app SHALL NOT claim to survive Force stop. Nothing can, and saying otherwise costs
  the user the one action that would fix it — reopening the app.

### FR-5 Community reports
- FR-5.1 The user SHALL be able to report a number as spam with a category
  (telemarketing, promotional, fraud/scam, survey, debt collection, other).
- FR-5.2 The app SHALL maintain a locally cached table of community-reported numbers, synced periodically.
- FR-5.3 A number SHALL be treated as spam when its report count and score exceed a configurable threshold.
- FR-5.4 Sync SHALL be incremental, on Wi-Fi by default, and SHALL NOT upload the user's contacts or call log.

### FR-6 Reputation lookup (pluggable)
- FR-6.1 The app SHALL expose a provider interface for an external caller-reputation service.
- FR-6.2 Lookups SHALL be asynchronous and cached; a cache miss SHALL NOT delay the screening decision.
- FR-6.3 The feature SHALL be off by default and require explicit user opt-in (a lookup discloses the calling number to a third party).

### FR-7 History & transparency
- FR-7.1 A history screen SHALL list screened calls with outcome and reason.
- FR-7.2 The user SHALL be able to undo any decision (allowlist the number, delete the rule that matched).
- FR-7.3 History SHALL be clearable by the user, and SHALL be pruned automatically to the last 8
  days. The window SHALL cover the 7 days the home screen counts blocks over, and SHALL NOT be a
  stored preference: nothing in the UI sets it, and a stored copy outranks a later default.
- FR-7.4 Pruning SHALL run on its own daily schedule with no network or feature-flag conditions on
  it, and once at app start so a shortened window takes effect on the next launch.

### FR-8 Onboarding & permissions
- FR-8.1 On first run the app SHALL request the call-screening role and the required permissions with a plain-language rationale.
- FR-8.2 The home screen SHALL always show whether protection is actually active and, if not, exactly what is missing.

### FR-9 Backup
- FR-9.1 The user SHALL be able to export and import block/allow lists and custom rules as JSON.

## 5. Non-functional requirements

- **NFR-1 Latency:** a screening decision SHALL be returned in < 100 ms at p95, from local storage only.
- **NFR-2 Privacy:** contacts, call log and history SHALL NOT leave the device. Only a salted hash of a
  number is sent when the user explicitly reports it. No analytics on call content.
- **NFR-3 Reliability:** engine failure SHALL fail *open* (allow the call) — never silently swallow calls.
- **NFR-4 Battery:** no foreground service, no polling; sync via WorkManager with constraints.
- **NFR-5 Storage:** blocklist lookups indexed; O(log n) or better for 100k community entries.
  Screened-call history SHALL be bounded by the retention window rather than growing with the age
  of the install.
- **NFR-6 Accessibility:** TalkBack labels on all controls, minimum 48dp touch targets, dynamic type.
- **NFR-7 Testability:** the decision engine SHALL be a pure Kotlin module, unit-testable with no Android dependencies.

## 6. Decision precedence (normative)

Evaluated top-down; the first match wins.

| # | Layer | Outcome |
|---|---|---|
| 0 | Emergency number | Allow (final, not user-overridable) |
| 1 | User allow list | Allow (final) |
| 2 | Contact, when "trust contacts" is on | Allow (final) |
| 3 | Whitelist-only mode, when on | Reject everything that got this far |
| 4 | User block list (exact / prefix / regex) | Reject (always a hard reject) |
| 5 | Numbers this user reported as spam | Reject (always a hard reject) |
| 6 | Hidden or withheld caller ID | Per setting |
| 7 | Pattern rules, built-in and custom | Per rule |
| 8 | International call | Per setting |
| 9 | Shared spam list (spamlist.json) | Silence only |
| 10 | Community reports at or above threshold | Reject, or silence |
| 11 | Cached reputation score above threshold | Silence only |
| 12 | Strict mode: not a known contact | Silence |
| — | Default | Allow |

Layers 3, 6, 8 and 10 honour the "silence instead of disconnect" setting. Layers 9 and 11 only ever
silence, whatever the setting says, because neither the shared list nor a reputation vendor is the
user. Layers 4 and 5 deliberately do
not: both are a direct instruction from the user about a caller they named.

Layer 5 is why a reported number never joins the block list. The report is the block, so the number
is listed once — under Reported — and removing the report is the single action that lets the caller
back through.

Whitelist-only sits at layer 3 — directly under the two "always allow" layers and above everything
else — because that position *is* the promise the feature makes: if a caller is not allow-listed
(or a trusted contact), nothing further is consulted.

This table is implemented as the layer list in `AppContainer.screeningEngine` — the two must be
changed together.

## 7. Acceptance criteria (v1)

- A call from a number on the blocklist never rings the device, and appears in history marked `BLOCKLIST`.
- A call from a contact rings normally even when it matches a pattern rule.
- With protection granted and airplane-mode data off, blocking still works.
- An emergency number rings even when every rule is enabled and the number is blocklisted.
- Revoking the call-screening role shows an unmistakable "not protected" state on the home screen.
- With whitelist-only mode on: an allow-listed number rings; an unknown number is blocked and
  appears in history as `WHITELIST-ONLY`; an emergency number still rings; turning "trust contacts"
  off blocks contacts too.

## 8. Open questions

1. Community backend — build in-house, or start local-only and add the server in v1.1?
2. Reputation provider — which vendor (and is per-lookup cost acceptable)?
3. Target market: India-first (TRAI 140-series, DND registry) or multi-region from day one?
4. Distribution: Play Store (needs the sensitive-permissions declaration) or enterprise/internal?
