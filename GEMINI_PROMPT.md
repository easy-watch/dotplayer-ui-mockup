# DotPlayer UI Mockup - Gemini Generation Prompt

## Context
You are a senior Android UI/UX designer specializing in Jetpack Compose and Material Design 3. Your task is to generate a complete, production-ready UI mockup system for **DotPlayer**, a premium offline-first local music player for Android.

## Design Philosophy
- **Visual Language**: Minimal industrial design inspired by OLED black, white, gray with a single red accent
- **Feel**: Premium, tactile, mechanical precision
- **Platform**: Android 8.0+ with Jetpack Compose
- **Accessibility**: TalkBack ready, large text support, reduced motion respect
- **Performance**: Optimized for mid-range devices, smooth 60fps scrolling

---

## Color Palette (Exact Hex Values)

```
Background:          #050505
Panel:               #101010
Surface:             #181818
Surface 2:           #202020
Primary Text:        #F5F5F5
Secondary Text:      #A3A3A3
Tertiary Text:       #6F6F6F
Accent Red:          #D9362E
Accent Red Soft:     #FF5A4B
Divider:             #2A2A2A
Playing/Active:      #D9D9D9
Error:               #FF6B5E
```

---

## Typography

- **Headings**: Condensed geometric sans-serif or dot-matrix-inspired (Doto / DotGothic16)
- **Body**: Inter / Roboto / System sans at 14-16sp
- **Metadata**: Small 12sp uppercase with wide letter-spacing
- **Mono/Track time**: 12-13sp monospace for precise timing

---

## Core App Structure

### Bottom Navigation (Stable Sections)
1. **Home** - Recent, favorites, quick actions, continue listening
2. **Library** - Tracks, albums, artists, folders, playlists
3. **Search** - Global instant search
4. **Settings** - App configuration

### Always-Visible Components
- **Mini Player** (bottom, 72-80dp height)
- **App Bar** (top, contextual)

### Sheet/Full-Screen Overlays
- **Now Playing** - Full-screen immersive player
- **Queue** - Draggable reorderable list
- **Playlist Detail** - Edit, add, remove tracks

---

## Screen Specifications

### Screen 1: Permission / Onboarding
**Purpose**: Request local music access with privacy assurance

**Layout**:
- Full black background (#050505)
- Centered decorative dot-matrix pattern (optional)
- Large headline: "MUSIC ACCESS REQUIRED"
- Supporting text: "DotPlayer reads music saved on your device. No cloud, no account, no tracking."
- Primary CTA button (red accent): "ALLOW ACCESS"
- Secondary CTA: "LEARN MORE"
- Small footer: OS permission disclaimer + privacy statement

**Visual Details**:
- Thin white divider line above buttons
- Button height: 56dp
- Button corner radius: 12dp
- Accent red (#D9362E) filled primary button
- Text color: #F5F5F5

---

### Screen 2: Home
**Purpose**: Dashboard with quick access to recent, favorites, and current session

**Top Bar**:
- Title: "HOME" (14sp uppercase, bold)
- Right icons: Search (24dp), Settings (24dp)

**Sections** (scrollable):

1. **Continue Listening**
   - Single album cover + metadata
   - Title, artist, duration
   - Play button overlay
   
2. **Recently Played**
   - Horizontal carousel: 4-5 album covers
   - Each 100x100dp
   - Tap to play
   
3. **Favorites**
   - Compact list: 4-6 tracks
   - Each row: 56dp height
   - Columns: favorite icon (red if saved), title, artist, duration
   
4. **Quick Actions**
   - 2x2 grid of large buttons:
     - Shuffle All
     - Playlists
     - History
     - Rescan
   - Each button: 100x100dp, center icon + label below

5. **Mini Player**
   - Sticky at bottom
   - Layout: [Album Art] | [Title\nArtist] | [Play/Pause] [Queue]
   - Progress bar: 2dp red line at bottom

---

### Screen 3: Library (Main View)
**Purpose**: Browse local collection across tracks, albums, artists, folders

**Top Section**:
- **Search Bar**: 40dp height, gray (#202020) background, magnifying icon, placeholder "Search tracks..."
- **Sort Menu**: Dropdown, sort by: Title, Artist, Album, Date Added, Duration
- **Filter Chips** (horizontal scroll):
  - All | Tracks | Albums | Artists | Folders | Favorites

**Content Sections** (tabbed or dynamic):

#### 3a. Tracks Tab
- **Row Height**: 64dp
- **Columns**: 
  - Track number (gray, 12sp)
  - Album cover thumbnail (40x40dp, left padding 12dp)
  - Vertical stack: Title (16sp bold) / Artist (12sp gray)
  - Duration (12sp gray, right padding 12dp)
  - Favorite icon (24dp touch target, red if saved)
- **Row Interaction**: Tap to play, swipe right to add to queue, swipe left to remove
- **Divider**: 1px #2A2A2A between rows
- **Empty State**: "NO MUSIC FOUND" + "RESCAN" button

#### 3b. Albums Tab
- **Grid Layout**: 2 columns
- **Card Height**: 180dp
- **Card Contents**:
  - Album cover (full width, aspect 1:1)
  - Album name (12sp bold, centered)
  - Artist name (10sp gray, centered)
  - Track count (10sp gray, centered)
- **Interaction**: Tap to view album detail screen

#### 3c. Artists Tab
- **Row Height**: 72dp
- **Columns**:
  - Artist initials circle (48dp, accent red or gray)
  - Artist name (16sp bold)
  - Album count (12sp gray)
  - Track count (12sp gray)
- **Interaction**: Tap to view artist detail

#### 3d. Folders Tab
- **Row Height**: 64dp
- **Columns**:
  - Folder icon (24dp)
  - Folder path (16sp)
  - Track count (12sp gray)
  - Storage label optional (10sp)
- **Interaction**: Tap to view folder contents

---

### Screen 4: Search
**Purpose**: Real-time global search across all content types

**Layout**:
- **Search Bar** (always active, full width, 48dp):
  - Query input field
  - Clear button (X icon, visible when text)
  - Search icon (left aligned)
- **Result Groups** (expandable sections):
  - Tracks (max 5 results shown)
  - Albums (max 4)
  - Artists (max 4)
  - Playlists (max 4)
  - Folders (max 4)
- **Each Result Row**:
  - Small icon / thumbnail (40dp)
  - Primary label + secondary label
  - Play button overlay (on hover or always visible)

**Empty Search State**:
- "No matches for '[query]'"
- "Try different keywords"

**Suggested Searches** (when empty):
- Popular tags: "Classical", "Jazz", "Pop", "Rock"

---

### Screen 5: Now Playing (Full-Screen)
**Purpose**: Immersive playback control and visualization

**Layout**:

**Top Bar**:
- Back button (left)
- "NOW PLAYING" (center, 14sp)
- Queue button (right)
- More menu (right, 24dp)

**Center Content** (main focus):
- **Album Art**:
  - Large cover image (300x300dp minimum)
  - If missing: Generated dot-matrix fallback pattern
  - Subtle shadow / depth effect
  
- **Metadata** (below art):
  - Track Title (18sp bold, white)
  - Artist Name (14sp, gray)
  - Album Name (12sp, tertiary gray)
  - Small line divider (#2A2A2A)

**Playback Controls**:
- **Seek Bar**:
  - Full width with 16dp padding
  - Progress thumb: 12dp red circle
  - Current time (left, 12sp mono) | Total time (right, 12sp mono)
  - Track underline: thin red line
  
- **Transport Controls** (5 buttons, equal spacing):
  - Previous (32dp icon)
  - Play/Pause (48dp icon, filled red when playing)
  - Next (32dp icon)
  - Shuffle (24dp toggle, accent red if on)
  - Repeat (24dp toggle, accent red if on)

**Secondary Row** (below transport):
- Favorite button (24dp, red if favorited)
- Add to queue button (24dp)
- Sleep timer button (24dp, shows remaining time in toast)

**Visualizer** (optional, v0.3):
- Animated dot-pattern equalizer
- Horizontal bars, 8-12 bands
- Only animate when visible (reduce motion respected)
- Toggle in settings

**Bottom Sheet** (swipe up):
- Queue preview
- Lyrics placeholder (v1.x feature)

---

### Screen 6: Queue (Bottom Sheet / Full Screen)
**Purpose**: Manage up-next queue and track ordering

**Header**:
- Title: "QUEUE"
- Close button (X, top right)
- Song count (12sp gray)

**Current Track Section**:
- Highlighted with accent red bar (left side, 4dp width)
- Same row format as library
- "NOW PLAYING" label above

**Upcoming Queue**:
- Draggable list
- Each row: drag handle (6 dots, left), track info, duration, remove button (X)
- Row height: 56dp
- Divider: thin gray between rows

**Bottom Actions**:
- Shuffle Queue button
- Clear Queue button (with confirmation)
- Add Tracks button

**Empty Queue State**:
- "Queue is empty"
- "Add tracks or tap Play All"

---

### Screen 7: Favorites
**Purpose**: Personal collection of marked tracks

**Header**:
- Title: "FAVORITES"
- Sort menu
- Count: "12 tracks"

**List**:
- Track rows (same format as Library)
- Each row has filled red heart icon
- Tap heart to remove from favorites

**Empty State**:
- Large heart outline icon
- "No favorites yet"
- "Tap the heart on any track to save it"

---

### Screen 8: Playlists
**Purpose**: Manage custom playlists

**Top Section**:
- Title: "PLAYLISTS"
- "Create Playlist" FAB or button

**Playlist List**:
- Card layout, 2 columns:
  - Playlist cover (composite or default art)
  - Playlist name (14sp bold)
  - Song count + duration (12sp gray)
  - Last modified date (10sp gray)

**Playlist Detail** (full screen):
- **Header**:
  - Playlist cover (200x200dp)
  - Playlist name (18sp bold)
  - Song count + total duration
  - Buttons: Play All | Shuffle | Edit | Delete
  
- **Tracks**:
  - Draggable list (if edit mode)
  - Same format as queue
  - Each row: handle, track info, remove button
  
- **Actions**:
  - Add Tracks (open library picker modal)
  - Remove multiple (swipe or checkbox mode)
  - Reorder (drag handle)

---

### Screen 9: Settings
**Purpose**: App configuration and privacy

**Sections** (scrollable):

#### Playback
- Resume playback (toggle)
- Autoplay (toggle)
- Repeat mode default (dropdown: off / all / one)
- Sleep timer (time picker)

#### Appearance
- Theme (toggle: dark / light - dark primary)
- Accent color (radio: red / other options)
- Art style (dropdown: cover / dot-matrix fallback)
- Visualizer (toggle, v0.3)
- Reduce motion (toggle, respect system setting)

#### Library
- Minimum track length (slider: 10s - 5m)
- Rescan Library (button + last scan timestamp)
- Excluded Folders (list with add/remove)
- Sort order (dropdown)

#### Privacy & Legal
- "No accounts, no cloud, no tracking"
- Crash logs (local only, user shares)
- Share Logs button
- App version (14sp gray)
- Open Source Licenses button

#### About
- "About DotPlayer"
- Privacy Policy link
- Open Source Font Attribution (Doto / DotGothic16)
- Legal notices

---

## Component Specs

### Mini Player (Always Visible)
- **Height**: 76dp (64dp content + 12dp padding)
- **Background**: #101010
- **Layout**:
  - Left: Album cover 52x52dp, rounded 4dp
  - Center: Title (14sp bold) / Artist (12sp gray), left-aligned, 1 line each (truncate)
  - Right: Play/Pause icon (28dp) + Queue icon (24dp)
- **Progress Bar**: 2dp red line at bottom
- **Touch**: Entire player opens Now Playing
- **Interaction**:
  - Tap play/pause zone stops full event propagation
  - Tap queue icon opens queue sheet
  - Tap elsewhere opens Now Playing

### Button Styles

**Primary CTA Button**:
- Background: #D9362E (red accent)
- Text color: #F5F5F5 (white)
- Height: 56dp
- Corner radius: 12dp
- Font: 14sp bold, uppercase
- Padding: 16dp horizontal
- Ripple: subtle lighter red

**Secondary Button**:
- Background: #202020
- Text color: #F5F5F5
- Border: 1px #2A2A2A
- Height: 48dp
- Corner radius: 12dp
- Font: 14sp, normal case

**Icon Button**:
- Background: transparent
- Touch target: 48x48dp minimum
- Icon: 24-28dp
- Ripple: subtle white overlay

### Input Fields

**Text Input**:
- Background: #181818
- Border: 1px #2A2A2A on focus
- Text color: #F5F5F5
- Placeholder: #6F6F6F
- Corner radius: 8dp
- Padding: 12dp
- Height: 48dp

**Search Bar**:
- Background: #202020
- Magnifying icon (left): 20dp, gray
- Clear button (right, if text): X icon, 20dp, gray
- Cursor: red accent color

### Chips / Filters

**Active Chip**:
- Background: #D9362E (red)
- Text: #F5F5F5
- Border: none
- Corner radius: 24dp
- Padding: 8dp horizontal, 6dp vertical
- Font: 12sp, bold

**Inactive Chip**:
- Background: #202020
- Text: #A3A3A3
- Border: 1px #2A2A2A
- Corner radius: 24dp
- Padding: 8dp horizontal, 6dp vertical

### List Rows

**Track Row**:
- Height: 64dp
- Padding: 12dp horizontal, 8dp vertical
- Divider: 1px #2A2A2A at bottom
- Highlighted row: subtle red left border (4dp)
- Selected: light red background (#1A0B0B)

**Album Row**:
- Grid card: 2 columns on mobile
- Height: 180dp
- Padding: 8dp
- Shadow: subtle drop shadow on cover
- Corner radius: 8dp

**Artist Row**:
- Height: 72dp
- Artist avatar: 56dp circle
- Content: vertical stack, center aligned
- Divider: 1px #2A2A2A

### Dividers & Separators
- Color: #2A2A2A
- Thickness: 1px (0.5dp effective on 2x displays)
- Margin: 12dp (full bleed for list dividers, 12dp padding for section dividers)

---

## Animation & Motion Guidelines

### Principles
- Reduce motion respected (check `reduceMotionEnabled` system setting)
- All transitions: 150-300ms cubic-bezier easing
- Avoid flashy or loud motion
- Movement should feel mechanical and intentional

### Specific Animations
1. **Play/Pause Icon**: Crossfade 150ms
2. **Favorite Heart**: Scale + color change 200ms
3. **Queue Reorder**: Smooth item shift 200ms
4. **Sheet Open/Close**: Slide + fade 250ms
5. **Visualizer Bars**: Smooth wave motion, only when visible
6. **Screen Transitions**: Fade or subtle scale

---

## Accessibility Requirements

### WCAG AA Compliance
- **Color Contrast**:
  - Text on background: white (#F5F5F5) on black (#050505) = 19.95:1 ✓
  - Text on surface: white on #181818 = 15.5:1 ✓
  - Gray text (#A3A3A3) on #050505 = 8.9:1 ✓
  - Accent red text is avoided; use only as background fills or icons

### TalkBack Support
- All clickable elements labeled: "Play/Pause", "Add to Favorites", etc.
- List items announce: "Track name, Artist name, Duration"
- Navigation items: "Home tab", "Library tab", etc.
- State changes announced: "Playing", "Paused", "Loading"

### Large Text Support
- Base font size: 16sp (respects system text scaling up to 125%)
- Headings scale proportionally
- Button minimum height: 48dp (never compressed)
- Touch target minimum: 48x48dp

### Reduced Motion
- Disabled: Visualizer animation, micro-interactions
- Respected: System `reduceMotionEnabled` state
- Fallback: Static dot pattern instead of animated visualizer

---

## State Indicators & Feedback

### Playing State
- Play/Pause icon: filled red when playing
- Mini player: progress bar animates
- Now Playing: visualizer animates (if enabled, not on reduce motion)
- Lock screen: media notification shows playing state

### Buffering / Loading
- Circular progress indicator (36dp)
- Color: red accent
- Position: center of relevant container
- No splash screen or full-screen loading

### Error States

| Condition | Message | Action | Icon |
|---|---|---|---|
| Permission not granted | MUSIC ACCESS REQUIRED | ALLOW | Exclamation |
| Permission denied | MUSIC ACCESS DENIED | OPEN SETTINGS | Exclamation |
| Empty library | NO MUSIC FOUND | RESCAN | Music note |
| File missing | TRACK UNAVAILABLE | SKIP / REMOVE | Warning |
| Unsupported audio | Silent skip, optional toast | Continue | None |

---

## Device Compatibility & Responsiveness

### Screen Sizes Tested
- **Small** (4–5"): Pixel 3a, Moto G6
- **Medium** (5–6"): Pixel 5, Samsung A12
- **Large** (6.5"+): Pixel 6 Pro, Samsung S21+

### Orientation
- **Portrait** (primary): All screens fully optimized
- **Landscape** (secondary): No breaks, content reflows; mini player may hide

### Safe Areas
- Status bar: 24dp (with notch awareness)
- Bottom nav: 56dp + navigation bar (system insets)
- Padding: 16dp sides on larger screens, 12dp on smaller

---

## Performance Targets

- **Cold Start**: Interactive list < 2s on mid-range hardware
- **Library Scroll**: 10,000+ tracks smooth 60fps
- **Image Decode**: Only at display size (no upsampling)
- **Visualizer**: Stops rendering when screen off or Now Playing not visible
- **Battery**: No unnecessary animations or tickers

---

## Design Tokens Summary

```
Colors: 8 base colors + derivations
Typography: 4 font sizes (12, 14, 16, 18sp) + weights
Spacing: 4dp, 8dp, 12dp, 16dp, 24dp units
Corner Radius: 4dp (icons), 8dp (cards), 12dp (buttons)
Touch Targets: 48x48dp minimum
Shadows: Subtle, dark-adapted (not bright halos)
Dividers: 1px #2A2A2A
Transitions: 150–300ms cubic-bezier
```

---

## Deliverables Expected

1. **Mockup Images / Screens**:
   - All 9 main screens rendered at 1080x1920px (mobile standard)
   - 2-3 alternative orientations or device sizes
   - High-fidelity visual representation (not wireframe)

2. **Component Library**:
   - Reusable button styles (primary, secondary, icon)
   - Track row, album card, artist row components
   - Mini player component
   - Search bar and input fields
   - Chip / filter components
   - Visualizer component

3. **Interaction Specs**:
   - Tap, swipe, and long-press behaviors
   - Sheet animations and transitions
   - Error state displays
   - Loading states

4. **Color & Typography Guide**:
   - Hex codes for all colors
   - Font sizes and weights
   - Spacing and alignment grid

5. **Responsive Mockups** (optional):
   - Tablet layout (10" screen)
   - Landscape orientation

---

## Tone & Voice

- **Minimal**: No unnecessary text, icons do the work
- **Clear**: Labels are concise, all-caps for headers
- **Respectful**: No ads, no tracking language, privacy-first messaging
- **Mechanical**: Motion feels precise, not organic
- **Premium**: High contrast, clean spacing, industrial feel

---

## Example Micro-Copy

- "MUSIC ACCESS REQUIRED" (permission screen)
- "No cloud, no account, no tracking." (onboarding)
- "NO MUSIC FOUND" (empty library)
- "PLAY ALL" (album / artist detail)
- "UP NEXT" (queue header)
- "RECENTLY PLAYED" (home section)
- "Add to Favorites" (tooltip)
- "Shuffle: On/Off" (toggle label)
- "Sleep in 15 min" (timer notification)

---

## Next Steps for Gemini

1. Generate all 9 screen mockups as high-fidelity images
2. Create component variations library
3. Produce state variation mockups (loading, error, empty)
4. Optional: Generate Figma file export or Android XML UI specifications
5. Provide responsive variants for tablet and landscape

---

## Questions to Clarify (If Needed)

- Should the visualizer be FFT-based realistic or pure dot-matrix aesthetic? (Recommend: dot-matrix for v0.3, real FFT for v1.x)
- Primary font choice: Doto, DotGothic16, or custom? (Recommend: open-license OFL option)
- Landscape layout: mini player always visible or fold into top bar? (Recommend: fold into app bar)

