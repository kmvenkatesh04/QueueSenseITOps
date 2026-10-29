# Privacy Policy: QueueSenseITOps

**Effective date:** 2026-10-29
**Extension version:** 0.5.1


QueueSenseITOps ("the extension") is a browser extension for service-desk agents. It reads
incidents that you can already open in your own ServiceNow instance and helps you follow them up. It
shows the contact person's local time and business hours, flags tickets that need a follow-up, tracks
your queues, and lets you post a comment or work note.

This policy describes what the extension reads, what it stores, where it stores it, and when any data
leaves your computer. It applies to the extension only. Your ServiceNow instance, Microsoft Teams and
your e-mail program have their own privacy terms.

> The extension is not made, endorsed or supported by ServiceNow, Inc. "ServiceNow" is a trademark
> of ServiceNow, Inc.

---

## 1. Summary

- **No server, no collection by the publisher.** The extension has no server or back end. [Publisher
  name] receives none of your data.
- **No network requests of its own.** The extension does not call any web service or API. It opens your
  ServiceNow pages in background tabs of your own signed-in browser session and reads them.
- **Stored only in this browser.** Everything the extension keeps is stored in this browser profile's
  extension storage (`chrome.storage.local`) on your computer. It is never synced to other devices.
- **No analytics, no telemetry, no advertising, no sale of data.**
- **No remote code.** All of the extension's code is in the package you install from the Chrome Web
  Store. It does not download or run code from anywhere else.
- **You decide when data leaves your computer:** posting an update to your ServiceNow instance, opening
  a Teams or e-mail link, copying data to the clipboard, or exporting a file.

## 2. Who the data is about

The extension is a workplace tool for service-desk agents. Most of the data it handles is about the
people on your tickets: callers, impacted users and the agents who work on them. The extension only
reads what you can already see in ServiceNow with your own account.

## 3. What the extension reads, and from where

The extension reads data **only** from ServiceNow pages that it opens itself, in a background tab, on
the ServiceNow address you entered:

- an address on `*.service-now.com`, or
- another address (a custom domain) that you entered and then confirmed in the extension and in the
  browser's own permission prompt.

It reads nothing from other websites and nothing while you browse normally. It has no content scripts
that run on the pages you visit.

| Source | What is read | Why |
|---|---|---|
| The incident form (`incident.do`) | Number, short description, state, priority, assignment group, assigned agent, opened and last-updated times, location, preferred contact method, contact number(s), and any extra field you map yourself in Settings | To show the ticket and work out the contact's local time and follow-up needs |
| The contact's user record (caller or impacted user), through ServiceNow's own page functions | Name, e-mail address, phone numbers, preferred contact method, VIP flag | To know who to contact, how, and when |
| The activity stream on the incident form | Work notes and additional comments, with author, time and type. To see the whole stream, the extension may click ServiceNow's own "Show more" control | To work out when the ticket was last updated, by whom, and whether a follow-up is due |
| The incident list (`incident_list.do`), only for "Load all open tickets" or auto refresh | For each row: number, state, assignment group and the text of up to 20 visible columns (up to 200 characters each); the column headings and the row count | To find every open ticket in the queues you chose |
| Field names of one incident form, only when you use Diagnostics or Settings → Column to field mapping | Field names and labels, and whether each field has a value. No field values | To help you map columns to fields. Kept only until the browser closes |

The extension **does not** read or store passwords, cookies, session tokens (such as ServiceNow's
`g_ck`), authorization headers or your browsing history. It never asks for your ServiceNow
credentials. Signing in is always done by you, in ServiceNow, as usual. If ServiceNow sends a
background tab to a sign-in page, the extension stops and tells you to sign in.

## 4. What the extension works out locally

All of the following is calculated **inside your browser**. None of it is sent anywhere:

- the contact person's likely country, time zone, local time and whether it is business hours there
  (from their phone number and the Location field);
- whether a follow-up is due, how often the agent has updated the ticket, and queue tracking (for
  example "new to queue" and "moved out");
- **engagement and sentiment labels** about people on the ticket, such as "Frustrated", "No response"
  or a negative-wording flag showing the matched phrase and who wrote it. These come from simple
  keyword and count rules, not from an AI model. They are shown only to you, to help you prioritise
  follow-ups.

## 5. What is stored, and where

All stored data is kept in `chrome.storage.local`, in this browser profile, on this computer only.
The extension does not use `chrome.storage.sync`, so nothing is synced to your other devices.

| Stored item | Contents | Limits and retention |
|---|---|---|
| Tickets you looked up or loaded | The fields and contact details in section 3, the results in section 4, the text of the 25 newest work notes and comments written by people (each cut to 500 characters), and the author, time and type of up to 300 activity entries | Kept until you delete them, use Clear all, or Settings → Data retention removes them. **By default, retention is "keep forever".** |
| A copy of the most recently looked-up ticket | The same data for one ticket | Removed with that ticket, by Clear all, or by Data retention |
| Follow-up history | For each update you posted or tested in dry-run mode: ticket number, time, channel (Teams, comment or work note), result, and the first 120 characters of your message | Up to 200 entries. Removed per entry or per ticket, by Clear all, or by Data retention |
| Queue history | Ticket numbers seen in each queue, with assignment group and dates. For tickets that then failed to load: the error and the cells of their ServiceNow list row (which can include, for example, the caller and short description) | Up to 5,000 tickets. A ticket is removed 30 days after it leaves your queues. Removed by Clear all |
| Diagnostics logs | Ticket numbers, times and status codes for the last 20 look-ups; for the last 50 posting attempts, a one-way hash (SHA-256) of the text instead of the text | No ticket content. Removed by Clear all |
| Refresh status | The state of the current or last refresh: ticket numbers, instance address, counts and error codes | Replaced at each refresh. Removed by Clear all |
| Your settings | Instance address, queue names, business hours, time zone, service desk name, saved views, field mapping, dry-run choice and similar options | Kept until you change them, reset them, or remove the extension |

The extension also keeps a few short-lived values that are deleted when the browser closes
(`chrome.storage.session`: the IDs of its own background tabs, and the field-name list from section 3),
and a few display preferences in the dashboard page's own storage (page size, selected queues and an
ageing threshold). None of these contain ticket content.

## 6. How to see and delete your data

- **See it:** the dashboard's **Data** tab shows everything stored about your tickets.
- **Delete one ticket:** select the ticket on the Data tab and choose Delete. This removes the ticket
  and its follow-up history. (Remove on the Tickets tab removes the ticket but keeps its follow-up
  history.) In both cases the ticket's number, queue and dates stay in the queue history for up to 30
  days, and its number stays in the diagnostics logs, until you use Clear all.
- **Delete all ticket data:** **Clear all stored tickets** on the Data tab removes all tickets, the
  follow-up history, the queue history, the diagnostics logs and the refresh status. Your instance
  address and settings are kept.
- **Delete old data automatically:** set **Settings → Data retention** to a number of days.
- **Remove access to a custom ServiceNow address:** open the browser's extension details page and
  change **Site access**.
- **Delete everything:** removing the extension from the browser deletes all of its stored data.

## 7. When data leaves your computer

The extension never sends data to [Publisher name] or to any third party by itself. Data leaves your
computer only when you take one of these actions:

| Your action | What is sent, and to whom |
|---|---|
| **Post** a comment or work note | The extension fills in the comment or work note field on the ServiceNow incident form in a background tab and saves it, exactly as if you had submitted the form. Your text goes to **your ServiceNow instance**. Depending on how your instance is set up, ServiceNow may e-mail an additional comment to the caller. Dry-run mode, which is on by default, checks everything without saving. |
| **Teams message** | Your browser opens a Microsoft Teams link (`teams.microsoft.com`) that contains the contact person's **e-mail address** and, if you wrote one in the composer, **your message**. From then on, **Microsoft** handles that data under its own terms. |
| **Email** | Your browser opens a `mailto:` link in your own e-mail program, with the contact's e-mail address only. |
| **Copy JSON** | Ticket data, including personal data, is placed on your system clipboard. |
| **Export file** (JSON) or **Export to Excel** | Ticket data is saved to a file on your computer. |

What happens to copied or exported data, and to messages you send, is up to you.

## 8. Automatic activity

- **Refresh on browser start** (on by default): when the browser starts, the extension re-reads the
  stored tickets (up to 300) in background tabs. It does this only for ServiceNow addresses you have
  already allowed, and only if you are still signed in. You can turn it off with the **Refresh on browser
  start** switch at the top of the dashboard.
- **Auto refresh** (off by default): while a dashboard tab is open, the extension reloads your queues'
  open tickets at the interval you choose.

Background tabs are always closed when the extension has finished reading.

## 9. Permissions and why they are needed

| Permission | Why |
|---|---|
| `storage` | Keep the data in section 5 on your computer |
| `unlimitedStorage` | A working set of a few hundred tickets with their recent notes can exceed the browser's default storage limit. The data still stays on your computer |
| `scripting` | Run the extension's own packaged functions in the ServiceNow tabs it opened, to read the page or to post the update you asked for |
| `tabs` | Open ServiceNow pages in background tabs, notice when a tab is sent to a sign-in page or fails to load, close the tab afterwards, and bring an already-open dashboard tab to the front |
| Host access to `https://*.service-now.com/*` | ServiceNow cloud instances use this address. The extension only opens and reads pages on the one instance you enter |
| Optional host access to `https://*/*` | Never requested as a whole. It is used only to ask for the one address of a ServiceNow instance with a custom domain, after you enter that address and confirm it |

## 10. Limited Use

The use of information received from ServiceNow pages by this extension complies with the
[Chrome Web Store User Data Policy](https://developer.chrome.com/docs/webstore/program-policies/user-data-faq),
including the Limited Use requirements. In particular:

- data is used only to provide the extension's single purpose (helping you follow up ServiceNow
  incidents) and the features described above;
- data is not transferred to anyone, except when you take one of the actions in section 7;
- data is not used or transferred for advertising, including personalised advertising;
- data is not sold;
- data is not used or transferred to determine creditworthiness or for lending purposes;
- no person reads the data, because the publisher never receives it.

## 11. Security

- No credentials are stored.
- Before every script run, the extension checks that the tab is still on exactly the ServiceNow address
  you entered, and refuses to run otherwise.
- The extension's dashboard cannot be reached by websites: web pages cannot send it messages.
- Stored data is protected by your browser profile and your computer's own account security. It is not
  encrypted by the extension.

## 12. Children

The extension is a workplace tool and is not directed at children.

## 13. Changes to this policy

If this policy changes, the new version will be published at the same address with a new effective
date.

## 14. Contact

Questions about this policy: venkateshkmtemp43@gmail.com.
