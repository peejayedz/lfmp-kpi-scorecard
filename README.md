# Law Firm Marketing Pros · Intake & Signed Clients Scorecard

The intake KPI dashboard for the Law Firm Marketing Pros GHL sub-account (location `sQ2VSMkR7cLHb46Wdemi`). It's hosted on GitHub Pages and synced from GoHighLevel every 30 minutes by GitHub Actions. It uses the same engine as the Premier Builders scorecard.

- **Date selector:** presets (Today, Yesterday, This/Last week, Last 7/30/90 days, This/Last month, This/Last quarter) or any custom From–To dates.
- **Compare to prior:** compares two periods (Today vs Yesterday, This week vs Last week, This month vs Last month, This quarter vs Last quarter, or any two custom ranges). Every number shows the other period's value and the change, coloured by whether the change is good.
- **Refresh data:** starts a sync right away. New numbers land in 1–2 minutes.
- **Executive summary:** Going well / Needs attention / Recommended focus, rule-based and recomputed for the selected dates. Each line opens its list.
- **Drill-downs:** click any number, funnel step, turned-away reason or source to see the people behind it, with search, filters, sorting, CSV export and links that open the GHL contact.

## Files

```
index.html                       the dashboard page
config.json                      goals, stage mapping, ad spend (no secrets)
scripts/kpi-core.mjs             the only place KPIs are calculated (used by the page and the sync)
scripts/sync.mjs                 pulls GHL and writes the data files
.github/workflows/sync-kpis.yml  runs the sync every 30 min and on Refresh
data/kpis.json                   created by the first sync (numbers only)
data/details.enc.json            created by the first sync (encrypted contact records)
```

## Privacy model

A GitHub Pages site is public, so:

- `data/kpis.json` (public) holds **numbers only** for the preset ranges. No names, emails or phones.
- `data/details.enc.json` holds contact-level records, **encrypted** with AES-256-GCM using a key derived from the team passphrase (PBKDF2-SHA256, 210k iterations).
- The page asks for the passphrase once per device. Without it, people see only the preset numbers; custom dates, custom comparisons and drill-downs need it.
- `noindex` keeps the page out of search engines.
- The GHL token lives only in the repo's Actions secrets and is never written to any file.

## One-time setup

1. **Repository variable:** Settings → Secrets and variables → Actions → Variables → `GHL_LOCATION_ID` = `sQ2VSMkR7cLHb46Wdemi`
2. **Repository secrets** (same page, Secrets tab):
   - `GHL_TOKEN`: a GHL Private Integration Token from the LFMP sub-account (scopes below)
   - `DASHBOARD_PASSPHRASE`: the team passphrase (4+ random words)
   - `DISPATCH_TOKEN` (enables the Refresh button): a fine-grained GitHub token. Repository access: **only this repo**. Permission: **Actions: Read and write**. It's stored only inside the encrypted file.
3. **Workflow permissions:** Settings → Actions → General → Workflow permissions → "Read and write permissions" → Save
4. **Pages:** Settings → Pages → Deploy from a branch → `main` / `(root)` → Save
5. **First run:** Actions → "Sync KPIs from GoHighLevel" → Run workflow. Then open the run log and copy the "Pipelines and stages" and "Calendars" lines into the mapping step below.

## GHL Private Integration scopes (all read-only)

Create the token in the Law Firm Marketing Pros sub-account: Settings → Private Integrations → Create new integration.

| Scope | Used for |
| --- | --- |
| `opportunities.readonly` | Leads, pipeline stages, won/lost status |
| `contacts.readonly` | Name, firm, email, phone, tags, source |
| `users.readonly` | "Assigned to" names |
| `locations.readonly` | Location lookup |
| `locations/customFields.readonly` | Finding the custom field ids |
| `calendars.readonly` | Calendar names (which ones are consults) |
| `calendars/events.readonly` | Consult appointments, showed / no-show |
| `conversations.readonly` | Call log |
| `conversations/message.readonly` | Call direction and status, for speed to contact and missed calls |

## Mapping your pipeline (`config.json`)

The first sync prints every pipeline, stage and calendar name in the Action log. Edit `config.json` so the names match exactly (matching ignores upper/lower case):

- `pipelines`: the pipeline(s) that hold LFMP's new-client deals. Empty = every pipeline.
- `stages`: which stage names mean contacted / qualified / consult booked / consult held / agreement sent / signed / no-show. A later step automatically counts as every earlier step, so list only the stages that first prove each step.
- `tags`: tags that prove a step (e.g. `lfmp client` = signed, `showed` = consult held).
- `turnedAway`: a tag like `turned away: wrong practice area` sets the reason shown in "Why leads were turned away". Otherwise the stage name (if listed) or "No reason recorded" is used.
- `booking.calendarKeywords`: a consult is any appointment on a calendar whose name contains one of these words. Empty = every calendar.
- `adSpend.monthly`: `{ "2026-10": 6200, "2026-11": 6500 }`. Used for cost per signed client; spread evenly across each month's days.
- `targets`: the goals behind the On goal / Off goal badges.
- `recommendedFocus`: the advice shown when a goal is missed.

Commit the change and press Refresh data (or run the workflow) to recalculate.

## How the numbers are counted

- **Funnel and conversion rates** follow the leads (opportunities) created in the selected dates, so each step is part of the one before.
- **Speed to first contact:** minutes from lead created to the first outbound call, during business hours if `speedToLead.useBusinessHours` is true (Mon–Fri 9am–6pm in `timezone`). A "Speed to Lead (min)" contact field, if a GHL workflow fills it, takes priority.
- **Contacted within 5 min:** of all new leads, so leads never called count as not within 5 minutes.
- **Missed calls not returned:** inbound calls in the dates that nobody answered, with no outbound call back to that person since.
- **No-show:** a consult whose time has passed and that never reached Consult held.
- A rate based on fewer than `minSample` (5) shows "Too few" instead of a goal badge.

## Troubleshooting

- **"Couldn't load the dashboard data":** the first sync hasn't run. Run the workflow once.
- **Workflow fails with 401/403:** the GHL token is wrong or missing a scope from the table above.
- **Workflow fails at "Commit if changed":** step 3 (Read and write permissions) isn't set.
- **Refresh button only reloads:** `DISPATCH_TOKEN` isn't set, or you haven't unlocked names on this device.
- **A section shows "Not tracked yet":** that scope is missing, so the sync skipped it. The log says which.
