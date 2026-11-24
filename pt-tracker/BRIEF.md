# Build Brief: PT Tracker

**Folder:** `pt-tracker`
**Platform:** Home Screen web app
**Date:** 2026-10-13

## 1. What it does
Keeps my place through 5 daily physical therapy sessions: I tap a session, check off each set of each exercise, and finished sessions dim until I reset for the next day.

## 2. Why this platform
- The interface is the product: a list of session buttons and a grid of set buttons with done/undone states.
- Fully offline, no iOS data, notifications, widgets, or triggers needed.
- Runner-up: Scriptable. It could do this, but its interface (UITable/alerts) would be clunky for tap-heavy checklists. Shortcuts is ruled out by the custom interface.

## 3. How it's launched and used
- Launch: Home Screen icon
- Main flow:
  1. Sessions screen shows Sessions 1–5. Finished sessions are dimmed with a check; the first unfinished one shows an "Up next" highlight; a partly done session shows "X of Y sets".
  2. Tap a session → exercise screen for that session.
  3. Tap Set 1 / Set 2 for each exercise as I finish it. Tapping a done set undoes it.
  4. When every set in the session is done, return to the Sessions screen immediately (no pause or message).
  5. "‹ Sessions" back button leaves mid-session; partial progress is kept.
  6. "Reset sessions" button at the bottom asks "Reset all sessions?" and, if confirmed, clears all progress.

## 4. Features
**Must have (v1):**
- Sessions screen with 5 session buttons, dimmed state for completed sessions, "Up next" highlight, partial-progress text
- Exercise screen: one screen template that loads the odd list (sessions 1, 3, 5) or even list (sessions 2, 4)
- Two set buttons per exercise, tap to mark done, tap again to undo
- Auto-return to Sessions screen when the session's last set is marked done
- Dimmed (completed) sessions can still be opened, to undo a set
- Reset sessions button with a confirmation dialog
- Progress persists if the app is closed or the phone restarts
- Exercise lists defined in one clearly marked data block at the top of the code, so Claude Code can update them easily
- Works offline; version number in footer

**Nice to have (later):**
- None identified

**Deliberately out of scope:**
- History, streaks, or logs. I only need to track today.
- Reminders/notifications. My session times aren't regular.
- Automatic daily reset. I reset manually.
- In-app exercise editor. I'll ask Claude Code to update the lists.

## 5. Data
- Fixed/embedded data: two exercise lists; every exercise is 2 sets × 10 reps unless noted:
  - Odd sessions (1, 3, 5), 7 exercises:
    1. Seated elbow bend & straighten (no hand, thumb up)
    2. Standing elbow extend (start arm curled up, hand over, palm up)
    3. Seated elbow bend & straighten (hand under, palm up, elbow on table)
    4. Tendon glides (palm out, table/palm/claw/half-fist)
    5. Finger roll stretch (palm up)
    6. Finger extension (palm down on table)
    7. Elbow on towel on edge of table. Straighten-hold 2 min. Add weight-hold 5 min (1 set, timed holds, no reps)
  - Even sessions (2, 4), 8 exercises:
    1. Seated elbow bend & straighten (no hand, thumb up)
    2. Standing elbow extend (start arm curled up, hand over, palm up)
    3. Seated elbow bend & straighten (hand under, palm up, elbow on table)
    4. Tendon glides (palm out, table/palm/claw/half-fist)
    5. Finger extension (palm down on table)
    6. Elbow on towel on edge of table. Straighten-hold 2 min. Add weight-hold 5 min (1 set, timed holds, no reps)
    7. Resistance Extension-back straight
    8. Resistance Flex-back straight
  - Daily extra (not part of any session): Putty squeezes, 5-10 min, each hand, done twice a day. Shown as its own card under "Also today" on the Sessions screen with two tick boxes. It doesn't count toward sessions done; Reset clears it too.
  - Store reps/sets per exercise (e.g., `{name, sets: 2, reps: 10}`) so a future change to one exercise is easy.
- User data it stores: which sets are done in each session, in localStorage. No export needed; losing it only loses today's place.
- Live/online data: None (fully offline)

## 6. Interface notes
- Sessions screen: title "Today's PT", subtitle "N of 5 sessions done". Large full-width session cards (name + "8 exercises" / "5 exercises"). Done = dimmed + check; up next = accent border + "Up next" badge; others = chevron. Reset button centered below the list; version number in footer.
- Exercise screen: "‹ Sessions" back link, session title, "X of Y sets · 10 reps each". Header row "Exercise / Set 1 / Set 2". Each row: exercise name on the left (wraps to two lines), two large square set buttons on the right (min 44pt tap targets). Done set = green fill with check.
- If an exercise's reps differ from 10 in future, show reps on that row.
- Mobile-first, safe-area padding, dark mode support.

## 7. Edge cases
- Undoing a set in a completed session → session becomes un-dimmed and shows partial progress.
- Leaving a session mid-way → progress kept; card shows "X of Y sets".
- Reset tapped by accident → confirmation dialog; Cancel changes nothing.
- All 5 sessions done → no "Up next" badge; subtitle shows "5 of 5 sessions done".
- Exercise lists change in a future update → stored progress for removed/renamed exercises should be ignored or cleared without breaking the app.
- No network on first launch → must be opened online once to cache.

## 8. iOS features and permissions
None

## 9. Done when (acceptance tests on the iPhone)
- [ ] Works in airplane mode after the first launch
- [ ] Session 1 shows 7 exercises (11 sets); Session 2 shows 8 exercises (15 sets)
- [ ] Wrist flex/extension are gone; elbow-on-towel reads "…Straighten-hold 2 min. Add weight-hold 5 min" with one set button
- [ ] Putty squeezes card on the Sessions screen: two boxes tick/untick, and Reset clears them
- [ ] Tapping a set marks it done; tapping again undoes it
- [ ] Marking the last set of a session returns to the Sessions screen immediately, with that session dimmed
- [ ] Opening a dimmed session and undoing a set un-dims it
- [ ] Leaving mid-session and returning keeps checked sets
- [ ] Progress survives closing the app from the app switcher
- [ ] Reset asks for confirmation, then clears all sessions
- [ ] An update shows the new version number after one online launch

## 10. Open questions
- None

## Change Log
- 2026-10-13: Brief created.
- 2026-10-13: v1.0 built as specified. Progress is stored per exercise name, so renaming or removing an exercise just drops its checkmarks.
- 2026-10-14: v1.1 – reworded five exercises after phone testing (both lists). Checkmarks on renamed exercises reset once.
- 2026-10-31: v1.2 – added the elbow-on-towel hold (1 set, no reps) to the end of both lists.
- 2026-11-13: v1.3 – wrist flex and wrist extension cut to 1 set. Added putty squeezes (twice a day) as a separate "Also today" card on the Sessions screen rather than inside sessions, because the timing doesn't follow the sessions.
- 2026-11-23: v1.4 – program change: removed wrist flex and wrist extension; elbow-on-towel shortened to "Straighten-hold 2 min. Add weight-hold 5 min" (still 1 set); added Resistance Extension and Resistance Flex (back straight, 2×10) to the end of sessions 2 and 4. Checkmarks on the renamed towel row reset once.
