# DotPlayer UI Mockup - Complete Storyboard Generation Prompt

## Master Brief: Full App Flow Storyboard

You are a senior Android UI/UX designer tasked with creating a **complete visual storyboard** for DotPlayer—a premium offline-first local music player for Android.

This is a **comprehensive, single-prompt request** for a full screen-flow mockup covering every major user journey from app start to playback to library browsing to settings.

---

## Part 1: Visual Design System

### Color Palette (Exact Hex Values)
```
#050505   Background (OLED black)
#101010   Panel (dark gray)
#181818   Surface (slightly lighter)
#202020   Surface 2 (input/chip backgrounds)
#F5F5F5   Primary text (white)
#A3A3A3   Secondary text (medium gray)
#6F6F6F   Tertiary text (dark gray)
#D9362E   Accent red (primary action color)
#FF5A4B   Accent red soft (hover/active)
#2A2A2A   Divider (thin separator lines)
#D9D9D9   Playing/Active state
#FF6B5E   Error state
```

### Typography
- **Headlines**: Condensed geometric sans or dot-matrix-inspired (Doto / DotGothic16 for branding)
- **Body text**: 14–16sp Inter / Roboto system sans
- **Metadata**: 12sp uppercase with tracking for album/artist labels
- **Time/duration**: 12–13sp monospace for precision timing

### Spacing Grid
- Base unit: 4dp
- Padding: 12dp, 16dp (content sides), 8dp (internal spacing)
- Touch targets: 48×48dp minimum
- Button heights: 48–56dp
- List row heights: 56–72dp
- Mini player height: 76dp (content + divider)

### Corner Radius
- Buttons: 12dp
- Cards/chips: 8dp
- Icons/small elements: 4dp

### Shadows & Depth
- Subtle, dark-adapted (not bright halos)
- Elevation used sparingly for modals and sheets
- Solid black (#000000) with 12% opacity for depth

---

## Part 2: Complete User Journey Storyboard

### Flow 1: Cold Start → Permission → Home

**Screen 1A: Splash / Launch**
- Full-screen black (#050505)
- Centered dot-matrix pattern (optional animation, very subtle)
- App name "DotPlayer" in dot-matrix font (12–18sp)
- Fade-in over 500ms
- Transition to Screen 1B after 1.5–2 seconds

**Screen 1B: Permission Gate (First Launch)**
- Full black background
- Centered layout:
  - Music note icon (48dp)
  - "MUSIC ACCESS REQUIRED" headline (18sp, bold, white)
  - Supporting text (14sp, gray): "DotPlayer reads music saved on your device. No cloud, no account, no tracking."
  - Two-line divider (thin red and white)
  - Primary CTA button: "ALLOW ACCESS" (red, 56dp, 100% width minus 32dp margin)
  - Secondary link: "Learn more" (12sp, gray, centered below)
- On allow: transition to Home
- On deny: show "Music Access Denied" screen with "Open Settings" button

**Screen 1C: Home (Empty State)**
- Bottom nav visible (4 items: Home, Library, Search, Settings)
- Mini player at bottom (72dp, empty/no playback state)
- Top app bar: "HOME" title (14sp), search icon, settings icon
- Main content area (black #050505):
  - Large icon: music notes (64dp, gray #6F6F6F)
  - "NO MUSIC FOUND" (16sp, bold white)
  - Supporting text: "Add music to your device or tap RESCAN"
  - Primary button: "RESCAN LIBRARY" (red, 56dp, centered)
  - Secondary text: "Last scanned: never" (12sp gray, below)

---

### Flow 2: Library Scan → Home with Content

**Screen 2A: Home (Content Loaded)**
- Top app bar: "HOME" (14sp uppercase)
- Section 1: "CONTINUE LISTENING" (12sp uppercase label)
  - Single card: album art (100×100dp, rounded), title (14sp bold), artist (12sp gray), play button overlay
  - Tap opens Now Playing
- Section 2: "RECENTLY PLAYED" (horizontal scroll carousel)
  - 4–5 album covers (100×100dp each, rounded)
  - Tap to play from that album
- Section 3: "FAVORITES" (6-item compact list)
  - Each row: favorite icon (red heart), track title (14sp), artist (12sp gray), duration (12sp)
  - Tap to play, swipe for options
- Section 4: "QUICK ACTIONS" (2×2 grid)
  - Buttons: "Shuffle All", "Playlists", "History", "Rescan"
  - Each 100×100dp, icon + label centered
- Bottom: Mini player (76dp, shows current playback or empty)

**Screen 2B: Home (Playing State)**
- Same layout as 2A
- Mini player now active:
  - Left: album cover (52×52dp rounded)
  - Center: title (14sp bold) / artist (12sp gray), left-aligned
  - Right: play/pause icon (28dp, filled red), queue icon (24dp)
  - Progress bar at bottom: thin red line, animated
- All sections still scrollable above mini player

---

### Flow 3: Library Browsing (Main Feature)

**Screen 3A: Library - Tracks Tab (Default)**
- Top section (sticky):
  - Search bar (40dp, magnifying icon left, clear button right when text entered)
  - Below: sort menu (dropdown), filter chips (All | Tracks | Albums | Artists | Folders | Favorites)
- Content: Scrollable list of tracks
  - Each row (64dp):
    - Track number (12sp gray, left)
    - Album cover (40×40dp rounded, left)
    - Vertical text stack: title (14sp bold white) / artist (12sp gray)
    - Duration (12sp gray, right)
    - Favorite icon (24dp, gray heart or red if favorited)
  - Dividers: thin gray (#2A2A2A) between rows
  - Tap row to play; long-press for context menu
  - Swipe right to add to queue
  - Swipe left to remove favorite

**Screen 3B: Library - Albums Tab**
- Same top bar with search/sort/filters
- Grid layout: 2 columns
- Each card (180×180dp):
  - Album cover (full width)
  - Album title (12sp bold, centered)
  - Artist name (10sp gray, centered)
  - Track count badge (10sp gray)
- Tap to open album detail

**Screen 3C: Library - Artists Tab**
- Same top bar
- List layout (72dp rows):
  - Artist initials circle (48dp, background red or gray)
  - Artist name (14sp bold)
  - Album count (12sp gray)
  - Track count (12sp gray)
- Tap to open artist detail

**Screen 3D: Library - Folders Tab**
- Same top bar
- List layout (64dp rows):
  - Folder icon (24dp)
  - Folder path / name (14sp)
  - Track count (12sp gray)
  - Storage label optional (10sp)
- Tap to open folder contents

**Screen 3E: Album Detail (Full Screen)**
- Header:
  - Back button (24dp, left)
  - Title: album name (16sp bold)
  - More menu (right)
- Content:
  - Large album cover (240×240dp, centered, subtle shadow)
  - Album title (18sp bold, centered)
  - Artist (14sp gray, centered)
  - Track count + total duration (12sp gray, centered)
  - Two action buttons: "PLAY ALL" (red, 48dp) | "SHUFFLE" (gray outline, 48dp)
- Tracks list (same format as library):
  - Each row (56dp)
  - Track number, title, artist, duration
  - Tap to play from this track
  - Favorite icon on right

---

### Flow 4: Search

**Screen 4A: Search (Empty / Focus)**
- Full-width search bar at top (48dp):
  - Magnifying icon (left)
  - Input field (placeholder: "Search tracks, albums, artists…")
  - Clear button (X, appears when text entered)
- Below search bar: recent searches or suggested tags (optional)
  - Chips: "Classical", "Jazz", "Rock", "Pop", "2024"
- Keyboard visible (on device)

**Screen 4B: Search (Results)**
- Search bar (active with query text)
- Result groups (each expandable):
  - **TRACKS** (max 5):
    - Row format: cover (32dp), title, artist, duration
    - Tap to play
  - **ALBUMS** (max 4):
    - Card format: cover, title, artist, track count
    - Tap to open detail
  - **ARTISTS** (max 4):
    - Row format: initials circle, name, track count
    - Tap to open artist
  - **PLAYLISTS** (max 3):
    - Row format: icon, playlist name, item count
    - Tap to open playlist
  - **FOLDERS** (max 3):
    - Row format: folder icon, path, item count
    - Tap to open folder

**Screen 4C: Search (No Results)**
- Search bar with failed query
- Large message: "No matches for '[query]'"
- Supporting text: "Try different keywords"
- Suggested alternatives (if available)

---

### Flow 5: Now Playing (Immersive Playback)

**Screen 5A: Now Playing - Compact**
- Full-screen black (#050505)
- Top bar (48dp):
  - Back button (24dp, left)
  - "NOW PLAYING" (14sp uppercase, center)
  - Queue button (24dp, right)
  - More menu / options (right)
- Center content (main focus):
  - Album art (280×280dp, centered, rounded 12dp, subtle shadow)
  - If no art: dot-matrix fallback pattern (same size)
  - Metadata below (centered):
    - Track title (18sp bold, white)
    - Artist name (14sp gray)
    - Album name (12sp tertiary gray)
  - Small divider line (1px red)
- Seek bar (full width minus 16dp padding):
  - Progress bar (4dp high, red)
  - Thumb (12dp circle, red)
  - Current time (left, 12sp mono) | Total time (right, 12sp mono)
- Transport controls (5 buttons, equal spacing, centered):
  - Previous (32dp icon)
  - Play/Pause (48dp icon, filled red when playing, hollow when paused)
  - Next (32dp icon)
  - Shuffle toggle (24dp, red if on, gray if off)
  - Repeat toggle (24dp, red if on, gray if off)
- Secondary row (below transport):
  - Favorite button (24dp heart, red if saved)
  - Add to queue button (24dp)
  - Sleep timer button (24dp, shows remaining time if active)
- Bottom sheet indicator (swipe up for queue/lyrics)

**Screen 5B: Now Playing - Visualizer (v0.3)**
- Same layout as 5A
- **Dot visualizer**: Below album art
  - Animated equalizer bars (8–12 bands)
  - Each bar: 4dp wide, variable height (0–100dp)
  - Color: red accent
  - Animation: smooth wave, ~60fps, synced to audio
  - Only animates while visible and playing
  - Tap to toggle (if setting enabled)

**Screen 5C: Queue Sheet (Bottom Sheet)**
- Header (sticky):
  - Drag handle indicator (3 dots, top center)
  - Title: "QUEUE" (14sp uppercase)
  - Song count (12sp gray)
  - Close button (X, right)
- Current track row (highlighted):
  - Left red bar (4dp accent)
  - "NOW PLAYING" label (12sp uppercase red)
  - Row content: track info, duration
- Upcoming tracks (draggable):
  - Each row (56dp):
    - Drag handle (6 dots, left)
    - Track info (title / artist)
    - Duration (right)
    - Remove button (X, right)
  - Divider: thin gray between rows
- Bottom actions (sticky):
  - "SHUFFLE QUEUE" button (outline)
  - "CLEAR QUEUE" button (outline, with confirmation modal)

**Screen 5D: Queue - Empty**
- Sheet layout as 5C
- Center message: "QUEUE IS EMPTY"
- Supporting text: "Add tracks or tap Play All from an album"
- Button: "BROWSE LIBRARY"

---

### Flow 6: Favorites / Playlists / History

**Screen 6A: Favorites (Full Screen)**
- Top app bar: "FAVORITES" (14sp uppercase)
  - Sort menu (dropdown)
  - Count badge: "12 tracks"
- List of favorited tracks:
  - Same row format as library (56dp)
  - Each row has filled red heart (always visible as state)
  - Tap to play
  - Swipe to remove favorite or add to playlist
- Empty state:
  - Large heart icon (64dp, outline)
  - "NO FAVORITES YET" (16sp bold)
  - Supporting text: "Tap the heart on any track to save it"

**Screen 6B: Playlists (Tab / Full Screen)**
- Top section:
  - Title: "PLAYLISTS" (14sp uppercase)
  - "CREATE PLAYLIST" button (red, 48dp)
- Playlist cards (2-column grid on mobile):
  - Each card (160×160dp):
    - Playlist art / default icon (100×100dp)
    - Playlist name (12sp bold)
    - Item count (10sp gray)
    - Last modified date (10sp gray)
  - Tap to open detail
  - Long-press for rename/delete

**Screen 6C: Playlist Detail**
- Header:
  - Playlist cover / art (200×200dp)
  - Playlist name (16sp bold)
  - Item count + duration (12sp gray)
  - Buttons: "PLAY ALL" (red) | "SHUFFLE" (outline) | "EDIT" (outline)
- Tracks in playlist:
  - Draggable list (if in edit mode)
  - Same row format as album detail
  - Each row (56dp)
  - Handle icon (left, if editable)
  - Track info, duration
  - Remove button (right, if editable)
- Bottom (edit mode):
  - "ADD TRACKS" button
  - "DONE" / "CANCEL" buttons

**Screen 6D: Create / Edit Playlist Modal**
- Modal sheet (bottom half of screen)
- Header: "NEW PLAYLIST" or "EDIT PLAYLIST"
- Input field:
  - Label: "NAME"
  - Text input (48dp, placeholder: "My Playlist")
  - Characters: max 100
- "CREATE" button (red, 56dp) or "SAVE" button
- "CANCEL" button (outline)

**Screen 6E: History - Recently Played**
- Top app bar: "HISTORY" (14sp uppercase)
  - Tabs below: "RECENT" | "MOST PLAYED" | "CONTINUE"
- List of recent tracks:
  - Each row (56dp)
  - Track info (title / artist)
  - Last played time (12sp gray, right)
  - Tap to play
- "CLEAR HISTORY" button (outline, bottom, with confirmation)

**Screen 6F: History - Most Played**
- Same layout as 6E
- Tracks sorted by play count (descending)
- Each row shows play count (12sp gray, right) instead of last played time

**Screen 6G: History - Continue Listening**
- Same layout as 6E
- Tracks with saved playback position
- Each row shows last position badge (12sp gray, right) or "50%" indicator
- Tap to resume from that position

---

### Flow 7: Settings

**Screen 7A: Settings - Main**
- Top app bar: "SETTINGS" (14sp uppercase)
- Scrollable list of sections (each with icon and label):
  - **PLAYBACK** (settings icon)
  - **APPEARANCE** (palette icon)
  - **LIBRARY** (music note icon)
  - **PRIVACY** (shield icon)
  - **ABOUT** (info icon)
- Each section tap opens a detail screen (7B–7F)

**Screen 7B: Settings - Playback**
- Back button and title: "PLAYBACK"
- Toggle items (48dp rows):
  - "Resume playback" (toggle, ON by default)
  - "Autoplay next album" (toggle, ON)
- Dropdown items:
  - "Repeat mode" (dropdown: Off / All / One) with current selection shown
  - "Default shuffle" (toggle, OFF)
- Slider:
  - "Sleep timer default" (range: 5–120 min, default 30)
  - Display: "30 min"
- Bottom text: "Media session gapless by default for supported formats."

**Screen 7C: Settings - Appearance**
- Back button and title: "APPEARANCE"
- **THEME** section:
  - Radio buttons: Dark (selected) | Light | System
  - Note: "Dark theme optimized for OLED"
- **ACCENT COLOR** section:
  - Radio buttons: Red (selected) | Orange | Blue | Green
- **ART STYLE** section:
  - Dropdown: "Cover" (default) | "Dot Matrix fallback"
- **VISUALIZER** section:
  - Toggle: "Enabled" (ON)
  - Note: "Simulated dot-pattern visualizer, v0.3 feature"
- **MOTION** section:
  - Toggle: "Reduce motion" (OFF, respects system setting)
  - Note: "Disables animated visualizer and micro-interactions"

**Screen 7D: Settings - Library**
- Back button and title: "LIBRARY"
- Section: "DISCOVERY"
  - Slider: "Minimum track length" (range: 10s–5m, default 30s)
  - Display: "30 seconds"
  - Button: "RESCAN LIBRARY" (outline, 48dp)
  - Small text: "Last scanned: Today, 2:30 PM"
- Section: "ORGANIZATION"
  - Dropdown: "Sort order" (Title | Artist | Album | Date Added | Duration)
  - List: "EXCLUDED FOLDERS" (empty by default)
    - Each row (48dp): folder path, remove button (X)
    - Button at bottom: "ADD FOLDER" (outline)

**Screen 7E: Settings - Privacy**
- Back button and title: "PRIVACY"
- Info block (gray background, 12sp text):
  - "DotPlayer is completely offline."
  - "No accounts. No cloud. No tracking."
  - "Your music, your device, your choice."
- Section: "CRASH REPORTS"
  - Toggle: "Save local crash reports" (ON)
  - Note: "Reports are stored locally and never uploaded automatically."
  - Button: "SHARE LOGS" (outline, 48dp)
    - Opens share sheet with local crash log file
- Section: "DATA"
  - Button: "EXPORT FAVORITES & PLAYLISTS" (outline)
  - Button: "CLEAR ALL DATA" (red outline, destructive)
    - Confirmation: "This will delete all favorites, playlists, and history. Music files are not affected."

**Screen 7F: Settings - About**
- Back button and title: "ABOUT"
- App logo (48×48dp, centered)
- App name: "DotPlayer" (16sp bold, centered)
- Version: "1.0.0" (12sp gray, centered)
- Build: "Build 2024-10-12" (10sp gray, centered)
- Divider line
- Section: "LICENSES"
  - "Open Source Font: DotGothic16"
    - License: "SIL Open Font License (OFL)"
    - Link: "View License"
  - "ExoPlayer (Media3)" → opens license text
  - "Jetpack Compose" → opens license text
  - "Room, Coil, Hilt" → nested list with links
- Section: "LEGAL"
  - Link: "Privacy Policy"
  - Link: "License Agreement"
  - Link: "Credits & Acknowledgments"
- Divider
- Footer text (10sp tertiary gray):
  - "Made with precision for music lovers."
  - "No telemetry. No ads. Pure listening."

---

### Flow 8: Error States and Modals

**Screen 8A: Permission Denied (After First Deny)**
- Full-screen modal or dialog
- Icon: exclamation or warning (48dp)
- Title: "MUSIC ACCESS DENIED" (16sp bold)
- Message: "DotPlayer cannot access your music without permission. Go to Settings to allow access."
- Button: "OPEN SETTINGS" (red, 56dp)
- Button: "DISMISS" (outline)

**Screen 8B: Track Unavailable**
- In-queue error state, or toast notification
- Toast overlay (bottom-center, 48dp height):
  - Icon: warning (24dp)
  - Text: "TRACK UNAVAILABLE • SKIPPING" (12sp white on red #D9362E)
  - Auto-dismiss after 3 seconds
- Queue continues to next playable item

**Screen 8C: Empty Queue**
- Mini player still visible at bottom
- Center message (above mini player):
  - "QUEUE IS EMPTY"
  - "BROWSE LIBRARY OR ADD A TRACK"
  - Button: "PLAY SOMETHING" (red, 56dp)

**Screen 8D: Confirmation Dialogs**
- Modal with title, message, two buttons
- Example: "Clear History?"
  - Message: "This action cannot be undone."
  - Button: "CANCEL" (outline)
  - Button: "CLEAR" (red)
- Animation: slide up from bottom, 200ms

---

### Flow 9: Widgets and Lock Screen

**Screen 9A: Compact Widget (Lock Screen)**
- Display size: ~240×80dp
- Layout:
  - Left: album art (60×60dp, rounded)
  - Center: track title (12sp bold) / artist (10sp gray), left-aligned
  - Right: play/pause button (40dp touch target, red if playing)
- Tap anywhere: opens Now Playing
- Tap play/pause: toggles playback
- Update: every 250ms while playing

**Screen 9B: Expanded Widget (Home Screen)**
- Display size: ~240×200dp
- Layout:
  - Top: track title (12sp bold)
  - Middle: album art (120×120dp, centered)
  - Artist name (10sp gray, centered)
  - Bottom: three buttons (32dp each):
    - Previous (left)
    - Play/Pause (center, red if playing)
    - Next (right)
- Tap widget: opens Now Playing
- Tap buttons: control playback
- Update: every 250ms while playing

---

## Part 3: Screen State Matrix

Create a complete visual reference for every screen in all possible states:

| Screen | Idle | Loading | Playing | Error | Empty |
|---|---|---|---|---|---|
| Home | Content visible | Skeleton loader | Mini player active | Permission prompt | "No music found" |
| Library | List view | Spinner | Highlighted row | "No access" | Empty message |
| Search | Empty state | Results load | Results visible | No matches | Clear |
| Now Playing | Paused | Buffering | Visualizer active | Track unavailable | Queue empty |
| Queue | List | Adding items | Current highlighted | Unplayable items | Empty |
| Favorites | List of hearts | Removing | Favorited highlight | Missing tracks | Empty |
| Playlists | Grid | Creating | Playing highlight | Rename error | Empty |
| Settings | All toggles | Saving | All visible | Permission denied | All defaults |

---

## Part 4: Component Library

Create a complete **component design system** showing:

### 4.1 Button Variants
- Primary (red accent, 56dp, 100% width)
- Secondary (gray outline, 48dp, 100% width)
- Tertiary (text-only, no background)
- Icon button (24–32dp, circular touch target)
- Floating Action Button (56dp circle, red, centered)

### 4.2 List Components
- Track row (64dp, normal state + selected state)
- Album card (grid, 2-column on mobile)
- Artist row (72dp)
- Folder row (64dp)
- Header row (sticky, 40dp, gray background)

### 4.3 Input Components
- Text input (48dp, focus state, error state)
- Search bar (40dp, active state, clear button)
- Slider (48dp touch, 4dp track)
- Toggle switch (48dp, on/off states)
- Dropdown (48dp, open/closed states)
- Chip / filter (28dp, active/inactive states)

### 4.4 Mini Player Component
- Default state (76dp, no playback)
- Playing state (76dp, progress animated)
- Paused state (76dp, progress static)

### 4.5 Modal / Sheet Components
- Bottom sheet (rounded top, overlay)
- Confirmation dialog (2 buttons, centered)
- Toast notification (bottom center, red accent)
- Loading spinner (36dp, red)
- Progress bar (4dp, red track)

### 4.6 Navigation Components
- Bottom navigation bar (56dp, 4 items)
  - Item: icon (24dp) + label (10sp)
  - Active: red accent icon + bold label
  - Inactive: gray icon + normal label
- App bar (56dp, title + icons)
- Tab bar (40dp height, scrollable)

---

## Part 5: Animation & Micro-Interaction Specs

### 5.1 Transitions
- **Screen change**: fade + slide 250ms (cubic-bezier easing)
- **Sheet open/close**: slide from bottom 300ms
- **Modal fade**: 150ms

### 5.2 Micro-interactions
- **Button press**: scale 0.95 + color shift, 100ms
- **Favorite toggle**: heart scale 1.2 + red fill, 200ms
- **Play/pause**: icon crossfade + color change, 150ms
- **Queue reorder**: smooth item shift 200ms
- **Mini player progress**: continuous smooth line (no stuttering)

### 5.3 Reduced Motion
- All animations disabled if system `reduceMotionEnabled` is true
- Fallback: instant state changes, no animation

---

## Part 6: Responsive Breakpoints

### 6.1 Mobile (4–6 inches, primary)
- All screens optimized for 1080×1920px
- Landscape: mini player folds into app bar, content reflows

### 6.2 Tablet (7–10 inches, secondary)
- 2-column album grid
- Split view: library on left, detail on right (if space permits)
- Mini player sticky at bottom or folded into a rail

---

## Part 7: Accessibility Specifications

### 7.1 Color Contrast
- White (#F5F5F5) on black (#050505): 19.95:1 ✓
- Gray (#A3A3A3) on black: 8.9:1 ✓
- Red accent only used as background fills or icons, never text

### 7.2 TalkBack Labels
- All buttons: "Play", "Next", "Add to favorites", "Queue"
- List items: "Track title, Artist, Duration"
- Navigation: "Home tab", "Library tab", "Search tab", "Settings tab"

### 7.3 Large Text Support
- Text scales up to 125% system scaling
- Button heights never compress below 48dp
- Icons scale proportionally with text size

### 7.4 Reduced Motion
- Visualizer animation disabled
- Sheet transitions become instant
- All micro-animations disabled

---

## Part 8: Interaction Patterns & User Flows

### 8.1 Play a Song (Core Scenario)
1. User opens Library → Tracks tab
2. Sees list of songs
3. Taps a song → song plays immediately
4. Now Playing auto-opens (or stays in background)
5. Mini player shows album art + progress
6. Seek bar shows current position
7. User can pause/play from mini player or Now Playing

### 8.2 Create a Playlist
1. User taps Playlists tab
2. Taps "CREATE PLAYLIST" button
3. Modal opens: input playlist name
4. User types "Workout" and taps "CREATE"
5. Playlist detail screen opens (empty)
6. User taps "ADD TRACKS"
7. Library picker opens; user selects songs
8. Songs added to playlist in order

### 8.3 Search and Play
1. User taps Search in bottom nav
2. Taps search bar
3. Begins typing "Nirvana"
4. Results update in real-time
5. Sees tracks, albums, artists matching
6. Taps "Smells Like Teen Spirit" track
7. Song plays immediately
8. Now Playing screen visible

### 8.4 Queue Management
1. User is playing a song
2. Taps queue icon (in Now Playing or mini player)
3. Queue sheet opens
4. Sees current song highlighted
5. Sees 10+ upcoming songs
6. User drags a song to reorder
7. User swipes left on a song to remove
8. Taps "SHUFFLE QUEUE"
9. Queue reorders randomly

### 8.5 Lock Screen Control
1. User locks device while playing
2. Lock screen shows compact widget
3. Album art visible
4. Play/pause button accessible
5. User taps play/pause
6. Playback toggles
7. User taps album art
8. Now Playing screen opens (app unlocks)

---

## Part 9: Detailed Visual Specifications

### 9.1 Mini Player (Sticky, Always Visible)

**Height**: 76dp (64dp content + 12dp system inset consideration)

**Composition**:
```
┌─────────────────────────────────────────────────────────┐
│ [Album Art]  Title         [▶]  [≡]                  │ 64dp
│              Artist                                      │
├─────────────────────────────────────────────────────────┤
│ ████████████░░░░░░░░░░░░░░░░░░░░░░░ (progress bar)    │ 2dp
├─────────────────────────────────────────────────────────┤
│ System nav bar area (16dp)                              │
└─────────────────────────────────────────────────────────┘
```

**Colors**:
- Background: #101010
- Progress: #D9362E (red)
- Text: #F5F5F5 (white) for title, #A3A3A3 (gray) for artist
- Icon: #D9D9D9 (light gray) or #D9362E (red if playing)

**Interactions**:
- Tap player area: open Now Playing
- Tap play/pause: toggle playback (prevent event bubbling)
- Tap queue icon: open Queue sheet
- Long-press: context menu (add to playlist, add to favorites)

---

### 9.2 Now Playing Screen (Full-Screen Immersive)

**Layout zones** (from top):
1. **Header** (56dp): Back | "NOW PLAYING" | Queue | ⋮
2. **Album Art** (300×300dp): Large cover, rounded 12dp
3. **Metadata** (72dp): Title (18sp) + Artist (14sp) + Album (12sp)
4. **Seek Bar** (48dp): Progress bar + time display
5. **Transport** (64dp): 5 buttons in a row
6. **Secondary Actions** (48dp): Favorite | Add to Queue | Sleep Timer
7. **Visualizer** (0–160dp, optional v0.3): Animated dot equalizer
8. **Bottom Hint** (16dp): Swipe up for queue

**Colors**:
- Background: #050505 (full black)
- Text: #F5F5F5 (white)
- Progress bar: #D9362E (red)
- Dividers: #2A2A2A (dark gray)

**Interactions**:
- Seek bar: drag thumb to seek
- Transport buttons: tap to control
- Favorite button: tap to toggle + heart animation
- Swipe up: show queue sheet
- Swipe down: close Now Playing (if opened as modal)
- Back button: close and return to previous screen

---

### 9.3 Visualizer Component (Dot-Matrix Style, v0.3)

**Design**:
- 8–12 vertical bars, each 4dp wide
- Spacing: 2dp between bars
- Height: variable (0–80dp max), animated
- Color: red accent (#D9362E)
- Animation: smooth easing, 60fps

**Behavior**:
- Simulated visualizer (no `RECORD_AUDIO` permission needed)
- Animate based on playback state and track metadata
- Stop rendering when screen is off or Now Playing is not visible
- Respect reduced motion: static bars if enabled
- Optional toggle in Settings

**Size**: Fits in Now Playing below album art, max 160dp height

---

## Part 10: Complete Mockup Deliverables

Please generate **all of the following**:

### 10.1 Screen Mockups (High-Fidelity Mobile)
1. **Splash / Launch** (1 screen)
2. **Permission Gate** (2 screens: allow prompt + denied)
3. **Home** (3 screens: empty, content, playing)
4. **Library** (5 screens: tracks, albums, artists, folders, album detail)
5. **Search** (3 screens: empty, results, no matches)
6. **Now Playing** (3 screens: compact, with visualizer, queue sheet)
7. **Favorites** (2 screens: with content, empty)
8. **Playlists** (3 screens: list, detail, create modal)
9. **History** (3 screens: recent, most played, continue)
10. **Settings** (6 screens: main, playback, appearance, library, privacy, about)
11. **Error States** (3 screens: permission denied, track unavailable, empty queue)
12. **Widgets** (2 screens: compact, expanded)

**Total: ~40–45 distinct, polished mockup screens**

All rendered at **1080×1920px** (mobile standard), high-fidelity, production-quality visual design.

### 10.2 Component Library
- Button variants (primary, secondary, tertiary, icon, FAB)
- List row types (track, album, artist, folder, header)
- Input controls (text, search, slider, toggle, dropdown, chip)
- Mini player (idle, playing, paused)
- Modals and sheets (bottom sheet, dialog, toast)
- Navigation bar + tab bar
- Badge and indicator styles

### 10.3 State Variations
- Active / inactive states
- Enabled / disabled states
- Hover / pressed states
- Error states
- Loading states
- Empty states

### 10.4 Interaction Documentation
- Tap, swipe, long-press behaviors for each screen
- Modal transitions and animations
- Gesture directions and speeds (250ms, 300ms, etc.)
- Focus states for accessibility
- Ripple and feedback effects

### 10.5 Design Guide Summary
- Color palette with hex codes
- Typography scale (font sizes, weights, line heights)
- Spacing and grid system
- Corner radius and shadows
- Responsive breakpoints
- Accessibility checklist

### 10.6 User Journey Storyboards
- 9 complete user flows (cold start, library browse, search, playback, queue, favorites, playlists, history, settings)
- Showing screen transitions and state changes
- With annotations for key interactions

---

## Part 11: Design Quality Checklist

Before finalizing, ensure every mockup adheres to:

- ✓ **Visual consistency**: Same colors, typography, spacing across all screens
- ✓ **Premium aesthetic**: High contrast, clean, minimal, industrial
- ✓ **Dark theme optimization**: Optimized for OLED (pure blacks #050505)
- ✓ **Touch target size**: All clickable elements ≥48×48dp
- ✓ **Readable text**: Primary text ≥14sp, secondary ≥12sp
- ✓ **Color contrast**: Text passes WCAG AA (≥4.5:1 for small text)
- ✓ **Smooth animations**: 150–300ms transitions, smooth easing
- ✓ **Accessibility**: All interactive elements labeled, icons clear, reduced motion respected
- ✓ **Responsive**: Works on 4–6" phones; gracefully degrades on landscape
- ✓ **Consistent interaction**: Same patterns for same actions across screens
- ✓ **Error handling**: All error states graceful and non-destructive
- ✓ **Empty states**: All empty states guide user to next action
- ✓ **No third-party branding**: No Apple, Google, Samsung, Spotify logos; only DotPlayer branding

---

## Part 12: Final Summary

**Goal**: Deliver a complete, production-ready UI mockup storyboard for DotPlayer that:
1. Looks premium, minimal, and distinctly industrial
2. Covers every major user flow from cold start to advanced features
3. Demonstrates proper app architecture (bottom nav, modals, flows)
4. Shows all error states and edge cases
5. Includes complete component library for handoff to development
6. Meets accessibility standards (WCAG AA)
7. Adheres to the exact color palette and design language specified
8. Provides clear interaction specs and animation timings

---

## Gemini Execution Instructions

Generate:
1. All mockups as high-fidelity PNG or JPG images
2. Each screen labeled with its name and purpose
3. All component library illustrations
4. A summary design guide (text + visual)
5. An annotated storyboard showing user flows

**Style**: Professional, production-ready, premium Android app aesthetic. Dark theme, red accent, minimal industrial. No generic templates—make it distinctive and recognizable as DotPlayer.

**Resolution**: 1080×1920px for phones (standard mobile), high DPI (2x effective)

**Format**: Deliver as a coherent visual deck showing all screens in logical order, with optional annotations.

