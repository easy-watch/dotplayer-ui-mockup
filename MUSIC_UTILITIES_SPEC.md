# MUSIC UTILITIES SPECIFICATION

Project: MusicPlayer
Document: MUSIC_UTILITIES_SPEC.md
Version: 3 (revised after implementation review)
Status: Proposed: Feature Expansion
Purpose: Define functional, architectural, UI, system-integration, and testing requirements for everyday music utilities without destabilizing the Release Candidate baseline.

Related docs:
- PRODUCT_SPEC.md
- ARCHITECTURE.md
- DESIGN_SYSTEM.md
- SCREEN_SPECIFICATIONS.md
- INTERACTION_SPEC.md
- DATA_MODEL.md
- PLAYBACK_ARCHITECTURE.md
- TEST_PLAN.md

> Revision note (v3):
> This revision incorporates:
> - explicit handling for WRITE_SETTINGS and pending-action re-entry after Settings
> - exact API-level behavior for copying vs direct URI for system sounds
> - registry tracking for created ringtone/notification/alarm copies
> - clearer failure/success read-back verification
> - better null/unsupported handling for DRM and unreadable files
> - explicit notification-channel limits under Android O+
> - stronger sorting guidance with locale-aware collation and deterministic tie-breakers
> - a single action framework for all track utilities
> - improved P0/P1 ordering and milestone structure

---

## 1. Scope

This document covers:
- sorting (tracks, albums, artists, folders, playlists, search)
- track action framework and menu
- share (single and batch)
- set as ringtone / notification sound / alarm sound
- song details
- show in files (best effort)
- multi-select and batch actions
- entity navigation shortcuts
- Android system integration, error handling, persistence, accessibility, and testing

This specification is implementation-focused. It does not describe only mockups.

---

## 2. Protected Baseline

The current Release Candidate is the protected baseline. New features must not unnecessarily modify or replace:
- Media3 ExoPlayer architecture and service ownership
- MediaSession / MediaLibraryService / MediaController communication
- PlaybackController design and authoritative player ownership
- audio focus handling and audio-becoming-noisy behavior
- background playback, queue synchronization, shuffle, repeat, and resumption
- MediaStore scanning and ContentObserver architecture
- Room and DataStore architecture
- library navigation, Now Playing behavior, widgets, and visualizer
- the privacy model, except for the single documented addition in section 4

New utilities must communicate through existing domain/repository abstractions. They must never create a second ExoPlayer or a second authoritative playback instance.

---

## 3. Product Philosophy

MusicPlayer is a fast, private, offline-first local music player with excellent Android system integration. Utilities must remain:
- offline-first
- fast
- minimal
- predictable
- privacy-preserving
- visually consistent with the existing design system

No accounts, online services, analytics, or network dependencies are allowed.

---

## 4. Permissions and Privacy Impact

This expansion adds one special-access permission. It must be documented in PRODUCT_SPEC.md and the privacy/About screen.

| Permission | Needed for | Type | When requested |
|---|---|---|---|
| READ_MEDIA_AUDIO (existing) | library | runtime | existing flow |
| POST_NOTIFICATIONS (existing) | playback notification | runtime | existing flow |
| WRITE_SETTINGS (new) | set default ringtone, notification sound, or alarm sound | special app access: Modify system settings | only when the user first chooses a Set as... action |

Rules:
- Declare:
  <uses-permission android:name="android.permission.WRITE_SETTINGS" />
- Check Settings.System.canWrite(context) before every system-sound operation.
- If permission is not granted:
  - show an explanation
  - launch Settings.ACTION_MANAGE_WRITE_SETTINGS with the app package URI
  - re-check permission on return
  - continue only if granted
- If user declines:
  - feature stays unavailable with a clear message
  - nothing else in the app is affected
- Still forbidden:
  - INTERNET
  - RECORD_AUDIO
  - location
  - contacts
  - camera
  - broad storage permissions
- On API 29+ no additional storage permission is needed for MediaStore-managed copies.

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

Song Details is placed ahead of ringtone work because it has no special permission and is the lowest-risk P0 utility. All P0 features ship before P1.

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

All entities support Ascending and Descending. The UI must clearly show the selected key and direction.

Example:
- Sort Tracks
  - Title
  - Artist
  - Album
  - Duration
  - Date Added
  - Year
  - File Size
- Order:
  - Ascending
  - Descending

Inside detail screens:
- Album detail stays in disc/track-number order
- Playlist detail stays in user-defined order
- Folder detail uses the Tracks sort preference

## 7. Defaults and Upgrade Safety

- Default sort is Title ascending, unless the app already defines a different ordering.
- Existing DataStore key sort_order is migrated into trackSortKey / trackSortDirection on first launch of the new version, so user library order is not silently reordered.
- Migration must be idempotent and covered by a test.

## 8. Persistence

Persist sort state in DataStore, not Room:
- trackSortKey
- trackSortDirection
- albumSortKey
- albumSortDirection
- artistSortKey
- artistSortDirection
- folderSortKey
- folderSortDirection
- playlistSortKey
- playlistSortDirection

Search sort is not persisted; it resets to Relevance for each search session.

## 9. Sorting Requirements

Sorting must be deterministic, stable, null-safe, efficient, and consistent across refreshes.

- Locale-aware collation:
  - Use java.text.Collator or ICU Collator for the user's locale
  - Do not use naive lowercase() for ordering
  - Cache the Collator instance
  - Precompute collation keys when sorting large lists
  - This is required for non-Latin titles (for example Bangla, Arabic, accented Latin titles)
- Unknown values:
  - missing artist/album/year are treated as "unknown" and placed last in both ascending and descending order
  - do not fabricate metadata to overcome nulls
- Tie-breaker:
  - after primary key, sort by title (collated), then by id
  - this ensures stable order across refreshes
- Numeric keys:
  - duration, size, date added, year use numeric comparison
- Queue independence:
  - changing sort must not reorder or mutate the active playback queue
  - only a new “Play all / Shuffle” action started from the visible list uses displayed order
- Performance:
  - sorting runs off the main thread
  - sorted lists are exposed as observable state
  - re-sort only when the snapshot or sort preference changes
  - do not sort inside a Composable or on recomposition
- Collator cache invalidation:
  - if the locale changes, rebuild the collator and sort cache
  - a locale change should invalidate in-memory sort results

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

A single TrackActionHandler (injected into ViewModels) maps actions to existing use cases:

| Action | Delegates to |
|---|---|
| Play / Play Next / Add to Queue | existing PlaybackController queue API |
| Favorite | existing favorites repository |
| Add to Playlist | existing playlist selection UI and repository |
| Share | ShareTracksUseCase |
| Set system sound | SetSystemSoundUseCase |
| Details | GetTrackDetailsUseCase |
| Go to ... | existing navigation routes |

No duplicate playback, queue, favorite, or playlist logic is permitted.

## 11. Menu Content and Context Rules

Standard menu order:
1. Play
2. Play Next
3. Add to Queue
4. divider
5. Add to Favorites / Remove from Favorites
6. Add to Playlist
7. divider
8. Share
9. Set as Ringtone
10. Set as Notification Sound
11. Set as Alarm Sound
12. divider
13. Song Details
14. Show in Files
15. divider
16. Go to Artist
17. Go to Album
18. Go to Folder

Availability depends on context via a pure function:
- availableActions(track, context)

Rules:
- If already on the album/artist/folder screen, hide the matching “Go to ...”
- Playlist screen adds Remove from Playlist
- Unavailable track:
  - disable Play, Share, Set as..., Details, Show in Files
  - allow Remove from Playlist and Remove Favorite
- Favorite state:
  - show Add or Remove according to actual state
- Show in Files:
  - hide or disable on unsupported devices / APIs
- System-sound actions:
  - disable on unreadable or missing track
  - provide reason in confirmation sheet or toast

## 12. Entry Points

Use the same TrackAction set in:
- long press menus
- overflow menus
- Search result menus
- Playlist / Folder / Album track menus
- mini-player and Now Playing menu
- selection mode context bar

All entry points must share one action framework.

## 13. Play, Play Next, Add to Queue, Favorites, Playlists

- Play:
  - uses existing playback abstraction
  - never creates a separate player
- Play Next:
  - inserts directly after current item using existing queue API
  - preserves queue semantics including shuffle/repeat
- Add to Queue:
  - appends using queue API
- Favorites:
  - existing repository remains authoritative
- Add to Playlist:
  - opens existing selection UI
  - includes “+ New Playlist”
  - reuses existing playlist repository

---
# Part C: Share

## 14. Behavior

Share sends the actual audio file through the system Sharesheet.

Rules:
- Use each track’s MediaStore content URI
- Do not expose app-private paths
- Use Intent.ACTION_SEND or ACTION_SEND_MULTIPLE
- Use EXTRA_STREAM with a valid URI
- Add ClipData to the intent when needed
- Set FLAG_GRANT_READ_URI_PERMISSION
- Use correct MIME type from MediaStore MIME_TYPE if possible
- fallback order:
  1. MediaStore MIME_TYPE
  2. MimeTypeMap from extension
  3. audio/*
- Never hardcode audio/mpeg for everything
- Do not log file paths or raw content URIs beyond sanitized debug logs

## 15. Batch Share

- use ACTION_SEND_MULTIPLE
- `EXTRA_STREAM` is an ArrayList<Uri>
- `ClipData` contains all URIs
- MIME type:
  - if all selected items share a MIME type, use it
  - otherwise use audio/*
- cap:
  - max 100 tracks per share
  - constant: SHARE_MAX_ITEMS = 100
- if user selects more than the cap:
  - show message: “Select up to 100 tracks to share at once.”
- skip unavailable tracks and report how many were skipped
- if no shareable items survive:
  - show “No valid files could be shared.”

## 16. Failure Handling

| Condition | Message |
|---|---|
| File not accessible | “Unable to share this track. The original audio file could not be accessed.” |
| No app can handle the intent | “No app is available to share this file.” |
| Failed batch share | “Shared N tracks. M could not be accessed.” |
| Unsupported share target | “This file format cannot be shared from this device.” |

No crash on any failure. Log sanitized cause only.

---
# Part D: System Sounds (Ringtone, Notification, Alarm)

## 17. One Mechanism, Three Types

Ringtone, notification sound, and alarm sound all use the same API family:
- RingtoneManager.setActualDefaultRingtoneUri(..., TYPE_RINGTONE)
- TYPE_NOTIFICATION
- TYPE_ALARM

They all require WRITE_SETTINGS. They are implemented in SystemSoundManager and shipped together.

What it changes:
- the device’s default sound for that category
- not a per-app channel override
- not a per-contact ringtone override
- not a per-alarm override inside third-party alarm apps

Important device caveat:
- on Android O+ many apps use their own notification-channel sound and ignore the user’s default notification tone
- the app must not claim that all notifications will change when the system default notification sound is updated

## 18. Strategy: Copy vs Direct URI

| API level | Strategy |
|---|---|
| 29+ | Copy track to the appropriate system collection and set the copied entry as the default |
| 26-28 | Use original MediaStore URI directly when possible, because legacy storage permission is not requested |
| lower than 26 | Not supported by the project baseline |

### API 29+ copy policy
When changing a ringtone/notification/alarm:
- copy the track into:
  - Ringtones/
  - Notifications/
  - Alarms/
- use MediaStore audio collection with proper flags
- set the copied entry as the default
- do not require broad storage permission
- do not deny a feature because the user has no writable folder; use standard system collections

### Why copy on 29+
This keeps the setting stable if the original file moves or is deleted and matches Android system expectations for ringtone collections.

## 19. Copy Reuse and Registry

The app must keep track of copies it created so it can:
- avoid duplicates
- detect preexisting copy from the same source
- delete created copies later if a cleanup feature is added

Recommended registry:
- a small app-managed registry in Room or DataStore to track:
  - sourceTrackId
  - sourceUri
  - copiedUri
  - copiedType (RINGTONE / NOTIFICATION / ALARM)
  - systemCollectionPath
  - createdAt
  - sizeBytes
- if the same source track is set again:
  - reuse the existing copiedUri if it still exists
  - else create a new copy
- matching rules:
  1. exact sourceTrackId match is preferred
  2. fallback: same file name + size + same collection path + same app owner
  3. choose only a single unambiguous result

This should be in a dedicated table or small datastore index, not ad hoc in memory only.

## 20. Permission Flow and Pending Action Pattern

```text
User taps "Set as Ringtone"
        │
        ▼
Source readable? ── no ─► Failure: file unavailable
        │
        └─ yes
            │
            ▼
        Confirmation sheet
        "Set 'Song Name' as your default ringtone?"
        (API 29+: "A copy will be saved in Ringtones.")
            │
            ▼
        canWrite() ? ── no ─► Explain + ACTION_MANAGE_WRITE_SETTINGS
            │
            └─ yes
                │
                ▼
            Pending action stored
            ▼
            Processing: "SETTING RINGTONE..."
            │
            ▼
            readBack check
            │
            ├─ success → success message
            └─ failure → failure message and cleanup of partial copy
```

Critical rule:
- the app must not lose the pending action when returning from Settings
- ViewModel or use-case layer should save a one-shot pending action as a small ephemeral state object
- on permission grant, re-run exactly once
- if the user leaves Settings without granting, do not retry automatically
- if permission is denied, show “Permission needed to change system sounds.”

## 21. Success Verification

Never claim success unless the read-back confirms the change.
- after setActualDefaultRingtoneUri / setActualDefaultNotificationUri / setActualDefaultAlarmUri
- read back the default value from the system
- compare to the URI that was actually written
- if mismatch:
  - treat as failure
  - if a copy was created, delete the partial entry
  - show failure message

### Messages
| Outcome | Message |
|---|---|
| ringtone success | “Default ringtone updated.” |
| notification success | “Default notification sound updated.” |
| alarm success | “Default alarm sound updated.” |
| permission not granted | “Permission needed to change system sounds.” |
| source missing/unreadable | “This track’s file could not be accessed.” |
| system rejected change | “Could not set this track as the ringtone.” (type adjusted per action) |
| unsupported on device | “This device does not support changing this sound.” |
| notification channel caveat | “Some apps use their own channel sound and may not be affected.” |

## 22. Copy and Cleanup Behavior

Copy rules:
- copy runs on IO
- shows loading state
- blocks duplicate taps
- if copy fails midway:
  - remove partial file
  - clear pending action
- if the copied file exists and matches the source:
  - reuse it
- if the user wants to remove created copies:
  - use the app-managed registry and delete only files created by the app

Important:
- The app must not delete files it did not create.
- This is a privacy-preserving and predictable policy.

## 23. Architecture

```text
UI
  └─ ViewModel
      └─ SetSystemSoundUseCase
          └─ SystemSoundManager
              ├─ canWrite()
              ├─ prepareSource(track, type)
              ├─ setDefault(type, uri)
              ├─ readBack(type)
              └─ created-copy registry
```

Rules:
- no RingtoneManager or Settings calls from Composables or ViewModels
- the domain layer exposes result types independent of Android classes
- view-model emits effect to launch Settings if permission is missing
- data layer performs copy and resolution

---
# Part E: Song Details

## 24. Data Model

MediaStore does not provide everything needed. Details are loaded on demand off the main thread through an inspector.

```kotlin
data class TrackDetails(
    val title: String?,
    val artist: String?,
    val album: String?,
    val albumArtist: String?,
    val genre: String?,
    val year: Int?,
    val trackNumber: Int?,
    val discNumber: Int?,
    val durationMs: Long?,
    val mimeType: String?,
    val formatLabel: String?,
    val bitrateKbps: Int?,
    val sampleRateHz: Int?,
    val channelCount: Int?,
    val fileName: String?,
    val sizeBytes: Long?,
    val relativePath: String?,
    val dateAddedSec: Long?,
    val dateModifiedSec: Long?
)
```

| Field | Source |
|---|---|
| title, artist, album, year, track number, duration, MIME, size, date added/modified, file name, relative path | MediaStore |
| albumArtist, genre, disc number, bitrate | MediaStore columns when available, otherwise MediaMetadataRetriever |
| sample rate, channel count | MediaExtractor / MediaFormat |
| format label | derived from MIME / container |

`AudioFileInspector` (in data/media/) wraps MediaMetadataRetriever and MediaExtractor. It:
- always releases resources
- runs on Dispatchers.IO
- uses timeouts
- returns partial results when some fields are unreadable
- never crashes the app

`GetTrackDetailsUseCase` merges MediaStore metadata with inspector output.

## 25. Display Rules

- Show only values reliably obtained
- omit or show “Unknown” as appropriate
- never fabricate metadata
- do not display the content URI
- relative path may be shown if valid
- include a “Copy path” action if needed
- load with a short loading state
- use existing menu actions: Share, Add to Playlist, Favorite, Set as Ringtone, Show in Files

Example:
- SONG DETAILS
- Title
- Artist
- Album
- Duration
- Format
- Bitrate
- Sample Rate
- Channels
- File Size
- Location
- Date Added

## 26. Song Details Entry Points

- context menu from any track row
- overflow menu in Now Playing
- details screen or sheet
- selection mode “Details” if exactly one item selected

---
# Part F: Show in Files (P2, Best Effort)

## 27. Behavior

Android has no standard “show this file in a file manager” guarantee. Therefore:

- label action as “Show in Files”
- build an ACTION_VIEW intent using a folder document URI derived from relative path
- resolve the intent before launching
- if no file manager can handle it:
  - show: “No compatible file manager is available.”
- reliable fallback:
  - show the relative path in Song Details with Copy action
- hide the action for:
  - removable volumes
  - invalid or unmappable paths
  - unsupported APIs
  - private app paths
- never expose private paths in the UI
- never crash on failure

This is P2 and best effort.

---
# Part G: Navigation, Multi-Select, Batch

## 28. Entity Navigation

“Go to Artist / Album / Folder” navigates using the existing navigation graph.

Rules:
- if the target no longer exists after a refresh, show a short message
- keep the user on the current screen
- no duplicate screens or new graph branches

## 29. Multi-Select

Applies to:
- Tracks
- Search results
- Playlist tracks
- Folder tracks

Long-press enters selection mode.

Selection mode requirements:
- selection state is temporary UI state only
- selection is tracked by track id, not UI position
- selection survives sort changes as long as track ids remain valid
- if library refresh removes some items:
  - drop missing ids
  - update count
  - exit selection if empty
- back press exits selection first
- selection mode shows a contextual top bar; normal navigation is disabled while selection is active

## 30. Batch Actions

| Action | Notes |
|---|---|
| Add to Queue | existing queue API, selection order preserved |
| Add to Playlist | one picker for all selected |
| Favorite toggle | if any selected item is not favorite, show Add; otherwise Remove |
| Share | up to 100 URIs |
| Remove from Playlist | direct playlist remove, not file delete |

Out of scope:
- batch delete / move / rename
- system-sound batch changes

## 31. Search Results and Playlist Integration

- Search results expose the same menu and action framework
- Playlist tracks expose same menu plus Remove from Playlist
- sorting defaults to Relevance for search
- playlist sort and folder sort remain separate from search sort

---
# Part H: Cross-Cutting Requirements

## 32. UI and Accessibility

- follow DESIGN_SYSTEM.md and SCREEN_SPECIFICATIONS.md
- black/white base with restrained red accent
- strong contrast
- clear labels
- 48 dp touch targets minimum
- explicit content descriptions for actions
- selection mode announces “N selected”
- respect reduced-motion settings
- ensure no visual clutter

## 33. Performance

- large libraries must not slow down
- no repeated sorting on recomposition
- no repeated MediaStore queries on each render
- sort results are cached by snapshot + sort state
- details extraction is off-main-thread
- copy operations are off-main-thread

## 34. Error Handling

Every system integration must have explicit success/failure behavior with a result type:
- Idle → Processing → Success
- Idle → Processing → Failure(reason)

Failure handling rules:
- user-readable, non-crashing, actionable when possible
- no stack traces in the UI
- long operations show a loading state
- duplicate taps are ignored while processing
- log sanitized failure cause

## 35. Launching Intents from the UI Layer

ViewModels and use cases must not call Activity APIs. Sharesheet, ACTION_MANAGE_WRITE_SETTINGS, and file-manager intents require Activity context. Pattern:

```kotlin
sealed interface UtilityEffect {
    data class LaunchShare(val intent: Intent) : UtilityEffect
    data class RequestWriteSettings(val pendingAction: TrackAction) : UtilityEffect
    data class OpenInFiles(val intent: Intent) : UtilityEffect
    data class ShowMessage(val message: UiText) : UtilityEffect
}
```

Rules:
- effects are consumed exactly once
- pending action is re-run after permission is granted
- rotation must not replay effect unexpectedly
- the screen collects effects and calls startActivity / launcher

## 36. Repository, Room, DataStore, Logging

- MediaStore data flows through existing data source and repository
- no new Room tables for sort preferences or temporary UI state
- DataStore holds sort preferences
- system-sound copy registry is stored in Room or DataStore, not ad hoc memory
- log share failures, write-settings failures, unreadable-file failures, unsupported operations, and permission outcomes
- do not log private paths or user audio contents
- follow the project’s release logging policy

## 37. Android Version Compatibility

- detect and handle unsupported operations
- use supported APIs
- avoid deprecated APIs when a replacement exists
- handle version-specific differences:
  - copy behavior on 29+ vs direct URI on 26-28
  - MediaStore column availability on API 30+
  - WRITE_SETTINGS behavior across Android versions
  - file manager capability differences
- never report success if Android rejected the operation

---
# Part I: Testing and Delivery

## 38. Automated Tests

| Area | Required Cases |
|---|---|
| Sorting | each key and direction per entity; locale-aware ordering with Bangla and accented titles; null/unknown last; tie-breaker; migration from sort_order; queue unaffected by sort change |
| Action framework | availableActions for library, search, playlist, album/artist/folder, unavailable track; handler delegates correctly |
| Share | URI/MIME resolution, invalid URI, missing file, multi-URI intent, 100-item cap, skipped-track count |
| System sound | success, permission-needed, source missing, unsupported, read-back mismatch, copy reuse, partial copy cleanup, API-specific strategy |
| Song Details | merge MediaStore + inspector, partial failure, no fabricated values, timeout |
| Multi-select | selection survives sort change, ids removed on refresh, favorite toggle rule, selection exited on back |
| Effects | effect consumed once; no re-fire on rotation; pending action resumes after permission |
| Migration | sort_order migration is idempotent and deterministic |

## 39. Physical Device Tests

Required physical tests:
- sorting with non-Latin titles
- share with at least two receiving apps
- ringtone set and verify in system settings
- notification sound change and verify in system settings
- alarm sound change and verify in system settings
- permission grant and deny flows
- track unavailable and unreadable file handling
- song details with missing tags
- file manager support / Show in Files
- multi-select on a list with >20 items
- large batch share boundary (99, 100, 101)
- external SD card / removable volume behavior
- work profile / multi-user scenario if available
- accessibility pass with TalkBack

## 40. Regression Testing

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

Any prior manual or automated verification gaps must be explicitly recorded as NOT VERIFIED until actually run.

## 41. Build Verification

```bash
./gradlew testDebugUnitTest --rerun-tasks
./gradlew assembleDebug
./gradlew assembleRelease
./gradlew lintVitalRelease
```

No new fatal lint errors.

## 42. Definition of Done (per feature)

A feature is done only when:
- architecture still matches the project structure
- no duplicate playback architecture
- UI follows the design system and accessibility rules
- automated tests pass
- physical-device verification is completed for system integrations
- no unnecessary permissions are added
- offline behavior is preserved
- failure states are handled
- AGENT_STATE.md and CHANGELOG.md are updated

## 43. Completion Matrix

| Feature | Priority | Automated | Physical |
|---|---|---|---|
| Track sorting | P0 | Required | Required |
| Track action framework | P0 | Required | Required |
| Share audio | P0 | Required | Required |
| Song Details | P0 | Required | Required |
| Set as ringtone | P0 | Required | Required |
| Set as notification sound | P1 | Required | Required |
| Set as alarm sound | P1 | Required | Required |
| Multi-select | P1 | Required | Required |
| Batch actions | P1 | Required | Required |
| Entity navigation | P1 | Required | Required |
| Show in Files | P2 | Required | Best effort |

## 44. Milestones

All P0 features ship before P1. Compile, run relevant tests, inspect changes, and update AGENT_STATE.md and CHANGELOG.md after each milestone.

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

## 45. State Accuracy Rule

Never claim “Complete”, “Verified”, “Passed”, or “Release Ready” unless validation actually happened. If physical verification was not performed:
- write: Physical Verification: NOT VERIFIED
- do not infer success from unit tests

---

## 46. Release Protection

Do not begin release signing while these features are still changing. After M24 passes, create RELEASE CANDIDATE 2 and then resume release engineering.

---

## 47. Explicit Non-Goals

Streaming, online catalogs, cloud sync, accounts, lyrics scraping, AI recommendations, audio editing, tag editing, album-art editing, batch copy/delete/move/rename file operations, social features, music recognition, automatic online metadata refresh.

---

## 48. Implementation Rules for the Coding Agent

Before changing code, read:
- AGENT_STATE.md
- PRODUCT_SPEC.md
- ARCHITECTURE.md
- DESIGN_SYSTEM.md
- SCREEN_SPECIFICATIONS.md
- INTERACTION_SPEC.md
- DATA_MODEL.md
- PLAYBACK_ARCHITECTURE.md
- TEST_PLAN.md
- this document

Then audit the current implementation.

Do not assume a feature is missing simply because it is listed here. If part of a feature already exists, extend or reuse it.

---

## 49. Final Product Goal

Discover → Sort → Search → Play → Queue → Favorite → Playlist → Share → Set System Sound → Inspect Details → Manage Library

The app stays:
- fast
- local
- private
- mechanical
- focused on the music itself

No feature may compromise playback reliability or the offline-first philosophy.

---

## 50. Open Decisions (owner sign-off required)

1. API 26-28 sound strategy:
   - this spec uses the original URI directly on 26-28
   - confirm or revise if you want a legacy write-permission path instead
2. copy cleanup:
   - whether to ship a “Remove created system sound copies” action in Settings
3. share cap:
   - 100 is default; adjust after actual binder-limit verification
4. Show in Files:
   - keep as P2 best effort or drop if device testing is poor
5. Song Details UI:
   - sheet vs screen; choose and keep consistent with SCREEN_SPECIFICATIONS.md

---

## 51. Change List from v1

| Area | Change |
|---|---|
| Permissions | adds WRITE_SETTINGS and explicit permission flow |
| System sounds | merges ringtone/notification/alarm into one mechanism |
| API behavior | defines copy behavior for 29+ vs direct URI for 26-28 |
| Copy tracking | adds registry of created copies for reuse and cleanup |
| Result handling | adds pending-action pattern after Settings return |
| Song Details | adds AudioFileInspector and TrackDetails model |
| Show in Files | downgraded to P2 best effort |
| Intents | adds ViewModel-effect pattern; ClipData + grant flag |
| Sorting | adds locale-aware collation, unknown-last, tie-breaker, migration, queue independence |
| Action framework | single TrackAction sealed type with shared handler |
| Multi-select | selection by id, refresh-safe behavior, selection exit rules |
| Milestones | P0 before P1; Song Details precedes ringtone work |
| Process | introduces stricter NOT VERIFIED rules and physical-device QA requirements |

---

## 52. Implementation Checklist

Before starting development of any item in this spec, confirm:
- [ ] the feature fits the protected baseline
- [ ] the feature uses the existing playback architecture
- [ ] DataStore/Room boundaries are respected
- [ ] permission requirements are documented
- [ ] failure states are handled
- [ ] device verification plan is recorded
- [ ] AGENT_STATE.md and CHANGELOG.md will be updated after implementation

---

## 53. Final Rule

Do not claim utility support is complete unless:
- the feature was implemented and built
- the relevant tests were run
- the relevant physical-device verification was performed
- the app still meets the protected baseline requirements

If any verification is missing, record it honestly as NOT VERIFIED. The app must remain fast, offline, private, and listening-first.

End of document.
