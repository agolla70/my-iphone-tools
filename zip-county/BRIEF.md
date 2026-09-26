# Build Brief: Oregon ZIP → County

**Folder:** `zip-county`
**Platform:** Home Screen web app
**Date:** 2026-09-25

## 1. What it does
I type a 5-digit Oregon ZIP and instantly see which county it's in. If the ZIP spans more than one county, it lists every county with its share of residential addresses and flags that I need to check the specific address another way.

## 2. Why this platform
- The interface matters: instant results as I type and a clear multi-county display.
- Works fully offline with a small fixed data set (~400–480 ZIPs), and also works on the Mac at the same URL.
- No widgets, triggers, Siri, or iOS data needed, so nothing favors Scriptable or Shortcuts.
- Runner-up: Shortcuts, which lost because I don't need voice and split ZIPs display poorly there.

## 3. How it's launched and used
- Launch: Home Screen icon.
- Main flow: open app → numeric keypad is already up → type 5 digits → the result appears automatically (no button) → tap the field again to clear it and enter a new ZIP.

## 4. Features
**Must have (v1):**
- Lookup of Oregon ZIPs from embedded HUD-USPS data.
- Single-county result shown large, e.g., "Multnomah County".
- Multi-county result: a clear banner such as "Spans 2 counties — check the address", then every county with its share, largest first (e.g., "Clackamas 92% · Multnomah 8%").
- Out-of-state counties labeled with their state (e.g., "Modoc Co., CA").
- A refresh script in the repo that pulls fresh data from the HUD API (see Section 5).
- Footer showing the app version and data vintage (e.g., "v1.0 · Data: HUD-USPS 2026 Q2").
- Works offline after the first load.

**Nice to have (later):**
- City or place name for each ZIP.
- A link to an address-level county lookup for split ZIPs.
- An Action button launcher (a one-action shortcut that opens the URL).

**Deliberately out of scope:**
- PO-Box-only ZIPs: not in HUD's data, and not needed for residential addresses.
- Non-Oregon ZIPs that reach into Oregon (e.g., 99362 Walla Walla, WA): I only look up Oregon ZIPs.
- Automatic or scheduled refresh: a manual refresh about once a year is enough.

## 5. Data
- **Fixed/embedded data:**
  - Source: HUD-USPS ZIP Code Crosswalk, ZIP-COUNTY, via the HUD User API: `https://www.huduser.gov/hudapi/public/usps?type=2&query=OR`, sending the header `Authorization: Bearer <token>`.
  - Share measure: `res_ratio` (residential addresses). If a ZIP's `res_ratio` values total 0, use `tot_ratio` and mark the result "(by all addresses)".
  - Processing:
    - Keep ZIPs whose preferred state is OR.
    - Keep every county row for those ZIPs, including counties in other states.
    - Drop county rows under 1%, but always keep the largest county.
    - Rescale the remaining shares to total 100% and round to whole percents.
  - County names: HUD returns FIPS codes only. Map them to names using a lookup table in the script (Oregon's 36 counties plus any neighboring-state counties that appear).
  - Output: a compact JSON object embedded in `index.html` between marker comments. It is stamped with the source and quarter (e.g., "HUD-USPS ZIP-COUNTY 2026Q2").
- **Token handling:**
  - The HUD API token is stored in macOS Keychain; the script reads it with `security find-generic-password`.
  - Keychain service name: hud-api-token
  - Don't use security add-generic-password ... -w with a hidden prompt to store it; that prompt stops at 128 characters and the token is longer.
  - The token must never be written to any file in the repo, since the repo is public.
  - Add `.gitignore` entries for any temporary or raw download files.
- **Refresh process (on request, about yearly):**
  1. I tell Claude Code to refresh the ZIP data.
  2. It runs `zip-county/tools/refresh` (Python or Node, whichever the repo's `CLAUDE.md` prefers).
  3. The script fetches the data, processes it, and prints a diff against the current embedded data: ZIPs added or removed, county lists that changed, and shares that moved more than 5 points.
  4. After I approve, it rewrites the embedded JSON, bumps the footer version and the service worker cache version, commits, and pushes.
  5. If the API fails, it stops and reports the error. As a fallback, I can download `ZIP_COUNTY_<MMYYYY>.xlsx` from HUD User after logging in, and the script accepts that file as input.
- **User data it stores:** None.
- **Live/online data:** None at runtime. The app is fully offline; the API is used only by the refresh script on the Mac.

## 6. Interface notes
- Single screen with one large input: `inputmode="numeric"`, `maxlength=5`, focused on launch.
- The result area sits under the input and uses big, readable type.
- Split ZIPs get a distinct color and the banner "Spans N counties — check the address", followed by the county/share list.
- Mobile-first layout with safe-area padding and dark-mode support.
- About line: "This product uses the HUD User Data API but is not endorsed or certified by HUD User." (required by HUD's API terms)

## 7. Edge cases
- Fewer than 5 digits: show nothing yet (no error).
- A 5-digit ZIP outside 970xx–979xx: "Not an Oregon ZIP."
- An Oregon-range ZIP not in the data: "Not found — may be a PO Box-only or new ZIP."
- A ZIP with only a tiny out-of-state sliver: after filtering it shows as a single county. (As of 2026 Q2, HUD's data has no out-of-state county rows for any Oregon ZIP; 97913 Nyssa is Malheur only.)
- Business-only ZIP (residential share is 0): use the total-address share and label it.
- No network on first launch: the app can't cache yet. Only a one-time online visit is required.

## 8. iOS features and permissions
None.

## 9. Done when (acceptance tests on the iPhone)
- [ ] Works in airplane mode after the first online launch.
- [ ] "97220" returns "Multnomah County" as a single county.
- [ ] Known split ZIP 97321 (Albany) shows the banner and "Linn 69% · Benton 31%".
- [ ] "98101" shows "Not an Oregon ZIP."
- [ ] The footer shows the version and the data quarter.
- [ ] Running the refresh script prints a diff, and after approval an update shows the new version after one online launch.
- [ ] The HUD token does not appear anywhere in the repo or its git history.

## 10. Open questions
- ~~Confirm at build time that `query=OR` returns all Oregon ZIPs.~~ Resolved 2026-09-25: `query=OR` matched `query=All` filtered to preferred state OR exactly (474 ZIPs, 568 rows, identical shares, 2026 Q2). No Oregon ZIP has an out-of-state county row.

## Change Log
- 2026-09-25: Brief created.
- 2026-09-25: v1.0 built. Refresh script is Python at `zip-county/tools/refresh` (dry run by default; `--apply` writes data and bumps the version; `--xlsx FILE` fallback, untested against a real HUD file). Resolved the `query=OR` question (see Section 10). Split test ZIP set to 97321. Data: 2026 Q2, 474 ZIPs, 60 multi-county, 19 business-only. Commit and push stay manual steps done by Claude after approval, not by the script.