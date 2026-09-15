# Call Blocker — Architecture

## 1. Stack

| Concern | Choice |
|---|---|
| Language | Kotlin 2.0 |
| UI | Jetpack Compose (Material 3) |
| Min / Target SDK | 24 / 35 |
| Persistence | Room (SQLite) |
| Settings | DataStore (Preferences) |
| Async | Coroutines + Flow |
| Background sync | WorkManager |
| DI | Hand-rolled `AppContainer` (no Hilt — keeps the build simple; swap in Hilt later if the module count grows) |
| Tests | JUnit4 + kotlinx-coroutines-test; engine is pure Kotlin |

## 2. Module / package layout

```
com.rlakkam.callblocker
├── CallBlockerApp.kt          Application; owns AppContainer
├── di/AppContainer.kt         manual DI graph
├── domain/
│   ├── model/                 CallDecision, ScreeningOutcome, MatchReason, Rule types
│   ├── engine/                ScreeningEngine + the individual rule layers  ← pure Kotlin, no Android
│   └── repo/                  repository interfaces the engine depends on
├── data/
│   ├── db/                    Room entities, DAOs, AppDatabase
│   ├── prefs/                 SettingsRepository (DataStore)
│   ├── local/                 new_additions.json store — the user's own reports, held on the device
│   ├── remote/                CommunityApi (interface + local stub), shared spam list + sync workers
│   └── repo/                  repository implementations
├── service/
│   ├── CallBlockerScreeningService.kt   CallScreeningService entry point
│   ├── BackgroundReliability.kt         battery, background-restriction and OEM autostart checks
│   ├── ReportNotifier.kt                confirms a spam report back to the user
│   └── ScreeningNotifier.kt             post-block notification
├── ui/                        Compose screens + ViewModels
└── util/PhoneNumbers.kt       normalisation (E.164-ish), emergency check
```

The dependency direction is strictly `ui → domain ← data`. The engine knows only
repository *interfaces*, so it can be tested with fakes and later reused on another surface
(a server-side screening service, for example).

## 3. Call flow

```
Incoming call
      │
      ▼
CallScreeningService.onScreenCall(Call.Details)
      │  normalise number (util/PhoneNumbers)
      ▼
ScreeningEngine.decide(NumberInfo)            ← must return fast, local data only
      │  layer 0..11, first match wins (see REQUIREMENTS §6)
      ▼
CallDecision(outcome, reason, ruleId, confidence)
      │
      ├── ALLOW    → respondToCall(default response)
      ├── SILENCE  → setSilenceCall(true)
      └── REJECT   → setDisallowCall(true) + setRejectCall(true)
      │                 + setSkipNotification per settings
      ▼
ScreenedCallEntity written to Room (off the critical path, on IO dispatcher)
      │
      ▼
Notification "Blocked call from X — Unblock / Report" (if enabled)
```

Anything thrown inside the engine is caught at the service boundary and downgraded to
**Allow** (NFR-3: fail open). A blocked legitimate call is a far worse failure than a spam call
that gets through.

## 4. The screening engine

`ScreeningEngine` is an ordered list of `ScreeningLayer` objects:

```kotlin
interface ScreeningLayer {
    val id: String
    suspend fun evaluate(ctx: ScreeningContext): CallDecision?   // null = no opinion, fall through
}
```

Layers in order: `EmergencyLayer`, `AllowlistLayer`, `ContactsLayer`, `AllowlistOnlyLayer`,
`BlocklistLayer`, `ReportedNumberLayer`, `HiddenNumberLayer`, `PatternRuleLayer`,
`InternationalLayer`, `SpamListLayer`, `CommunityReportLayer`, `ReputationLayer`,
`StrictModeLayer`. A call that reaches the end of the list is allowed.

`ReportedNumberLayer` reads the user's own reports and is why reporting a number blocks it without
adding it to the block list: the report *is* the block. That keeps the number in one place — the
Reported tab — and makes removing the report the one action that undoes it, rather than leaving two
entries that have to be deleted in the right order.

`AllowlistOnlyLayer` is the whole of whitelist-only mode: three lines that reject anything still
undecided at that point. Because the layers above it are the ones that say "always allow", putting
the gate there gives the feature its guarantee without touching any other layer — which is the
argument for the pipeline shape in the first place.

Adding a detection strategy = adding one layer and one line in the list. No other code changes.

`ScreeningContext` carries the normalised number, the raw number, presentation
(allowed / restricted / payphone / unknown), and a snapshot of settings — so the engine never
does I/O it cannot afford.

## 5. Data model

| Table | Key columns |
|---|---|
| `blocked_numbers` | `normalized` (PK), `matchType` (EXACT/PREFIX/REGEX), `pattern`, `label`, `createdAt` |
| `allowed_numbers` | `normalized` (PK), `label`, `createdAt` |
| `pattern_rules` | `id`, `name`, `pattern`, `matchType`, `action`, `enabled`, `builtIn` |
| `community_reports` | `normalized` (PK), `reportCount`, `score`, `category`, `updatedAt` |
| `spam_list` | `normalized` (PK), `name`, `details`, `category`, `updatedAt` — replaced wholesale by each sync |
| `reputation_cache` | `normalized` (PK), `score`, `label`, `fetchedAt` |
| `screened_calls` | `id`, `number`, `outcome`, `reasonLayer`, `ruleId`, `timestamp` |

All number columns store the normalised form; indices on `normalized`.

## 6. Privacy posture

- Contacts are read on demand through `ContactsContract.PhoneLookup`, never copied into the database.
- The call history table stores only numbers the app itself screened, and only for the last
  `HistoryRetention.DAYS` — eight. `HistoryPruneWorker` deletes past that daily, and once at app
  start. Deleting rows frees pages inside the database file rather than shrinking it, so the file
  settles at the size of a week's screening instead of growing for the life of the install.
- Reports made in the app are written to `new_additions.json` in app-private storage. The app then
  hands the report to the phone's own mail app, addressed to the list's maintainer, and the user
  presses send — it never sends anything itself. Shipping a mailbox or repository credential in the
  APK would hand it to anyone who unzipped it, and a report that leaves from the user's own address
  is one a maintainer can reply to. `mergeNewAdditions` folds the queue into `spamlist.json` at
  release-build time.
- The shared spam list is a plain HTTPS GET of a public file — `spamlist.json` in the separate
  public repository `call-blocker-spam-list`, addressed at `HEAD` so a branch rename cannot strand
  the installs that hardcode the URL — refreshed daily by `SpamListSyncWorker` and conditional on an ETag. Nothing is sent; nothing identifies the
  install. It is replaced wholesale on each sync so an entry deleted upstream stops being silenced.
- The community sync pulls a delta list; the only upload is an explicit user report, containing a
  salted SHA-256 of the number plus a category — no device or account identifier.
- Reputation lookups are opt-in, with the third-party disclosure shown before the toggle flips.

## 7. Build & run

1. Open the folder in Android Studio (Ladybug or newer).
2. Let it generate the Gradle wrapper if prompted (`gradle wrapper` was not run here — no Android SDK on the sync host).
3. Run on a device with API 29+ and grant the "Caller ID & spam app" role when asked.
4. `./gradlew test` runs the engine unit tests — they need no device.

## 8. Roadmap after v1

- v1.1 Community backend (Ktor + Postgres), signed delta feed, abuse-resistant reporting.
- v1.2 On-device heuristic scoring (call frequency, short-duration patterns, number churn).
- v1.3 iOS Call Directory extension sharing the same blocklist export format.
- v1.4 Enterprise/MDM profile: org-managed blocklist pushed to devices.
