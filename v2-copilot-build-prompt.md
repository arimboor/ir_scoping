# Build prompt: IR Scoping v2 (offline incident response case manager)

> **How to use this file**
> Paste everything from "PART 1" to the end into Microsoft Copilot (or GitHub Copilot Chat in VS Code).
> If Copilot rejects it as too long, paste PART 1 first and say "Wait for the remaining parts before writing code", then paste each further part, and finish with: "All parts sent. Now write the complete v2.html file."
> If the output is cut off, reply: "Continue exactly where you stopped, inside the same code block."
>
> The finished file is roughly 3,500 lines. Microsoft 365 Copilot chat may not output that much in one go. GitHub Copilot in VS Code (agent or edit mode, writing straight to `v2.html`) handles a file this size far better. If you stay in chat, ask it to build in stages: "Write sections 1 to 4 of the script first", then "Now sections 5 to 9", and so on, and paste the pieces together in order.

***

## PART 1: Role, goal and hard constraints

You are a senior front-end engineer. Build a complete, production-quality **single-file web app** called **IR Scoping v2**: an offline case manager for an incident response (DFIR) consultant. The user picks an incident type, walks through a scoping questionnaire, then manages the case: engagement timeline, attack timeline (MITRE ATT&CK), requests to the client, IOCs, affected assets, a decision log and an hours budget, and finally prints a report.

Hard constraints:

1. Output **one file, `v2.html`**, with inline CSS and vanilla JavaScript. No frameworks, no libraries, no CDNs, no web fonts, no network calls of any kind.
2. It must run by double-clicking the file (file:// URL) in current Chrome and Edge.
3. All data persists in **IndexedDB** across browser restarts and reboots. Never use localStorage for case data (only for the theme preference).
4. Never lose typed data: autosave with a short debounce, and flush saves when the tab is hidden or closed.
5. Everything is config-driven so the user can edit questions, lists, colours and statuses without touching logic.
6. Use `'use strict'`, `const`/`let`, small functions, and section banner comments. Code must be well commented but not noisy.
7. Escape all user-provided text before inserting it into HTML or SVG (write an `esc()` helper and use it everywhere).
8. Never use the em dash character anywhere in UI text or generated output. Use a comma, colon, period or single hyphen instead.
9. Do not use `alert()`, `confirm()` or `prompt()`. Use a custom modal built on the `<dialog>` element.

***

## PART 2: Design system

### 2.1 Theme tokens

Implement these as CSS custom properties on `:root` / `[data-theme="dark"]` (default) and `[data-theme="light"]`. The `<html>` element carries `data-theme`. A small inline script in `<head>` applies the saved theme before first paint (read from localStorage key `ir-scoping-theme`, wrapped in try/catch).

Dark theme (default):
- bg `#0e1319`
- surface `#151c25`
- surface-2 `#1c2531`
- surface-3 `#243041`
- border `#2b3747`
- text `#e4ebf2`
- muted `#8f9db0`
- accent `#4f9cf9`
- accent-strong `#7ab4ff`
- accent-contrast `#06121f` (text on accent buttons)
- danger `#f0616d`
- success `#3fb97a`
- warn `#e3b341`
- focus `#7ab4ff`
- shadow `0 8px 30px rgba(0,0,0,.45)`
- color-scheme: dark

Light theme:
- bg `#f3f5f8`
- surface `#ffffff`
- surface-2 `#f6f8fb`
- surface-3 `#e9eef5`
- border `#d3dbe6`
- text `#17202b`
- muted `#5b6878`
- accent `#1d64d8`
- accent-strong `#134ea9`
- accent-contrast `#ffffff`
- danger `#c7323f`
- success `#1e8a55`
- warn `#a5710a`
- focus `#1d64d8`
- shadow `0 8px 30px rgba(20,30,50,.15)`
- color-scheme: light

Secondary accent used in gradients: `#8a5cf6` (violet).

### 2.2 Typography and spacing

- Body font: `"Segoe UI", system-ui, -apple-system, Roboto, "Helvetica Neue", Arial, sans-serif`, 15px, line-height 1.5.
- Monospace (IOC values, hostnames, kbd): `ui-monospace, Consolas, "Cascadia Mono", monospace`.
- h1 1.45rem, h2 1.2rem, line-height 1.25, no margins by default.
- Section labels / table headers: 0.72 to 0.78rem, uppercase, letter-spacing 0.5px, weight 650, muted colour.
- Main container: max-width 1240px, centred, padding 24px 20px 64px.
- Radii: buttons and inputs 8px, cards 12px, modal 14px, pills and badges 999px.
- Use `font-variant-numeric: tabular-nums` on counts, hours and times.

### 2.3 Components

- **Top bar**: sticky, surface background, bottom border. Left: brand button (26px rounded square "IR" mark with a 135deg gradient from accent to `#8a5cf6`, white bold text) + "IR Scoping" + small accent "v2". Right: save status text and a "Light mode / Dark mode" toggle (ghost button).
- **Save status**: shows "Unsaved changes" / "Saving..." (amber 8px dot), "Saved HH:MM:SS" (green dot), "Save failed" (danger, bold), "Held in memory only" (warn).
- **Buttons**: min-height 38px, padding 7px 14px, weight 550, surface-2 background, border, hover surface-3. Variants: primary (accent bg, accent-contrast text), danger (danger text, tinted border), danger solid (danger bg, white text), success (success text), small (min-height 32px, 0.85rem), ghost (transparent).
- **Inputs, selects, textareas**: full width, min-height 40px, surface-2 background, border, radius 8px; textarea min-height 96px (64px in register forms), vertical resize.
- **Focus**: `:focus-visible` outline 2px focus colour, offset 2px. Inputs get the outline at offset 0 and a focus-coloured border.
- **Cards**: surface background, 1px border, radius 12px.
- **Badges**: pill, 0.78rem, weight 650, uppercase. "Tint badge" for phases and statuses: background = colour mixed 22% with transparent, text = colour mixed 70% with the text token (use `color-mix(in srgb, ...)`), colour passed via an inline CSS variable.
- **Flags** (home list): tiny pills (0.72rem) in warn or danger tint, e.g. "1 overdue request", "Hours at 85% of cap", "Backup due".
- **Stat tiles**: grid `repeat(auto-fill, minmax(150px, 1fr))`, each tile surface-2 with border and radius 10px; big number 1.55rem bold, label 0.82rem muted, optional sub line 0.78rem. Number turns warn or danger colour for warning states.
- **Toasts**: fixed bottom-right stack, surface-3, 4px left border coloured by kind (info accent, success, warn, error danger), slide-up 0.18s animation, auto-dismiss (3.5s default).
- **Modal**: `<dialog>` with blurred dark backdrop, width min(560px, 100vw minus 32px). Inside a `<form method="dialog">`: title, body, actions right-aligned. Submit buttons are placed first in the DOM (so Enter triggers the primary action) and the actions row uses `flex-direction: row-reverse`. Cancel buttons are `type="button"`. Escape cancels. Helper `openModal({title, body, buttons, onOpen})` returns a Promise of `{value, form}`; `confirmModal(title, message, label, danger)` returns a boolean. For destructive confirmations, autofocus the Cancel button.
- **Banner**: full-width alert under the top bar (danger tint, danger border) for "IndexedDB unavailable" and "Database upgraded in another tab".
- **Notice**: inline warn-tinted box with a button, used for the backup reminder.
- **Tabs**: horizontal row that wraps, bottom border; each tab is a borderless button with a 2px bottom border (accent when active, accent-strong text). Each tab can show a small count pill (surface-3) or a warn/danger tinted pill.
- **Yes/No/Unknown control**: segmented control of three radio labels (min-height 38px, padding 6px 18px); the checked label gets accent background and accent-contrast text; a small underlined "Clear" link appears when answered.
- **Multi-select**: grid of checkbox "choice" cards `repeat(auto-fill, minmax(230px, 1fr))`, checked card gets accent border and 12% accent tint.
- **Conditional questions**: indented 14px with a 2px left border in 45% accent.
- **Progress bar**: 8px pill, surface-3 track, fill gradient accent to `#8a5cf6`, animated width.
- **Empty states**: centred muted text, 48px vertical padding.
- **Keyboard hints**: `<kbd>` styled with 1px border, 2px bottom border, 4px radius.

### 2.4 Responsive rules

- At 900px and below the wizard's section sidebar is hidden and a "Jump to section" select appears above the form.
- At 820px and below, list tables become stacked cards: hide thead, each row is a block, each cell shows its label via `td[data-label]::before`.
- At 700px and below, register forms become single column and summary definition lists stack.
- Touch targets at least 38px. Only autofocus fields when `matchMedia('(pointer: fine)')` matches, so tablets do not pop the keyboard on every screen.

### 2.5 Domain colours

Phases: Scoping `#3b82f6`, Collection `#d97706`, Analysis `#8b5cf6`, Reporting `#0d9488`, Closed `#6b7280`.

Request statuses: Open `#3b82f6`, Chased `#d97706`, Received `#16a34a`, Not available `#6b7280`.

Asset statuses: Suspected `#d97706`, Confirmed `#dc2626`, Contained `#8b5cf6`, Remediated `#16a34a`.

Engagement timeline groups: Engagement `#3b82f6`, Evidence `#d97706`, Analysis `#8b5cf6`, Containment `#dc2626`, Reporting `#16a34a`, Other `#6b7280`.

ATT&CK tactic colours: Reconnaissance `#64748b`, Resource Development `#78716c`, Initial Access `#dc2626`, Execution `#ea580c`, Persistence `#d97706`, Privilege Escalation `#a16207`, Defense Evasion `#65a30d`, Credential Access `#16a34a`, Discovery `#0d9488`, Lateral Movement `#0891b2`, Collection `#2563eb`, Command and Control `#4f46e5`, Exfiltration `#9333ea`, Impact `#db2777`.

PNG export palette (always light): bg `#ffffff`, surface `#f6f8fb`, border `#d3dbe6`, text `#17202b`, muted `#5b6878`, line `#c3cdd9`.

***

## PART 3: Storage and data model

### 3.1 IndexedDB

- Database `ir-scoping`, version 1, object store `incidents`, keyPath `id`, index `updatedAt`.
- Write a small promise-based DB layer: `open()`, `getAll()`, `get(id)`, `put(rec)`, `putMany(recs)` (one transaction), `delete(id)`. Each operation resolves on transaction complete and rejects on error or abort.
- If IndexedDB is missing or fails to open (private window, blocked storage), fall back to an in-memory Map, show a red banner explaining data will be lost on close, and add a `beforeunload` prompt.
- On successful open, call `navigator.storage.persist()` (ignore failures).
- Handle `onversionchange` by closing the DB and showing a banner asking the user to reload.

### 3.2 Incident record

```
{
  id: UUID (crypto.randomUUID with a getRandomValues fallback),
  title, clientName, incidentType,              // incidentType is a type id
  phase: 'scoping'|'collection'|'analysis'|'reporting'|'closed',
  status: 'draft'|'complete',                   // kept in sync: closed => complete, else draft
  createdAt, updatedAt,                          // ISO strings
  currentStep: number,                           // wizard section index
  answers: { [questionId]: string | number | string[] },
  budget: { hoursCap: number|'', warnPct: number },   // warnPct default 80
  lastExportedAt: ISO string | null,             // set on JSON export only
  timeline:  [ { id, datetime, type, details, createdAt, updatedAt? } ],
  attack:    [ { id, datetime, tactic, technique, host, account, artefact, description, ... } ],
  requests:  [ { id, description, owner, raised, due, status, notes, ... } ],
  iocs:      [ { id, value, type, confidence, firstSeen, lastSeen, context, ... } ],
  assets:    [ { id, kind, name, identifier, role, privileged, status, notes, ... } ],
  decisions: [ { id, datetime, kind, who, summary, details, ... } ],
  hours:     [ { id, date, person, hours, activity, description, ... } ]
}
```

- `datetime` values are `YYYY-MM-DDTHH:MM` strings (datetime-local format) in the **incident timezone**. `date` values are `YYYY-MM-DD`.
- Every register item has `createdAt`, and `updatedAt` after any edit. Empty fields are removed from stored items.
- Write `normalizeRecord(rec)` that upgrades any stored or imported record to this shape (missing arrays become `[]`, missing phase is derived from status, budget defaults, invalid dates become null). Apply it whenever records are loaded.

### 3.3 Autosave

- `markDirty(touch = true)` sets dirty, shows "Unsaved changes", and debounces a save by 400ms.
- `touch` means content changed and `updatedAt` must be bumped. Navigation-only saves (changing wizard step) use `touch = false` so simply opening or browsing a case does not count as a change.
- `commit()` = immediate content save (used for register add/edit/delete and phase change).
- `flushSave()` serialises saves through a promise chain so writes never overlap; it snapshots the record with structuredClone before writing. On failure: show "Save failed", toast once, retry every 5 seconds.
- Flush on `visibilitychange` (hidden), `pagehide` and `beforeunload`.

***

## PART 4: Screens and behaviour

### 4.1 Home screen

- Header: "Incidents" title, subtitle "Cases saved in this browser.", buttons "Import JSON", "Export all", "New incident" (primary).
- **Backup reminder banner** (see 6.6) with an "Export all now" button.
- **Phase filter chips**: All, Open cases (not closed), Scoping, Collection, Analysis, Reporting, Closed, each with a count. Active chip uses accent background.
- **Search box** with placeholder "Search everything: titles, answers, events, IOCs, hosts, requests..." and an "N of M" counter.
- **Table** sorted by last updated (newest first). Columns: Title, Type, Client, Phase (tint badge), Last updated (`07 Oct 2026, 09:15` en-GB), actions: Open, Duplicate, JSON, Markdown, Delete (danger, confirmation modal).
- Under each title: flags (overdue requests count, hours at or over the warning level, backup due) and, when the search matched inside the case rather than the title/client/type/phase, a line like "IOCs: ...evil.example.com..." with the match wrapped in `<mark>`.
- Search covers: title, client, type label, phase label, every visible questionnaire answer (formatted), and every field of every register item (select values shown as labels). Search the typed term and its refanged form (so `evil[.]com` finds `evil.com`). Case-insensitive.
- **New incident** modal: Incident type select (required), Client name, Incident title (optional; default "Client - Type label - YYYY-MM-DD"). Creating opens the questionnaire.
- **Duplicate** copies everything with a new id, "Copy of " title, phase scoping, lastExportedAt null.

### 4.2 Case shell (all case views)

- Header: h1 title; meta row with incident type badge, client name (or "No client set"), timezone badge (or "Timezone not set"), "Phase" select (left border coloured with the phase colour), backup chip ("Last JSON export 2d ago, changed since" / "Never exported to JSON", warn style and "Backup due:" prefix when stale). Right side: "All incidents", "Export JSON", "Export Markdown".
- Tabs: Questionnaire (badge answered/total), Summary, Engagement timeline (count), Attack timeline (count), Requests ("2 open, 1 overdue", danger if overdue), IOCs (count), Assets ("compromised/total"), Decisions (count), Hours ("11.5/10h", warn or danger by budget level), Report. Empty counts show no badge. Switching tabs flushes saves.
- Changing phase saves immediately and toasts "Phase set to X."
- Document title: "<case title> - IR Scoping v2".

### 4.3 Questionnaire (wizard)

- Progress area: "Section 3 of 13: <title>" left, "X of Y questions answered (Z%)" right, progress bar for section position.
- Two-column body: left sticky sidebar card (280px) listing sections grouped under headings Common / <Type label> / Scope, each with "answered/total" count (green with a check mark when complete; active section highlighted with 18% accent tint); right the section form card.
- Form header: small uppercase accent group label, h2 title, muted description.
- Footer: Back (disabled on first), Next (last section says "Review summary"), plus hint "Enter in a single-line field: next section. Ctrl + Enter: next from anywhere."
- Enter in a single-line input (text, number, date, time, datetime-local) goes Next, except inputs with a datalist. Ctrl/Cmd + Enter goes Next from anywhere in the wizard.
- Field changes update answers, autosave, update conditional visibility in place (toggle `hidden`, do not re-render, so focus is kept), refresh sidebar counts, progress and tab badges.
- Questions with `bind` store on the record (`title`, `clientName`, `incidentType`) instead of answers. Changing incident type switches the type-specific sections; answers for the old type are kept but hidden.

### 4.4 Summary

- One card per section: title with group in brackets, "answered/total", "Edit" button that jumps to that wizard section. Body is a two-column definition list (label muted, answer with pre-wrap); unanswered shows italic "Not answered". Hidden conditional questions are not shown.

### 4.5 Register views (generic engine)

Every register (Part 5) renders with the same engine:

1. Muted intro paragraph.
2. If the register has datetime fields and no incident timezone is set: warn notice "No incident timezone set, so times are stored as entered and no UTC is shown. Set it in the Questionnaire under Engagement & client."
3. Optional **panel** card (stats, budget inputs).
4. **Form card**: "Add <singular>" heading; two-column grid of fields (wide fields span both columns); required fields marked with a red asterisk; datetime fields show the incident timezone after the label, a "Now" button, and a live UTC hint under the input; inline error paragraph; buttons "Add <singular>" (primary), "Cancel edit" (hidden unless editing), hint "Ctrl + Enter to save".
5. Optional **chart card** (both timelines) with "Export PNG".
6. **Table card**: title with count, optional toolbar buttons, table with the register's columns plus row actions (optional quick actions, Edit, Delete with confirmation). Empty state "Nothing recorded yet. Add the first <singular> above."

Engine behaviour:
- Defaults: `'now'` = current time in incident timezone, `'today'` = today in incident timezone, or a literal.
- Validate required fields ("Required: A, B."), then the register's optional `normalize(item)` which can return an error.
- Optional `dedupeKey(item)`: if a different item has the same key, either merge into it with the register's `merge(a, b)` (toast "Merged with existing IOC.") or block with "This <singular> is already in the register. Edit the existing entry instead."
- Edit fills the form, changes the heading to "Edit <singular>" and the button to "Update <singular>", scrolls to the form.
- After any change: commit, reset the form to defaults, refresh panel, chart, table and tab badges.
- Field definition: `{ id, label, type: text|textarea|number|date|datetime|select, required, default, options | groups(rec) (for optgroups), suggestions(rec) (datalist), placeholder, wide, step, onChange(value, form, rec) }`.
- Column definition: `{ label, text(item, rec), html(item, rec)?, exportText(item, rec)?, cls? }`. `text` is used for search and Markdown, `html` for the screen, `exportText` (e.g. defanged IOC) for Markdown, report and sharing.

### 4.6 Report (print-ready)

- Toolbar: explanation "Print-ready view of the whole case. Use Print / Save as PDF and choose Save as PDF as the destination. IOCs are defanged." and a primary "Print / Save as PDF" button calling `window.print()`.
- The report article always uses light colours (override the theme tokens on the `.report` element), white "paper" card, padding 36px 40px, max-width 1040px.
- Content: kicker "INCIDENT RESPONSE CASE REPORT" (accent, uppercase), h1 title, metadata list (Client, Incident type, Phase, Timezone with note "local times shown, UTC in brackets", Created, Last updated, Generated, Record ID).
- "Key figures" stat tiles: Hosts compromised (sub "N suspected"), Accounts compromised, Privileged accounts compromised, IOCs, Attack timeline events, Open requests (sub "N overdue"), Hours used (sub "of Xh cap (Y%)" or "No cap set"), Questions answered.
- "Scoping questionnaire": answered questions only, grouped by section (h3 + definition list), then "N unanswered questions omitted."
- One section per register (Engagement timeline, Attack timeline, Requests, IOCs, Assets, Decisions, Hours) rendered as tables (not charts, so they split across pages). Hours section starts with "Xh logged against a cap of Yh (Z%)." Empty registers say "None recorded."
- Section h2 has a 2px solid text-colour bottom border.
- Print CSS: `@page { margin: 14mm }`, force light tokens, white body, 12px font, hide top bar, banners, tabs, case header, toolbar, footer, toasts and all buttons; tables keep header rows on each page; avoid breaking inside rows, stat tiles and Q/A pairs; avoid breaks after headings; `print-color-adjust: exact`; stat grid 4 columns.

***

## PART 5: Registers

### 5.1 Engagement timeline (`timeline`)
- Intro: "Engagement milestones: access and evidence requested or received, analysis, updates and reports shared."
- Fields: datetime (required, default now); type (required select with optgroups by group, see 7.4); details (textarea, wide, placeholder "e.g. Memory images for DC01 and FS02 requested from client IT").
- Sort by datetime. Columns: When (local + UTC on second line), Event (coloured dot + label), Details.
- Chart: title "Engagement timeline", file slug `engagement-timeline`; card title = event label, tag = group label, colour = group colour, details = details; legend = groups used.

### 5.2 Attack timeline (`attack`)
- Intro: "Threat actor activity reconstructed from evidence, mapped to MITRE ATT&CK tactics. Host and account suggestions come from the Assets register."
- Fields: datetime (required, no default); tactic (required select, label "Initial Access (TA0001)"); technique (text, datalist of "T1566.002 Spearphishing Link" style entries from 7.6; when a known technique id is typed and tactic is empty, set the tactic automatically); host (datalist of asset host names); account (datalist of asset account names); artefact (label "Source artefact", placeholder "e.g. Security.evtx 4624, $MFT, UAL, EDR telemetry"); description (textarea, wide).
- Sort by datetime. Columns: When, Tactic (dot + label), Technique, Host, Account, Source, Description.
- Chart: title "Attack timeline", slug `attack-timeline`; card title = technique or tactic label, tag = tactic label, colour = tactic colour; details = description plus a line "Host: X  |  Account: Y  |  Source: Z"; legend = tactics used, in tactic order.

### 5.3 Requests to client (`requests`)
- Intro: "Everything we have asked the client for. Open or chased requests past their due date are flagged as overdue."
- Fields: description (label "Request", required, wide textarea); owner; raised (date, required, default today); due (date); status (required select, default open); notes (wide textarea).
- Overdue = status open or chased AND due date earlier than today (incident timezone).
- Sort: open/chased first, then by due date (empty last), then raised.
- Columns: Request, Owner, Raised, Due (with red "Overdue" flag), Status (tint badge), Notes. Overdue rows get an 8% danger background.
- Quick row actions: "Chased" (only when open), "Received" (when open or chased).
- Panel: stat tiles per status plus "Overdue" (danger when non-zero).
- Tab badge: "N open" or "N open, M overdue" (danger).

### 5.4 IOCs (`iocs`)
- Intro: "Paste values as-is: defanged input such as hxxp or [.] is converted back, the type is detected automatically, and duplicates are merged into the existing entry."
- Fields: value (required, wide); type (required select, see 7.3, auto-detected from value unless the user changed it manually); confidence (High/Medium/Low, default Medium); firstSeen (datetime); lastSeen (datetime); context (wide textarea).
- **Refang** on input: `hxxp` to `http`, `[.]` `(.)` `{.}` `[dot]` to `.`, `[:]` to `:`, `[@]` `[at]` to `@`, `[/]` to `/`.
- **Auto-detect**: 32/40/64 hex = md5/sha1/sha256; IPv4 or IPv6; scheme:// = url; x@y.z = email; common file extensions (exe, dll, ps1, bat, js, vbs, hta, lnk, msi, iso, zip, 7z, rar, docm, xlsm, pdf, scr, etc.) or slashes = filename; else domain pattern = domain.
- **Validate**: hashes exact hex length ("SHA256 must be 64 hexadecimal characters."), IPv4 octets 0 to 255 or valid IPv6 ("Not a valid IPv4 or IPv6 address."), email, URL needs a scheme, domain pattern, last seen not before first seen. Lowercase domains, emails and hashes.
- **Dedupe** key `type|value`; merge keeps earliest first seen, latest last seen, highest confidence, and combines distinct context lines.
- Sort by type order then value. Columns: Type, Value (monospace; exportText = defanged), Confidence, First seen, Last seen, Context.
- **Defang** for sharing: IP and domain dots to `[.]`; URL `http` to `hxxp` and dots to `[.]`; email `@` to `[@]` and dots to `[.]`; hashes and filenames unchanged.
- Toolbar: "CSV (raw, for EDR / SIEM)" and "CSV (defanged, for sharing)". Columns: `type,value,confidence,first_seen_local,first_seen_utc,last_seen_local,last_seen_utc,context,case`. Quote cells containing comma, quote or newline. Prefix free-text cells starting with `= + - @` with an apostrophe (formula injection guard) but never alter the value column. Use CRLF line endings, no BOM.
- Panel: tiles for total and per type (non-zero only).

### 5.5 Assets (`assets`)
- Intro: "Hosts and accounts in scope. Confirmed, contained and remediated all count as compromised; suspected does not."
- Fields: kind (Host/Account, required, default host); name (required, placeholder "Hostname, or account (UPN / DOMAIN\user)"); identifier (IP, SID or other); role ("e.g. Domain controller, finance user"); privileged (Yes/No/Unknown); status (required, default suspected); notes (wide).
- Dedupe key `kind|lowercase(name)`, blocked (no merge).
- Sort by kind, then status order, then name. Columns: Type, Name (mono), Identifier (mono), Role, Privileged, Status (tint badge), Notes.
- Panel ("blast radius"): tiles Hosts compromised (sub "N suspected"), Accounts compromised, Privileged accounts compromised (accounts with privileged Yes and status not suspected), Contained, Remediated; a small table Hosts/Accounts by status with totals; and the line "Scoping estimate from the questionnaire: X systems, Y user accounts." (from answers `affected_systems_count` and `affected_users_count`, "?" if one is missing, or "Scoping estimate: not recorded in the questionnaire.").
- Tab badge "compromised/total".

### 5.6 Decisions and communications log (`decisions`)
- Intro: "Who decided or approved what, and when. Entries show when they were last edited, to support a defensible record."
- Fields: datetime (required, default now); kind (required, default Decision, see 7.5); who (label "Made / approved by", required, placeholder "e.g. Client CISO (J. Smith)"); summary (required, wide, "e.g. Approved isolation of FS02"); details (label "Rationale, participants, channel", wide textarea).
- Sort by datetime. Columns: When, Type, By, Summary (plus muted "Edited <date time>" line when updatedAt exists), Details.

### 5.7 Hours (`hours`)
- Intro: "Time booked against this case. Set the hours cap (e.g. the insurer panel limit) to get warnings as you approach it."
- Fields: date (required, default today); person (required, datalist of people already used); hours (number, step 0.25, required, must be more than 0 and no more than 24); activity (required select, see 7.5); description (wide).
- Sort by date, newest first. Columns: Date, Person, Hours (right-aligned), Activity, Description.
- Panel: "Hours cap" and "Warn at (% of cap)" number inputs (saved to `budget` on input, without re-rendering the inputs); a 12px meter (green, warn colour at or above the warning %, danger at or above 100%); text "11.5h of 10h used (115%). 1.5h over cap." or "Xh remaining." or "Xh logged. No cap set."; small tables "By activity" and "By person".
- After adding or editing an entry, toast once when crossing the warning level ("Hours at 85% of the 10h cap.") and when crossing 100% ("Hours cap exceeded: 11.5 of 10h used.").

***

## PART 6: Timezone, charts, exports

### 6.1 Incident timezone

- Question `discovery_tz` (type `timezone`) in the first section is the incident timezone. Render it as a select of all IANA zones (`Intl.supportedValuesOf('timeZone')`, with UTC first, de-duplicated, and a short fallback list) plus a "Use browser timezone (<zone>)" button. Unknown stored values show as "<value> (not a recognised timezone)".
- Convert a local datetime string in an IANA zone to UTC without libraries: compute the zone offset with `Intl.DateTimeFormat(...).formatToParts` (hourCycle h23), apply it, then re-check the offset at the resulting instant (two-pass, correct across DST).
- Every datetime field shows a live hint: "Europe/London (UTC+01:00) = 2026-10-07 08:30 UTC", or "Enter in Europe/London.", or "Set the incident timezone (Engagement & client) to see UTC."
- Formatted answers: "2026-10-07 09:30 Europe/London (2026-10-07 08:30 UTC)" or "... (timezone not set)". The timezone answer itself shows "Europe/London (currently UTC+01:00)".
- Display local datetimes as "07 Oct 2026 09:30" by string parsing (never through a Date object, to avoid shifting).

### 6.2 Timeline chart (SVG)

`buildTimelineSVG(rec, palette, { events: [{datetime, title, tag, color, details}], legend: [{label, color}] }, heading)` returns `{ svg, width, height }`. Geometry:
- Width 960, padding 32. Title 20px bold (wrapped), sub line 13px muted: "<heading> · <client> · <type> · Timezone: <tz>". Legend: coloured circles r=6 with 12px labels, wrapping. 1px separator line.
- Event rows: date column right-aligned at x=220 (local datetime 13px weight 650; UTC 11.5px muted beneath). Vertical line at x=250 (2px, line colour) from the first to the last dot; dots r=8 in the event colour with a 3px stroke of the background colour.
- Card from x=280 to the right padding: rect radius 8, surface fill, border stroke, 5px accent bar in the event colour; title 14px bold (wrapped, leaving room for the tag); tag uppercase 10.5px bold in event colour, right-aligned; details 13px wrapped with 18px line height; minimum card height 48; 30px gap between cards.
- Gap labels between rows ("+1h 30m", "+1d 5h", "+3d", "same time"), 11px muted, right-aligned at x=234, vertically centred in the gap.
- Footer 11px muted: "5 events · Span 6d 5h · Generated 07 Oct 2026, 08:50".
- Measure text with a hidden canvas 2D context using the same font stack. Wrap by words; split unbreakable long strings (hashes, paths) by characters.
- On screen use the current theme tokens (read via getComputedStyle); re-render on theme change. The SVG scales to the card width (max 960px).
- **PNG export**: build with the light export palette, load the SVG as a `data:image/svg+xml` Image, draw onto a canvas at 2x (1x if the height at 2x would exceed 16000px), `canvas.toBlob` PNG, download as `ir-<slug>-<case-title-slug>-<YYYY-MM-DD>.png`.

### 6.3 Downloads

All downloads go through one helper: create a Blob, `URL.createObjectURL`, a hidden `<a download>`, click, then revoke after 1.5s. Import uses a hidden `<input type="file" accept=".json,application/json">` that is reset after each use.

### 6.4 JSON export and import

- Export wrapper: `{ schemaVersion: 3, app: 'ir-scoping', appVersion: '2.0.0', exportedAt, count, incidents: [...] }`, pretty-printed. Single case file `ir-scoping-<slug>-<date>.json`; all cases `ir-scoping-all-<date>.json`. Flush saves and re-read from the DB before exporting.
- After a JSON export, set `lastExportedAt` on each exported case **without changing updatedAt**, then refresh the header chip and home list.
- Import validation: must be an object with numeric `schemaVersion` and an `incidents` array (fatal otherwise). Newer schema = warning. Each incident needs a string `id` and `incidentType`; `answers` must be an object. Keep only string, number, boolean or string-array answers. Sanitise every register against its field definitions (coerce numbers, require valid datetime/date formats, drop items missing required fields) and report counts of dropped items. Unknown incident types import with a warning. Older schema 1 and 2 files must import (missing registers become empty, phase derived from status).
- Import modal shows counts, notes, and when ids already exist a radio choice: **Merge** (default: the more recently updated record wins on top-level fields and on answers present in both; answers only in the older one are kept; register items are combined by id with the newer winning; duplicate IOCs merged; local lastExportedAt kept) or **Skip** (leave existing cases untouched). Toast "Import complete: X added, Y merged, Z skipped, N invalid."

### 6.5 Markdown export

File `ir-scoping-<slug>-<date>.md`:
- `# IR Case Report: <title>`, then bullets: Client, Incident type, Phase, Incident timezone (with note), Created, Last updated, Questions answered, Record ID.
- One `##` per questionnaire section, each question as bold label then the answer or `_Not answered_` (hidden conditional questions omitted; multi-line answers use Markdown line breaks).
- One `##` per non-empty register; each item is a bullet of `**Column:** value` pairs joined with ` | `, using exportText (IOCs defanged). Hours section starts with the logged-versus-cap sentence.
- Footer: `***` then `_Generated <date> by IR Scoping v2.0.0 (schema 3)._`

### 6.6 Backup reminder

- `BACKUP_WARN_DAYS = 3`. A case is "backup due" when it has changed since its last JSON export (or was never exported) AND that export (or its creation, if never exported) is more than 3 days old.
- Home banner: "Backup reminder: N cases have changes not exported to JSON in over 3 days." with "Export all now". Row flag "Backup due". Case header chip as in 4.2.

***

## PART 7: Configuration lists (put all of these in one CONFIG section)

### 7.1 Incident types
ransomware "Ransomware", bec "Business Email Compromise", malware "Malware / Endpoint Compromise", exfiltration "Data Exfiltration", insider "Insider Threat", other "Other".

### 7.2 Phases, statuses
Phases and colours as in 2.5. Request statuses: open, chased, received, not_available (labels "Open", "Chased", "Received", "Not available"). Asset kinds: host "Host", account "Account". Asset statuses: suspected, confirmed, contained, remediated.

### 7.3 IOC types and confidence
ip "IP address", domain "Domain", url "URL", email "Email address", md5 "MD5", sha1 "SHA1", sha256 "SHA256", filename "File name / path", other "Other". Confidence: High, Medium, Low.

### 7.4 Engagement event types (id, label, group)
engagement_start "Engagement started" (engagement); scoping_call "Scoping call" (engagement); client_meeting "Client meeting / status call" (engagement); access_requested "Access requested" (evidence); access_granted "Access granted" (evidence); evidence_requested "Evidence requested" (evidence); evidence_received "Evidence received" (evidence); collection_started "Collection started" (evidence); collection_done "Collection complete" (evidence); analysis_started "Analysis started" (analysis); analysis_done "Analysis complete" (analysis); key_finding "Key finding" (analysis); containment_action "Containment / remediation action" (containment); interim_shared "Interim update shared" (reporting); draft_report "Draft report shared" (reporting); final_report "Final report shared" (reporting); engagement_closed "Engagement closed" (reporting); other "Other" (other).

### 7.5 Decision kinds and hour activities
Decision kinds: Decision, Approval, Instruction from client, Advice given, Communication, Other.
Hour activities: Scoping, Evidence collection, Analysis, Containment / remediation support, Reporting, Meetings / comms, Project management, Travel, Other.

### 7.6 ATT&CK tactics and techniques
Tactics (id, label): TA0043 Reconnaissance, TA0042 Resource Development, TA0001 Initial Access, TA0002 Execution, TA0003 Persistence, TA0004 Privilege Escalation, TA0005 Defense Evasion, TA0006 Credential Access, TA0007 Discovery, TA0008 Lateral Movement, TA0009 Collection, TA0011 Command and Control, TA0010 Exfiltration, TA0040 Impact. (Colours in 2.5.)

Techniques as `[id, name, tactic]`:
- Initial Access (TA0001): T1566 Phishing; T1566.001 Spearphishing Attachment; T1566.002 Spearphishing Link; T1190 Exploit Public-Facing Application; T1133 External Remote Services; T1078 Valid Accounts; T1199 Trusted Relationship; T1189 Drive-by Compromise.
- Execution (TA0002): T1059 Command and Scripting Interpreter; T1059.001 PowerShell; T1059.003 Windows Command Shell; T1047 Windows Management Instrumentation; T1204 User Execution; T1569.002 Service Execution.
- Persistence (TA0003): T1053.005 Scheduled Task; T1543.003 Windows Service; T1547.001 Registry Run Keys / Startup Folder; T1136 Create Account; T1098 Account Manipulation; T1505.003 Web Shell.
- Privilege Escalation (TA0004): T1068 Exploitation for Privilege Escalation; T1548.002 Bypass User Account Control; T1134 Access Token Manipulation.
- Defense Evasion (TA0005): T1562.001 Disable or Modify Tools; T1070.001 Clear Windows Event Logs; T1070.004 File Deletion; T1027 Obfuscated Files or Information; T1036 Masquerading; T1218 System Binary Proxy Execution; T1564.008 Email Hiding Rules.
- Credential Access (TA0006): T1003.001 LSASS Memory; T1003.003 NTDS; T1003.006 DCSync; T1110 Brute Force; T1558.003 Kerberoasting; T1555 Credentials from Password Stores; T1557 Adversary-in-the-Middle; T1621 Multi-Factor Authentication Request Generation; T1539 Steal Web Session Cookie.
- Discovery (TA0007): T1087 Account Discovery; T1018 Remote System Discovery; T1046 Network Service Discovery; T1482 Domain Trust Discovery; T1083 File and Directory Discovery.
- Lateral Movement (TA0008): T1021.001 Remote Desktop Protocol; T1021.002 SMB/Windows Admin Shares; T1021.006 Windows Remote Management; T1570 Lateral Tool Transfer; T1550.002 Pass the Hash.
- Collection (TA0009): T1560 Archive Collected Data; T1114 Email Collection; T1114.003 Email Forwarding Rule; T1005 Data from Local System; T1039 Data from Network Shared Drive.
- Command and Control (TA0011): T1071.001 Web Protocols; T1219 Remote Access Software; T1572 Protocol Tunneling; T1090 Proxy; T1105 Ingress Tool Transfer.
- Exfiltration (TA0010): T1041 Exfiltration Over C2 Channel; T1567.002 Exfiltration to Cloud Storage; T1048 Exfiltration Over Alternative Protocol.
- Impact (TA0040): T1486 Data Encrypted for Impact; T1490 Inhibit System Recovery; T1489 Service Stop; T1485 Data Destruction; T1657 Financial Theft.

### 7.7 Other settings
`APP_VERSION = '2.0.0'`, `SCHEMA_VERSION = 3`, `BACKUP_WARN_DAYS = 3`, `DEFAULT_BUDGET_WARN_PCT = 80`, autosave debounce 400ms, theme key `ir-scoping-theme`, `TZ_QUESTION_ID = 'discovery_tz'`.

***

## PART 8A: Questionnaire config and common questions

### 8.1 Question schema
Define all questions in one `QUESTIONNAIRE` object: `{ commonSections: [...], typeSections: { ransomware: [...], bec: [...], ... }, closingSections: [...] }`. Wizard order = common, then the selected type's sections, then closing. Each section: `{ id, title, description, questions }`. Each question:
- `id` (unique across all sections; prefix type-specific ids), `label`, `type`, optional `options`, `help`, `placeholder`, `suggestions`, `bind`, `showIf`.
- Types: `text`, `textarea`, `number` (min 0), `date`, `time`, `datetime`, `select` (with "Select..." empty option), `multiselect` (checkbox cards), `yesno` (Yes/No/Unknown segmented), `timezone`.
- `showIf`: `{ q, equals }`, `{ q, in: [...] }`, `{ q, notEquals }`, `{ q, notEmpty: true }`. For multiselect controllers, match if any selected value matches. If the controller is hidden, the dependent is hidden. Build a question index at startup and console.warn on duplicate ids or unknown showIf targets.
- Empty answers are deleted from `answers`.

Notation below: `id | type | label | options | condition | help`. "yn" = yesno.

### 8.2 Common sections

**engagement: "Engagement & client"** (desc: "Core engagement details. Title, client and incident type are also shown on the incident list.")
- _title | text, bind title | Incident title | placeholder "e.g. ACME Corp ransomware Oct 2026"
- _clientName | text, bind clientName | Client name
- _incidentType | select, bind incidentType | Incident type | the 6 types | help "Changing this switches the type-specific sections. Answers already given for another type are kept but hidden."
- discovery_tz | timezone | Incident timezone | help "All date/time answers in this incident are recorded in this timezone and also shown in UTC."
- client_industry | select | Client industry / sector | Financial services, Insurance, Healthcare, Pharma / life sciences, Manufacturing, Retail / e-commerce, Technology / SaaS, Professional services / legal, Education, Public sector / government, Energy / utilities, Telecoms, Transport / logistics, Charity / non-profit, Other
- client_size | select | Organisation size (employees) | 1-49, 50-249, 250-999, 1,000-4,999, 5,000-19,999, 20,000+
- client_hq | text | Headquarters country and other operating countries
- engagement_ref | text | Internal case / matter reference
- engaged_via | select | Engaged via | Client directly, Outside counsel, Cyber insurer / panel, Broker, MSSP / partner, Other
- privilege | yn | Is the engagement under legal privilege? | help "If yes, confirm labelling and distribution requirements for all work product."
- counsel_details | text | Outside counsel firm and lead contact | if privilege = Yes

**contacts: "Points of contact"** (desc: "Who we work with day to day, and how we talk to them safely.")
- poc_name | text | Primary contact name
- poc_role | text | Primary contact role | placeholder "e.g. CISO, Head of IT"
- poc_email | text | Primary contact email
- poc_phone | text | Primary contact phone
- poc_tech | text | Technical lead (name, role, contact)
- poc_exec | text | Executive sponsor / decision maker
- comms_compromised | yn | Is corporate email or chat believed to be compromised?
- comms_channel | select | Agreed communication channel | Corporate email, Corporate Teams / Slack, Phone only, Out-of-band (Signal / WhatsApp), Separate clean tenant / bridge, Other
- comms_oob_details | textarea | Out-of-band channel details | if comms_compromised in Yes, Unknown | help "Assume the threat actor may be reading corporate comms until proven otherwise."
- poc_availability | textarea | Availability, working hours and escalation path

**discovery: "Discovery & detection"** (desc: "When and how the incident came to light.")
- discovery_dt | datetime | Date and time of discovery
- earliest_activity_dt | datetime | Earliest known suspicious activity (if known)
- detection_source | multiselect | How was it detected? | EDR / AV alert, SIEM alert, MDR / SOC notification, User report, IT noticed outage / performance issue, Ransom note / extortion message, Third-party notification (customer, supplier), Law enforcement / NCSC notification, Bank / financial anomaly, Threat intel / dark web monitoring, Audit / internal review, Other
- detection_details | textarea | Detection narrative | help "Who noticed what, when, and what happened next."
- activity_ongoing | yn | Is malicious activity believed to be ongoing?
- prior_incidents | yn | Any related or prior incidents in the last 12 months?
- prior_incidents_details | textarea | Prior incident details | if prior_incidents = Yes

**environment: "Environment & affected systems"** (desc: "Size and shape of the estate, and what is known to be affected.")
- affected_systems_count | number | Number of systems known or suspected to be affected
- affected_users_count | number | Number of user accounts known or suspected to be affected
- total_endpoints | number | Total endpoints (workstations / laptops)
- total_servers | number | Total servers (physical and virtual)
- env_platforms | multiselect | Platforms in the environment | On-prem Active Directory, Entra ID (Azure AD), Hybrid identity (AD Connect), Microsoft 365, Google Workspace, AWS, Azure, GCP, VMware ESXi / vCenter, Hyper-V, Citrix / VDI, OT / ICS, Mainframe
- env_os | multiselect | Operating systems in scope | Windows, Linux, macOS, ESXi, Network appliances, Mobile
- edr_product | text | EDR / AV product(s) deployed
- edr_coverage | select | Estimated EDR coverage | 95-100%, 75-94%, 50-74%, Under 50%, No EDR, Unknown
- siem_product | text | SIEM / log platform
- msp | text | MSP / MSSP involvement (name, scope of service)
- sites_affected | textarea | Sites, regions or business units affected

**impact: "Business impact"** (desc: "Operational and data impact as currently understood.")
- impact_level | select | Overall business impact | Critical: core operations halted, High: significant degradation, Moderate: limited disruption, Low: no noticeable disruption, Unknown
- impacted_functions | multiselect | Business functions impacted | Production / manufacturing, Customer-facing services, Finance / payments, Payroll / HR, Email / collaboration, Logistics / supply chain, Clinical / patient care, Website / e-commerce, Internal IT only, None
- critical_systems_down | textarea | Critical systems currently unavailable
- data_at_risk | multiselect | Data types potentially at risk | DATA_TYPES (see 8.5)
- records_estimate | number | Estimated number of individuals / records affected
- impact_cost | text | Estimated cost of disruption (per day or total, with currency)
- media_attention | yn | Is there media or public attention?

**regulatory: "Regulatory & notification"** (desc: "Notification obligations and clocks that may already be running.")
- reg_frameworks | multiselect | Potentially applicable regimes | UK GDPR / DPA 2018 (ICO), EU GDPR, NIS Regulations / NIS2, DORA, FCA / PRA, PCI DSS, HIPAA, SEC cyber disclosure (8-K), US state breach laws, Sector regulator (other), None identified, Unknown
- dpo_engaged | yn | Has the DPO / privacy team been engaged?
- reg_notified | yn | Has any regulator been notified?
- reg_notified_details | textarea | Regulator(s), date notified and reference | if reg_notified = Yes
- reg_deadline | datetime | Earliest notification deadline | help "e.g. 72 hours from awareness for UK/EU GDPR."
- contract_notify | yn | Contractual notification obligations to customers or partners?
- contract_notify_details | textarea | Contractual obligations details | if contract_notify in Yes, Unknown

**insurance: "Cyber insurance & broker"** (desc: "Policy, panel and approval requirements.")
- insured | yn | Does the client hold cyber insurance?
- insurer | text | Insurer | if insured = Yes
- policy_number | text | Policy number | if insured = Yes
- broker | text | Broker (firm and contact) | if insured = Yes
- insurer_notified | yn | Has the insurer been notified? | if insured = Yes
- claim_ref | text | Claim reference | if insurer_notified = Yes
- panel_requirements | textarea | Panel / pre-approval requirements (rates, scope caps, reporting) | if insured = Yes

**law_enforcement: "Law enforcement"** (desc: "Current or planned law enforcement and government agency involvement.")
- le_involved | yn | Is law enforcement involved?
- le_agencies | multiselect | Agencies involved | Action Fraud / NFIB, Regional Organised Crime Unit (ROCU), NCA, Local police, NCSC, FBI, CISA / US Secret Service, Europol / national police (EU), Other | if le_involved = Yes
- le_reference | text | Crime / case reference and officer contact | if le_involved = Yes
- le_planned | select | Does the client intend to report? | Yes, No, Undecided, Awaiting legal advice | if le_involved in No, Unknown
- le_constraints | textarea | Any constraints from law enforcement (e.g. evidence handling, do-not-alert) | if le_involved = Yes

**containment: "Containment actions taken"** (desc: "What has already been done. Note anything that may have destroyed or altered evidence.")
- containment_actions | multiselect | Actions already taken | Hosts isolated (network / EDR), Internet egress blocked or restricted, VPN / remote access disabled, Accounts disabled, Password resets (users), Password resets (privileged / service accounts), krbtgt reset, Sessions / tokens revoked, MFA re-registered, IOCs blocked at firewall / proxy, Systems powered off, Backups isolated, External access (RDP, etc.) closed, None yet, Other
- containment_details | textarea | Containment details and timings
- systems_rebuilt | yn | Have any affected systems been wiped, reimaged or restored?
- systems_rebuilt_details | textarea | Which systems, and was anything captured first? | if systems_rebuilt = Yes
- ta_access_removed | yn | Is the threat actor believed to be locked out?
- third_party_ir | yn | Has any other IR firm or MSP performed work so far?
- third_party_ir_details | textarea | Who, what was done, and are their outputs available? | if third_party_ir = Yes

**evidence: "Logs & evidence preserved"** (desc: "What evidence exists, how long it will survive, and how we can get it.")
- evidence_preserved | yn | Has any evidence been preserved?
- evidence_types | multiselect | Evidence preserved so far | Memory images, Full disk images, Triage collections (KAPE / Velociraptor / CyLR), VM snapshots, EDR telemetry export, Windows event logs, Firewall logs, Proxy / web gateway logs, VPN logs, DNS logs, NetFlow, M365 Unified Audit Log, Entra sign-in / audit logs, Cloud audit logs (CloudTrail / Activity Log), Email samples, Malware samples | if evidence_preserved = Yes
- log_retention | textarea | Retention of key log sources | help "e.g. firewall 30 days, EDR 7 days, SIEM 90 days. Flag anything about to roll over."
- chain_of_custody | yn | Is chain of custody being recorded?
- evidence_location | text | Where is preserved evidence stored?
- remote_collection | yn | Can we deploy collection tooling remotely (EDR / Velociraptor)?
- access_requirements | textarea | Access requirements (accounts, VPN, jump host, approvals)

***

## PART 8B: Type-specific and closing questions

### 8.3 Type-specific sections

**Ransomware**
- rw_actor "Ransom note & threat actor" (desc "Threat actor identification and extortion details."): rw_note_found yn "Has a ransom note been found?"; rw_note_details textarea "Ransom note filename, contents or contact method" (if rw_note_found = Yes); rw_variant text "Suspected variant / group" (placeholder "e.g. Akira, LockBit, Play"); rw_extension text "Encrypted file extension"; rw_ta_contact yn "Has anyone contacted the threat actor or visited the negotiation portal?"; rw_negotiator text "Is a negotiator engaged? (firm and contact)" (if rw_ta_contact = Yes); rw_demand text "Ransom demand (amount and currency)"; rw_deadline datetime "Threat actor deadline"; rw_exfil_claimed yn "Does the threat actor claim data theft?"; rw_leak_site yn "Is the client listed on a leak site?"; rw_payment_stance select "Client position on payment" (Will not pay, Considering, Undecided, Awaiting legal / insurer advice); rw_sanctions yn "Has sanctions screening (OFSI / OFAC) been considered?" (if rw_payment_stance in Considering, Undecided, Awaiting legal / insurer advice).
- rw_scope "Encryption scope & recovery" (desc "Blast radius, identity infrastructure and recoverability."): rw_encryption_start datetime "When did encryption start (first observed)?"; rw_encrypted_types multiselect "System types encrypted" (Workstations, File servers, Database servers, Application servers, Domain controllers, ESXi / hypervisors, NAS / SAN, Backup servers, Cloud VMs, OT / ICS); rw_encrypted_count number "Approximate number of encrypted systems"; rw_dc_impact yn "Are domain controllers encrypted or compromised?"; rw_dc_details textarea "DC details (how many, which sites, any healthy DCs left)" (if rw_dc_impact in Yes, Unknown); rw_hypervisor yn "Were hypervisors / vCenter targeted?"; rw_backup_status select "Backup status" (Intact, offline / immutable; Intact but online / reachable; Partially affected; Encrypted or deleted; No backups; Unknown); rw_backup_product text "Backup product and storage location"; rw_last_good_backup datetime "Last known good backup"; rw_backup_tampering yn "Evidence of backup tampering (deleted jobs, VSS deletion, console access)?"; rw_initial_access multiselect "Suspected initial access vector" (VPN / firewall vulnerability, Exposed RDP, Phishing, Valid credentials (no MFA), Third-party / supplier access, Unpatched public-facing application, Unknown); rw_decryptor yn "Has a public decryptor been checked (e.g. No More Ransom)?"; rw_recovery_priorities textarea "Recovery priorities (systems in order)".

**Business Email Compromise**
- bec_tenant "Mail platform & identity" (desc "Tenant, logging and identity controls."): bec_platform select "Mail platform" (Microsoft 365 / Exchange Online, Exchange on-prem, Hybrid Exchange, Google Workspace, Other); bec_tenant text "M365 / Entra tenant ID and primary domain" (if bec_platform in Microsoft 365 / Exchange Online, Hybrid Exchange); bec_licence select "Licence level" (E5 / A5 / G5, E3 / A3 / G3, Business Premium, Business Standard / Basic, Google Workspace Enterprise, Google Workspace Business, Mixed, Unknown; help "Drives log availability (e.g. MailItemsAccessed, retention)."); bec_ual yn "Is the Unified Audit Log enabled?" (same condition as bec_tenant); bec_admin_access yn "Can we be granted read-only admin roles (Global Reader, Security Reader)?"; bec_affected_count number "Number of affected mailboxes"; bec_affected_mailboxes textarea "Affected mailboxes / accounts" (placeholder "One per line"); bec_privileged yn "Are any affected accounts privileged (admin roles)?"; bec_mfa_status select "MFA status for affected users" (Enforced for all users, Enforced for some users, Not enabled, Unknown); bec_mfa_methods multiselect "MFA methods in use" (SMS / voice, Authenticator push, Push with number matching, TOTP code, FIDO2 / passkey, Windows Hello; if bec_mfa_status in Enforced for all users, Enforced for some users); bec_ca yn "Are Conditional Access policies in place?"; bec_legacy_auth yn "Is legacy authentication blocked?"; bec_aitm yn "Is AiTM phishing or token theft suspected?".
- bec_activity "Mailbox activity & financial loss" (desc "Attacker actions in the mailbox and any resulting fraud."): bec_rules yn "Suspicious inbox rules found?"; bec_rules_details textarea "Inbox rule details (names, conditions, actions)" (if bec_rules = Yes); bec_forwarding yn "External forwarding configured?"; bec_oauth yn "Suspicious OAuth app consents or new app registrations?"; bec_mfa_added yn "New MFA methods or devices registered by the attacker?"; bec_phish_sent yn "Was the mailbox used to send phishing (internal or external)?"; bec_phish_count number "Approximate number of recipients" (if bec_phish_sent = Yes); bec_file_access yn "Evidence of SharePoint / OneDrive / Drive access by the attacker?"; bec_fraud yn "Fraudulent payment or bank detail change?"; bec_loss text "Financial loss (amount and currency)" (if bec_fraud = Yes); bec_payment_date date "Date of fraudulent payment" (if bec_fraud = Yes); bec_bank_recall yn "Has the bank been contacted to recall funds?" (if bec_fraud = Yes); bec_recall_status text "Recall status / bank reference" (if bec_bank_recall = Yes); bec_third_parties textarea "Suppliers or customers impersonated or targeted"; bec_remediated yn "Have sessions been revoked and passwords reset for affected accounts?".

**Malware / Endpoint Compromise**
- mw_details "Malware details" (desc "What was found and where."): mw_family text "Suspected malware family / tooling" (placeholder "e.g. Qakbot, Cobalt Strike, infostealer"); mw_detection_name text "Detection name(s) from security tooling"; mw_sample yn "Is a sample available?"; mw_iocs textarea "Known IOCs (hashes, file paths, domains, IPs)"; mw_delivery multiselect "Suspected delivery vector" (Email attachment, Malicious link, Drive-by / SEO poisoning / malvertising, Removable media, Software supply chain, Exploit of public-facing service, Cracked / pirated software, Unknown); mw_host_count number "Number of hosts affected"; mw_host_roles multiselect "Roles of affected hosts" (User workstations, Privileged admin workstations, Servers, Domain controllers, Web servers, Developer machines, Cloud workloads); mw_hosts textarea "Affected hostnames" (placeholder "One per line"); mw_patient_zero text "Suspected patient zero (host / user)".
- mw_behaviour "Behaviour & spread" (desc "Post-compromise activity and current host state."): mw_edr_result select "Security tooling outcome" (Blocked, Detected, not blocked, Not detected, No EDR on host, Unknown); mw_c2 yn "Command and control traffic observed?"; mw_c2_details textarea "C2 indicators and timeframe" (if mw_c2 = Yes); mw_persistence multiselect "Persistence mechanisms identified" (Services, Scheduled tasks, Run keys, WMI subscriptions, Startup folder, Remote access tools (AnyDesk, etc.), None identified, Unknown); mw_cred_access yn "Indicators of credential access (LSASS dumping, browser credential theft)?"; mw_lateral yn "Indicators of lateral movement?"; mw_lateral_details textarea "Lateral movement details (protocols, accounts, hosts)" (if mw_lateral = Yes); mw_host_state select "Current state of affected hosts" (Running, connected; Running, isolated; Powered off; Reimaged; Mixed); mw_user_impact yn "Were affected users privileged, or did they access sensitive data?".

**Data Exfiltration**
- dx_data "Data & indicators of theft" (desc "What is believed to have left the environment and why we think so."): dx_how_known multiselect "Why is exfiltration suspected?" (Threat actor claim / leak site, Data found online, Extortion email, DLP alert, Unusual egress volume, Third-party notification, Forensic finding (e.g. rclone, archives), Other); dx_claim_details textarea "Claim / alert details"; dx_data_types multiselect "Data types believed exfiltrated" (DATA_TYPES); dx_source textarea "Source repositories (file shares, databases, SaaS)"; dx_volume text "Estimated volume (e.g. 120 GB, 40k files)"; dx_records number "Estimated number of data subjects / records"; dx_proof yn "Has the threat actor provided proof (file tree, samples)?"; dx_proof_validated yn "Has the proof been validated as genuine client data?" (if dx_proof = Yes).
- dx_channel "Exfiltration channel & timeline" (desc "How and when data moved, and what telemetry covers it."): dx_channels multiselect "Suspected exfiltration channel" (Cloud storage (MEGA, Dropbox, etc.), rclone / similar sync tool, FTP / SFTP, HTTP(S) upload, Email, C2 channel, Removable media, Personal cloud / SaaS account, Unknown); dx_tools text "Tools or destinations identified"; dx_window_start datetime "Exfiltration window start"; dx_window_end datetime "Exfiltration window end"; dx_egress_logs yn "Firewall / proxy logs available covering the window?"; dx_netflow yn "NetFlow or bandwidth data available?"; dx_cloud_logs yn "Cloud / SaaS audit logs available covering the window?"; dx_extortion yn "Is there an extortion demand?"; dx_extortion_details textarea "Extortion details (amount, deadline, contact)" (if dx_extortion = Yes); dx_affected_parties textarea "Third parties whose data may be involved".

**Insider Threat**
- it_subject "Subject & allegation" (desc "Handle with care: confirm need-to-know and HR / legal oversight before any action."): it_subject_role text "Subject role / department (avoid names unless required)"; it_subject_status select "Subject employment status" (Current employee, Serving notice, Departed, Contractor, Third-party / supplier staff); it_leave_date date "Leaving / departure date" (if it_subject_status in Serving notice, Departed); it_allegation select "Nature of allegation" (Data theft, Sabotage, Fraud, Unauthorised access, Policy violation, Harassment / misconduct (digital evidence), Other); it_allegation_details textarea "Allegation summary and trigger"; it_privileged yn "Does the subject hold privileged or admin access?"; it_subject_aware yn "Is the subject aware of the investigation?"; it_covert yn "Must the investigation remain covert?"; it_hr yn "Is HR engaged?"; it_legal yn "Is legal (internal or external) engaged?".
- it_evidence "Devices, evidence & constraints" (desc "Sources available and legal constraints on collection."): it_devices multiselect "Devices and accounts in scope" (Corporate laptop, Corporate desktop, Corporate mobile, Personal device (BYOD), Removable media, Corporate email, Corporate cloud storage, Collaboration / chat, Physical access records); it_devices_secured yn "Have relevant devices been secured?"; it_access_revoked yn "Has the subject's access been revoked or restricted?"; it_monitoring_policy yn "Is there an acceptable use / monitoring policy the subject was notified of?"; it_jurisdiction text "Employment jurisdiction(s)"; it_works_council yn "Works council or union consultation required?"; it_dlp yn "DLP / CASB / UEBA data available?"; it_outcome select "Anticipated outcome" (Disciplinary, Civil litigation, Criminal referral, Undecided); it_evidential_standard yn "Is the work expected to support expert witness / court proceedings?".

**Other**
- ot_details "Incident details" (desc "For incidents that do not fit the other categories."): ot_category select "Closest category" (Cloud account / tenant compromise, Web application compromise, Edge device exploitation (VPN, firewall), DDoS, Supply chain / third-party compromise, OT / ICS incident, Credential stuffing / account takeover, Lost or stolen device, Unknown, Other); ot_description textarea "Description of the incident"; ot_assets textarea "Key assets involved"; ot_indicators textarea "Known indicators or observables"; ot_vendor yn "Is a third-party vendor or supplier involved?"; ot_vendor_details textarea "Vendor details and their engagement so far" (if ot_vendor = Yes).

### 8.4 Closing section (all types)

**scope: "Objectives & scope"** (desc "What the client needs from us, so scope and estimate are right first time.")
- objectives | textarea | Key questions the client needs answered | help "e.g. How did they get in? Was data taken? Are they still in?"
- services | multiselect | Services requested | Forensic investigation, Threat hunting / compromise assessment, Containment & eradication support, Recovery support, Threat actor communications support, Dark web monitoring, Regulatory / legal reporting support, Executive briefing
- deliverables | multiselect | Expected deliverables | Verbal briefings, Daily status updates, Interim findings, Full technical report, Executive summary, IOC list, Remediation recommendations
- onsite | yn | Is on-site presence required?
- onsite_location | text | On-site location(s) | if onsite = Yes
- urgency | select | Urgency | Immediate (start today), Within 24 hours, Within the week, Planned
- budget | text | Budget or hours constraints
- next_steps | textarea | Agreed next steps and owners
- scoping_notes | textarea | Other scoping notes

### 8.5 DATA_TYPES list
Personal data (PII), Special category / health data (PHI), Payment card data (PCI), Credentials / secrets, Intellectual property, Financial records, Customer data, Employee / HR data, Legal / privileged material, None identified, Unknown.

***

## PART 9: Code organisation, quality and acceptance tests

### 9.1 Script sections (in this order, each with a banner comment)
1. Configuration (constants, incident types, timeline groups and event types, export palette, data types, QUESTIONNAIRE, phases, statuses, IOC types, ATT&CK lists, timezone list) with a header comment explaining the question schema and how to add an incident type.
2. Config index and question logic (question index, getSections, visibility, stats, timezone helpers, answer formatting).
3. Utilities (`$`, `$$`, `esc`, `uuid`, `nowISO`, `clone`, `errMsg`, date formatting, file slug, download).
4. Database layer.
5. State and autosave.
6. UI helpers (theme, banner, toast, modal, formatters).
7. Case metrics (request stats, asset stats, hours info, backup info, case flags).
8. Registers (helpers for IOCs and CSV, the `REGISTERS` definitions, `REGISTER_ORDER`).
9. Home rendering and search.
10. Case shell, wizard and summary.
11. Generic register views.
12. Timeline SVG and PNG export.
13. Report.
14. Export / import (JSON, CSV, Markdown, sanitise, merge).
15. Init (apply theme, wire top bar, open DB, banner on failure, footer text "IR Scoping v2.0.0 · schema 3 · storage: IndexedDB (this browser profile)", render home).

### 9.2 Acceptance tests (the result must pass all of these)
1. Opening the file shows the home screen with no console errors (a missing favicon is acceptable).
2. Create a Ransomware case for "ACME": the title defaults to "ACME - Ransomware - <today>" and the questionnaire opens on Engagement & client.
3. Answering "Yes" to legal privilege reveals the counsel question immediately without losing focus; reloading the page keeps the answer and the current section.
4. Setting the timezone to Europe/London and entering 2026-07-01 09:30 shows "= 2026-07-01 08:30 UTC"; 2026-12-01 09:30 shows 09:30 UTC; America/New_York 2026-10-07 09:30 shows 13:30 UTC.
5. Requests: a request due in the past with status Open is flagged Overdue, the tab reads "N open, 1 overdue", and the home row shows "1 overdue request". "Chased" quick action changes its status.
6. IOCs: entering `hxxps://evil[.]example[.]com/x` stores `https://evil.example.com/x` as type URL; adding `185.220.101[.]45` twice with different confidence and context results in one entry with the highest confidence and both contexts; `999.1.1.1` is rejected; raw CSV contains `185.220.101.45`, defanged CSV contains `185[.]220[.]101[.]45`.
7. Assets: adding DC01 then dc01 (both hosts) is blocked; tiles show correct compromised counts; privileged account with status Confirmed counts as privileged compromised.
8. Attack timeline: typing "T1566.002 Spearphishing Link" with no tactic selected sets Initial Access; host suggestions list the asset hosts; chart renders and "Export PNG" downloads a non-empty PNG.
9. Hours: with a 10h cap, adding 6h then 2.5h toasts the 85% warning; adding 3h more toasts the cap exceeded message; 30h is rejected.
10. Phase change persists; the home filter "Analysis" shows that case only; "Open cases" excludes closed ones.
11. Searching `adm_jsmith`, `evil[.]example`, `T1021` or a request description finds the case and shows where it matched.
12. A case created more than 3 days ago and never exported shows "Backup due" and the banner; "Export all now" clears it without changing the case's last-updated time.
13. JSON export then re-import with Merge combines entries without duplicates; a schema 1 file with only `{id, title, incidentType, status: 'complete', answers}` imports as phase Closed with empty registers; invalid register entries are dropped and reported.
14. Markdown export contains every questionnaire section, every non-empty register, defanged IOCs and the hours summary.
15. Report tab shows all sections in light colours; print preview hides the app chrome and tables continue cleanly across pages.
16. Theme toggle switches all colours (including the on-screen timeline chart) and is remembered after reload.
17. Everything works at 400px width (tables become cards, forms single column) and with the keyboard alone.

### 9.3 Deliverable
Return the complete `v2.html` in one code block. After the code, list any assumptions you made, and explain briefly how to add a new incident type, a new register field, and a new IOC type.
