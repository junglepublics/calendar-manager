# Handover: junglepublics repos and the Calendar Manager project

Written 2026-11-11 by Claude (Claude Code cloud session), at Prateek Shankar's request, so a personal wiki LLM has full context. Everything below was checked against the repos, the live Apps Script project (read through the Google Drive connector) or the live calendar (read through the Google Calendar connector) unless it is marked **unverified**. No secrets are included.

---

## 1. People, accounts and machines

| Thing | Value |
|---|---|
| Owner | Prateek Shankar (Dark Matter Labs, "DML") |
| Work account (hosts everything below) | `prateek@darkmatterlabs.org` (Google Workspace). The Apps Script project, the web app and the work calendar all live here. |
| Personal calendar ("Play") | `prateek.shankar@gmail.com`. It is the *source* for Busy sync and must be shared with the work account. |
| GitHub | user/org `junglepublics`, six public repos (below) |
| Local machine | Windows, **PowerShell 5.1** (no `&&`; run commands one per line), Node v24, Git 2.53, `clasp` installed and logged in as the DML account |
| Local folders | `C:\Users\prate.MOTHERSHIP\Documents\vibe` (git clone of the repo), `C:\Users\prate.MOTHERSHIP\Documents\calendar-manager-clasp` (clasp project folder, created with `clasp clone`) |

Gotcha that cost a lot of time: early in the work Prateek said the script lived in the personal Google account, then the DML one. **It is the DML account.** The web app link has `/a/macros/darkmatterlabs.org/` in it.

---

## 2. The six repos under github.com/junglepublics

All six are public, default branch `main`. Only `vibe` was worked on in this session. The other five were cloned read-only today and skimmed (titles, README, file list), so those descriptions are shallow.

| Repo | Last push | What it is |
|---|---|---|
| **vibe** | 2026-11-10 | **Source for Calendar Manager** (Google Apps Script + web UI). Active work; see sections 3 to 6. Branches: `main`, `calendar-manager` (old), `ccr-95e9c42c-5gf8ti` (this session's branch). |
| **calendar-manager** | 2026-10-22 | One file, `index.html`: a redirect page that sends to the Calendar Manager web app (meta refresh + `location.replace`). A short, memorable link. The code is *not* here; it is in `vibe`. `vibe/index.html` on `main` is the same redirect. The Pages URL is probably `junglepublics.github.io/calendar-manager` (**unverified**). |
| **prtk.xyz** | 2026-10-15 | Prateek's portfolio ("prateek shankar: portfolio of work"). A static copy of the Cargo site at prtk.xyz, taken 2026-10-15, served by GitHub Pages. ~107 files: project folders (the-iworker-project, polycene-design-manual, worlding-englishes, ganga-jamuna-market, etc.), `bio`, `media`, `fonts`, `css`. **Password-locked**: each page is encrypted with `tools/lock.mjs` and decrypted in the browser. Each page has `index.html` (wide) and `m.html` (768px and under). To serve at prtk.xyz you add a `CNAME` file and point DNS at GitHub Pages. The last commit mentions "the pixel dinosaur, as on the wiki lens" (a favicon), so there is already a wiki connection. |
| **dml** | 2026-10-22/23 | Four standalone HTML pages for Dark Matter Labs work: `learning-system-v3.html` ("QNO Knowledge Engine", 84 KB), `m2m-learning-agenda.html` ("M2M Learning Pop-Up"), `risksense-revised-operating-model.html` ("RiskSense: Revised Operating Model | 8 July 2026"), `timeline.html` ("Timeline Live Embed"). Last commit: "Enhance timeline layout and SVG graphics". Contents not reviewed beyond titles. |
| **india-advocacy** | 2026-10-14 | "Berlin Action Guide: India": an interactive public page (`index.html`, 56 KB) about advocacy from Berlin on India's voter roll crisis. README says it is live at `https://junglepublics.github.io/india-advocacy/`, served by GitHub Pages **from a `gh-pages` branch that must be kept in sync with `main`** (only `main` was visible in the shallow clone, so **unverified**). |
| **neukin-film-club** | 2026-08-14 | "NeuKin Film Club": a single-page voting app (`index.html`, 55 KB) with a Google Apps Script backend (`neukin-apps-script.gs`) that stores data in a Google Sheet ("data" tab). Deployed as a web app, executing as the owner, access "Anyone" (so voters need no sign-in). The setup steps are in the `.gs` header comment. Editing the script does not update the live URL; you must make a new deployment version. |

---

## 3. Calendar Manager: what it is

A personal automation for the work calendar. A Google Apps Script web app with four automations:

1. **Busy sync**: copies accepted, weekday, timed events from the personal (Play) calendar to the work calendar as blocks titled **"Busy"** (description `synced: <calendar>:<eventId>`), coloured with the Hold colour. Each occurrence of a recurring event syncs separately. Blocks are removed when the source event is deleted, declined or moved.
2. **Colour coding**: classifies work-calendar events and sets their Google colour. See section 5.
3. **Lunch scheduler**: places a 1-hour **"Lunch"** block (description `lunch-block: managed by Calendar Manager`) each weekday in a window (default 12:30–14:30), around accepted meetings. Any lunch event you have not declined counts as covering the day. A lunch already under way is never moved. Prateek currently has the Lunch colour set to **grey on purpose**.
4. **Slack status**: sets Slack status from the calendar ("In a meeting: …", "In focus time", "At lunch", moon emoji "After hours" outside 09:00–18:30). **A status Prateek sets manually in Slack is never overwritten** (the script remembers the text it last set and compares). Needs the Slack **user token** (`xoxp-…`) with scopes `users.profile:read` and `users.profile:write`; both have been added. The token is stored in Apps Script user properties (never in the repo).

### Where it lives (IDs)

| Thing | Value |
|---|---|
| Apps Script project | "Calendar Manager", script ID `1VI8d9K16gKlycBMAF7lgPVhx6XY-T0G9J4z4QzlO1DxOfL95PFhCyNGC`, owner `prateek@darkmatterlabs.org` |
| Backup copy (made 2026-10-22 before changes) | "Copy of Calendar Manager", `1VGd0sJGhU7ovMTVT2IIK8QdmP9UC9x30olZeUHSbNA8_rTXY72rxfSZM` |
| Web app deployment ID | `AKfycbzFgHGLoSGmKO3W453dRrmUtuMh67ju3oP9asUJ2PU0UmGiZAzw_6HCSdHgDGLcnqhD` |
| Web app URL | `https://script.google.com/a/macros/darkmatterlabs.org/s/<deployment ID>/exec` (executes as owner, only the owner can open it) |
| Files in the project | `Code.gs` (server), `index.html` (UI), `appsscript.json` (manifest: timezone Europe/Berlin; the Advanced Calendar service v3 is enabled) |

---

## 4. How code gets from the repo to the live script

Claude cannot reach Prateek's computer or sign in to Google, so Prateek runs the commands. (The only way to give Claude that reach is the Claude desktop app's Remote Control folders; none were connected.)

**Source of truth is the git repo, deployed with clasp.** From PowerShell, one command per line:

```
cd $HOME\Documents\vibe
git fetch
git checkout ccr-95e9c42c-5gf8ti
git pull
.\sync.ps1 ..\calendar-manager-clasp
cd $HOME\Documents\calendar-manager-clasp
clasp deploy --deploymentId AKfycbzFgHGLoSGmKO3W453dRrmUtuMh67ju3oP9asUJ2PU0UmGiZAzw_6HCSdHgDGLcnqhD --description "<what changed>"
```

- `sync.ps1` (also `sync.sh`) copies `CalendarManager-Code.gs` to `Code.js` and `calendar-manager-index.html` to `index.html` inside the clasp folder, then runs `clasp push -f`.
- "**Script is already up to date**" means the files being pushed are identical to what is already in the project. It does **not** mean "my latest changes are there". It once meant the local clone was on `main` (older code), so nothing differed.
- `clasp deploy --deploymentId …` makes a new version and points the existing web-app URL at it. Editing code alone does not change what the web app serves.
- **Triggers** (`syncCalendars` hourly; `onCalendarChange` on both calendars; `updateSlackStatus` every 5 minutes) are created by running `installTriggers()` once in the Apps Script editor. Whether Prateek ever ran it on the current version is **unverified**; the Triggers sidebar should show 4.
- **How Claude verifies the live script:** the Drive connector can export the project (`application/vnd.google-apps.script+json`, base64 JSON of the three files). Claude compares it to the repo files (ignoring Windows line endings). This was done on 2026-11-10 and the live code was identical to the repo at `19282da`.
- The Calendar connector can also read events including `colorId`, which is how the colour behaviour was checked.

---

## 5. Architecture of the live code (`CalendarManager-Code.gs`, ~46 KB)

### Colour classification (`classifyEvent_` → `classify_`)
Order of decision for each timed work-calendar event:
1. Our own **Busy** block (title `Busy`, description starts `synced:`) → Hold colour. Our own **Lunch** block → Lunch colour. These are always enforced.
2. A **hashtag** that belongs to a rule (e.g. `#recreation`, `#lunch`) in the title or description → that rule's colour.
3. **Rules** (editable in the UI under "Colour rules"), checked top to bottom, first keyword match on the title wins. Each rule has `keywords`, `color`, `guests` and a `tag`. `guests:true` means it also applies when other people are invited; `guests:false` means solo events only. Defaults: Lunch (solo only), Focus time, Hold/Busy (solo only), Reminder, Priority (solo only), Events & talks (blue), Recreation (yellow).
4. Otherwise **who is invited** decides: a Teams/Zoom link or any outside guest → External; only `@darkmatterlabs.org` guests → DML team; solo with no rule → Default (native colour).
- **`#exception`** in an event's description skips all keyword rules (Lunch included, fixed 2026-11-10); the event is then coloured by attendees. It is listed in the **Exceptions** tab.
- Attendees come from the Calendar API with the `self` flag (not by comparing emails) and exclude room/resource calendars.
- Colours are Google "named" colours mapped to event colour IDs 1–11 (PALE_BLUE=1 Lavender, PALE_GREEN=2 Sage, MAUVE=3 Grape, PALE_RED=4 Flamingo, YELLOW=5 Banana, ORANGE=6 Tangerine, CYAN=7 Peacock, GRAY=8 Graphite, BLUE=9 Blueberry, GREEN=10 Basil, RED=11 Tomato). `NONE` clears the colour so the calendar's own colour shows.

### Manual colours ("recolour it yourself and it sticks"), added 2026-11-10
The script stores `eventId → colorId` it last applied (user property `calendarManagerColorMap`; `…ColorMapInit` marks the first run). If an event's colour no longer matches, Prateek changed it by hand: it is marked `'M'` and left alone until **"Hand back"** in the Schedule tab (`releaseManualColor`). The first run after deploy "takes over" every colour once. A new event that already has a colour is treated as Prateek's. Failed writes stay retryable. Works for events organised by others, where descriptions cannot be edited (the reason `#exception` is awkward).

### Efficiency decisions (done 2026-10-22)
- One `Calendar.Events.list` per calendar per job (with a fields mask) instead of per-event reads; writes only when something differs (`Calendar.Events.insert/patch/remove` with `sendUpdates:'none'`).
- Triggers are event-driven: `onCalendarChange` (debounced, with `LockService` in `syncCalendars`) plus an hourly safety net.
- `testRun()` runs the three calendar jobs from the editor and logs results.

### Config and state (Apps Script *user properties*, not in the repo)
`calendarManagerConfig` (calendar IDs, work domain, days ahead, keywords, colours, rules, lunch, work hours, Slack token), `…LastRun`, `…LastMsg`, `…ColorMap`, `…ColorMapInit`, `…LastSync`, `…SlackSet`. Rules are versioned (`rulesVersion`, currently 4) with an `upgradeRules_` migration. A user-property value is capped at **9 KB**; the colour map is ~3 KB for ~44 events in 2 weeks, so it could hit the cap around 150 events (**risk if the sync window is widened**).

### The web UI (`calendar-manager-index.html`, ~55 KB)
Tabs: **Schedule** (banner "N events to recolour", Apply colours, 7/14-day preview with reason per event and "Your colour / Hand back"), **Colour rules**, **Exceptions**, **Lunch**, **Accounts** (calendar IDs, Slack token), **Activity** (log). Per-automation "Run now" cards plus "Run everything". All user-supplied text is inserted with `textContent` (no HTML injection).

---

## 6. History of this work (what was done, in order)

- **2026-06-25 / 08-03 / 08-04**: initial upload of `Code.gs` and `index.html` to `vibe` by Prateek (also the old `calendar-manager` branch). Original version used `CalendarApp`, per-event API calls, a six-panel UI.
- **2026-10-22 (this session, first part)**: read and analysed the code; listed bugs. Fixed: lunch not grey, `#exception`, Slack manual status preserved, "Clear status" actually clears. Then the **efficiency rewrite** (above), `testRun()`, Prateek tested it (Busy 0 created/8 skipped, Colour 0 updated/84 unchanged, Lunch 0/0/3) with no errors. Built a first redesigned UI with a dry-run preview. Set up **clasp** with Prateek step by step (Node REPL mistake, PowerShell `&&` not supported, `clasp login` as DML, `clasp clone`, `sync.ps1`).
- **2026-10-22 (a separate, parallel session)**: built a *different, newer* version straight onto `main` (it merged this session's branch first): redesigned UI with **editable colour rules**, lunch always yellow, Recreation and event words, hashtag tags (`#recreation`), an **Exceptions tab**, and a redirect page. That is the version Prateek actually deployed and was running.
- **2026-10-24**: this session added "faster load" (embedded settings and a cached preview in `doGet`). **It never reached the live script** because the live script was the `main` version.
- **2026-11-10**: Prateek asked why `#exception` did not work. Claude's first attempt (a `#colour:` / `#skip` tag system) was written against the old code and is **not** in the live script. Claude then discovered that the live script (read from Drive) was the `main` version, not its own. Root cause of `#exception` failing in the live code: the Lunch rule applied even with guests and was explicitly exempt from `#exception`. Fixed on the `ccr-95e9c42c-5gf8ti` branch (made it `main` + fix, via a merge that keeps history), then added **manual colours**. Prateek deployed; Claude confirmed via Drive export that the live files equal repo commit `19282da`.

### Branch situation (important)
- `origin/main` = `60a51f2` (the rules-based version, **without** the `#exception`/Lunch fix and **without** manual colours).
- `origin/ccr-95e9c42c-5gf8ti` = `19282da` = `main` + both changes. **This is what is live.** Its history contains an earlier abandoned stretch (tags system, snapshot commit) before the merge with `main`.
- **No pull request has been opened.** The clean way to finish is a PR from `ccr-95e9c42c-5gf8ti` into `main` (the diff against `main` is only `CalendarManager-Code.gs` and `calendar-manager-index.html`). Until then, a local clone on `main` and `sync.ps1` will push the *older* code over the live script.

---

## 7. Open items and unverified things

1. **Manual colour test (open):** Prateek reported an event recoloured yellow "did not stick". The event is "Forro Dance Warming afterwards Dance Class" (organiser is Prateek, but it has outside guests, so the rules say Sage). As of 2026-11-10 22:03 UTC it was still **yellow (colorId 5)**, even after its guest responses changed at 21:16 UTC; nothing seen reverted it. Whether the script ran since is unknown. A live test was requested (click "Run now" on Colour coding, then check the Schedule tab shows "Your colour") but not completed. Suspects if it does revert: the first-run takeover overwrote a colour set *before* the first run; or an old deployment serving older code.
2. `installTriggers()` on the current version: unconfirmed.
3. Clearing a colour back to native (`colorId: null` via `Calendar.Events.patch`): the calendar contained native-coloured events, but a clear-by-script was never confirmed.
4. The Slack manual-status protection was only exercised once ("No change (no active meeting)").
5. A private interactive artifact of this session's *old* UI exists at `https://claude.ai/artifact/SJvh2chMuHWGZveZFSpza9`. It does **not** match the live UI.
6. `preview.html` on `main` was made by the parallel session and may not match the live UI.
7. Other repos (`dml`, `prtk.xyz`, `india-advocacy`, `neukin-film-club`) were not worked on and only skimmed.
8. The "personal" colour category exists in config but is not used by any rule (Busy blocks use the Hold colour, as asked).

---

## 8. Working-with-Prateek notes (for the wiki LLM)

*(Inferred from this one long conversation; treat as a starting point, not settled preferences.)*

- Prateek wants **plain, copy-paste instructions**, one command per line, and gets frustrated by long or hedged explanations. Say exactly which folder a command runs in and what output to expect.
- Verify things yourself when you can (Drive export, Calendar connector) instead of asking Prateek to check, and say plainly what you cannot do (no access to their computer, Google login, or Apps Script editor).
- Do not assume Prateek skipped a step; check the evidence first. Several confusions came from the live script differing from the repo.
- Seems to like: explicit rules over hidden magic, a preview before anything changes, a dark/light-mode-aware clean UI, colours sticking when set by hand, lunch handled sensibly.
- Never commit tokens. The Slack token lives only in Apps Script user properties.
