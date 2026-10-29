### QueueSenseITOps

A Brave/Chromium (Manifest V3) extension for support agents who follow up on ServiceNow incidents. It
reads an incident from your own already-logged-in ServiceNow session, works out the reporting user's
location and whether it's currently business hours there, and helps you follow up — a Teams message
during their business hours, a ticket update (comment/work note) otherwise — from a dashboard that
tracks every ticket you've looked up.

It makes no network requests of its own and has no server: no analytics, no telemetry. It opens your
ServiceNow pages in **background tabs** of your own signed-in session and reads them, including
automatically when the browser starts (turn that off with the **Refresh on browser start** switch at the
top of the dashboard). Those pages load data from your instance as usual: the form looks up the caller's
record, and the extension clicks the activity stream's "Show more" to read every note. Posting a note
submits the ServiceNow form, only when you click Post. Teams and `mailto:` links open only when you click
them; a Teams link carries the contact's e-mail address and your message to Microsoft Teams. All data is
stored locally (`chrome.storage.local`) on your machine. See [Privacy](#privacy) and `PRIVACY.md`.

## What it does

- **Look up an incident** by number. The extension opens the classic incident form (`incident.do`) in a
  background tab and reads it via your existing session — it never asks for or stores a password. It
  does not read Workspace pages or forms inside frames.
- **Work out the user's location and timezone** from their phone number's country code, refined by the
  ServiceNow **Location** field when the phone number alone can't narrow a multi-timezone country down
  to a city (e.g. a US number narrowed to `Austin, TX` by the Location field). Falls back to the Location
  field alone when there's no usable phone number.
- **Classify business hours** at that location right now, and at the time the ticket was last updated by
  the assigned agent, with weekends adjusted per-country (e.g. Friday–Saturday across most of the Middle
  East) when you turn that on in Settings.
- **Resolve the preferred contact method** from the *label* "Preferred Contact Method(Impacted User)" (a
  custom field some instances have), not a hardcoded technical field name — so it still works if your
  instance names the underlying field differently.
- **Recommend and help you send a follow-up**: a Teams deep link during the user's business hours, or a
  ticket comment/work note update otherwise. A guided, branching questionnaire can compose one of six
  built-in, generic template messages (user education, alternate ticket created, requested a call, needs more
  info, etc.) instead of writing free text. The messages are signed with the service desk name you set in
  **Settings → Service desk name** (default "Service Desk"); the wording itself lives in
  `lib/messageTemplates.js`.
- **Track every ticket you've looked up** in a dashboard (its own page, opened from the toolbar icon —
  there's no popup): search, a live table for today's tickets with Status, Assignment group, location/
  timezone, a red/green last-contact indicator, a VIP star, and a priority highlight for users whose
  business hours end within 2 hours.
- **Bulk fetch** up to 50 incident numbers at once (comma/newline/file-separated). Up to **5 tickets are
  fetched in parallel** through a rolling pool (the next one starts as soon as any finishes). While it runs,
  the input collapses to a one-line progress summary; **Edit list** opens it again.
- **User engagement** per ticket, from the activity stream: how many times the caller updated the ticket,
  shown as **Frustrated** (4+ updates, or 3+ within 24 hours), **Sporadic response** (1–3 updates) or
  **No response** (none).
- **Negative sentiment**: a red ☹ icon next to the ticket number when anyone *other than the assigned
  agent* (the caller, other agents, watchers) used wording like "misrouted", "incorrect", "disappointed",
  "still not working", "third time", "no response", "closed without my confirmation", "escalate" …
  Hover the icon to see each phrase, who wrote it, and the sentence around it. The phrase list is in
  `lib/engagement.js` (`NEGATIVE_PATTERNS`).
- **Remove** a single ticket from your searched list (row → *Remove*, click twice), or a single entry
  from the History view. Removing a ticket keeps its follow-up history.
- **Export the visible table to Excel** (`.xlsx`, generated locally — no server, no library dependency),
  from the **Export to Excel** button in the Tickets toolbar (also on the Data tab). Includes User Updates,
  User Response, Negative Sentiment and Sentiment Keywords columns.
- **Refresh automatically** when the browser restarts (on by default; turn it off with the **Refresh on
  browser start** switch at the top of the dashboard), if you're still logged in — only for instances you
  already allowed, checked per ServiceNow instance, in background tabs, with per-ticket error isolation and
  throttling so it doesn't hammer your instance.

## What's new in 0.5.1

Chrome Web Store readiness release; no change to how tickets are read or analysed.

- **New name and look: QueueSenseITOps.** New toolbar and store icon, the QueueSenseITOps logo in the
  dashboard header, and a short splash with the full artwork while the dashboard loads (skipped when the
  system asks for reduced motion). Export files are now named `queuesenseitops-YYYYMMDD-HHMM.json` /
  `queuesenseitops-tickets-YYYYMMDD-HHMM.xlsx`; exports from earlier versions still import. The source
  artwork and the Web Store promo tiles are in `dev/brand/`.
- **Generic wording only.** The six built-in follow-up templates are neutral service-desk English, and
  **Settings → Service desk name** sets the name used in greetings and signatures (default "Service Desk").
- **Privacy.** A privacy policy ([PRIVACY.md](PRIVACY.md)), a **Settings → Privacy** summary, and clearer
  wording about background tabs, browser-start refresh and Teams links.
- **Less data read.** The incident form's description, impact, urgency, category, subcategory and active
  fields are no longer read by default (they were never used).
- **Delete is more thorough.** Deleting or removing a ticket also removes the stored copy of the
  last-searched ticket when it is that ticket; the confirmation says what the queue history keeps.
- **Hardening.** The incident list is accepted only from a frame on your instance's own origin; an unused
  injectable script was removed; bulk-fetch files over 1 MB are refused.
- **Packaging.** `npm run pack` builds the Web Store zip from the shipped files only.

## What's new in 0.5.0

- **Auto refresh** (top of the Overview tab). Turn it on and choose an interval from 15 minutes to 2 hours, in steps of 15 minutes. Each refresh does the same as **Load all open tickets**. **Refresh now** starts one straight away.
  - Auto refresh runs only while a dashboard tab is open, in any window. The next run is due one interval after the previous run ended, whichever tab ran it, and only one tab loads at a time.
  - An automatic run never asks for a permission, never moves the keyboard focus, never switches the tab you are on and never changes your Tickets filters. It shows the usual progress panel, marked "Auto refresh:", and you can Cancel it there.
  - The status line explains a pause, for example a missing ServiceNow address or queue, an instance you have not yet allowed with Load all open tickets, or a ServiceNow login page. After a login page it tries again at the next interval.
- **New to the queue and moved out.** Every complete refresh is compared with the ones before:
  - **Baseline.** The first complete refresh of a queue only records which tickets are there and when they were first seen. None of them is "new", because the extension cannot know when they arrived. A queue added to Settings later gets its own baseline.
  - **New to queue.** A ticket that appears after the baseline is tagged **New to queue** for 24 hours. The time it entered is shown in the ticket's details and in the Excel export.
  - **Moved out.** A tracked ticket that a later complete refresh no longer finds is tagged **Moved out**, with the reason from the re-check: resolved (with its state), moved to another group, or no longer listed (for example, a state you do not load). If it comes back, it counts as entering the queue again.
  - Only a complete refresh can mark tickets moved out or set a baseline. A refresh that was cut off by the ticket limit, cancelled or stopped by a login page cannot. A ticket the re-check finds still open in your queue, or cannot re-read, is never marked moved out.
  - The Tickets toolbar has a **Queue** filter: New to queue, Entered in past 24 h, Moved out in past 24 h, Moved out, In queue and Not moved out.
- **Overview boxes:**
  - **Entered the queue (past 24 h)** and **Moved out (past 24 h)**, each with "+N since the previous refresh".
  - **Unassigned** open tickets.
  - **Red flags**: how many agents are flagged.
  - Click any box to see exactly those tickets in the Tickets table. Overview counts leave out tickets that moved out of the queue.
- **Red flags tab.** It covers open tickets in the chosen queues. A flagged agent is marked "Flagged". Click a name or count to see those tickets.
  - **Most ageing tickets**: open tickets at least 5, 10 or 30 days old (you choose). An agent is flagged with 2 or more of them and more than the team average.
  - **Not updating tickets daily**: open tickets opened at least 24 working hours ago where the agent has not written an update for 24 working hours, or never has. Working hours leave out Saturday and Sunday. An agent is flagged when half or more of their open tickets qualify, or 3 or more do.
  - **Mostly following up outside business hours**: the agent's own updates, judged against each user's business hours at the time of the update. An agent is flagged with at least 4 judged updates and more than half of them outside business hours.
  - The tab also shows the number of unassigned tickets.
- **Every ticket fetched, or listed.** Tickets that fail to load are retried automatically after the main pass: 2 rounds by default, 2 tickets at a time, with a short pause before each round. You can change the number of rounds (0–5) in **Settings → Open tickets to load**. The summary says, for example, "Listed 275 · fetched 275 (15 after retry)".
- **Fetch errors tab** (with the count in its name). It lists the tickets that still failed, as ServiceNow's own list showed them (same columns, same text), with the error in plain words and the number of attempts.
  - Each Number opens that incident in ServiceNow. **Open these tickets in ServiceNow** opens ServiceNow's incident list filtered to exactly these tickets.
  - **Retry** (one ticket) and **Retry all** fetch them again.
  - Tickets that ServiceNow reports as not found are listed but not retried automatically.
- **Excel export** adds First seen in queue, Entered queue, New to queue, Moved out at, Moved out reason, Agent quiet (working hours) and Agent not updating daily.
- **Clear all stored tickets** (Data tab) also clears the queue history and the fetch errors. The next refresh starts a new baseline.

## What's new in 0.4.0

- **Overview tab** (after Tickets). It counts every stored open ticket, meaning any ticket that is not Resolved, Closed or Canceled:
  - Open tickets, VIP users, Follow up now, Updated in past 24 hours, Over 30 days old and Negative sentiment.
  - **Status** split.
  - **Last updated**: past 24 hours, 24–48 hours, 2–5 days, more than 5 days. "Last updated" is the newest update written by a person. When none could be read, ServiceNow's own Updated time is used.
  - **Ticket age by priority**: under 3 days, 3–5, 5–10, 10–30 and over 30 days, one column per priority.
  - **By agent**: both bucket sets per assigned agent, plus how many of the agent's own updates fell inside or outside the user's business hours.
  - Pick the **Queues** (assignment groups) to count. Click any count to open the Tickets table showing exactly those tickets.
- **Timeline shows people only.** Updates by "System", integrations, workflows and other automated authors are left out. Searches and your own follow-ups are no longer timeline entries (the Searched time is a fact in the drawer). Each update is one short line; hover or focus it to read its text.
- **Every note is read.** When the activity stream shows only its newest entries, the extension clicks "Show more" until everything is loaded, up to 40 times or 20 seconds per ticket. The drawer says "not every page could be loaded" when it had to stop early.
- **Multi-select filters.** Business hours now, Ticket status, Priority, Assignment group, Assigned agent, Age, Last updated and Saved view each take several values. Within one filter any chosen value matches; different filters must all match.
- **Quick filters**: VIP users, Follow up now, Negative sentiment and Open only, each with a live count. You can combine them; **Clear all filters** resets everything except the Searched day.
- **Follow up now** badge on a row: the ticket is open and assigned, the user is in business hours now, and either the user replied after the agent's last update or the agent has not updated it for 24 hours. Hover the badge for the reason.
- **Priority** pill (P1–P5) next to the Status.
- **Excel export** adds Priority, Age (Days), Age Bucket, Last Update, Last Update By, Last Updated, Follow Up Now, Follow Up Reason, and Agent Updates In Business Hours / Off Hours. "Update 1-4" now lists updates by people only.
- Tickets stored by 0.3 show the new columns once they are refreshed. Until then the extension works from what was stored.

## What's new in 0.3.0

- **Load all open tickets** (button next to Search). This lists every open ticket of your queues from ServiceNow's own incident list, page by page, in a background tab. It keeps the tickets whose State is one of your chosen states, and fetches them into the Tickets table, 5 at a time.
  - Set the queues (assignment groups) and states in **Settings → Open tickets to load**.
  - The queues are prefilled from the assignment groups of your saved views.
  - The default states are New, Open, Assigned, In Progress, Pending and On Hold. On standard ServiceNow, Open is New and Pending is On Hold.
  - Tickets that were loaded before and are no longer listed are re-checked, so their State updates.
- **Paginated Tickets table.** Choose 25, 50 or 100 rows per page, or All, and move with First / Prev / Next / Last. Filters, counts and Excel export always cover all filtered rows, not just the current page.
- **The user is the Impacted User.** When the incident's "Impacted User" field names someone other than the caller, that person's name, email, phone, country, business hours, VIP flag and sentiment are used. The caller is shown as "reported by". Settings → Contact to use → "Always use the caller" switches this off.
- **Preferred contact actions.** "Teams message" opens `https://teams.microsoft.com/l/chat/0/0?users=<email>`, "Email" opens `mailto:<email>`, and Phone shows the number.
- **User name colour by sentiment.**
  - Red: frustrated (frequent updates) or negative wording written by the user.
  - Green: responding.
  - Grey: no response yet.
- **VIP star** is bigger and blinks red and gold. It stays steady when "reduce motion" is turned on in your operating system.
- **Status shows the State label** ("In Progress"), never the bare number. This also applies to tickets stored by older versions.
- **Settings → Column to field mapping.**
  1. Click **Load fields** to list the fields on one incident's form.
  2. Choose a field for each column (User, Status, Assignment group, Assigned agent, Ticket summary, Preferred contact, Location), or leave it on Automatic.
  3. Click **Test mapping** to see the value each column would show for any incident, then **Save mapping**.
- **Excel export** adds "User Is" and the last 4 ticket updates (Update 1-4 Time and Update 1-4 text).
- **Diagnostics → Discover fields** now works.
- **Out-of-date background warning.** A banner appears when the browser is still running an older copy of the extension's background. Reload the extension on `brave://extensions`, then reopen the dashboard.
- **Data retention** (Settings → Data retention) is now actually applied.

## Install (load unpacked)

1. Open `brave://extensions` (or `chrome://extensions`).
2. Turn on **Developer mode** (top right).
3. Click **Load unpacked** and select this folder (`ServiceNow Ticket update`).
4. Click the extension's toolbar icon to open the dashboard. If it isn't pinned, find it under the
   puzzle-piece icon in the toolbar and pin it.

The extension ships with `host_permissions` for `*.service-now.com` only. The first time you look up a
ticket on an instance with a different domain, the browser will ask you to allow that specific site —
always reviewed by you, never requested silently.

To build the Chrome Web Store package, run `npm run pack`: it zips exactly `manifest.json`,
`background.js`, `README.md`, `bg/`, `dashboard/`, `icons/` and `lib/` into `dist/` (never `dev/`,
`tests/` or `package.json`). `node dev/pack.mjs --list` shows the files without writing the zip.

## First-run checklist (on a real ServiceNow instance)

This has been tested extensively against a local mock ServiceNow server with realistic scenarios, but
every real instance is customized. Before relying on it day to day:

1. **Open an incident and look it up once.** Check the Diagnostics tab's "Discover fields" / field
   inventory if a value looks wrong — it shows what the extractor actually found on the page, with no
   PII beyond what's already in the ticket you have open.
2. **Confirm the Location field path.** If your instance's Location field isn't labelled exactly
   `Location`, the fallback won't fire — the phone-number-based result is still used, just without the
   city-level refinement.
3. **Confirm "Preferred Contact Method(Impacted User)"** matches your instance's actual field label
   (case and punctuation matter for the label-scan fallback). If it's named differently, the extension
   falls back to the incident's standard `contact_type` field.
4. **Check your timezone/date-format assumptions** in Settings if your instance's date fields render in
   a non-default format — the extension tries to read your ServiceNow date-format preference from the
   page itself first.
5. **Turn on "weekend by country"** in Settings if you support regions where the weekend isn't
   Saturday/Sunday.
6. **Review the Settings → View settings** predefined filters (Status / Assignment group) once you know
   which values your instance actually uses — they're free text matched against what the ticket shows,
   not a fixed list.
7. **Try one real ticket update in dry-run mode first** (Settings has a dry-run switch, and the default
   is dry-run **on**) before sending anything for real — a dry run checks everything (field detection,
   permissions, text length) without posting.

## The dashboard

Open it from the toolbar icon. Six tabs:

- **Tickets** — today's tickets in a table: Incident, Status, Assignment group, User (with a VIP star
  when flagged in ServiceNow), Last contact (red/green, by business hours), Business hours now (with a
  priority alert when under 2 hours remain, and a "back in Xh Ym" pill when it's currently off-hours),
  Preferred contact, and the recommended action. The User column also shows the caller's response pill
  (Frustrated / Sporadic response / No response), and a ☹ next to the ticket number flags negative
  sentiment. Filter by text, Status, or a saved view (Settings). Bulk fetch is next to Search; Export to
  Excel is in the toolbar above the table. Each row's *Remove* takes it off the list.
- **Overview** — counts of the open tickets: status, last updated, age by priority, and a per-agent
  table. Every count opens the Tickets table filtered to those tickets.
- **History** — past interactions and follow-ups you've sent (or dry-run checked), per ticket. Each entry
  has its own *Remove*.
- **Data** — the raw local JSON for a ticket, and import/export of your whole local dataset (export is a
  safe way to back up or move your tracked tickets; import always shows a confirmation summary before
  merging anything).
- **Diagnostics** — a content-free fetch log (incident numbers and status codes only, not PII) and a
  "Discover fields" tool for comparing what the extractor found against your instance's actual field
  names.
- **Settings** — timezone/business-hours overrides, weekend-by-country, the service desk name used in the
  guided follow-up messages, dry-run default, note type (comment vs. work note), and predefined
  Status/Assignment-group filter views.

## Privacy

The full privacy policy is in [PRIVACY.md](PRIVACY.md) (the publisher hosts it at a public URL for the
Web Store listing). In short:

- **No server, no analytics, no telemetry, no sale of data.** The publisher receives no data. The
  extension makes no network requests of its own and runs no remote code; it reads ServiceNow pages it
  opens in background tabs of your signed-in session, and never reads passwords, cookies or session
  tokens.
- **What is read:** incident fields (number, summary, state, priority, assignment group, assigned agent,
  times, location, contact method and number), the contact person's name, e-mail, phones and VIP flag,
  the activity stream (work notes and comments with authors), and the rows of your queues' incident list.
- **What is worked out locally:** location, time zone and business hours, follow-up needs, queue tracking
  and engagement / sentiment labels (for example "Frustrated", from keyword and count rules). These
  labels are shown only to you.
- **What is stored, on this computer only** (`chrome.storage.local`): the tickets you looked up or loaded
  (including the text of up to 25 recent notes each), your follow-up history, the queue history (ticket
  numbers and dates, plus the list-row cells of tickets that failed to load; up to 30 days), content-free
  diagnostics logs and your settings. Delete one ticket or **Clear all** on the Data tab; **Settings →
  Data retention** removes old tickets automatically.
- **When data leaves this computer, always because you chose it:** posting a comment or work note (it is
  saved to your ServiceNow instance); a **Teams message** link, which sends the contact's e-mail address
  and your message to Microsoft Teams (`teams.microsoft.com`), where Microsoft handles it; an **Email**
  (`mailto:`) link with the contact's address; **Copy JSON**, which puts ticket data (personal data
  included) on your clipboard; and **Export file** / **Export to Excel**, which save ticket data to a file.
- **Automatic activity:** **Refresh on browser start** (on by default, switch at the top of the dashboard)
  and **Auto refresh** (off by default, Overview tab) open background tabs to your instance.
- **Retention:** stored tickets are kept until you delete them unless you set **Settings → Data
  retention**; queue history is dropped 30 days after a ticket leaves your queues; removing the extension
  deletes all of its data. Site access granted to a custom ServiceNow address can be removed on the
  browser's extension details page (**Site access**).

## Development

No build step, no runtime dependencies — everything is plain ES modules.

```bash
npm test          # unit tests (tests/run.mjs) — run under TZ=UTC / Asia/Kolkata / America/New_York / Pacific/Auckland
npm run mock       # a local, zero-dependency mock ServiceNow server for manual testing
npm run e2e        # end-to-end tests against the mock server in real headless Brave (dev/e2e)
npm run demo       # dev/demo.mjs — a quick manual smoke-test script
npm run pack       # dev/pack.mjs — the Web Store zip (shipped files only) in dist/
```

The unit suite must pass identically in all four timezones above (business-hours/DST logic is
timezone-sensitive). The e2e suite drives a real headless Brave instance over the Chrome DevTools
Protocol against the mock server in `dev/mock-servicenow/` — no real ServiceNow access is needed to run
either suite.

Since 0.5 the mock's classic incident list has the columns of a real default list, and three test controls
(GET or POST, JSON answers; `/__mock/reset` does not reset them, each has its own `reset=1`):
`/__mock/list-control?hide=&show=` (tickets the list leaves out), `/__mock/incident-control?number=&state=&assignment_group=`
(overrides shown on both the list and the form) and `/__mock/fail-control?number=&count=&kind=503|noform|reset`
(the next loads of an incident page fail). The `dashboard-queue-flags` e2e group uses all three.

## Known limitations

- Business-hours/location logic depends on a usable phone number and/or Location field; with neither, the
  status is reported as `unknown` rather than guessed.
- A country whose phone prefix spans multiple timezones (and whose Location field doesn't narrow it to a
  city) is reported as `uncertain` rather than picking one zone arbitrarily.
- Only `*.service-now.com` is a built-in host permission; any other domain (including on-prem or custom
  DNS ServiceNow instances) requires a one-time, visible browser permission prompt.
- The extension clicks the activity stream's "Show more" (or "Load more", "View all", "Show older") control
  up to 40 times or for 20 seconds per ticket. A longer stream is read only partly, and the drawer says
  "not every page could be loaded". A control inside an activity entry, a dialog, or a form's submit button
  is never clicked. If your instance pages the stream some other way, older updates are not counted.
- "Person or system" is decided from the author's name. "System", "System Administrator", integration and
  workflow users, e-mail addresses and login ids (such as `svc_snow`) count as automated. A real person whose
  display name looks like a login id (two or more digits, or an underscore) or contains a word such as
  "integration" or "workflow" is treated as automated too, and their updates do not appear in the timeline.
- Tickets fetched before this version show no pill or icon until they are refreshed.
- Negative sentiment is keyword-based, not a language model: it can miss sarcasm and can flag a phrase
  used neutrally (e.g. a work note saying a ticket was "reopened" for a valid reason).
- Queue tracking compares complete reads of ServiceNow's incident list. A ticket missing from the list is
  re-read first: if it is still open, in a saved queue and in a saved state (the list skipped it while paging,
  or came back short or empty), or the re-read fails, it is not marked "Moved out". This re-check only covers
  tickets stored in the dashboard and is capped at the ticket limit.
- "New to queue" and "Moved out" are only as fine-grained as the auto refresh interval: a ticket that came
  and went between two refreshes is never seen.
- Auto refresh runs only while a dashboard tab is open. Turning it on, or clearing all stored data while it
  is on, with no earlier run starts a refresh straight away.
- VIP status, Status, and Assignment group all require the signed-in user to have read access to those
  fields in ServiceNow; if ACLs block them, the extension reports that plainly rather than guessing.

