# Troop Meeting Board — User Guide

The **Troop Meeting Board** (https://troop402.github.io/meeting-time/) is an ambient digital agenda display designed for youth-led troop meetings. It runs on phones, tablets, laptops, and large cabin TV displays via Chromecast.

---

## 1. Quick Start for Troop Leaders & Scouts

### Adding to Home Screen (Mobile Web App)
Running the board as an installed web app hides browser address bars and provides a clean, full-screen experience.

- **iPhone / iPad (Safari):**
  1. Open https://troop402.github.io/meeting-time/ in Safari.
  2. Tap the **Share** button (the square with an arrow pointing up).
  3. Scroll down and tap **Add to Home Screen**.
  4. Tap **Add** in the top right.
- **Android (Chrome):**
  1. Open https://troop402.github.io/meeting-time/ in Chrome.
  2. Tap the three dots menu (top right).
  3. Tap **Install app** or **Add to Home screen**.

---

## 2. Loading the Weekly Agenda

Before each meeting, the SPL or Scribe distributes the meeting link or raw agenda text.

### Option A: 1-Tap Link Import (Fastest)
1. Copy the shared meeting link (e.g. `https://troop402.github.io/meeting-time/#agenda=...`) from text message, email, or chat.
2. Open the Meeting Board on your phone.
3. Tap **📥 Import / Edit Raw** at the top.
4. Tap **📋 Paste from Clipboard** (or paste the link into the box).
5. Confirm the prompt to import. The entire agenda, tabs, and meeting title will load immediately.

### Option B: Raw Text Import
If working from plain text or an email summary:
1. Tap **📥 Import / Edit Raw**.
2. Paste the formatted text into the box (see formatting reference below).
3. Tap **Save to Agenda**.

---

## 3. TV Casting & Cabin Presentation

The app is built to cast to a Chromecast or Google TV screen while letting the meeting facilitator maintain control from their phone.

### Starting a Cast Session
1. Connect your phone or laptop to the cabin Wi-Fi.
2. Open the Meeting Board in Google Chrome.
3. Tap the **Cast** button in the top navigation.
4. Select the cabin display or Chromecast from the list.

### Autonomous TV Mode (Phone Sleep Resilience)
* **Phone Can Sleep:** Once casting begins, the TV runs completely independently. You can lock your phone screen or put it in your pocket—the TV will continue cycling through topics and tabs.
* **Auto-Rejoin:** When you unlock your phone, the app automatically reconnects within a few seconds and shows **`[ 📡 Reconnecting... ]`** until live control is restored.
* **Disconnecting vs. Stopping:**
  * Tap the Cast button and select **🔌 Disconnect Phone (Keep TV Running)** if you want to leave the cabin or close your browser without interrupting the TV display.
  * Tap **🛑 Stop Casting** only when the meeting has ended and you want to shut off the TV receiver.

---

## 4. Running the Meeting (SPL / Facilitators)

### Ambient Time Tracking
* **Clock Sync:** The board automatically advances the highlighted topic based on real-world wall clock time.
* **Approaching Transition Alert:** 1 minute before the next agenda milestone, the **Next Up** card gently pulses in size to prompt instructors and patrols to wrap up.

### Navigation & Manual Override
* **Advancing Topics:** Tap the **Next Up** card, tap **▶**, or swipe left on the screen to manually move forward.
* **Previous Topic:** Tap **◀** or swipe right.
* **Returning to Auto-Advance:** If you manually navigate away from the current schedule, tap the **Resume Auto** button to re-sync with the clock.

### Side Cards (Patrols, Dates, Announcements)
* In presentation mode, the board automatically cycles between the main agenda (40s) and active side cards (20s).
* Tapping any colored tab on the left immediately brings that card into focus on both phone and TV.

---

## 5. Live Editing During the Meeting

Need to change an announcement or shift a time slot on the fly?
1. In the **Elements & Agenda** timeline, tap the item you want to edit.
2. Update the time, title, or details in the editor panel below.
3. Tap **Save**.
4. If actively connected to the TV, the update pushes to the TV screen instantly.

---

## 6. Raw Agenda Text Format Reference

The bulk importer reads simple plain text structured as follows:

```text
🦉 September 23, 2026 - Wood Tools & Fire Building

7:00 PM Meeting Set Up
- ASMs: Unlock Cabin, Plug in Outdoor Lights, Open Doors
- Troop Guides: Verify materials for Skills Stations
- Athena Patrol: Flag Ceremony setup

7:15 PM Opening
- Flag Ceremony
- Scout Oath and Law
- Announcements

7:30 PM Patrol Breakout
- Grab Patrol Book, Take Attendance & Uniform Check
- Upcoming Overnight Event Details

8:00 PM Skills Instruction - Stations
- Green Station: Pocket knife safety & fire safety
- Blue Station: Fire building & safe extinguishing
- Black Station: Campfire environmental guidelines

8:20 PM Advancements
- JASMs, Golden Eagles: Sign offs at the stage
- ASMs: Scoutmaster Conferences at the fireplace

8:40 PM Closing Ceremony
- Flag Ceremony & SM Minute

=== TABS ===
# Patrols [x]
- Sister Patrols
-- Falcon / Red Panda
-- Honey Badger / Mustang
-- Gator / Sun Bear

# Dates [ ]

# Announcements [x]
- Parent’s Meeting: Sep 27
- Mount Lassen Campout: Oct 10-12
```
