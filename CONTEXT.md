# Project Context & AI Guidance: `meeting-time`

> **NOTICE FOR AI ASSISTANTS (Gemini, Claude, ChatGPT, Copilot, etc.):**  
> This document is the comprehensive ground-truth context and design intent for the `meeting-time` project. Read this document thoroughly before proposing code changes, answering questions, or refactoring. It captures the project history, hardware constraints, user philosophy, past AI regression pitfalls, and immutable design contracts.

---

## 1. Project Overview & URLs
- **Repository:** `troop402/meeting-time` (GitHub)
- **Live Deployment:** [https://troop402.github.io/meeting-time/](https://troop402.github.io/meeting-time/)
- **Core Principle:** **Zero-build, zero-server, single-file application**. The entire app runs directly in client browsers from a single [`index.html`](file:///workspaces/meeting-time/index.html) file hosted on GitHub Pages, accompanied only by standard PWA assets ([`site.webmanifest`](file:///workspaces/meeting-time/site.webmanifest) and app icons). No Node.js build steps, no webpack/vite bundles, no npm runtime dependencies, and no backend database.

### Organization Transfer Checklist (Troop 402)
1. **GitHub Pages Re-Enablement:** In https://github.com/troop402/meeting-time/settings/pages, ensure GitHub Pages is enabled (Source: `Deploy from a branch`, Branch: `main`, Folder: `/ (root)`).
2. **Google Cast Developer Console (`1909D2D6`):** In https://cast.google.com/publish/, update application `1909D2D6` Receiver Application URL to:
   `https://troop402.github.io/meeting-time/?view=presentation`
   *(Changes typically take 10-15 minutes to propagate across Google's Cast servers).*
3. **Codespaces Access & Policies:** In organization settings (https://github.com/organizations/troop402/settings/codespaces), ensure member access is permitted and billing limits are active.
4. **Git Remote URL:** Configured to `https://github.com/troop402/meeting-time.git`.
5. **Cast Namespace:** Configured to `urn:x-cast:troop402.meetingtime` across sender and receiver.

---

## 2. Real-World Use Case & User Intent

### The Problem Being Solved
The app was created specifically for **Scout Troop meetings** (and board-meeting-style agendas) where youth leadership and adult advisors run structured, multi-topic schedules together. 

### Hardware & Environment Constraints
1. **Display & Lighting Environments:** Meetings happen in halls, classrooms, or dim boardrooms. **Dark mode is the default application theme** (providing high-contrast legibility, energy efficiency on OLED displays, and an ambient presentation glow), with a **Light mode toggle** available for high-ambient projector environments where maximum lumens are desired.
2. **Projector Displays (4:3 and 16:9):** The app is projected onto a large screen, typically driven by a mobile phone, tablet, or laptop plugged into an HDMI cable or wireless casting receiver.
3. **Passive Ambient Display:** During the meeting, attendees and leaders should not have to manually advance slides or fiddle with a computer. The screen operates hands-off:
   - Displays **"Now"**: The current agenda item, its scheduled time, system clock, and detailed markdown notes.
   - Displays **"Next Up"**: A prominent alert card showing what topic starts next to keep speakers on time.
   - **Side Cards (Tabs)**: Intermittently rotates in general troop reference cards (**Patrols**, **Dates**, and **Announcements**) without interrupting the flow of the meeting.

---

## 3. History of Pain Points & Preventing Regressions

Over hundreds of iterations, multiple AI coding sessions suffered from recurring regressions. **Any AI working on this codebase must understand these past pitfalls and avoid repeating them:**

### 1. Inadvertent Loss of Working Features (The Need for "App Contracts")
- In earlier iterations, LLMs rewriting `index.html` frequently deleted subtle logic (e.g., losing visible button borders, breaking wake lock retention, breaking swipe deadzones, or ruining aspect-ratio typography).
- **The Solution:** Lines 11–69 of [`index.html`](file:///workspaces/meeting-time/index.html#L11-L69) contain the formal **APP CONTRACTS**. These rules represent hard-fought requirements. **Never alter or remove code satisfying these rules without the user's explicit consent.**

### 2. Screen Wake Lock Regressions
- **Requirement:** The device screen **must never sleep** during a meeting.
- **The Trap:** AIs frequently attached `wakeLock.release()` to the "Pause" button, causing tablets to go dark when paused.
- **The Rule:** The Screen Wake Lock API (`navigator.wakeLock`) must be acquired upon play or user resume, **must persist while paused**, and must automatically re-acquire via `document.addEventListener('visibilitychange')` whenever the browser returns to the foreground.

### 3. Dual-View Snapping vs. Halfway Scrolling
- **Requirement:** The interface is strictly **two screens stacked vertically**:
  1. **Top Screen (Presentation View):** Projector presentation.
  2. **Bottom Screen (Management View):** Timeline editor, element controls, and bulk text import.
- **The Rule:** Uses CSS scroll snapping (`scroll-snap-type: y mandatory`). Vertical scroll locks to either view and must **never rest halfway between them**. Do NOT break `height: calc(var(--vh, 1vh) * 100)` or add root scrollbars.

### 4. Touch Gestures & Diagonal Swipe Conflicts
- **Requirement:** Horizontal swiping (◀ / ▶) on the presentation screen shifts previous/next agenda items.
- **The Trap:** Diagonal swiping while trying to scroll down to the Management View accidentally skipped agenda items.
- **The Rule:** Touch handlers have a minimum deadzone (25px) and an axis-determination check. If vertical movement exceeds horizontal movement, the touch locks to vertical scrolling and horizontal swipe gestures are suppressed.

### 5. Playback Auto-Advance vs. Manual Inspection
- **Requirement:**
  - Real-time time sync: When playing, the app checks the system clock against agenda item scheduled times and auto-advances at real-time milestones.
  - **Approaching Milestone Size Pulse:** Exactly 1 minute (60 seconds) prior to the next scheduled agenda milestone, the Next Up card pulses in size (`pulse-active` with `alertPulse` keyframes). This visual warning fires **regardless of whether playback is playing or paused** to signify real-world time relative to the schedule.
  - Intentional navigation (clicking a tab, swiping to another topic, clicking a timeline item) **pauses** auto-advancing.
  - Passive actions (viewing, scrolling down to manage) do **not** pause.
  - **Auto-Resume:** Navigating back to the currently scheduled timeline item **automatically unpauses** and resumes live auto-advance.

### 6. Rejecting External Storage & API Dependencies
- Pastebin, GitHub Gists, Google Keep, and external database APIs were evaluated and rejected. External APIs introduce API keys, rate limits, offline vulnerability, and service deprecation risks.
- **The Rule:** Lossless persistence uses **`localStorage`** for offline device saving, and **`CompressionStream('deflate-raw')`** to compress the entire meeting title, agenda items, and tab contents into the URL hash (`#agenda=...`). The entire meeting state travels within the link itself.

### 7. Bidirectional Import / Export
- The plain-text format exported in the **"Import / Edit Raw"** modal must be parseable back into the app without data loss.
- Format: `HH:MM AM/PM Topic Name` followed by `-` bullets, `--` sub-bullets, and `=== TABS ===` with `# Tab Name [x]` for side-cards.
- Destructive imports wipe the in-memory undo stack and require a confirmation dialog.

### 8. Casting & Dual Authority Remote Control (CAF & W3C Presentation API)
- **Dual Authority Model:**
  - **Playing / Unpaused State:** The TV presentation is autonomous and is the primary Source of Truth, driven by its local wall clock. Whenever it advances to a new agenda milestone, it broadcasts `RECEIVER_STATUS` to the phone controller so an awake phone immediately mirrors the active topic.
  - **Interaction / Paused State:** When an intentional user action occurs on the phone controller (tapping a side tab, clicking next/prev, or toggling pause), both the TV and phone transition to `isPlaying: false`. In this paused state, the phone controller is authoritative, and the TV presentation faithfully follows every manual navigation step.
  - **Unpausing / Resuming:** Resuming playback restores the TV presentation as the autonomous authority on wall-clock time.
- **Pocket Independence & Battery Preservation:**
  - When actively casting, the phone intentionally releases its screen wake lock so the device can sleep naturally in the user's pocket.
  - The TV presentation receiver independently maintains its own keep-alive (`disableIdleTimeout = true`).
  - **Android PWA Setting:** The user should set the installed Chrome app's battery usage to **"Unrestricted"** (instead of "Optimized"). While mobile OSs still pause background WebSockets when the screen turns off, "Unrestricted" prevents Android from killing the Chrome tab process in RAM, allowing instant wake-up without full-page reloads.
- **Silent Auto-Rejoin on Wake:**
  - When the phone is unlocked (`visibilitychange === 'visible'`), it silently probes the Cast connection and requests live position (`REQUEST_STATUS`).
  - Under W3C Presentation API, `reconnect(savedSessionId)` runs programmatically without requiring user gestures.
  - Under Google Cast SDK, `autoJoinPolicy: ORIGIN_SCOPED` re-attaches to the existing running TV session without reloading or interrupting the TV screen.
- **Ghost State Purging & Permanent Disconnect Detection:**
  - If the TV is powered off, or the user leaves the venue/Wi-Fi, the app **must never pretend it is still connected**.
  - **Triggers:** `CAST_STATE_CHANGED` (`NO_DEVICES_AVAILABLE`), `SESSION_STATE_CHANGED` (`NO_SESSION`, `SESSION_ENDED`, `SESSION_START_FAILED`), action errors in `sendMessage`, and a 3-second probe timeout automatically clear `isCasting = false` and wipe stored session IDs.
- **Receiver Kiosk Experience:** In presentation-only / receiver view (`?view=presentation`), interactive controls (play/pause buttons, cast buttons, edit triggers) are stripped from the DOM. If paused by the controller, an unobtrusive non-interactive `⏸ Paused` status badge is displayed.
- **Disabled Tab DOM Exclusion:** Disabled/unchecked side tabs are completely excluded from the presentation DOM so they do not show buttons or interfere with automated card cycling on either device.

### 9. Management View Virtual Keyboard Bouncing Fix (`.is-editing`)
- **The Problem:** On mobile devices, focusing an `<input>` or `<textarea>` caused the screen to wildly bounce and jitter on every keystroke.
- **The Root Cause:** `body, html` had `scroll-snap-type: y mandatory; scroll-behavior: smooth;`. When the mobile soft keyboard opens and the user types, the browser automatically adjusts scroll position to keep the caret visible. The CSS scroll-snap engine immediately fought this adjustment, snapping back to align-start and animating the fight on every keystroke.
- **The Rule:** An `.is-editing` class is toggled on `html` and `body` on `focusin`/`focusout` of any `INPUT` or `TEXTAREA`. When active, `scroll-snap-type: none !important;` and `scroll-behavior: auto !important;` are enforced, completely eliminating keyboard bouncing.

### 10. Legacy Chromecast (2018 3rd Gen) Twemoji Polyfill
- **The Problem:** The 2018 3rd Generation Chromecast runs Eureka OS, which lacks native OS color emoji glyphs. Emojis rendered as empty rectangular boxes ("tofu").
- **The Rule:** Twemoji is loaded via CDN (`@twemoji/api`). Calling `twemoji.parse(document.body)` dynamically converts Unicode emojis to inline SVG/PNG images with zero local bundling or build steps.

### 11. Side-Tab Emoji Stripping vs. Card Header Retention
- **The Rule:** Side-tab pill buttons use `stripEmojis(tab.title)` so pills like `🚩 Announcements` display cleanly as `Announcements` on the vertical pill without rotated/misaligned emoji artifacts. The expanded card header (`.tab-content-area h2`) retains the full title with the emoji.

### 12. Agenda Distillation (Lesson Plan vs. Presentation Display)
- **The Rule:** Long-form troop agenda documents contain detailed facilitator guides, instructor lists, advancement requirements, and setup protocols. When converting them for the app, distill them into high-level, presenter-friendly bullet points suitable for quick ambient scanning on a shared screen.
- **Golden Format Reference:** See [`docs/agendas/2026-09-23-wood-tools-fire-building.txt`](file:///workspaces/meeting-time/docs/agendas/2026-09-23-wood-tools-fire-building.txt) for the standard troop format, including the regular `8:20 PM Advancements` slot (JASMs/Golden Eagles sign-offs on stage, ASMs conferences at fireplace), concise sub-bullet hierarchies, and untouched sister patrol pairings.

---

## 4. Technical Architecture & Tech Stack

### Libraries (Loaded via CDN)
- **CSS:** [Pico CSS v2](https://picocss.com/) (`https://cdn.jsdelivr.net/npm/@picocss/pico@2/css/pico.min.css`) provides clean semantic CSS variables and dark/light mode foundations.
- **Reactivity & State:** [Alpine.js v3](https://alpinejs.dev/) (`https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js`) provides lightweight, declarative reactivity directly in HTML attributes (`x-data`, `x-bind`, `x-on`, `x-show`, `x-model`).

### Key Configuration Block: `APP_CONFIG`
Located at lines 96–121 in [`index.html`](file:///workspaces/meeting-time/index.html#L96-L121):
```javascript
const APP_CONFIG = {
    appearance: {
        defaultTheme: 'dark'     // Options: 'dark' (default) or 'light'
    },
    timing: {
        agendaDuration: 40,      // Seconds the main agenda is displayed
        cardDuration: 20,        // Seconds each sidecard tab is displayed
        debugDuration: 5,        // Seconds for fast cycle simulation mode
        bounceLeadTime: 3,       // Seconds before tab transition to trigger the "bounce/tug" animation
        carouselInterval: 8000,  // Milliseconds between flipping pages of long text
        refreshTimeout: 180000   // Milliseconds before auto-refresh check gives up
    },
    swipe: {
        deadzone: 25,            // Pixels to swipe before registering the action
        dragDistance: 70         // Pixels to swipe to complete the transition
    },
    storage: {
        agendaKey: 'boardAgenda',
        tabsKey: 'boardTabs',
        themeKey: 'boardTheme_v2',
        titleKey: 'boardMeetingTitle',
        castSessionKey: 'meeting_presentation_id'
    },
    cast: {
        appId: '1909D2D6',       // Google Cast SDK Developer Console Custom Web Receiver Application ID
        namespace: 'urn:x-cast:troop402.meetingtime',
        maxInactivity: 10800     // Seconds (3 hours) of inactivity before receiver shuts down
    },
    history: {
        maxItems: 20             // Undo history stack limit
    }
};
```

### Side Cards / Rotating Tabs Mechanics
- Three fixed tabs: **Patrols** (`#16a34a` green), **Dates** (`#2563eb` blue), and **Announcements** (`#ea580c` orange).
- **Deck-Dealing Animation:** Enabled side cards slide in from the right edge (`translateX(0)`), overlaying the main agenda, and slide back out when returning to the agenda.
- **Visual Cues:** Tab progress bars fill during the display cycle; a `@keyframes softTug` bounce triggers 3 seconds before transition as a visual notice.
- **Remote Side-Tab Control:** Clicking the "Agenda" tab on a connected controller normalizes target to `null`/`agenda`, clears `visibleTabs` on both phone and TV, and applies direct DOM `.dealt` cleanup to immediately collapse side-cards on the TV.
- **Approaching Milestone Pulse:** 60 seconds before an upcoming agenda milestone, the Next Up card pulses in size (`alertPulse`) across both controller and TV presentations. Can be tested manually in the Debug Panel via "Simulate 1-Min Auto-Advance Pulse (Next Up)".
- **Debug Panel:** Includes "Fast Cycle Simulation" (5s mode), "Simulate 1-Min Auto-Advance Pulse", and real-time wake lock telemetry accessible via the discreet `⚙️ Debug` link in the Management View.

### Typography & Mobile Responsiveness
- **Landscape Scaling:** Text and UI scale proportionally via `clamp()` and `vh` units so projector displays (low or high res) remain legible.
- **TV Details Viewport:** Dedicated sizing on `body.presentation-only .details-panel .details-viewport` via `clamp(0.95rem, 4.4vh, 3.5rem)`, producing ~47.5px at 1080p. This hits the ideal balance between long-distance readability and preventing multi-page carousel blowouts.
- **Portrait Lock:** In portrait orientation (used when editing on a phone), typography decouples from `vh` to prevent oversized text from overflowing the viewport.
- **Management View Usability:** In mobile portrait, the Management View uses dedicated compact typography and tightened padding matching landscape usability to maximize screen real estate for the timeline and editor.
- **Mobile Input Stability & Keyboard Protection:** Form inputs and textareas use a fixed base font size (16px) to eliminate mobile browser auto-zoom, and `--vh` calculation is suppressed while text inputs are actively focused to prevent keyboard open/close viewport bouncing and scroll re-snapping.
- **Fluid Lists:** List indents scale with font size so bullets never clip outside containers.
- **Mobile Viewport Fix (`--vh`):** A custom JS handler calculates real viewport height on resize and orientation shifts to counteract mobile browser UI address bars and Android PWA launch rendering races.

---

## 5. Live UI Sizing Prototyper & Active Default Config

To eliminate past trial-and-error around screen scaling, a **Live Sizing & Layout Prototyper** is embedded directly into the Developer Debug modal (`⚙️ Debug`).

### Current Active Default Settings
The app is currently configured with the **Harmonic Proportional** base profile and user-tuned scale overrides:
```json
{
  "preset": "harmonic",
  "scale": "100%",
  "leftCol": "35%",
  "titleScale": "95%",
  "detailsScale": "100%",
  "nowScale": "140%",
  "nextUpHeaderScale": "140%",
  "nextUpTitleScale": "130%",
  "mgmtContinuity": true
}
```
- **CSS Custom Properties on `:root`**:
  - `--proto-scale`: `1.0` (Global scale factor)
  - `--proto-left-col`: `35%` (Left column width in landscape grid)
  - `--proto-title-scale`: `0.95` (Title heading size multiplier)
  - `--proto-details-scale`: `1.0` (Details body font multiplier)
  - `--proto-now-scale`: `1.4` (NOW heading & system clock scale multiplier)
  - `--proto-nextup-header-scale`: `1.4` (Next Up header and time multiplier)
  - `--proto-nextup-title-scale`: `1.3` (Next Up title text multiplier)
  - `--proto-nextup-scale`: `0.95` (Legacy Next Up card multiplier)
- **Management View Continuity (`body.proto-mgmt-continuity`)**: Active by default. Aligns Management View column width, card borders, and timeline row font sizing with the presentation styling.
- **Dynamic Text Pagination**: `renderPresentationTextPages()` uses computed style font sizing and line height to guarantee accurate page splits across all scales.
- **Live TV Synchronization over Cast**: When actively connected to a TV (`isCasting`), slider adjustments and default resets stream to the cast receiver in real-time (`SET_PROTO` with a 50ms trailing debounce) and are bundled into `SYNC_STATE`. This allows hands-on tuning of TV presentation typography and layout directly from a phone controller.
- **Further Prototyping on Mobile**: The user can open `⚙️ Debug` on their phone to adjust sliders, test in landscape/portrait, or tap `📋 Copy Config` to export updated values.

---

## 6. Guidelines for Future AI Assistance

When interacting with the user or modifying this codebase:
1. **Respect the App Contracts:** Review lines 11–69 of [`index.html`](file:///workspaces/meeting-time/index.html#L11-L69) before touching layout, state, or event handling.
2. **Be Surgical & Concise:** Explain the "why" behind changes before outputting code.
3. **Preserve Comments & Structure:** Do NOT strip out configuration blocks, inline comments, or contract definitions.
4. **Never Force External Dependencies:** Do not recommend npm packages, node servers, or backend databases unless explicitly requested. Everything must remain self-contained in static client-side files.
5. **Acknowledge Mobile Vibe Coding:** The user frequently codes from mobile devices and phone browsers. Keep solutions practical, clean, and easily testable on GitHub Pages.

