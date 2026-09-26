# iPhone Utility Platform Decision Guide

**Purpose:** Help decide which of three free, persistent platforms best fits a proposed personal iPhone utility, then start building it with Claude.

**Audience:** Written for Claude to use when helping the user evaluate a new project idea. It is also readable by the user directly.

**Last researched:** September 2026 (iOS 26). If a question hinges on a platform capability that may have changed, verify it before relying on this guide.

---

## Instructions for Claude

When the user brings a new project idea and this guide is in context:

1. **Restate the idea** in one or two sentences to confirm understanding.
2. **Run the intake questions** (Section 3). Answer as many as possible from what the user already said and ask only about the ones that change the outcome. Ask no more than three at a time.
3. **Check the hard requirements** (Section 4). A single disqualifier removes an option, however good it looks otherwise.
4. **Score the remaining options** against the fit signals (Section 5) and recommend one. Name a runner-up if the choice is close, and say what would tip it the other way.
5. **Consider a hybrid** (Section 6) when one platform covers the core and another adds a missing feature, such as a Home Screen icon, Siri, or a widget.
6. **Flag when none of the three fit** (Section 7) and say plainly what the alternative costs.
7. **Once the user agrees on a platform, plan the app** (features, data, what to leave out). Then write a **build brief** using the template in Section 11.

**Where each step happens:** Deciding and planning happen in a chat conversation inside a Claude Project. Building happens in Claude Code, in the user's `my-iphone-tools` repo, where the tool's folder contains the brief as `BRIEF.md`. A chat conversation should therefore end with a finished brief, not with code. If this guide is being read in Claude Code and a `BRIEF.md` exists, skip Steps 1–7 and build from the brief, using the workflow in Section 8.

Keep the recommendation short: the pick, two or three reasons tied to the user's answers, and any trade-off they should know about.

---

## 1. User Context

- **Personal use only.** Apps are for the user's own iPhone, with no App Store, sharing, or distribution needs unless the user says otherwise.
- **Must stay installed.** Anything requiring a reinstall or re-signing every 7 days is ruled out.
- **Tools available:** a Mac (comfortable with macOS and automation), Claude Desktop, Claude Code, iCloud Drive, and a GitHub account or other free static host.
- **Pythonista** is installed and holds older tools such as the original zip-to-county lookup. It is legacy only: no new projects there.
- **Offline use** matters to the user and should be treated as a common requirement.

---

## 2. The Three Options at a Glance

| | **Home Screen Web App (PWA)** | **Scriptable** | **Apple Shortcuts** |
|---|---|---|---|
| **What it is** | An HTML/CSS/JS page, hosted free (e.g., GitHub Pages) and added to the Home Screen from Safari | A free app that runs JavaScript scripts with access to many native iOS features | Apple's built-in automation app; shortcuts are chains of actions |
| **Cost** | Free | Free (optional tip) | Free, built in |
| **Stays installed** | Yes; iOS exempts Home Screen web apps from Safari's 7-day data deletion | Yes | Yes |
| **Offline** | Yes, after the first online visit (a service worker caches the files) | Yes | Yes, for on-device actions |
| **Best at** | Rich, custom interfaces: forms, lists, lookups, calculators, charts, small games | Native data plus light interfaces: widgets, calendars, reminders, files, notifications | System automation: Siri, triggers, the Action button, working across apps |
| **Interface quality** | Excellent, and fully customizable | Moderate (alerts, tables, or a web view) | Basic (prompts, menus, alerts) |
| **Home Screen / Lock Screen widgets** | No | **Yes** | Via the Shortcuts widget only (a launcher button) |
| **Siri / Shortcuts integration** | Can only be opened by URL | Yes, runs inside Shortcuts and accepts and returns values | **Native** |
| **Automatic triggers** (time, location, arriving home, NFC, opening an app) | No | Only when launched by a Shortcuts automation | **Yes** (Personal Automations) |
| **Local notifications** | No, unless you run a push server | Yes (can schedule them) | Yes (Show Notification) |
| **Access to Calendar, Reminders, Contacts, Health, Photos** | No | Calendar, Reminders, Contacts, Photos, Files, location (**not Health**) | Broad, including logging to Health |
| **Camera, microphone, location** | Yes, through the browser with permission | Photos, camera picker, location | Yes, via actions |
| **Data storage** | Browser storage on the device; **deleting the icon deletes the data** | Plain files in iCloud Drive (synced and easy to back up) | Files, Notes, or data embedded in the shortcut |
| **How Claude's work gets to the phone** | Push files to the host, then open the URL in Safari once | Save a `.js` file to iCloud Drive → Scriptable | Shortcuts Playground in Claude Code produces a signed `.shortcut` file; AirDrop or iCloud imports it |
| **Updating** | Push new files and bump the cache version; the app updates on its next online launch | Overwrite the `.js` file | Regenerate and re-import, or edit on the phone |
| **Easy to edit on the phone** | No | Yes (built-in editor) | Yes (visual editor) |
| **Could share later if wanted** | Easiest: send a link | Must send the script, and the other person needs Scriptable | Can share an iCloud link |

---

## 3. Intake Questions

Answer these from the user's description first; ask only the ones that matter.

1. **Interaction:** Is it mostly *look something up or calculate*, *enter and review data over time*, *automate or trigger something*, or *glance at information*?
2. **Interface:** Does it need a real screen (multiple fields, a list, a chart, custom layout), or is one question and one answer enough?
3. **Glanceable:** Should the information appear on the Home Screen or Lock Screen without opening anything? (Points to a widget.)
4. **Voice or hands-free:** Should Siri, the Action button, or Back Tap run it?
5. **Triggers:** Should it run automatically at a time, a place, when an app opens, or when the phone connects to something?
6. **iOS data:** Does it read or write Calendar, Reminders, Contacts, Health, Photos, or Files?
7. **Notifications:** Does it need to alert the user at a time or on a condition?
8. **Stored data:** Does it keep data the user creates? How much, and would losing it matter?
9. **Offline:** Must it work with no connection? Does it ever need live internet data?
10. **Size of embedded data:** Is there a fixed data set (like the zip table)? Roughly how many rows?
11. **Mac or other devices:** Would it be useful on the Mac or iPad too?
12. **Editing on the go:** Will the user want to tweak it on the phone without Claude?

---

## 4. Hard Requirements (Disqualifiers)

| If the project needs… | Rule out | Because |
|---|---|---|
| A Home Screen or Lock Screen widget showing live content | Web app | Web apps can't create widgets |
| Automatic triggers (time of day, location, arriving/leaving, NFC, app opened) | Web app; Scriptable on its own | Only Shortcuts Personal Automations provide triggers (they can launch a Scriptable script) |
| Scheduled local notifications without a server | Web app | Web Push needs a sending server |
| Reading or writing Calendar, Reminders, or Contacts | Web app | The browser has no access |
| Writing to Health | Web app, Scriptable | Only Shortcuts has Health actions |
| A rich custom interface (multi-field forms, filterable lists, charts, custom visuals) | Shortcuts | Its interface is limited to prompts, menus, and alerts |
| Large or growing user data that must not be lost | Web app *alone* | Deleting the icon wipes its data; add an export/backup feature or choose Scriptable (iCloud files) |
| True background processing or continuous tracking | **All three** | See Section 7 |

---

## 5. Fit Signals

After the disqualifiers, lean toward the option whose signals match best.

**Choose a Home Screen web app when:**
- The interface *is* the product: several inputs, results that update as you type, lists, tables, charts, or visual polish.
- It's a self-contained tool (lookup, calculator, converter, checklist, reference, tracker, quiz, flashcards, timer).
- Its data is fixed and embedded, or small user data with an export button is acceptable.
- It would also be handy on the Mac, iPad, or another computer (same URL).
- The user may want to share it someday.
- Claude can build and test it fully on the Mac before it goes on the phone (preview it in a browser).

**Choose Scriptable when:**
- A widget is central, such as a glanceable countdown, a daily figure, or a summary of the next event.
- It needs Calendar, Reminders, Contacts, Files, or Photos, with more logic than Shortcuts handles comfortably.
- Its logic is real code (parsing, calculations, loops over data) but the interface is simple.
- It should keep user data as files in iCloud Drive.
- It should be callable from Siri or Shortcuts while doing substantial work in JavaScript.

**Choose Apple Shortcuts when:**
- It's mainly *doing things across apps and the system*, such as setting focus, logging to Health, texting, starting a timer, or saving to Notes.
- It should run automatically on a trigger, or from Siri, the Action button, Back Tap, or Control Center.
- Its logic is short, roughly under 30–40 actions without heavy branching.
- One prompt and one result is enough interface.
- The user wants to tweak it on the phone without Claude.

**Watch-outs:**
- **Web app:** It must be opened online once to cache, and updates arrive only on the next online launch. Bump the service worker cache version on every change, or the old version keeps loading.
- **Scriptable:** The interface is utilitarian. Widgets refresh on iOS's schedule, not on demand. The app is maintained by one developer.
- **Shortcuts:** Large embedded data (hundreds of rows) is awkward to edit visually. Complex logic becomes hard to read. Some actions ask for permission on first run.

---

## 6. Hybrids

These often combine the strength of one option with a missing piece from another.

- **Scriptable + Shortcuts (most common).** Scriptable does the logic; a one-action shortcut ("Run Script") gives it a Home Screen icon, a Siri phrase, the Action button, or an automation trigger. The shortcut can pass input in and receive a result back.
- **Web app + Shortcuts.** A shortcut that opens the web app URL adds Siri or Action-button launching. It can't pass data into the page beyond URL parameters.
- **Shortcuts + web app for display.** A shortcut gathers data (for example, from Health) and opens the web app with it in the URL. Treat this as advanced and fragile; prefer a single platform when one is enough.
- **Scriptable widget + web app.** A widget shows a glanceable figure; tapping it opens the full web app via URL.

---

## 7. When None of the Three Fit

Say so plainly if the project needs:
- Background location tracking, continuous sensors, Bluetooth or NFC hardware beyond Shortcuts' support, or HealthKit *reading* with custom analysis.
- A native app with its own widgets, App Intents, or Live Activities.

The honest path is a native Swift app, which Claude Code can write in Xcode, **plus the $99/year Apple Developer Program**, which keeps the app installed for a year. A free Apple ID build expires after 7 days, so it doesn't meet the user's requirement. Before recommending this, offer a simplified version that fits one of the three options, if one exists.

---

## 8. Build Workflows

### A. Home Screen web app

**Default structure** (one folder per tool in a single GitHub Pages repo, e.g. `my-iphone-tools/zip-county/`):
- `index.html`: all HTML, CSS, and JS inline, with any fixed data embedded as a JSON object
- `manifest.webmanifest`: name, short_name, `display: "standalone"`, theme colours, icons
- `sw.js`: service worker that pre-caches every file, serves cache-first, and uses a versioned cache name (e.g., `v3`)
- `icon-192.png`, `icon-512.png`, and an `apple-touch-icon`

**Requirements for Claude's code:**
- Works fully offline after the first load; no external CDNs at runtime.
- Mobile-first layout: large tap targets, the right keyboard type (`inputmode="numeric"` for numbers), safe-area padding, and dark-mode support.
- User data in `localStorage` or IndexedDB, plus an **Export** button (downloads or shares JSON) and an **Import** option whenever the data matters.
- Visible version number in the footer so the user can confirm an update arrived.

**Deploy:** Push to the repo; GitHub Pages serves it. On the iPhone, open the URL in Safari → Share → Add to Home Screen (leave "Open as Web App" on). Launch once online, then test in airplane mode.

**Update:** Edit the files, bump the cache version in `sw.js` and the footer version, and push. Open the app online once; it may take a second launch to show the new version.

### B. Scriptable

**Default structure:** one `.js` file named for the tool, saved to `iCloud Drive/Scriptable/`, with any fixed data embedded or read from a JSON file in the same folder.

**Requirements for Claude's code:**
- Use Scriptable's APIs (`Alert`, `UITable`, `ListWidget`, `Calendar`, `Reminder`, `FileManager.iCloud()`, `Notification`, `Location`), and check them against the Scriptable documentation, since Claude's memory of them may be imperfect.
- Detect how it was launched: `config.runsInWidget` for the widget layout, `args.shortcutParameter` for input from Shortcuts, and interactive mode otherwise. Return results to Shortcuts with `Script.setShortcutOutput()`.
- Call `Script.complete()` at the end.

**Deploy:** Save the file into the iCloud Drive Scriptable folder from the Mac; it appears in the app on the phone. Add a widget (long-press the Home Screen → Scriptable widget → choose the script) and/or a one-action shortcut for a Home Screen icon or Siri.

**Update:** Overwrite the file.

### C. Apple Shortcuts (via Shortcuts Playground in Claude Code)

**Setup (one time):** Install Federico Viticci's `shortcuts-playground-plugin` in Claude Code on the Mac (needs Python 3.10+). It generates the shortcut, validates it, and signs it with macOS's `shortcuts` command-line tool.

**Build:** In Claude Code, describe the shortcut in plain English (the plugin's build command). Include any fixed data as a JSON text block for "Get Dictionary from Input." Keep dictionary values simple strings.

**Deploy:** Import the resulting `.shortcut` file (AirDrop it, or open it in Shortcuts on the Mac so it syncs through iCloud). Then add it to the Home Screen, assign it to the Action button, or create a Personal Automation.

**Without the plugin:** Claude can write numbered, action-by-action instructions for the user to build it by hand in the Shortcuts app. This works well for short shortcuts.

**Update:** Regenerate and re-import (delete the old one to avoid duplicates), or edit on the phone.

---

## 9. Worked Example: Oregon ZIP → County Lookup

- **Answers:** look-up interaction; one input and one answer, though showing multi-county splits benefits from a clean display; no widget, triggers, or iOS data; fixed data of about 480 ZIPs; must work offline; handy on the Mac too.
- **Disqualifiers:** none apply.
- **Recommendation:** **Home Screen web app.** It has instant as-you-type results, a clean display of multi-county ZIPs (e.g., "Mostly Clackamas 92% · also Multnomah 8%"), works offline, and runs on every device.
- **Runner-up:** **Shortcuts,** if the user wants to say "Hey Siri, Oregon county" and hear an answer. It's feasible with the data embedded as a dictionary, but multi-county results are harder to present.
- **Hybrid option:** add a shortcut that opens the web app URL for Action-button launching.
- **Data source:** HUD-USPS ZIP–County Crosswalk (quarterly), filtered to county FIPS codes beginning with `41`, keeping every county per ZIP with its residential share. Refresh about once a year.

---

## 10. Quick Decision Flow

```
Needs background tracking, Health reading, or native-only features?
  └─ Yes → Section 7 (native + $99/yr) or simplify the idea
  └─ No ↓
Needs automatic triggers, or is it mainly cross-app/system actions?
  └─ Yes → Shortcuts (use Scriptable inside it if the logic is heavy)
  └─ No ↓
Needs a widget, or Calendar/Reminders/Contacts/Files access?
  └─ Yes → Scriptable (+ a one-action shortcut for an icon/Siri)
  └─ No ↓
Needs a real interface, or useful on other devices too?
  └─ Yes → Home Screen web app
  └─ No → Simple one-question tool: Shortcuts (quickest) or web app (nicer)
```

---

## 11. Build Brief Template

**Instructions for Claude (in chat):** When planning is done, fill in this template and present it as a single Markdown code block so the user can copy it. Fill every section from the conversation. Write "None" where something doesn't apply, and ask the user about anything still undecided rather than guessing. Keep it short and concrete: it is a spec for Claude Code, not a pitch. The user saves it as `<tool-folder>/BRIEF.md` in the `my-iphone-tools` repo.

**Instructions for Claude (in Claude Code):** Treat `BRIEF.md` as the source of truth for scope. If something in it is ambiguous or can't be built as described, ask before improvising. When the build changes a decision, update the brief's Change Log.

```markdown
# Build Brief: <Tool Name>

**Folder:** `<tool-folder-name>`   (lowercase-with-hyphens)
**Platform:** Home Screen web app | Scriptable | Shortcuts | <hybrid: which parts on which>
**Date:** <YYYY-MM-DD>

## 1. What it does
<One or two sentences, from the user's point of view.>

## 2. Why this platform
<Two or three reasons tied to the decision guide, plus the runner-up and why it lost.>

## 3. How it's launched and used
- Launch: <Home Screen icon / widget / Siri phrase "..." / Action button / automation trigger>
- Main flow: <step-by-step: what the user does, what they see>

## 4. Features
**Must have (v1):**
- <feature>
**Nice to have (later):**
- <feature>
**Deliberately out of scope:**
- <thing we decided not to do, and why>

## 5. Data
- Fixed/embedded data: <what, source URL or file, size, how to prepare it, how often to refresh>
- User data it stores: <what, where, backup/export plan> or "None"
- Live/online data: <what, from where> or "None (fully offline)"

## 6. Interface notes
<Layout, inputs (e.g., numeric keypad), result display, empty/error states, dark mode, any visual preferences.>

## 7. Edge cases
- <e.g., input that isn't valid, ZIP not in Oregon, ZIP spanning several counties, no network on first launch>

## 8. iOS features and permissions
<Widgets, notifications, Calendar/Reminders/Location access, Shortcuts input/output, and so on> or "None"

## 9. Done when (acceptance tests on the iPhone)
- [ ] <e.g., works in airplane mode after the first launch>
- [ ] <e.g., "97220" returns "Multnomah County">
- [ ] <e.g., an update shows the new version number after one online launch>

## 10. Open questions
- <anything to resolve during the build> or "None"

## Change Log
- <YYYY-MM-DD>: Brief created.
```
