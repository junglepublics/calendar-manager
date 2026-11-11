# Calendar Manager

A Google Apps Script web app that looks after Prateek's work calendar. It runs in the Dark Matter Labs Google account, not on GitHub.

What it does:

- **Busy sync**: copies accepted weekday events from the personal calendar to the work calendar as "Busy" blocks.
- **Colour coding**: colours work events by editable rules (keywords, hashtags, who is invited). `#exception` in a description skips the rules. A colour you set by hand sticks until you press "Hand back".
- **Lunch**: places a one-hour Lunch block each weekday around meetings.
- **Slack status**: sets your Slack status from the calendar, and never overwrites a status you set yourself.

## Files

| File | What it is |
|---|---|
| `CalendarManager-Code.gs` | The server code (becomes `Code.js` in the Apps Script project) |
| `calendar-manager-index.html` | The web UI (becomes `index.html`) |
| `index.html` | A redirect to the web app, served by GitHub Pages as the short link |
| `sync.ps1`, `sync.sh` | Copy the two files into the clasp folder and run `clasp push` |

## Deploying (PowerShell, one line at a time)

```
cd $HOME\Documents\vibe
git checkout main
git pull
.\sync.ps1 ..\calendar-manager-clasp
cd $HOME\Documents\calendar-manager-clasp
clasp deploy --deploymentId <web app deployment ID> --description "<what changed>"
```

`main` is always the live version. Secrets (the Slack token) live only in Apps Script user properties, never in this repo.
