# Project Tracker

A dashboard for tracking projects, their tasks, who owns what, and how far along everything is. The page lives on GitHub Pages. The data lives in a Google Sheet. Apps Script joins them and sends the weekly summary email.

## How the pieces fit

| Piece | Where it lives | What it does |
|---|---|---|
| `index.html` | GitHub Pages | The dashboard everyone opens |
| `Code.gs` | Apps Script, attached to the Sheet | Reads and writes the Sheet, sends email |
| Google Sheet | Your Drive | The actual data, editable by hand |

Nothing else to install. No build step, no dependencies, no npm.

## Try it before you wire anything up

Open `index.html` in a browser. With `CONFIG.API_URL` left empty it runs in demo mode and saves to that browser only. Add projects, add tasks, move sliders, look at the weekly summary. Nothing you do here touches the Sheet.

When you're happy with it, connect the Sheet.

## Connecting the Sheet

**1. Make the Sheet and add the script**

New Google Sheet → Extensions → Apps Script. Delete the starter code, paste in all of `Code.gs`, save.

**2. Build the tabs**

In the Apps Script editor, pick `setUp` from the function dropdown and run it. Approve the permissions prompt. It creates three tabs (`Projects`, `Subprojects`, `Meta`) and generates your edit key.

Open View → Logs to see the key. Write it down. It also lives in the `Meta` tab if you lose it.

**3. Deploy it**

Deploy → New deployment → type: Web app.

- Execute as: **Me**
- Who has access: **Anyone**

Copy the URL it gives you. It ends in `/exec`.

**4. Point the dashboard at it**

Open `index.html`, find the `CONFIG` block near the bottom, paste the URL into `API_URL`:

```js
API_URL: "https://script.google.com/macros/s/AKfy.../exec",
```

**5. Turn on the weekly email**

Back in Apps Script, run `installWeeklyTrigger`. It's set to Monday at 7am. To move it, change `EMAIL_DAY` and `EMAIL_HOUR` at the top of `Code.gs` and run it again.

Set who gets it in the dashboard's Settings, or directly in the `Meta` tab's `recipients` row. Start with just your own address.

**6. Publish to GitHub Pages**

Push `index.html` to a repo. Settings → Pages → deploy from your main branch, root folder. It'll be live in a minute or two.

## Who can change things

Everyone who opens the page gets view mode. Clicking **Unlock editing** and entering the edit key switches that browser into edit mode, and it stays unlocked until they lock it again.

Be clear-eyed about what this is. The edit key stops accidents and casual meddling. It is not real security. Anyone determined enough can read the key out of network traffic, and the deployment setting means anyone with the URL can read the data. Don't put anything confidential in here.

To rotate the key, run `resetEditKey` in Apps Script. Everyone will need the new one.

## How the percentage works

A project with no tasks uses whatever percentage you set on it directly.

A project with tasks ignores that and calculates from the tasks instead, as a weighted average. Weight is how much of the project each task represents. Leave every weight at 1 and it's a plain average. Set one task to 3 and it counts triple.

A task marked Complete counts as 100% no matter where its slider sits.

## Changing the look

The colours are six CSS variables at the top of `index.html`:

```css
--paper:  #FFF4EC;   /* background */
--card:   #FFFFFF;   /* cards */
--ink:    #2E1F3D;   /* text */
--ink-2:  #6E5C7C;   /* secondary text */
--flame:  #FF6B35;   /* primary accent */
--grape:  #7B3FA0;   /* progress bars */
```

Change those six and the whole thing reskins. Status colours (`--mint`, `--amber`, `--rose`) sit just below.

Fonts are Bricolage Grotesque for headings and numbers, Figtree for everything else, loaded from Google Fonts in the `<head>`.

## Changing the email's look

The email has its own copy of the palette, in the `EC` block near the top of `Code.gs`. It has to be separate because email clients can't read a stylesheet reliably, so every colour is written inline as the HTML is built. If you reskin the dashboard, change `EC` to match or the two will drift apart.

The email is built from nested tables rather than modern CSS. That's deliberate. Outlook renders HTML through Microsoft Word, which has no flexbox, no grid, and no web fonts, so tables with background colours are the only layout approach that behaves the same everywhere. The progress bars are two table cells with percentage widths.

To see your changes without emailing anyone, run `previewSummaryHtml` in Apps Script, open View → Logs, paste the output into a `.html` file and open it in a browser.

`email-preview.html` in this repo is a sample of the output with fake data, so you can see the design before setting anything up.

## Two people editing at once

The Sheet tracks a revision number. If someone saves while you have the page open, your next save is rejected and the dashboard reloads their version rather than overwriting it. You'll see a message when that happens.

The page also re-checks the Sheet every two minutes so you're rarely working from a stale copy. Change `REFRESH_SECONDS` in `CONFIG` to adjust, or set it to `0` to turn it off.

## Backups

Settings → Download a backup writes a JSON file of everything. Restore from backup replaces the current contents with that file. The Sheet itself also keeps full version history through File → Version history, which is usually the faster way to undo something.

## If something breaks

**"Can't reach the Sheet"** — the API URL is wrong, or the deployment isn't set to Anyone. Redeploy and copy the URL fresh. Note that editing the script requires a *new deployment version* before changes go live.

**"That key didn't work"** — check the `editKey` row in the `Meta` tab.

**Dates showing oddly** — the `due` columns are meant to be plain text in `YYYY-MM-DD` format. If someone typed a date and Sheets reformatted it, set that column's format back to Plain text.

**The email didn't arrive** — run `sendWeeklySummary` manually in Apps Script and read the log. Usually it's an empty `recipients` row. Gmail also caps daily sends, though you're nowhere near it with a weekly summary.
