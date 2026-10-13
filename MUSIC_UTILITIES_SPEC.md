# MUSIC UTILITIES SPECIFICATION

Project: MusicPlayer
Document: MUSIC_UTILITIES_SPEC.md
Version: 5
Status: Proposed: Feature Expansion (not yet implemented)
Purpose: Define the functional, architectural, UI, system-integration and testing requirements for everyday music utilities without destabilizing the Release Candidate baseline.

Related docs:
- PRODUCT_SPEC.md
- ARCHITECTURE.md
- DESIGN_SYSTEM.md
- SCREEN_SPECIFICATIONS.md
- INTERACTION_SPEC.md
- DATA_MODEL.md
- PLAYBACK_ARCHITECTURE.md
- TEST_PLAN.md

> Revision note (v5).
> Revised before implementation. Compared with v4, this version:
> - adds a pending system-sound action TTL and a single-pending-action rule
> - requires the pending action to be stored before leaving for Settings and persisted in saved state with a createdAt timestamp
> - requires re-checking permission on resume instead of trusting the activity result
> - adds a read-back algorithm based on MediaStore id resolution, with Success / Failed / Unverified outcomes
> - clarifies that copy and inspector operations have explicit timeouts
> - names the required constants for share cap, copy timeout, inspector timeout, and pending TTL
> - clarifies backup exclusion for system-sound copies and the performance safety of the copy registry
> - adds work-profile / multi-user handling and an OEM caveat test requirement
> - adds the missing "only one pending action at a time" rule
> - strengthens the "may not affect everything" caveat for ringtone, notification and alarm sounds
> - removes ambiguity around track list ordering and playlist item id selection
> - adds exact QA reporting language for previously unverified items
>
> Section 47 contains the full change list.

---

## 1. Scope

This document covers:

- Sorting (tracks, albums, artists, folders, playlists, search)
- Track action framework and menu
- Share (single and batch)
- Set as ringtone / notification sound / alarm sound
- Song Details
- Show in Files (best effort)
- Multi-select and batch actions
- Entity navigation shortcuts
- Android system integration, error handling, persistence, accessibility and testing

It is an implementation specification, not a design-only concept.

---

## 2. Protected Baseline

The current Release Candidate is the protected baseline. New features MUST NOT unnecessarily modify or replace:

- Media3 ExoPlayer architecture and service ownership (`MusicPlaybackService` owns the only authoritative player)
- MediaSession / `MediaLibraryService`, `MediaController` communication, `PlaybackController`
- Audio focus and audio-becoming-noisy handling
- Background playback, queue synchronization, shuffle, repeat, playback resumption
- MediaStore scanning and `ContentObserver`
- Room and DataStore architecture (additions allowed only as stated in this document)
- Library navigation, Now Playing behavior, widgets, visualizer
- The privacy model, except for the single documented addition in section 4

New utilities use existing domain/repository abstractions. They never create a second `ExoPlayer`.

---

## 3. Product Philosophy

MusicPlayer is a fast, private, offline-first local music player with excellent Android system integration. Utilities must remain offline-first, fast, minimal, predictable, privacy-preserving and visually consistent with the existing design system. No accounts, online services, analytics or network dependencies.

---

## 4. Permissions and Privacy Impact

This expansion adds one special-access permission. It must be documented in `PRODUCT_SPEC.md` and in the privacy/About screen.

| Permission | Needed for | Type | When requested |
|---|---|---|---|
| `READ_MEDIA_AUDIO` (existing) | Library | Runtime | Existing flow |
| `POST_NOTIFICATIONS` (existing) | Playback notification | Runtime | Existing flow |
| `WRITE_SETTINGS` (new) | Setting the default ringtone, notification or alarm sound | Special app access ("Modify system settings") | Only when the user first chooses a Set as... action |

Rules:

- Declare:
  `android.permission.WRITE_SETTINGS`
- Check `Settings.System.canWrite(context)` before every system-sound operation.
- If not granted, explain the reason, then launch `Settings.ACTION_MANAGE_WRITE_SETTINGS` with the URI `package:<applicationId>`.
- `ACTION_MANAGE_WRITE_SETTINGS` does not return a reliable result code. Never infer the outcome from the result; re-check `canWrite()` when the screen resumes.
- If the user does not grant it, the Set as... actions stay unavailable with a clear message. Nothing else in the app is affected.
- Still forbidden:
  - `INTERNET`
  - `RECORD_AUDIO`
  - location
  - contacts
  - camera
  - broad storage permissions
- API 29+ requires no extra storage permission for entries the app creates itself.

---

## 5. Feature Priority

| Priority | Feature |
|---|---|
| P0 | 1. Track sorting |
|  | 2. Track action framework and menu |
|  | 3. Share audio file |
|  | 4. Song Details |
|  | 5. Set as ringtone |
| P1 | 6. Set as notification sound |
|  | 7. Set as alarm sound (same mechanism as ringtone; shipped with notification sound) |
|  | 8. Multi-select |
|  | 9. Batch actions |
|  | 10. Entity navigation shortcuts |
| P2 | 11. Show in Files (best effort) |

Song Details precedes the ringtone work: it needs no special permission and is the lowest-risk P0. All P0 features ship before P1.

---
# Part A: Sorting

## 6. Sort Targets and Keys

Each entity has its own key set. Keys that do not make sense for an entity are not offered.

| Entity | Sort keys | Default |
|---|---|---|
| Tracks | Title, Artist, Album, Duration, Date Added, Year, File Size | Title, ascending |
| Albums | Title, Artist, Year, Track Count | Title, ascending |
| Artists | Name, Track Count, Album Count | Name, ascending |
| Folders | Name, Track Count | Name, ascending |
| Playlists | Name, Date Created, Track Count | Name, ascending |
| Search results | Relevance (default), then Title / Artist | Relevance |

All entities support ascending and descending. The UI clearly shows the selected key and direction.

Example:

```text
SORT TRACKS
 ● Title   ○ Artist   ○ Album   ○ Duration
 ○ Date Added   ○ Year   ○ File Size

ORDER
 ● Ascending   ○ Descending
```

Inside detail screens:
- Album detail stays in disc/track-number order
- Playlist detail stays in user-defined order
- Folder detail uses the Tracks sort preference

## 7. Defaults and Upgrade Safety

- Default is Title ascending, unless the app already defines a different ordering.
- The existing DataStore key `sort_order` is migrated into `trackSortKey` / `trackSortDirection` on first launch of the new version, so no user's library is silently reordered.
- The migration is idempotent and covered by a test.

## 8. Persistence

Persist sort state in DataStore (not Room):

```text
trackSortKey,    trackSortDirection
albumSortKey,    albumSortDirection
artistSortKey,   artistSortDirection
folderSortKey,   folderSortDirection
playlistSortKey, playlistSortDirection
```

Search sort is not persisted; it resets to Relevance each search session.

## 9. Sorting Requirements

Sorting must be deterministic, stable, null-safe, efficient and consistent across refreshes.

- Locale-aware collation:
  - Use a `Collator` for the user's locale.
  - Do not use naive `lowercase()` for ordering.
  - Prefer `android.icu.text.Collator` (API 24+) with numeric collation enabled so `"Track 2"` sorts before `"Track 10"`.
  - Cache the collator and precompute collation keys for large lists.
  - Required for non-Latin titles and for accented Latin titles.
- Locale change:
  - When the system locale changes, rebuild the collator and invalidate cached sort results.
- Unknown values:
  - Missing artist/album/year are "unknown" and placed last in both ascending and descending order.
  - Never fabricate metadata.
- Tie-breaker:
  - After the primary key, order by collated title, then by `id` to ensure stable order across refreshes.
- Numeric keys:
  - Duration, size, date added and year compare numerically.
- Queue independence:
  - Changing a sort must not reorder or modify the active playback queue.
  - Only a new "Play all / Shuffle" started from the visible list uses the displayed order.
- Performance:
  - Sort off the main thread.
  - Expose results as observable state.
  - Re-sort only when the library snapshot, sort preference or locale changes.
  - Never sort inside a `Composable` or on recomposition.

---
# Part B: Track Action Framework

## 10. Single Definition of Actions

All track actions are defined once and handled once. Screens never implement their own copies.

```kotlin
sealed interface TrackAction {
    data class Play(val trackId: Long) : TrackAction
    data class PlayNext(val trackIds: List<Long>) : TrackAction
    data class AddToQueue(val trackIds: List<Long>) : TrackAction
    data class SetFavorite(val trackIds: List<Long>, val favorite: Boolean) : TrackAction
    data class AddToPlaylist(val trackIds: List<Long>) : TrackAction
    data class RemoveFromPlaylist(val playlistId: Long, val itemIds: List<Long>) : TrackAction
    data class Share(val trackIds: List<Long>) : TrackAction
    data class SetSystemSound(val trackId: Long, val type: SystemSoundType) : TrackAction
    data class ShowDetails(val trackId: Long) : TrackAction
    data class ShowInFiles(val trackId: Long) : TrackAction
    data class GoToArtist(val trackId: Long) : TrackAction
    data class GoToAlbum(val trackId: Long) : TrackAction
    data class GoToFolder(val trackId: Long) : TrackAction
}

enum class SystemSoundType { RINGTONE, NOTIFICATION, ALARM }
```

A single `TrackActionHandler` (injected into ViewModels) maps actions to existing use cases:

| Action | Delegates to |
|---|---|
| Play / Play Next / Add to Queue | Existing `PlaybackController` queue API |
| Favorite | Existing favorites repository |
| Add to Playlist | Existing playlist-selection UI and repository |
| Share | `ShareTracksUseCase` |
| Set system sound | `SetSystemSoundUseCase` |
| Details | `GetTrackDetailsUseCase` |
| Go to ... | Existing navigation routes |

No duplicate playback, queue, favorite or playlist logic is permitted.

## 11. Menu Content and Context Rules

Standard menu order:

```text
Play
Play Next
Add to Queue
-----
Add to Favorites / Remove from Favorites
Add to Playlist
-----
Share
Set as Ringtone
Set as Notification Sound
Set as Alarm Sound
-----
Song Details
Show in Files
-----
Go to Artist
Go to Album
Go to Folder
```

Availability comes from one pure function, `availableActions(track, context)`:

| Rule | Behavior |
|---|---|
| Already on that album/artist/folder screen | Hide the matching "Go to ..." |
| Playlist screen | Add Remove from Playlist (removes the entry only, never the audio file) |
| Track unavailable or file missing | Disable Play, Share, Set as..., Details, Show in Files; still allow Remove from Playlist and Remove from Favorites |
| DRM-protected or unreadable file | Disable Share and all Set as... actions with a reason shown |
| Favorite state | Show Add or Remove according to actual state |
| Show in Files unsupported | Hide or disable |

## 12. Entry Points

The same `TrackAction` set and `availableActions` are used by: long-press menus, overflow menu buttons, Search results, Playlist/Folder/Album track menus, Now Playing compact menu, and the selection-mode action bar. There is one framework, never one per screen.

## 13. Play, Play Next, Add to Queue, Favorites, Playlists

- Play: uses the existing playback abstraction; never creates a separate player.
- Play Next: inserts after the current item through the existing queue API; preserves shuffle/repeat and queue semantics.
- Add to Queue: appends through the existing queue API.
- Favorites: the existing repository stays authoritative.
- Add to Playlist: opens the existing selection UI (including "+ New Playlist") and uses the existing playlist repository.

---
# Part C: Share

## 14. Behavior

Share sends the actual audio file through the system Sharesheet.

Rules:
- Use the track's MediaStore `content://` URI. A separate `FileProvider` is not required for MediaStore URIs.
- Build the intent with `Intent.ACTION_SEND`, `EXTRA_STREAM`, the MIME type, and always set:
  - `ClipData` containing the URI
  - `FLAG_GRANT_READ_URI_PERMISSION`
- Wrap with `Intent.createChooser()`.
- Never expose raw file paths, app-private paths or database paths.
- MIME fallback order:
  1. MediaStore `MIME_TYPE`
  2. `MimeTypeMap` by file extension
  3. `audio/*`
- Never hardcode `audio/mpeg` for everything.
- Do not log raw content URIs beyond sanitized debug output.

## 15. Batch Share

- Use `Intent.ACTION_SEND_MULTIPLE` with `EXTRA_STREAM` as an `ArrayList<Uri>`.
- Include `ClipData` containing all URIs and the read-grant flag.
- MIME:
  - if all selected tracks share one MIME type, use it
  - otherwise `audio/*`
- Cap: `SHARE_MAX_ITEMS = 100`.
- If the user selects more: "Select up to 100 tracks to share at once."
- Skip unavailable, DRM-protected and unreadable tracks and report how many were skipped.
- If no shareable items survive: "No valid files could be shared."

## 16. Failure Handling

| Condition | Message |
|---|---|
| File not accessible | "Unable to share this track. The original audio file could not be accessed." |
| No app can handle the intent | "No app is available to share this file." |
| Batch with skipped items | "Shared N tracks. M could not be accessed." |

No crash on any failure. Log a sanitized cause only.

---
# Part D: System Sounds (Ringtone, Notification, Alarm)

## 17. One Mechanism, Three Types

Ringtone, notification sound and alarm sound use the same API family:
- `RingtoneManager.setActualDefaultRingtoneUri(..., TYPE_RINGTONE)`
- `TYPE_NOTIFICATION`
- `TYPE_ALARM`

All require `WRITE_SETTINGS`. They live in `SystemSoundManager`. Ringtone ships first (P0), notification and alarm reuse the same code (P1).

What this changes:
- the device's default sound for that category

What it does not change:
- apps that set their own notification-channel sound (common on Android 8.0+)
- clock/alarm apps that keep a per-alarm sound
- per-contact ringtones
- per-SIM ringtone settings on dual-SIM or OEM-customized devices

Therefore, every success message must be worded as a default change, and the confirmation sheet should include a one-line caveat.

Whole-track playback:
- The entire selected track plays from the beginning; there is no trimmer or start-point picker.
- This is a non-goal. It should be stated in the confirmation sheet.

## 18. Permission Flow and Pending Action

The pending action must be stored before leaving for Settings, and must survive process death.

Required constants:

```kotlin
const val PENDING_SYSTEM_SOUND_TTL_MS = 10 * 60_000L
const val COPY_TIMEOUT_MS = 30_000L
const val INSPECTOR_TIMEOUT_MS = 5_000L
const val SHARE_MAX_ITEMS = 100
```

Required state model:

```kotlin
data class PendingSystemSound(
    val trackId: Long,
    val type: SystemSoundType,
    val createdAtMs: Long
)
```

Rules:
- Only one pending system-sound action may exist at a time.
- If the user attempts to request another while one is pending, show a message: "A system-sound change is already pending. Finish or cancel it first."
- Persist the pending action in `SavedStateHandle` (or equivalent saved-state mechanism) before launching Settings.
- Include `createdAtMs` so stale actions can be discarded on resume or process restore.
- If `System.currentTimeMillis() - createdAtMs > PENDING_SYSTEM_SOUND_TTL_MS`, clear the pending action and show a timeout message.

Flow:

```text
User taps "Set as Ringtone"
        │
        ▼
Source readable and not DRM-protected? ─ no ─► Failure: file unavailable
        │ yes
        ▼
Confirmation sheet
  "Set 'Song Name' as your default ringtone?"
  "The whole track plays from the start. Some apps and per-contact
   or per-SIM settings may not be affected."
  (API 29+: "A copy will be saved to Ringtones.")
        │ confirm
        ▼
Settings.System.canWrite() ? ─ no ─► 1. Save PendingSystemSound(trackId, type, createdAtMs)
        │ yes                          2. Explain, then launch ACTION_MANAGE_WRITE_SETTINGS
        │                              3. On RESUME: re-check canWrite()
        │                                   granted → run the pending action once and clear it
        │                                   not granted → clear it and show
        │                                   "Permission needed to change system sounds."
        ▼
Processing: "SETTING RINGTONE..."  (duplicate taps ignored)
        │
        ▼
copy (API 29+) or direct URI (API 26-28) → setActualDefault...Uri
        │
        ▼
Read back and compare (section 19)
```

Important:
- The pending action must be saved before leaving for Settings.
- The app must not assume the Settings result is reliable.
- `ACTION_MANAGE_WRITE_SETTINGS` may return without a reliable result code, so `canWrite()` is the only valid check on resume.

## 19. Strategy, Verification and Outcomes

### Source strategy by API level

| API level | Strategy |
|---|---|
| 29+ | Copy the track into the matching MediaStore audio collection (`RELATIVE_PATH` = `Ringtones/`, `Notifications/`, `Alarms/`), with `IS_RINGTONE` / `IS_NOTIFICATION` / `IS_ALARM` and `IS_PENDING` while writing. Set the copied entry as the default. |
| 26-28 | Use the track's original MediaStore URI directly (no copy), because writing to shared storage would require a legacy permission the app is not requesting. |
| lower than 26 | Not supported by the project baseline. |

Copy-on-29+ policy:
- Saves a copy in the OS-managed ringtone collection.
- Keeps the setting working if the original file moves or is deleted.
- Matches Android's standard behavior for ringtone collections.

### Verification: intended behavior and result states

After setting the default, read it back with `RingtoneManager.getActualDefaultRingtoneUri(type)`.

Algorithm:
1. Resolve both the intended URI and the read-back URI to a MediaStore id.
2. Compare IDs.
3. If they match: `Success`
4. If they do not match: `Failed`
5. If read-back is null, cannot be resolved, or the MediaStore entry is temporarily unavailable: `Unverified`

Do not delete the copy for `Unverified`; the change might have succeeded but the system did not surface a resolvable value. Show the soft message below.

| Outcome | Message |
|---|---|
| Ringtone success | "Default ringtone updated." |
| Notification success | "Default notification sound updated." |
| Alarm success | "Default alarm sound updated." |
| Unverified | "Your device accepted the request, but we could not confirm the change. Check your sound settings." |
| Permission not granted | "Permission needed to change system sounds." |
| Source missing, unreadable or DRM-protected | "This track's file could not be accessed." |
| System rejected / setter failed | "Could not set this track as the ringtone." (type adjusted per action) |
| Unsupported on this device | "This device doesn't support changing this sound." |
| Notification channel caveat | "Some apps use their own channel sound and may not be affected." |

## 20. Copy Registry (Room)

The app records every system-sound copy it creates so it can avoid duplicates and clean up later. This is a Room table, not DataStore.

```kotlin
@Entity(
    tableName = "system_sound_copies",
    indices = [Index("sourceTrackId", "type")]
)
data class SystemSoundCopyEntity(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    val sourceTrackId: Long,
    val type: String,              // RINGTONE | NOTIFICATION | ALARM
    val copiedUri: String,
    val copiedMediaId: Long,
    val relativePath: String,      // Ringtones/ | Notifications/ | Alarms/
    val displayName: String,
    val sizeBytes: Long,
    val createdAt: Long,
)
```

Rules:
- Add it to `DATA_MODEL.md` with a migration test.
- Exclude it from Auto Backup and device transfer. The URIs and MediaStore ids are device-specific and not portable.
- Reuse the existing copy if `(sourceTrackId, type)` already exists and the entry still exists in MediaStore.
- Fallback match:
  - same display name + same size + same collection path + same app owner,
  - only when exactly one candidate matches
- Cleanup safety:
  - delete only entries created by the app
  - skip the current default if it is still active
  - tell the user if the system default is currently in use
- On API 26-28, no copies are created, so no registry rows are written.

## 21. Copy Execution

- Copy runs on `Dispatchers.IO` with a loading state and duplicate-tap protection.
- Write with `IS_PENDING = 1`; clear it only after the copy succeeds.
- On any failure, delete the pending entry and clear the pending action.
- Honor cancellation: leaving the screen does not leave partial files.
- On timeout:
  - show a timeout banner or message
  - clear the pending action
  - clean up partial copy if one was created

## 22. Architecture

```text
UI (screen)
   ▲ one-shot effect (launch Settings / show message)
ViewModel ──► SetSystemSoundUseCase                  (domain)
                      │
                      ▼
              SystemSoundManager                     (data/system)
              ├─ canWrite(): Boolean
              ├─ prepareSource(track, type): Result<Uri>   // copy (29+) or direct URI (26-28)
              ├─ setDefault(type, uri): Result<Unit>
              ├─ readBack(type): ReadBackResult             // Success | Failed | Unverified
              └─ copy registry access (Room DAO)
```

Rules:
- No `RingtoneManager`, `Settings` or system-setting `ContentResolver` calls inside Composables or ViewModels.
- The domain result type is independent of Android classes: `Success`, `Unverified`, `NeedsPermission`, `SourceUnavailable`, `Unsupported`, `Failed(cause)`.
- `NeedsPermission` makes the ViewModel save the pending action and emit an effect; the screen launches Settings.

---
# Part E: Song Details

## 23. Data Model and Sources

MediaStore does not provide everything needed. Details load on demand off the main thread through a separate model. Do not add these fields to `Track` or to library scanning.

```kotlin
data class TrackDetails(
    val title: String?, val artist: String?, val album: String?,
    val albumArtist: String?, val genre: String?,
    val year: Int?, val trackNumber: Int?, val discNumber: Int?,
    val durationMs: Long?,
    val mimeType: String?, val formatLabel: String?,
    val bitrateKbps: Int?, val sampleRateHz: Int?, val channelCount: Int?,
    val fileName: String?, val sizeBytes: Long?, val relativePath: String?,
    val dateAddedSec: Long?, val dateModifiedSec: Long?,
)
```

| Field | Source |
|---|---|
| Title, artist, album, year, track number, duration, MIME type, size, date added/modified, file name, relative path | MediaStore (available on all supported APIs; `relativePath` on API 29+, otherwise derived from `DATA`) |
| Album artist, genre, disc number, bitrate | MediaStore columns only on API 30+; on API 26-29 use `MediaMetadataRetriever` |
| Sample rate, channel count | `MediaExtractor` / `MediaFormat` (`KEY_SAMPLE_RATE`, `KEY_CHANNEL_COUNT`); not available from MediaStore on any API |
| Format label | Derived from MIME type / container |

`AudioFileInspector` (in `data/media/`) wraps `MediaMetadataRetriever` and `MediaExtractor`. It:
- always releases resources
- runs on `Dispatchers.IO`
- enforces `INSPECTOR_TIMEOUT_MS`
- returns partial results when only some fields can be read
- never crashes the app

`GetTrackDetailsUseCase` merges MediaStore data with the inspector result.

## 24. Display Rules

- Show only values reliably obtained; omit the row or show "Unknown" per the design system.
- Never fabricate artist, album, year, bitrate, sample rate or size.
- Do not show the content URI.
- The relative path may be shown if valid and copyable.
- Include a "Copy path" action.
- Load with a short loading state.
- If the inspector fails, MediaStore fields still display.
- Use the existing menu actions: Share, Set as Ringtone, Add to Playlist, Favorite, Show in Files.

Example:

```text
SONG DETAILS

JAAT GELO
Band Lalon
────────────────────
ALBUM        Bi Protip
DURATION     04:32
FORMAT       MP3
BITRATE      320 kbps
SAMPLE RATE  44.1 kHz
CHANNELS     2
FILE SIZE    10.4 MB
LOCATION     Music/Lalon/   [COPY]
DATE ADDED   12 Oct 2026
```

---
# Part F: Show in Files (P2, Best Effort)

## 25. Behavior

Android has no standard "show this folder" guarantee, and results vary by file manager.

- Label the action "Show in Files".
- Build an `ACTION_VIEW` intent with a folder document URI derived from `RELATIVE_PATH` for primary external storage only.
- Use the directory MIME type `DocumentsContract.Document.MIME_TYPE_DIR`.
- Resolve the intent before launching. If it cannot be resolved or fails: "No compatible file manager is available."
- The dependable fallback is the copyable relative path in Song Details.
- Hide the action for removable volumes, invalid or unmappable paths, unsupported APIs, and private app paths.
- Never expose private paths in the UI.
- Never crash.

---
# Part G: Navigation, Multi-Select, Batch

## 26. Entity Navigation

"Go to Artist / Album / Folder" navigates through the existing navigation graph.

Rules:
- If the target no longer exists after a library refresh, show a short message and stay on the current screen.
- No duplicate screens or new graph branches.

## 27. Multi-Select

Applies to Tracks, Search results, Playlist tracks and Folder tracks. Long-press enters selection mode.

```text
3 SELECTED        [Select All] [Clear]
Share · Add to Queue · Add to Playlist · Favorite
```

Selection keys:

| Screen | Selection is keyed by |
|---|---|
| Tracks, Search, Folder, Album | Track id |
| Playlist detail | Playlist item id |

Rules:
- Selection is temporary UI state, never stored in Room or DataStore.
- Keying by id means selection survives re-sorting.
- When a library refresh removes tracks, drop the missing ids, update the count, and exit selection mode if empty.
- Back press exits selection mode first.
- Leaving the screen clears selection.
- Selection mode shows a contextual top bar; normal navigation is disabled while it is active.

## 28. Batch Actions

| Action | Notes |
|---|---|
| Add to Queue | Existing queue API; order follows visible list order |
| Add to Playlist | One picker for all selected |
| Favorite toggle | If any selected item is not a favorite, show Add; otherwise Remove |
| Share | Up to 100 URIs |
| Remove from Playlist | Playlist screens only; uses playlist item ids; never deletes audio |

Out of scope:
- batch delete / move / rename
- batch system-sound changes

## 29. Search, Playlist and Now Playing Integration

- Search results expose the same menu through `availableActions`; defaults to Relevance.
- Playlist tracks expose the same menu plus Remove from Playlist.
- Now Playing gets a compact overflow menu: Add to Favorites, Add to Playlist, Share, Song Details, Set as Ringtone.
- Main controls remain unchanged.

---
# Part H: Cross-Cutting Requirements

## 30. UI, Accessibility, Performance, Errors

### UI and accessibility
- Follow `DESIGN_SYSTEM.md` and `SCREEN_SPECIFICATIONS.md`.
- Use black/white base with a restrained red accent.
- Strong contrast and clear labels.
- 48 dp touch targets minimum.
- Explicit `contentDescription` values.
- Selection mode announces "N selected".
- Respect reduced-motion settings.
- No visual clutter.

### Performance
- Large libraries must not slow down.
- No repeated sorting or MediaStore queries on recomposition.
- Sorted lists are cached per (library snapshot, sort state, locale).
- Song Details extraction and copy operations run off the main thread.

### Errors
Every system integration returns a result type, with explicit success/failure behavior:

```text
Idle → Processing → Success | Unverified | Failure(reason)
```

Rules:
- User-readable, non-crashing, actionable where possible.
- No stack traces in the UI.
- Long operations show a loading state.
- Duplicate taps are ignored while processing.
- Sanitized cause is logged.

## 31. Launching Intents from the UI Layer

ViewModels and use cases must not call Activity APIs. Sharesheet, `ACTION_MANAGE_WRITE_SETTINGS`, and file-manager intents need an Activity context.

```kotlin
sealed interface UtilityEffect {
    data class LaunchShare(val intent: Intent) : UtilityEffect
    data class RequestWriteSettings(val pending: PendingSystemSound) : UtilityEffect
    data class OpenInFiles(val intent: Intent) : UtilityEffect
    data class ShowMessage(val message: UiText) : UtilityEffect
}
```

Rules:
- Each effect is consumed exactly once.
- Rotation must not replay it.
- The pending system-sound action is in saved state, not in the effect itself.
- The screen re-checks permission on resume and tells the ViewModel to run or clear the pending action.

## 32. Repository, Room, DataStore, Logging

- MediaStore data flows through the existing data source and repository.
- System integrations flow `UI → ViewModel → Use Case → System abstraction → Android framework`.
- Room:
  - the only new table is `system_sound_copies`
  - no Room tables for sort preferences, selection state, share state or temporary UI state
- DataStore:
  - holds the sort preferences
- Logging:
  - log share failures, write-settings failures, unreadable-file failures, unsupported operations, and permission outcomes
  - do not log paths, audio contents, or private data
  - follow project release logging policy

## 33. Android Version Compatibility

- Detect unsupported operations.
- Use supported APIs.
- Avoid deprecated APIs when a replacement exists.
- Never report success when Android rejected the operation.
- Handle version-specific differences:
  - copy behavior on API 29+ vs direct URI on API 26-28
  - MediaStore columns only available on API 30+ for Song Details
  - `WRITE_SETTINGS` flow and its lack of a result code
  - file-manager capability differences
  - ICU `Collator` numeric collation availability (API 24+)

---
# Part I: Testing and Delivery

## 34. Automated Tests

| Area | Required Cases |
|---|---|
| Sorting | Each key and direction per entity; locale-aware ordering with Bangla and accented titles; numeric collation ("2" before "10"); locale-change invalidation; unknown last in both directions; stable tie-breaker; `sort_order` migration (idempotent); queue unaffected by sort change |
| Action framework | `availableActions` for library, search, playlist, album/artist/folder screens, unavailable track, DRM/unreadable track; handler delegates correctly |
| Share | URI and MIME fallback resolution; `ClipData` and grant flag always present; missing file; invalid URI; multi-URI intent; 100-item cap; skipped-track count |
| System sound | Success; Unverified; Failed with partial-copy cleanup; permission-needed; source missing/DRM; unsupported; read-back compared by media id with differently formatted URIs; copy reuse; registry row create/update; API-specific strategy via fakes; cleanup skips the current default |
| Pending action | Saved before Settings; restored after process death; runs exactly once after grant; cleared when not granted; permission decided by `canWrite()` on resume |
| Song Details | Merge MediaStore + inspector; API 26-29 vs 30+ field sources; partial inspector failure; timeout; no fabricated values |
| Multi-select | Selection survives re-sort; ids dropped on refresh; playlist selection by item id with duplicate tracks; favorite toggle rule; exit on back |
| Effects | Consumed once; no re-fire on rotation |
| Room | `system_sound_copies` DAO and migration test; `Auto Backup` exclusion configuration |

Unit tests do not replace device testing for system APIs.

## 35. Physical Device Tests

Required physical tests:
- Sorting with non-Latin titles
- Share with at least two receiving apps
- Ringtone set and verify in system settings
- Notification sound change and verify in system settings
- Alarm sound change and verify in system settings
- Permission grant and deny flows
- Track unavailable / unreadable / DRM-protected file handling
- Song details with missing tags
- File manager support / Show in Files
- Multi-select on a list with >20 items
- Large batch share boundary: 99, 100, 101
- External SD card / removable volume behavior
- Work profile / multi-user scenario if available
- Accessibility pass with TalkBack
- OEM customization pass on at least one non-stock/modified launcher or OEM skin if available

## 36. Regression Testing

All existing functionality must continue to pass:
- library
- search
- playback
- queue
- shuffle
- repeat
- favorites
- playlists
- history
- sleep timer
- widgets
- MediaSession
- notification
- lock-screen controls
- Bluetooth/headphone interruption
- process recreation
- background playback
- Room persistence

Any prior manual or automated verification gaps must be listed as `NOT VERIFIED` in the QA report until actually run.

Previously unverified items:
- Headphone unplug and Bluetooth disconnect behavior
- Phone-call audio-focus interruption
- Reboot and headset-button playback resumption
- Lock-screen controls
- 10,000-track queue test
- TalkBack accessibility pass
- Work-profile / multi-user behavior

## 37. Build Verification

```bash
./gradlew testDebugUnitTest --rerun-tasks
./gradlew assembleDebug
./gradlew assembleRelease
./gradlew lintVitalRelease
```

No new fatal lint errors.

## 38. Definition of Done (per feature)

A feature is done only when:
- architecture still matches the project structure
- no duplicate playback architecture
- UI follows the design system and accessibility rules
- automated tests pass
- physical-device verification is completed for system integrations
- no unnecessary permissions are added
- offline behavior is preserved
- failure states are handled
- `AGENT_STATE.md`, `CHANGELOG.md`, `PRODUCT_SPEC.md`, `DATA_MODEL.md` and the privacy/About text are updated

## 39. Completion Matrix

| Feature | Priority | Automated | Physical |
|---|---|---|---|
| Track sorting | P0 | Required | Required |
| Track action framework and menu | P0 | Required | Required |
| Share audio | P0 | Required | Required |
| Song Details | P0 | Required | Required |
| Set as ringtone | P0 | Required | Required |
| Notification sound | P1 | Required | Required |
| Alarm sound | P1 | Required | Required |
| Multi-select | P1 | Required | Required |
| Batch actions | P1 | Required | Required |
| Entity navigation | P1 | Required | Required |
| Show in Files | P2 | Required | Best effort |

## 40. Milestones

All P0 features ship before P1. After each milestone:
- compile
- run relevant tests
- inspect changes
- update `AGENT_STATE.md` and `CHANGELOG.md`
- do not combine milestones into one uncontrolled change

```text
M16  Library Sorting
M17  Track Action Framework
M18  Share
M19  Song Details
M20  System Sounds: Ringtone
M21  System Sounds: Notification + Alarm
M22  Multi-Select + Batch Actions + Entity Navigation
M23  Show in Files + UI polish
M24  Automated + Physical Regression QA
M25  Release Candidate 2
```

## 41. State Accuracy and Release Protection

- Never claim "Complete", "Verified", "Passed" or "Release Ready" unless validation actually happened.
- If physical verification was not performed, write `Physical Verification: NOT VERIFIED`.
- Do not infer device success from unit tests.
- Do not begin release signing while these features are changing.
- After M24 passes, create `RELEASE CANDIDATE 2`; only then resume release engineering.

---

## 42. Explicit Non-Goals

Streaming, online catalogs, YouTube integration, cloud sync, accounts, lyrics scraping, AI recommendations, audio editing or trimming, equalizer/DSP, tag editing, album-art editing, batch file delete/move/copy/rename, social features, music recognition, online metadata refresh.

---

## 43. Implementation Rules for the Coding Agent

Before changing code, read:
- `AGENT_STATE.md`
- `PRODUCT_SPEC.md`
- `ARCHITECTURE.md`
- `DESIGN_SYSTEM.md`
- `SCREEN_SPECIFICATIONS.md`
- `INTERACTION_SPEC.md`
- `DATA_MODEL.md`
- `PLAYBACK_ARCHITECTURE.md`
- `TEST_PLAN.md`
- this document

Then audit the current implementation.

Do not assume a feature is missing because it is listed here. If part of a feature already exists, extend or reuse it instead of duplicating it.

---

## 44. Final Product Goal

```text
DISCOVER → SORT → SEARCH → PLAY → QUEUE → FAVORITE → PLAYLIST
        → SHARE → SET SYSTEM SOUND → INSPECT DETAILS → MANAGE LIBRARY
```

The app stays fast, local, private, mechanical and focused. No feature may compromise playback reliability or the offline-first philosophy.

---

## 45. Open Decisions (owner sign-off required)

1. API 26-28 sound strategy:
   - This spec uses the original URI directly on 26-28.
   - Confirm or revise if you want a legacy write-permission path instead.
2. Copy cleanup UI:
   - Whether to ship a "Remove created system sound copies" action in Settings in v1 or defer it.
3. Share cap:
   - 100 by default; adjust after actual binder-limit verification.
4. Show in Files:
   - Keep as P2 best effort or drop it if device testing shows poor file-manager support.
5. Song Details UI:
   - Sheet or screen; choose one and keep it consistent with `SCREEN_SPECIFICATIONS.md`.
6. Numeric collation:
   - Confirm if numeric ordering is wanted in every title sort.
7. Pending cleanup UI:
   - Whether to show a "Cancel pending sound change" action in Settings flow.

---

## 46. Document Updates Required When Adopting This Spec

| Document | Update |
|---|---|
| `PRODUCT_SPEC.md` | Add `WRITE_SETTINGS` to the permission table; add the new feature requirements |
| `DATA_MODEL.md` | Add `system_sound_copies`; add backup-exclusion note; add the new DataStore sort keys; mark `sort_order` as migrated |
| `PLAYBACK_ARCHITECTURE.md` | No change expected; note that queue order is unaffected by sort |
| `TEST_PLAN.md` | Add the cases from sections 34-36 |
| Privacy / About screen | Explain why "Modify system settings" is requested and when |
| `CHANGELOG.md` / `AGENT_STATE.md` | Record each milestone as it completes |

---

## 47. Change List

### v4 → v5
| Area | Change |
|---|---|
| Pending action | Added TTL and single-pending-action rule |
| Settings return | Requires `SavedStateHandle` persistence before leaving Settings and re-check of `canWrite()` on resume |
| Read-back | Clarified algorithm using MediaStore id resolution; `Unverified` outcome added and full semantics defined |
| Timeouts | Added `COPY_TIMEOUT_MS`, `INSPECTOR_TIMEOUT_MS`, `SHARE_MAX_ITEMS`, `PENDING_SYSTEM_SOUND_TTL_MS` |
| Backup policy | States that `system_sound_copies` is excluded from Auto Backup and transfers |
| Work profile / multi-user | Added physical QA requirement |
| OEM caveat | Added test requirement for OEM-customized devices |
| DRM | Added read-only / DRM / unreadable file handling to the share and Set as... logic |
| Copy registry | Clarified reuse and cleanup rules, plus safe deletion policy |
| Share | Clarified mandatory `ClipData` and read-grant flags |
| QA / release | Added specific unverified items and stricter language around not reporting success without verification |

### v1 → v5
`WRITE_SETTINGS` added; ringtone/notification/alarm merged into a single mechanism; copy vs direct URI strategy by API; `AudioFileInspector` and `TrackDetails`; Show in Files downgraded to P2; ViewModel-effect pattern for intents; per-entity sort keys, unknown-last rule, tie-breaker and queue independence; single `TrackAction` framework; milestones reordered with P0 first; pending action TTL and re-check rules added; backup exclusion and device-specific caveats added.

End of document.
