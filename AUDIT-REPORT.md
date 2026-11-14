# LumaOS Pro — Privacy Policy Audit & Publisher Pack

**Prepared:** 14 November 2026
**Policy page updated:** [`index.html`](index.html) (effective 14 November 2026)
**Subject app:** LumaOS Pro `com.lumaos.pro.launcher`
**Source audited:** `rolexsir-org/lumaospro` @ `316f901` (default branch `main`, fetched 14 Nov 2026)
**Auditor note:** every technical statement in the published policy was read out of source, not out of documentation. Where documentation and source disagreed, **source won** and the disagreement is listed in §4.

> **Do not publish until the three BLOCKING items in §5 are resolved.** The page itself is
> complete and contains no unfinished placeholders, but it depends on facts only you can
> confirm (release version, Play listing state, in-app disclosure correction).

---

## 1. What was done

1. Cloned the Android repository **read-only** into a scratch directory outside this repo,
   added it to `.git/info/exclude`, and deleted it after the audit. No file in
   `rolexsir-org/lumaospro` was modified, created or pushed.
2. Read the merged manifest, the Gradle build, the dependency catalog, both Android backup
   rule files, the accessibility service config, both widget provider configs, the whole
   `vault/` package, `context/`, `notifications/`, `widgets/music/`, `core/`, `wallpaper/`,
   `diagnostics/`, `optimizer/`, `controls/`, `onboarding/`, `settings/` and the user-visible
   strings that carry the in-app disclosures.
3. Cross-checked three claims that could not be settled inside the app repo and that a
   generic policy would get wrong:
   * **Where DataStore actually writes.** Verified `androidx.datastore`'s
     `Context.dataStoreFile()` resolves to `File(applicationContext.filesDir, "datastore/…")`
     (androidx `datastore/datastore/src/androidMain/.../DataStoreFile.android.kt`), so the
     `<include domain="file" path="datastore/"/>` rule does match the real settings file.
   * **What a `<include>` can and cannot reach.** Verified `FullBackup.BackupScheme.getTokenForXmlDomain()`
     (AOSP `core/java/android/app/backup/FullBackup.java`) maps `file` → `filesDir`,
     `database` → `databases/`, `sharedpref` → `shared_prefs/`, and that `no_backup/` is a
     separate tree no rule here names. This is what makes "the Vault is never backed up"
     a structural fact rather than a promise — and what exposes finding **F-2**.
   * **Library supply chain.** The runtime dependency set is AndroidX + Kotlin coroutines
     only (`gradle/libs.versions.toml`). No Firebase, Play Services, ad, analytics, crash,
     HTTP or crypto library is present, which is what allows the policy to state "no data is
     uploaded" instead of "no data is uploaded for stated purposes".
4. Validated the produced HTML mechanically (§6).

---

## 2. Source files inspected

| Area | Files |
|---|---|
| Manifest / build | `app/src/main/AndroidManifest.xml`; `app/build.gradle.kts`; `build.gradle.kts`; `settings.gradle.kts`; `gradle/libs.versions.toml`; `gradle.properties` |
| Vault crypto | `vault/core/VaultCrypto.kt`; `VaultKeyring.kt` (incl. `VaultPinPolicy`); `VaultStreamAead.kt`; `VaultRecoveryCode.kt`; `VaultEncoding` (in `VaultCrypto.kt`) |
| Vault storage | `vault/core/VaultPaths.kt` (`VaultFiles.shred/writeAtomically/deleteRecursively`); `VaultObjectStore.kt`; `VaultMetadataStore.kt`; `VaultModels.kt` (`VaultAutoLock`, `VaultBackupSettings`, `MAX_EVENTS`) |
| Vault platform | `vault/platform/VaultManager.kt`; `VaultDeviceKeyGuard.kt`; `VaultBiometricGate.kt`; `VaultGuardSigner.kt`; `VaultConcealmentStore.kt`; `VaultLifecycleGuard.kt`; `VaultMedia.kt` |
| Vault backup | `vault/core/VaultBackup.kt`; `VaultManager.backupTo/restoreFrom/verifyContainer/exportTo`; `vault/ui/VaultActions.kt` |
| Vault UI | `VaultScreenActivity.kt`; `VaultLockActivity.kt`; `VaultHomeActivity.kt`; `VaultSecurityActivity.kt`; `VaultCaptureActivity.kt`; `VaultRecorderActivity.kt`; `VaultGalleryActivity.kt`; `VaultFilesActivity.kt`; `VaultNotesActivity.kt`; `VaultHiddenAppsActivity.kt`; `VaultViewerActivity.kt` |
| Focus | `context/FocusAccessibilityService.kt`; `FocusAccess.kt`; `FocusEnforcement.kt`; `ContextEngine.kt`; `ContextScheduleManager.kt`; `ContextAutomation.kt`; `focus/FocusRestrictionActivity.kt`; `res/xml/focus_accessibility_config.xml` |
| Notifications / music | `notifications/NotificationHub.kt`; `NotificationExtractor.kt`; `NotificationModels.kt`; `NotificationCards.kt`; `NotificationAccess.kt`; `widgets/music/MusicWidgetProvider.kt` (contains `MusicNotificationListener`); `MusicBridge.kt`; `MusicLogic.kt`; `MusicAccessActivity.kt`; `res/xml/luma_music_widget.xml`; `res/xml/luma_battery_widget.xml` |
| Launcher / data | `core/LumaApp.kt`; `core/LumaCore.kt`; `core/prefs/LumaPrefs.kt`; `core/apps/AppRepository.kt`; `DefaultDockResolver.kt`; `core/usage/LaunchHistoryStore.kt`; `Frecency.kt`; `core/model/ComponentKey`; `launcher/HomeActivity.kt`; `LauncherViewModel.kt`; `context/VoiceCommandParser.kt`; `diagnostics/DiagnosticsStore.kt`; `DiagnosticsRepository.kt`; `optimizer/Optimizer.kt`; `intelligence/Samplers.kt`; `backup/BackupCodec.kt`; `MiniJson.kt` |
| Media / surfaces | `wallpaper/WallpaperActivity.kt`; `WallpaperImageStore.kt`; `WallpaperCatalog.kt`; `ui/glass/LumaGlassNoise.kt`; `ui/theme/LumaThemeManager.kt`; `ui/theme/WallpaperPaletteExtractor.kt`; `icons/IconPackManager.kt`; `icons/LivingIconCatalog.kt`; `controls/DeviceControls.kt`; `QuickControls*.kt`; `widgets/WidgetHostController.kt` |
| Android data policy | `res/xml/backup_rules.xml`; `res/xml/data_extraction_rules.xml`; `application` attributes `allowBackup` / `fullBackupContent` / `dataExtractionRules` |
| Disclosures shipped to users | `res/values/strings.xml` (`onboarding_privacy_note`, `privacy_summary`, `voice_command_prompt`); `strings_v2.xml` (`focus_service_description`, `focus_access_disclosure*`, `focus_access_not_default`); `strings_shade.xml` (`notification_access_disclosure`, `qc_grant_*`) |
| Docs (corroboration only) | `docs/VAULT.md`; `docs/VAULT_THREAT_MODEL.md`; `docs/FOCUS_ENFORCEMENT.md`; `docs/QA_CHECKLIST.md`; `README.md`; `CHANGELOG.md`; `PRIVACY_POLICY_DRAFT.md` |
| Tests read as evidence | `app/src/test/.../vault/VaultPlatformIntegrationTests.kt` (asserts no backup rule names the Vault dir) + 18 further `vault/` test classes |

---

## 3. Verified facts → where the policy says them

| Claim published in `index.html` | Evidence in source |
|---|---|
| App holds **no** `INTERNET` / `ACCESS_NETWORK_STATE` permission | Only 8 `<uses-permission>` elements exist (manifest L29–49). No HTTP/socket/import anywhere in `app/src/main/java`; the single `https://` literal is `DefaultDockResolver.kt:36` (`Uri.parse("https://example.com")`, a browser-probe) |
| No analytics / ads / crash / billing SDK | `gradle/libs.versions.toml` + `app/build.gradle.kts` dependency block: androidx + coroutines only; no billing, no Firebase, no Play Services |
| App list comes from `LauncherApps`, not `QUERY_ALL_PACKAGES`; `<queries>` narrows visibility | `AppRepository.kt:22-34`; manifest `<queries>` L60–107 |
| Settings live in `files/datastore/lumaos_settings.preferences_pb` | `LumaPrefs.kt:67` `preferencesDataStore(name = "lumaos_settings")`; androidx resolves `filesDir/datastore/<name>.preferences_pb`; 62 fixed keys + per-profile prefixes |
| Launch history: count, last launch, decayed score, 24-hour histogram; only launches **through** Luma; opt-out + clear | `LaunchHistoryStore.kt` (schema, `retainOnly`, `clear`); `Frecency.kt` (`HALF_LIFE_MS` 7 d, `MIN_LAUNCHES_FOR_SUGGESTIONS`); gated by `LauncherViewModel.kt:489 if (suggestionsEnabled.value)`; `SettingsViewModel.kt:438`; no `PACKAGE_USAGE_STATS` in manifest |
| Diagnostics: ≤200 events, ≤30 startup samples, own-process PSS, system memory, battery; local only | `DiagnosticsStore.kt` (`DATABASE_NAME = "lumaos_diagnostics.db"`, `MAX_EVENTS=200`, `MAX_STARTUP_SAMPLES=30`, `clearHistory()`); `Samplers.kt`; `Optimizer.kt:217-290` |
| Vault = AES-256-GCM, 96-bit IV, 128-bit tag, per-object DEK, HKDF domain separation, PBKDF2 PIN, streaming AEAD for large objects | `VaultCrypto.kt` (`KEY_BYTES=32`, `GCM_IV_BYTES=12`, `GCM_TAG_BYTES=16`, `hkdf`, `pbkdf2Sha256`, `calibratePbkdf2Iterations` 350 ms → 120 000–800 000); `VaultStreamAead.kt`; `VaultKeyring.kt` (`metadataKey`, `objectKeyWrapperKey` bind the object id) |
| PIN policy 6–12 digits, rejects repeated/sequential/low-diversity; iterations floored at 100 000 and capped at 4 000 000 | `VaultKeyring.kt` `VaultPinPolicy`, `MIN/MAX_PBKDF2_ITERATIONS` |
| Recovery key = 256-bit CSPRNG, Crockford base32, 54 chars in 14 groups, 10-bit checksum, shown once | `VaultRecoveryCode.kt`; `VaultLockActivity.kt:167` copy |
| Keystore biometric wrapper: non-exportable AES-GCM 256, auth required for **every** use, invalidated on new enrolment, device-unlock required on Q+, unavailable on API 26–27 | `VaultDeviceKeyGuard.kt:110-125`, `availability()` L64-95; `VaultBiometricGate.kt` (`BiometricPrompt` + `CryptoObject`); minSdk 26 → `UNSUPPORTED_OS` path |
| Auto-lock 30 s/1/3/5/10 min, default 3 min; locks on screen-off, on leaving, on process death; `FLAG_SECURE`; excluded from Recents; all Vault activities `exported="false"` | `VaultModels.kt:118-128`; `VaultSession.kt` `LockReason`; `VaultScreenActivity.kt:58`; `AndroidManifest.xml` Vault activity block |
| Retry throttling in `guard.json`, Keystore-HMAC-signed, ≤15 min, deletion = maximum penalty, no key material | `VaultUnlockGuard.kt` (`MAX_DELAY_MS`), `VaultGuardSigner.kt`, `docs/VAULT_THREAT_MODEL.md` §3.1 (matches code) |
| No plaintext ever written by camera/scanner; recorder → vault `staging/` scratch, then encrypted; no MediaStore row; video capture **not** implemented | `VaultCaptureActivity.kt` header + `savePdf()` (L392-399, "PDF bytes never touch the filesystem"); `VaultRecorderActivity.kt` header; `VaultGalleryActivity.kt:23-25` |
| Decoded images/thumbnails RAM-only, bounded 4–24 MB, cleared on lock | `VaultMedia.kt` `LumaWallpaperBlurCache`… (`VaultMediaCache.budgetBytes`) |
| Deletion = shred best-effort; **what really deletes is losing the key**; no physical-erasure claim | `VaultObjectStore.kt:136-156`; `VaultPaths.kt:158-186` ("Flash storage may retain the physical blocks… stated plainly in the Privacy notes instead of being claimed") |
| Backup container `LVBK` v1: recovery-wrapped VMK, ciphertext body copied byte-for-byte, SHA-256 trailer, hostile-input caps, read-back verification; aborts if a referenced object is missing; 16 MB free-space margin; restore stages → verifies → swaps → rolls back | `VaultBackup.kt` (MAGIC/VERSION/limits/`write`/`restore`); `VaultManager.backupTo` + `verifyContainer` + `restoreFrom` (L1381-1460) |
| Export is explicit, via Android's document picker, and is stated to make content less private | `VaultActions.startExport` (`ACTION_CREATE_DOCUMENT`) + `VaultManager.exportTo:1226` |
| One notification listener powers shade **and** music; content in memory only; ≤60 cards; reply via the posting app's own `PendingIntent`; nothing written/logged | `MusicWidgetProvider.kt:35-80` doc + `NotificationHub.kt` (`MAX_ENTRIES=60`, `reply()`, clears listener on disconnect); `NotificationExtractor.kt` (fields read; "never written to disk, never logged"); grep confirms **no** `SharedPreferences`/file write in `notifications/` or `widgets/` |
| Accessibility service: `typeWindowStateChanged` only, `canRetrieveWindowContent="false"`, reads only `event.packageName`, nothing persisted, 2 s per-package cooldown, 350 ms post-Home presentation, safety allowlist, revocation = supported steady state | `res/xml/focus_accessibility_config.xml`; `FocusAccessibilityService.kt` (`onAccessibilityEvent`, `evaluate`, `intervene`, `FocusInterventionGate`); `FocusAccess.kt`; `FocusEnforcement.kt` |
| Voice = `RecognizerIntent`, provider does the capture; ≤3 transcripts; not retained | `HomeActivity.kt:3518-3522`, `:190` (`EXTRA_RESULTS`), `strings.xml:25` prompt text; `VoiceCommandParser.kt` |
| Icon packs read `appfilter.xml` locally; living icons are bundled assets; "premium" themes are local only | `IconPackManager.kt:37,84-87`; `assets/living_icons/*`, `assets/licenses/*`; no billing dependency |
| Vault excluded from **all** Android backup paths | Storage is `File(context.noBackupFilesDir, "vault")` (`VaultManager.kt:102`); neither rules file names `no_backup`; `VaultPlatformIntegrationTests.kt` guards it; AOSP domain mapping ((§1, item 3) |
| Wallpaper copies: `files/wallpaper_images`, ≤12 files, ≤12 MB, byte-for-byte (so embedded metadata travels), EXIF read for orientation only, excluded from backup, `setBitmap(..., allowBackup=false)` for custom photos | `WallpaperImageStore.kt:37,76-140,238-255,330-340`; `WallpaperCatalog.kt:644-645` |
| Blurred wallpaper derivative cached as PNG in `cacheDir` | `LumaGlassNoise.kt:95-135` + call site `LumaThemeManager.kt:567-570` (`cacheDir = context.cacheDir`) — **not** mentioned in `PRIVACY_POLICY_DRAFT.md`; added to the policy |
| Optimiser touches only LumaOS Pro's own caches/DB; no other app is killed or cleaned | `Optimizer.kt:83-113` (`scope` on every action); manifest comment: no `KILL_BACKGROUND_PROCESSES` / `FORCE_STOP` |
| No exported provider; only the home activity is a normal exported entry point; music control receiver + schedule receiver unexported | `grep -c "<provider"` = 0; manifest activity/service/receiver `exported` flags |

---

## 4. Contradictions and defects found (source vs. docs vs. draft policy)

`F-1`…`F-8`. **Severity: BLOCKER** = must resolve before publishing; **MAJOR** = fix in code or wording; **MINOR** = note only.

**F-1 · BLOCKER · Release version does not exist in the audited tree.**
The brief specifies **1.5.0**. The repository states **1.2.0 / `versionCode` 120** in
`app/build.gradle.kts:45-46`, and `CHANGELOG.md:330-334` says so explicitly and adds
*"Nothing below has been published"*. There is no 1.3.0, 1.4.0 or 1.5.0 section anywhere in
`CHANGELOG.md`; every headline feature this policy documents (Vault, Focus Protection layer 2,
notification shade, music widget, context engine) sits under **"Unreleased"** headings.
The policy page prints **1.5.0** as instructed and describes the feature set that is actually
in `main`. *Action:* confirm the release build is the audited tree (or re-run this audit against
the real 1.5.0 tag). If 1.5.0 differs, the policy claims in §6/§7 must be re-verified.

**F-2 · MAJOR · Device-to-device transfer cannot carry the history it says it will.**
`res/xml/data_extraction_rules.xml` `<device-transfer>` includes
`<include domain="file" path="databases/"/>`. Per AOSP `FullBackup.java`, the `file` domain
resolves to `filesDir`, so this names `files/databases/` — a path that does not exist.
The app's SQLite files (`lumaos_launches.db`, `lumaos_diagnostics.db`) live under
`getDatabasePath()`, i.e. the **`database`** domain (singular). Conclusion: launch history and
diagnostics are excluded from *both* cloud backup and device transfer — which is the more
private outcome, but contradicts the file's comment ("a direct switch should carry the
history too"), `PRIVACY_POLICY_DRAFT.md` and the in-app `privacy_summary` string
("device-to-device transfer can include diagnostics history").
*Action:* either change to `domain="database"` (and keep the wording), or delete the include and
correct the in-app string. The published policy is worded to stay true under either fix
(§13 names the caveat rather than asserting a behaviour) — but the in-app text is currently
inaccurate and, being the user-facing disclosure, matters more than the doc.

**F-3 · BLOCKER (compliance) · The in-app privacy summary under-discloses the shade.**
`strings.xml:188` `privacy_summary` states the notification-access grant is used "only for
media sessions and does not read or store notification contents". `NotificationExtractor.kt`
reads titles, bodies, sub-text, expanded text, conversation lines, message lines, timestamps
and `RemoteInput` in order to draw and reply to cards. "Does not **store**" is true;
"does not read" and "only for media sessions" are not. Under Play's User Data policy the
in-app disclosure is what is judged alongside this policy.
*Action:* correct `privacy_summary` to match §8 of the published policy (reads in memory,
stores nothing, ≤60 cards, revocable), then re-screenshot any store asset that shows that screen.

**F-4 · MAJOR · Do Not Disturb automation cannot currently be granted.**
`ContextAutomation.applyDoNotDisturb` and `DeviceControls.applyFocusFilter` gate on
`NotificationManager.isNotificationPolicyAccessGranted()` and then call
`setInterruptionFilter()`, which requires `ACCESS_NOTIFICATION_POLICY`. That permission is
**not declared** in the manifest, so the grant screen will not list LumaOS Pro and both paths
will stay in their `return`-early branch: the feature silently no-ops while `docs/` and
`README.md` describe it as working ("it restores the prior filter when leaving that context").
The policy was written conservatively (DND described as a user-granted special access,
revocable, with restore-only-if-still-owned semantics) so it does not overclaim — but the app
either needs the permission declared or the feature removed.

**F-5 · MAJOR · `WRITE_SETTINGS` used but not declared.**
Brightness and rotation-lock writes are gated on `Settings.System.canWrite()`
(`DeviceControls.kt:311,322-366`). `WRITE_SETTINGS` is absent from the manifest, so
`canWrite()` is permanently false, the tiles always fall back to
`TileTapOutcome.NeedsGrant`, and Android's "modify system settings" grant page will not list
the app. Same class of defect as F-4: a documented capability that cannot activate. Either
declare it (and disclose it — it *is* a special app access) or state plainly in the UI that
brightness/rotation open Android's panel instead. `WifiManager.isWifiEnabled` is likewise read
without `ACCESS_WIFI_STATE`; it is wrapped in `runCatching` and degrades to "unavailable",
so the honest wording "we ask Android and fall back to its panel" already holds.

**F-6 · MINOR · `UsageStatsManager` is queried despite a manifest comment to the contrary.**
`DefaultDockResolver.kt:164-176` calls `queryUsageStats(INTERVAL_MONTHLY, now-30d, now)` to
rank first-run home-grid seeding, while the manifest comment claims usage stats are
"deliberately not requested" and `Frecency.kt` repeats that. Both are true of *foreground
detection*, and no permission is held — so on stock Android the call yields nothing and the
curated fallback runs. No policy change needed (the policy never claims the API is untouched,
and holds out only that no usage-stats permission exists), but the stale comment invites a
future reviewer to "fix" the wrong thing. Prefer deleting the call: it is a permission-free
peek at aggregate foreground time that buys nothing when it returns empty.

**F-7 · MINOR · Dead and redundant configuration.**
(a) `backup_rules.xml` / `data_extraction_rules.xml` include `domain="sharedpref" path="."`,
but the app contains **zero** `getSharedPreferences` calls — the include is a permanent no-op.
(b) `VaultManager.PREFS = "lumaos_vault"` (L68) is unreferenced.
(c) `VaultBackupSettings.autoBackup` is parsed and stored but no code path consumes it —
matching `README.md` ("unattended scheduled backups are not implemented yet"). The policy says
exactly that; do not let a marketing copy "discover" an auto-backup toggle.
None affect accuracy; all three are worth cleaning so the next audit is shorter.

**F-8 · MINOR · `docs/VAULT.md` states "SQLite is used solely by the diagnostics store".**
`lumaos_launches.db` also exists. Documentation drift only — the policy follows the code.

**Also checked and found genuinely accurate** (worth recording, because these are the claims a
reviewer is most likely to challenge): the no-`INTERNET` statement; the Vault/`noBackupFilesDir`
isolation and its build test; `canRetrieveWindowContent="false"` and the single-field read;
notification content being memory-only with a 60-entry bound; the `LVBK` container carrying only
the recovery-wrapped master key; the "recovery-only, verified or failed" backup-state model; and
every "not implemented, never claimed" line in `docs/FOCUS_ENFORCEMENT.md` (no device owner, no
in-app screen blocking, no rebind API).

---

## 5. Unresolved facts requiring developer confirmation

**BLOCKING before publication**

1. **Release identity (F-1).** Is the audited tree the release build? Publish only when the
   policy's §6/§7 descriptions match the shipping APK. Nothing here is device-verified:
   `docs/VAULT.md` states the sandbox had no SDK/Gradle/emulator, so run `./build.sh test`
   (and `lint`) before the release that this policy ships with.
2. **Play listing state.** The policy directs users to "the developer contact shown on the
   LumaOS Pro listing on Google Play". `PRIVACY_POLICY_DRAFT.md` carries an unconfirmed
   publisher (`saqibtantray`) and support address (`sytmrx.dev@gmail.com`); `CHANGELOG.md`
   says nothing has been published, and `README.md:123-126` lists "complete the store listing
   (including publisher/support contact…)" as outstanding. If no live listing exists at publish
   time, either add the confirmed email here or publish the listing first. **Do not invent one.**
3. **F-3 correction** (in-app `privacy_summary`) must ship in the same release, otherwise the
   in-app disclosure and this policy disagree on notification access.

**Should be confirmed, not blocking**

4. **Intended audience / rating answers.** Nothing in the source constrains this; §18 is written
   for a general audience with no child-directed content. Confirm the Play age-content
   questionnaire and the "designed for families" answer agree.
5. **Publisher legal identity.** §1 names "the developer of LumaOS Pro" because no legal entity,
   address or DPO exists in the source and none may be invented. If you register a company, §1
   and §19 both need the entity name and postal address added.
6. **Canonical URL.** This repo's GitHub Pages site is live and building
   (`https://rolexsir-org.github.io/lumaOS-privacy/`, verified via the Pages API today), so that
   URL can go into Play Console as soon as this branch is merged to `main`. A custom domain would
   replace it — then update the `<link rel="canonical">`/`og:url` in `index.html` too.
7. **F-2 / F-4 / F-5 / F-6 code decisions.** Each changes a sentence in the policy once settled
   (device transfer of history; DND availability; WRITE_SETTINGS; usage-stats call).
8. **OEM behaviour.** `FLAG_SECURE`, screenshot blocking, D2D migration content and whether an OEM
   task manager rebinds the accessibility service are device-dependent; §7 and §13 already hedge
   them ("Android decides", "behaviour differs by OEM"). Keep that hedging unless you device-test.
9. **Old policy URL.** The previous page (dated 30 October 2026) is described as superseded; if it
   was never live, delete the "Replaces" chip from the header rather than referencing a URL that
   never existed.

---

## 6. Permission → purpose mapping

All eight runtime/normal permissions below are exactly what
`app/src/main/AndroidManifest.xml` declares — nothing more.

| Permission / access | Protection level | Declared | Requested | Purpose (verified) | Data it enables touching | Alternative if declined |
|---|---|---|---|---|---|---|
| `REQUEST_DELETE_PACKAGES` | normal (install-time) | ✅ L29 | n/a | Lets the long-press menu offer **Uninstall**, which opens Android's own confirmation | package name of the app you chose | The menu item is useless; nothing else changes |
| `RECEIVE_BOOT_COMPLETED` | normal | ✅ L30 | n/a | `ContextBootReceiver` rebuilds the next **inexact** profile alarm after boot / `TIME_SET` / `TIMEZONE_CHANGED` | your own schedules only | Context switches stop firing until the launcher next runs |
| `POST_NOTIFICATIONS` | **runtime** | ✅ L31 | at the moment context alerts are enabled (`SettingsActivity.kt:237-245`) | The launcher's *own* "Work profile active"-style alert (channel `context_changes`) | nothing beyond your schedule state | No alerts; everything else unaffected |
| `USE_BIOMETRIC` | normal | ✅ L33 | n/a | Ask Android to verify you for the optional Vault biometric wrapper (`BiometricPrompt` + `CryptoObject`) | a verified/not-verified result only. **No biometric sample is ever read, stored or compared** | Unlock with the PIN |
| `USE_FINGERPRINT` | normal (legacy, ≤API 27) | ✅ L34 | n/a | Same, on Android 8.1 and below; `availability()` returns `UNSUPPORTED_OS` on API 26–27 | as above | as above |
| `SET_WALLPAPER` | normal | ✅ L36 | n/a | Apply the wallpaper you chose to home / lock / both via `WallpaperManager.setBitmap` | the bitmap you selected | Keep Android's wallpaper |
| `CAMERA` | **runtime** | ✅ L45 | on entering the Vault camera/scanner (`VaultCaptureActivity.kt:109`) | Private capture + document scanner → JPEG/PDF **in memory** → encrypted object. No video. The Quick-Controls torch uses `CameraManager.setTorchMode` and needs no permission | camera frames, never persisted unencrypted, never in `DCIM`/`MediaStore` | Scanner + private camera say they can't run; rest of Vault works |
| `RECORD_AUDIO` | **runtime** | ✅ L46 | on starting the Vault recorder (`VaultRecorderActivity.kt:95`) | Vault voice memos, staged in `no_backup/vault/staging` then encrypted on stop | microphone audio for that one recording | Recorder explains itself; everything else works |

**Declared features (not permissions):** `camera.any`, `microphone`, both `required="false"` (L48-49) —
so devices without them still install and simply see those two surfaces unavailable.

**`<queries>` (Android 11 visibility, L60-107):** launchable apps · HOME role (for truthful
"am I the default launcher" status) · `WEB_SEARCH` · `VIEW http/https` browsable · `market://` ·
`STILL_CAMERA`, `IMAGE_CAPTURE`, `CAST_SETTINGS`, `DISPLAY_SETTINGS` (each Quick-Controls tile is
offered only if a real handler answers) · seven icon-pack theme actions
(`org.adw.launcher.THEMES`, `com.novalauncher.THEME`, `com.teslacoilsw.launcher.THEME`,
`com.anddoes.launcher.THEME`, `com.gau.go.launcherex.theme`, `com.fede.launcher.THEME_ICONPACK`,
`net.oneplus.launcher.icons.ACTION_PICK_ICON`). **No `QUERY_ALL_PACKAGES`.**

**Special accesses (user-switched in Android settings; no `<uses-permission>` can obtain them):**

| Binding | Guarded by | What the app can then do | Verified ceiling |
|---|---|---|---|
| Accessibility — "LumaOS Focus Protection" (service, `exported=true`, `BIND_ACCESSIBILITY_SERVICE`) | config `focus_accessibility_config.xml` | Receive `TYPE_WINDOW_STATE_CHANGED`; read `event.packageName`; `performGlobalAction(GLOBAL_ACTION_HOME)` | `canRetrieveWindowContent="false"`; no content/input access; nothing persisted/logged/transmitted; **but note in §7 that an accessibility binding is generically powerful — the narrowness is this config, not the API** |
| Notification access — `MusicNotificationListener` (`BIND_NOTIFICATION_LISTENER_SERVICE`) | Android's own toggle | `onNotificationPosted/Removed` + `getActiveSessions()` | ≤60 in-memory projections; content never written/logged; replies go back through the posting app's `PendingIntent` |
| Do Not Disturb access | `isNotificationPolicyAccessGranted()` | `setInterruptionFilter(PRIORITY)` / restore | **F-4:** `ACCESS_NOTIFICATION_POLICY` not declared ⇒ likely never grantable |
| Modify system settings | `Settings.System.canWrite()` | brightness, rotation lock | **F-5:** `WRITE_SETTINGS` not declared ⇒ always false, falls back to opening Android's panel |
| Usage access (not requested) | — | — | **F-6:** API called once at first run, returns nothing without the permission |

**Not requested, and therefore impossible:** location (fine or coarse), contacts, phone, SMS,
calendar, call log, body sensors, `READ_EXTERNAL_STORAGE`/`READ_MEDIA_*`, `MANAGE_EXTERNAL_STORAGE`,
`QUERY_ALL_PACKAGES`, `PACKAGE_USAGE_STATS`,
`KILL_BACKGROUND_PROCESSES`, `FORCE_STOP`, `DEVICE_ADMIN`, `SYSTEM_ALERT_WINDOW`, `INTERNET`.

---

## 7. Data-flow and third-party processing summary

### 7.1 Inbound — what enters the app

| Source | Enters as | Held where | Lifetime |
|---|---|---|---|
| Android `LauncherApps` | package, activity component, profile serial, label, icon | process memory; component ids embedded in layout/hidden/pinned keys and in launch rows | refreshed on package broadcasts; layout persists until you change it |
| Your gestures/settings | 62 fixed DataStore keys + per-profile `home_layout_*`, `hidden_apps_*`, `pinned_apps_*`, `drawer_hidden_*`, `gesture_*`, `schedule_*`, `focus_*`, `context_wallpaper_*`, `backdrop_*`, `profile_theme_*` | `files/datastore/lumaos_settings.preferences_pb` | until changed; eligible for Android backup |
| Your taps | launch events | `databases/lumaos_launches.db` | until cleared / app removed / suggestions off |
| Perf sampling | PSS, available memory, low-memory flag, startup ms, action outcomes | `databases/lumaos_diagnostics.db` | rolling caps 200 / 30 |
| Android battery + widget ticks | level, charging | memory → `RemoteViews` | per update |
| Notification listener | per-notification fields + actions + `RemoteInput` | **memory only**, ≤60 | with the notification |
| Media sessions | player label, title/artist/subtitle, state, position, duration, artwork | memory → `RemoteViews` | with the session |
| Accessibility service | one package name per window-state event | inside one decision | discarded immediately |
| Speech provider | up to 3 transcript strings | memory | discarded after matching |
| Your picks (SAF) | wallpaper photo bytes; Vault file bytes; PDF/backup containers | `files/wallpaper_images/` or encrypted `no_backup/vault/objects/` | until you delete |
| Camera2 / MediaRecorder | JPEG frames, scratch `.mp4`/audio | memory; staging dir | encrypted or shredded at lock/unlock |

### 7.2 Outbound — every path by which data can leave the process

| # | Destination | Data | Trigger | Governed by |
|---|---|---|---|---|
| 1 | The app you launch | an intent, nothing about you | any tap | Android |
| 2 | **Speech-recognition provider** (3rd-party app; may upload) | your speech + language | only an invoked voice command | that provider's policy — **the one genuinely external third-party processing in the app** |
| 3 | Browser / web-search app / Play Store | the drawer's search text | tapping a "search web"/"search in Play" row (`HomeActivity:2804-2805`) | that app |
| 4 | Posting app of a notification | your reply text | replying in the shade | that app |
| 5 | Player app | media key events | widget/row controls | that app |
| 6 | Third-party widget providers | widget id, bounds, options, update | placing/resizing | that app |
| 7 | **Android Auto Backup transport (Google)** | the whole DataStore file, incl. keys the JSON export strips (`context_dnd_*`, `focus_unlock_*`, `last_action_at`, cached palettes, prompt flags) | your device's backup setting | your Google account + Google's backup terms |
| 8 | Android device-to-device transfer | same file (not the DBs — see F-2) | your choice during setup | the migration tool |
| 9 | A user-chosen destination | one decrypted Vault item; a `LVBK` **encrypted** container; or the plaintext `lumaos-backup.json` | explicit export/backup, after confirmation | wherever you put it |
| 10 | Android system | wallpaper bitmap (its own copy; `allowBackup=false` requested for custom photos) | setting a wallpaper | Android / OEM |
| — | **Developer / any server** | **nothing** | — | no network permission, no SDK |

### 7.3 Storage map

```
/data/data/com.lumaos.pro.launcher/
  files/datastore/lumaos_settings.preferences_pb   settings+layouts      ── Android backup ✅  D2D ✅  app JSON export ✅ (subset)
  files/wallpaper_images/<uuid>.img               photo copies (metadata intact) ── backup ❌  export ❌ (refs stripped)
  databases/lumaos_launches.db                     launch history        ── backup ❌  D2D ❌ (F-2)  export ❌
  databases/lumaos_diagnostics.db                  measurements          ── backup ❌  D2D ❌ (F-2)  export ❌
  cache/luma_wp_blur_<hash>.png                    tiny blurred wallpaper derivative ── backup ❌ (cache is never backed up)
  no_backup/vault/keyring.json                     3 wrapped copies of the VMK: ciphertext + salts + iteration counts
  no_backup/vault/guard.json                       Keystore-HMAC-signed attempt counter + lockout deadline
  no_backup/vault/meta.bin(+.bak)                  encrypted index: names, folders, settings, ≤200 events
  no_backup/vault/objects/<32-hex>.obj              AES-256-GCM streaming payloads, opaque names
  no_backup/vault/staging/                         capture/scratch, shredded at lock and unlock
  no_backup/vault/concealment*                      device-bound Keystore-sealed hidden-app mirror (readable pre-auth, useless off-device)
```
Anything in `no_backup/` is outside both Android backup trees ((§1, item 3), which is why the policy can
state the Vault exclusion as a structural fact (see §1, item 3).

---

## 8. Google Play Data safety and sensitive-permission declaration checklist

Pre-filled from this audit. Play's forms are the authority on their own wording — where a
definition decides a box, that is flagged rather than assumed. **Do not submit until F-1…F-3 are
closed.**

### 8.1 Data safety form

- [ ] **Privacy policy URL** → `https://rolexsir-org.github.io/lumaOS-privacy/` (Pages verified live today; must be merged to `main` first, and must remain reachable at that URL forever after).
- [ ] **Account needed to use the app?** No — `grep` finds no auth, no `AccountManager`, no sign-in UI.
- [ ] **Data encrypted in transit?** The honest answer for this app is nuanced: nothing is transmitted by the app, and Android backup transport is Google's. Do not tick "all data encrypted in transit" as an app property; do not tick "not encrypted" either — read the current Console help text for the no-network case.
- [ ] **Users can request deletion?** Yes — deletion is self-service on-device (clear data / uninstall / clear history), stated in §12 and §17 of the policy.
- [ ] **Data types — recommended declarations.** Play counts *collection* as data leaving the app to a developer or third party. On the audited build nothing leaves to the publisher, so the expected answer for each of the following is **"Not collected / Not shared"**, with the two exceptions below:
  - App info & list of installed apps · Device or other IDs (**not read**: no ANDROID_ID, no serial, no advertising ID) · Name, email, phone · Photos/Videos, Audio files, Files/Folders · Messages, Notifications · App activity · Device or other identifiers · Precise/Coarse location · Calendar, Contacts · In-app search · Voice or audio · Performance data · Crash logs · Purchase history — all **not collected by the app**.
  - **Exception 1 — Voice:** invoking a voice command hands audio to another app on the device, which may upload it. Decide with the current Play guidance whether that is declared as sharing to a third party; the policy describes it openly in §10 either way.
  - **Exception 2 — Backup:** the settings file is handed to Android's backup agent. Whether Auto Backup must be declared is a definitional question for the current form; the policy's §13 is written so the answer can't contradict it.
- [ ] **Vault is not a data type to declare** — it never leaves the device except via a user-chosen file. Do not declare encrypted files as "collected" if you adopt the "nothing leaves to us" position; keep the two answers consistent.
- [ ] **Advertising ID / third-party ad SDK:** none (dependencies audited). Answer "no" everywhere it is asked.
- [ ] **Analytics SDK:** none.
- [ ] **Crash reporting:** none — state that diagnostics are local-only if the form offers a free-text note.

### 8.2 Sensitive API / permission declarations

- [ ] **Accessibility Service API** (`FocusAccessibilityService`). Declare use case = user-directed app-blocking/accountability for the user's own device, with the four facts the policy publishes: single event type; `canRetrieveWindowContent="false"`; only `event.packageName` read; nothing collected, stored, logged or transmitted. Prepare for the review to ask for: a screen recording showing the **in-app disclosure** (`focus_access_disclosure`) *before* the system-settings redirect, the system toggle's own service description, the revoked state, and layer 1 continuing to work. All four exist in code — do not demo a build where they were removed.
- [ ] **Notification Listener API** (`MusicNotificationListener`). Declare the two purposes (media sessions for the widget; shade rendering + reply). Expect to state, and to be checked against: no data retention (≤60 in RAM), no logging of content, no transmission, prominent disclosure (`notification_access_disclosure`), per-user grant/revocation, and that **one** listener serves both features. Play's policy on this API also bars using it to *collect* data — the memory-only design is the answer, and `privacy_summary` must stop implying the shade reads nothing (F-3).
- [ ] **Camera permission.** Justify: Vault private capture + document scanner only, requested at use, optional, no video, no background capture, no face detection, no EXIF/GPS tagging. ⚠️ There is **no dedicated pre-permission rationale sheet** before the system `CAMERA`/`RECORD_AUDIO` dialog — the rationale lives in the screen's caption and in the post-denial explanation. If Play's prominent-disclosure expectation for camera/mic requires an explicit pre-prompt, add it (small change, closes the gap).
- [ ] **Microphone permission.** Same justification, scoped to one user-started recording.
- [ ] **All Files Access (`MANAGE_EXTERNAL_STORAGE`)** — not requested: answer "no".
- [ ] **Device admin / lock task / device owner** — not used: answer "no" (`docs/FOCUS_ENFORCEMENT.md` §"Why not … DevicePolicy").
- [ ] **`QUERY_ALL_PACKAGES`** — not requested: nothing to justify.
- [ ] **SMS / Call log / Contacts / Calendar / Location** — not requested: nothing to justify.
- [ ] **`REQUEST_DELETE_PACKAGES`** — declare purpose = user-confirmed uninstall shortcut only (no silent uninstall path exists; `AppRepository.uninstallIntent` only fires `ACTION_DELETE`).
- [ ] **`RECEIVE_BOOT_COMPLETED`** — declare purpose = rebuilding one inexact alarm; there is no persistent background service loop beyond the two Android bindings, and no polling anywhere (accessibility and notification paths are both event-driven).
- [ ] **`SET_WALLPAPER`** — user-initiated static wallpaper only; no live/wallpaper-service loop.

### 8.3 Content, ratings and store-listing alignment

- [ ] Age rating questionnaire answers match §18 (general audience, no child-directed content, no UGC, no ads, no in-app purchases).
- [ ] "Designed for families": opt out unless you intentionally opt in — and if you opt in, §18 and the Family Link statements must be re-audited.
- [ ] Store listing "Safety" section and this policy must not diverge from each other or from `privacy_summary` (single source of truth = this repo).
- [ ] Promo screenshots/video must not show Vault content, and should show the disclosure screens the sensitive-API forms reference.
- [ ] `README.md:123-130` still lists trademark clearance and package-name availability for `com.lumaos.pro.launcher` as outstanding — a Play-blocking item, not a privacy one, but do not lose it.

---

## 9. Verification performed on the published page

| Check | Result |
|---|---|
| Tag balance / well-formedness (`html.parser` walk, void-element aware) | **no errors**, no unclosed elements |
| Every `href="#…"` resolves to an `id` | 74 internal links → 24 distinct targets, **0 broken** |
| Duplicate ids | 0 |
| Required sections | 20 `<section>` + header + footer; TOC has exactly 20 entries in document order |
| Heading hierarchy | h1 → h2 → h3, no level skips |
| `aria-labelledby` targets all present | yes (20 sections + glance card + TOC) |
| Scripts / inline handlers | **0** `<script>`, 0 `on*=` attributes |
| External resources | 0 (fonts are system stacks; no CDN, no image requests, no analytics) |
| Unfinished placeholders | 0 (`TODO/TBD/[insert]/{{}}/lorem` all absent) |
| WCAG 2.1 contrast (computed from the CSS tokens) | body 13.35:1, headings 18.0:1, muted 8.57:1, accent links 9.79:1, callout 13.07:1 — all pass AA, most pass AAA |
| Payload | 109 843 bytes raw, **32 127 bytes gzipped**, single file |
| Responsive | single column < 1000px (two-column TOC list < 1000px, one column < 460px); sticky rail ≥ 1000px; tables scroll horizontally rather than squeezing; `min-width:0` guards grid overflow |
| Print / PDF | dedicated `@media print` pass: light-on-white, chrome hidden, sections kept whole |
| Motion & focus | `prefers-reduced-motion` honoured; `:focus-visible` ring; skip link; `scroll-margin-top`; `:target` flash to orient after a jump |
| Metadata | `title`, `description`, `canonical`, Open Graph, Twitter card, `theme-color`, `color-scheme`, `robots` |

## 10. Changing this page

Keep it a single self-contained file. If you edit the text: re-run §9's checks (anchor targets and
"no placeholders" are the two that break silently), update the **Effective** date and the
"Replaces" chip together, and treat any change to §6/§7/§8 as a change to the Play declarations.
