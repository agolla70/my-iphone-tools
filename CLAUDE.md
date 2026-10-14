# my-iphone-tools

Small personal utilities for my iPhone (iOS 26), built with Claude. Every tool is for my own use only and must stay installed without re-signing. Most should work offline.

The decision guide is `iphone-utility-platform-guide.md` in this folder. Section 8 (Build Workflows) and Section 11 (Build Brief) apply to all work here.

## How work arrives

- Each tool lives in its own folder (`lowercase-with-hyphens/`) containing a `BRIEF.md` written during planning in chat.
- **Before building, read that tool's `BRIEF.md` and the relevant part of Section 8 of the guide.** The brief sets the scope. If something is ambiguous or can't be built as written, ask me rather than improvising.
- If I ask for a new tool that has no brief, don't start coding. Suggest planning it first in chat, or offer to draft a brief here using the guide's template and confirm it with me.
- When a build changes a decision in the brief, update the brief and add a line to its Change Log.

## Repo layout

```
my-iphone-tools/
├── CLAUDE.md
├── iphone-utility-platform-guide.md
├── index.html                 # simple list of links to the web-app tools
├── <web-app-tool>/            # served by GitHub Pages
│   ├── BRIEF.md
│   ├── index.html
│   ├── manifest.webmanifest
│   ├── sw.js
│   └── icons…
├── <scriptable-tool>/
│   ├── BRIEF.md
│   └── <Tool Name>.js         # source of truth; deployed by copying to iCloud
├── <shortcuts-tool>/
│   ├── BRIEF.md
│   └── <Tool Name>.shortcut   # signed output from Shortcuts Playground
└── data/                      # data-prep scripts and raw source files (not shipped)
```

**This repo is public** (GitHub Pages on a free account). Never commit personal data, API keys, passwords, or anything private. If a tool would need private data, stop and ask.

**Note:** API tokens go in Keychain and must be stored with -w "$VAR", not the hidden prompt. That way the 128-character snag never happens again.

## Platform conventions

### Home Screen web apps

- Each tool is a self-contained folder served from its own path (e.g. `https://<user>.github.io/my-iphone-tools/<tool>/`). Use relative paths everywhere, and set the manifest's `start_url` and `scope` to `./`. Register the service worker with the tool folder as its scope.
- All CSS and JS goes inline in `index.html`. No runtime CDNs or external requests unless the brief calls for live data. Fixed data is embedded as a JSON object.
- `sw.js` pre-caches every file, serves cache-first, and uses a versioned cache name. **Every change bumps the cache version** (e.g. `zip-county-v4`) **and the version shown in the footer.** Delete old caches on activate.
- All tools share one web origin, so prefix every `localStorage`/IndexedDB key with the tool name.
- Mobile-first design: large tap targets, the correct `inputmode`, safe-area padding (`viewport-fit=cover` + `env(safe-area-inset-*)`), light and dark mode, and a readable system font.
- If the tool stores user data, include **Export** (JSON download/share) and **Import**. Deleting the Home Screen icon deletes the data.
- Include `apple-touch-icon` and manifest icons (192 and 512 px). A simple generated icon is fine.
- Add the tool to the root `index.html` link list.

### Scriptable

- The source of truth is the `.js` file in the tool's folder. Deploy by copying it to Scriptable's iCloud folder:
  `~/Library/Mobile Documents/iCloud~dk~simonbs~Scriptable/Documents/`
  (If that path doesn't exist, find the Scriptable folder under `~/Library/Mobile Documents/` and tell me.)
- Check Scriptable APIs against the documentation (https://docs.scriptable.app) rather than relying on memory.
- Handle every launch mode: widget (`config.runsInWidget`), Shortcuts (`args.shortcutParameter`, return results with `Script.setShortcutOutput()`), and interactive. End with `Script.complete()`.
- Store user data with `FileManager.iCloud()` in a subfolder named for the tool.
- Put a comment header at the top of the file with the tool name, version, and date.

### Shortcuts

- Build with the Shortcuts Playground plugin (viticci/shortcuts-playground-plugin). The output is a signed `.shortcut` file saved in the tool's folder.
- Embed fixed data as a JSON Text action → "Get Dictionary from Input". Keep values as simple strings.
- Keep shortcuts short and linear. If the logic gets heavy, recommend moving it into a Scriptable script called from the shortcut.
- After building, tell me how to import it (open on the Mac so it syncs through iCloud, or AirDrop it to the phone) and where to delete the old version.

### Hybrids

Follow each part's conventions above. The brief says which platform owns which piece.

## Data preparation

- Put download and processing scripts in `data/` (Python is fine), with a comment naming the source URL and date. The processed output gets embedded in the tool.
- Record the data's source and "as of" date in the tool's footer or About screen.

## Testing and shipping

- **Web apps:** Test locally first (`python3 -m http.server` from the repo root), including a mobile-width view and an offline check using the service worker. Then show me what changed.
- **Before pushing to GitHub, ask me.** Pushing publishes the change. Use clear commit messages (`zip-county: v4 – show multi-county splits`).
- After deploying, list the brief's "Done when" checks for me to run on the iPhone, including the steps to see the update (e.g. "open the app online once, then relaunch").
- When I report problems from testing on my phone, fix them in this session. Update the brief if the scope changes.

## Working style

- I'm technically comfortable but not a professional developer. Explain what you did and anything I need to do on the phone in plain language. Keep it brief.
- Keep dependencies and build steps to a minimum: no bundlers or frameworks unless the brief truly needs them.
- Pythonista holds my older tools. Don't create new tools there.
