# IR Scoping v3: Copilot prompt (all in one, data included)

Attach this one file to Copilot (plus `v3.html` once it exists). Everything is inside it: the instructions, every build step and ALL the question data. The data is in the **DATA** section at the end: one JSON block per file, each headed `DATA FILE: <path>`.

This file is generated from the JSON files in `copilot-v3/json/`. Edit the questions in ONE place: either the JSON files (then ask Claude to regenerate this file), or the JSON blocks in the DATA section of this file (then use UPDATE). Do not mix the two.

## For you: how to use this file

**To change questions in an existing v3.html (most common)**

1. Change the data (in the DATA section of this file, or in `json/` and regenerate this file).
2. In Copilot, attach this file and `v3.html`, and type: **"Follow the attached prompt file. Do UPDATE."**

**To build v3.html from scratch**

1. Attach this file and type: **"Follow the attached prompt file. Do STEP 01."** Save the returned `v3.html`.
2. For each next step, attach this file and `v3.html`, and type: **"Do STEP 02A."** (then 02B, 03 and so on).
3. Paste each returned code block directly under its marker in `v3.html` (for example under `// @@JS-DB`); keep the marker line. "REPLACE FUNCTION x" means swap the whole function.
4. After each step, open `v3.html` in Chrome or Edge and run the step's CHECK (F12 shows errors).
5. Start a new chat every three or four steps: attach this file and the current `v3.html`, and type **"Continue with STEP NN."**
6. If a reply is cut off, type: "Continue exactly where you stopped, inside the same code block."

| Step | What it does | Also attach |
|---|---|---|
| 01 | Design system and skeleton | nothing else |
| 02A | Question bank, part 1 | v3.html (data comes from the DATA section) |
| 02B | Question bank, part 2 (incident types) | v3.html (data comes from the DATA section) |
| 03 | Utilities, database, state, autosave, home screen | v3.html |
| 04 | Question logic | v3.html |
| 05 | Case shell, questionnaire wizard, summary | v3.html |
| 06 | Incident timezone and UTC | v3.html |
| 07 | Register engine, case timeline, chart and PNG | v3.html |
| 08 | Incident timeline (MITRE ATT&CK) | v3.html |
| 09 | IOC register and CSV export | v3.html |
| 10 | Affected assets register | v3.html |
| 11 | Time log (hours) | v3.html |
| 12 | JSON export / import and Markdown report | v3.html |
| 13 | Case phases, full search, backup reminder | v3.html |
| 14 | Print-ready report | v3.html |
| 15 | Analytics screen | v3.html |
| 16 | Incident types screen | v3.html |
| 17 | Report checklist | v3.html |
| 18 | Final review against acceptance tests | v3.html |
| UPDATE | Update the question bank from the DATA section (use any time) | v3.html |

VS Code: attach with `#file:` (for example `#file:copilot-v3/IR-Scoping-v3-Copilot-Prompt-All-In-One.md #file:v3.html`). Microsoft 365 Copilot: upload the files with the attach button.

***

## For Copilot: how to work with this file

1. This file is the whole specification. When I say "Do STEP NN" (or "Do UPDATE"), carry out ONLY that section, then stop and wait. Never start the next step on your own.
2. The CONTRACT below applies to every step.
3. All question data is in the DATA section at the end of this file (one ```json block per file, headed "DATA FILE: <path>"). Use it exactly; never invent or retype question data, and never ask me for JSON files. If v3.html is needed and not attached, ask for it.
4. Return only what the step asks for: code blocks labelled with their marker, or the complete file for STEP 01.

## CONTRACT

We are building a single-file web app called "IR Scoping v3". The contract below applies to every step.

WHAT IT IS
An offline case manager for an incident response (DFIR) consultant: pick an incident type, walk through a scoping questionnaire, then manage the case (case timeline, incident timeline mapped to MITRE ATT&CK, IOCs, affected hosts and accounts, time log in minutes) and print a report. v3 shares its saved cases with v1 and v2 (same database and record shape), so it must read and write exactly the data model below.

HARD RULES (apply to every phase)
1. One file, v3.html, inline CSS and vanilla JavaScript. No frameworks, libraries, CDNs, web fonts or network calls.
2. Must run by double-clicking the file (file:// URL) in current Chrome and Edge.
3. Case data is stored in IndexedDB only. localStorage is used only for the theme preference.
4. Never lose typed data: debounced autosave, and flush saves when the tab is hidden or closed.
5. Data-driven: every question, Ask line, report check and shared answer list comes ONLY from the question bank embedded in the page (step 02), built from the JSON files in json/questionnaire/ and json/incident-types/. Never retype that data in JavaScript. Other settings (statuses, colours, page size) live in config constants, not in logic.
6. 'use strict', const/let, small functions, clear comments, no clutter.
7. Escape all user text before putting it into HTML or SVG, using esc().
8. Never use the em dash character in UI text or generated output.
9. Never use alert(), confirm() or prompt(). Use the custom <dialog> modal.
10. In every step after step 01, return ONLY new or changed code, as separate code blocks. Start each block with a comment naming the marker it goes under, for example "// PASTE BELOW: // @@JS-DB" or "/* PASTE BELOW: @@CSS-HOME */". If an existing function must change, say "REPLACE FUNCTION name" and give the whole new function. Do not repeat unchanged code.

MARKERS (created in step 01, never removed)
HTML: <script type="application/json" id="question-bank"> sits directly before the main <script>; step 02 fills it.
CSS: @@CSS-BASE, @@CSS-HOME, @@CSS-WIZARD, @@CSS-REGISTERS, @@CSS-REPORT, @@CSS-PRINT
JS, in this order: @@JS-CONFIG-CORE, @@JS-CONFIG-QUESTIONS, @@JS-CONFIG-CASE, @@JS-UTILS, @@JS-DB, @@JS-STATE, @@JS-UI, @@JS-QUESTION-LOGIC, @@JS-TIMEZONE, @@JS-METRICS, @@JS-REGISTER-HELPERS, @@JS-REGISTERS (inside the REGISTERS object literal), @@JS-HOME, @@JS-SHELL, @@JS-WIZARD, @@JS-SUMMARY, @@JS-REGISTER-VIEW, @@JS-CHART, @@JS-REPORT, @@JS-EXPORT, @@JS-INIT

DATA MODEL (one IndexedDB record per case; database "ir-scoping", version 1, store "incidents", keyPath "id", index "updatedAt")
{
  id (UUID), title, clientName, incidentType (the primary type), extraTypes (array of additional type ids, may be empty),
  phase: 'scoping' | 'collection' | 'analysis' | 'reporting' | 'completed' | 'closed',
  status: 'draft' | 'complete'   (kept in sync: completed and closed mean complete),
  createdAt, updatedAt (ISO strings), currentStep (wizard section index),
  answers: { questionId: string | number | string[] },
  budget: { hoursCap: number or '', warnPct: number (default 80) },
  lastExportedAt: ISO string or null (set only by JSON export),
  checks: { reportCheckId: 'Yes' | 'No' | 'N/A' } (Report checklist answers; step 17),
  timeline: [], attack: [], iocs: [], assets: [], hours: []
}
Register items always have id and createdAt, plus updatedAt after an edit. Datetimes are "YYYY-MM-DDTHH:MM" strings in the incident timezone; dates are "YYYY-MM-DD".

SHARED NAMES (use exactly these so steps fit together)
Bank: BANK, LISTS, OPTION_SOURCES, bankSections, INCIDENT_TYPES, PHASES, SEVERITIES, COUNTRIES, DATA_TYPES, TIMELINE_GROUPS, TIMELINE_EVENT_TYPES, IOC_TYPES, IOC_CONFIDENCE, ASSET_STATUSES, ATTACK_TACTICS, ATTACK_TECHNIQUES, QUESTIONNAIRE, REPORT_CHECKS_COMMON, REPORT_CHECKS
Utilities: $, $$, esc, uuid, nowISO, clone, errMsg, fmtDateTime, fileSlug, dateStamp, download(filename, content, mime), isEmpty, normOptions
DB: DB.open, DB.getAll, DB.get, DB.put, DB.putMany, DB.delete, DB.mode ('idb' or 'memory')
State: state, markDirty(touch = true), commit(), flushSave(), setSaveStatus, normalizeRecord, newRecord, upsertLocal
UI: applyTheme, showBanner, toast(msg, kind, ms), openModal({title, body, buttons, onOpen}), confirmModal(title, message, label, danger)
Questions: QINDEX, getSections(type), getVal, setVal, isVisible, sectionStats, overallStats, formatAnswer(q, v, rec), typeLabel
Timezone: incidentTz, isValidTz, zonedToUtc, utcHint, fmtUtc, fmtLocal, fmtDate, nowLocalString, todayLocal, utcOf
Views: renderHome, renderList, openIncident, renderShell(tab, html), refreshTabs, openTab, updateHeader, renderWizard, renderSummary, renderRegister(key), renderReport
Registers: REGISTERS (object keyed by register key), REGISTER_ORDER
Chart/export: buildTimelineSVG, exportChartPNG, exportIncidentsJSON, exportAllJSON, toMarkdown, exportMarkdown

***

## STEP 01: Design system and skeleton

STEP 01: Design system and skeleton. Return the COMPLETE v3.html for this step only.

Create the full page structure with all markers from the contract, the complete design system CSS, and a working top bar with a theme toggle. JS sections that later phases fill should contain just their marker comment (plus anything listed below).

PAGE STRUCTURE
<head>: charset, viewport, <title>IR Scoping v3</title>, a tiny inline script that reads localStorage key "ir-scoping-theme" (in try/catch) and sets data-theme on <html> before first paint, then <style>.
<body>:
- header.topbar: brand button #brandHome (26px rounded square mark with "IR", gradient 135deg from accent to #8a5cf6, white bold text; then "IR Scoping" and a small accent "v3"), spacer, span#saveStatus (aria-live polite), ghost button #themeToggle.
- div#dbBanner.banner (role alert, hidden)
- main#app (tabindex -1)
- footer#appFoot.app-foot
- dialog#modal.modal
- div#toasts.toasts (aria-live polite)
- input#importFile type file, accept ".json,application/json", hidden
- <script type="application/json" id="question-bank">{}</script> (empty for now; step 02 fills it)
- <script> with 'use strict' and the JS markers in contract order. In the @@JS-REGISTERS area write: const REGISTERS = { // @@JS-REGISTERS  }; and below it: const REGISTER_ORDER = ['timeline','attack','iocs','assets','hours'].filter(k => REGISTERS[k]);

THEME TOKENS
Define each token as a CSS custom property on :root and [data-theme="dark"] (default), overridden on [data-theme="light"].
Dark: bg #0e1319, surface #151c25, surface-2 #1c2531, surface-3 #243041, border #2b3747, text #e4ebf2, muted #8f9db0, accent #4f9cf9, accent-strong #7ab4ff, accent-contrast #06121f, danger #f0616d, success #3fb97a, warn #e3b341, focus #7ab4ff, shadow "0 8px 30px rgba(0,0,0,.45)", color-scheme dark.
Light: bg #f3f5f8, surface #ffffff, surface-2 #f6f8fb, surface-3 #e9eef5, border #d3dbe6, text #17202b, muted #5b6878, accent #1d64d8, accent-strong #134ea9, accent-contrast #ffffff, danger #c7323f, success #1e8a55, warn #a5710a, focus #1d64d8, shadow "0 8px 30px rgba(20,30,50,.15)", color-scheme light.

BASE CSS (under @@CSS-BASE)
- box-sizing border-box; [hidden] { display:none !important }.
- Body: bg token, text token, font 15px/1.5 "Segoe UI", system-ui, -apple-system, Roboto, "Helvetica Neue", Arial, sans-serif.
- h1 1.45rem, h2 1.2rem, line-height 1.25, margin 0. Classes .muted, .small (0.85rem), .sr-only.
- :focus-visible outline 2px focus token, offset 2px, radius 4px. kbd: monospace 12px, 1px border, 2px bottom border, radius 4px, surface-2.
- Top bar: sticky top 0, z-index 20, flex, gap 12px, padding 10px 20px, surface bg, bottom border.
- Save status 0.85rem muted; data-state "saved" shows an 8px green dot before the text, "pending"/"saving" an amber dot, "error" danger bold, "memory" warn.
- Banner: padding 12px 20px, background danger mixed 18% into surface, danger bottom border.
- main: max-width 1240px, centred, padding 24px 20px 64px. Footer: same width, 0.8rem muted.
- Buttons .btn: inline-flex centred, min-height 38px, padding 7px 14px, surface-2 bg, border, radius 8px, weight 550, hover surface-3. Variants: .primary (accent bg/border, accent-contrast text, hover accent-strong), .danger (danger text, border mixed 50% danger), .danger.solid (danger bg, white text), .success (success text), .small (min-height 32px, padding 4px 10px, 0.85rem), .ghost (transparent). Disabled opacity .45. .link-btn: unstyled text button, weight 600, hover accent-strong underline.
- Inputs (text, search, number, date, time, datetime-local, email), select, textarea: width 100%, min-height 40px, padding 8px 11px, surface-2 bg, border, radius 8px, inherit font. Textarea min-height 96px, vertical resize. Focus: outline 2px focus at offset 0 and focus border.
- .card: surface bg, 1px border, radius 12px.
- .badge: pill, padding 2px 9px, 0.78rem, weight 650, uppercase. .badge.type: surface-3 bg, normal case. .badge.tint: background = color-mix(in srgb, var(PC) 22%, transparent), text = color-mix(in srgb, var(PC) 70%, var(text)), where PC is a custom property set inline per badge.
- .flag (tiny pill 0.72rem weight 650) with .warn and .danger tints (20% mix). .chip (0.8rem pill with border, muted) and .chip.warn.
- .notice (flex row, padding 10px 14px, radius 8px, border) and .notice.warn (warn 50% border, warn 10% into surface bg).
- .stat-grid: grid repeat(auto-fill, minmax(150px, 1fr)) gap 10px. .stat: padding 12px 14px, border, radius 10px, surface-2. .stat .num 1.55rem bold tabular-nums; .lbl 0.82rem muted 600; .sub 0.78rem muted. .stat.warn/.danger colour the number.
- .empty-state: centred muted, padding 48px 20px.
- Toasts: fixed bottom-right 18px, grid gap 8px, max-width min(420px, 100vw minus 36px). .toast: padding 11px 14px, radius 10px, surface-3, border, 4px left border (accent; .success, .warn, .error variants), shadow, 0.18s slide-up animation.
- Modal: dialog width min(560px, 100vw minus 32px), padding 0, surface, border, radius 14px, shadow; backdrop rgba(5,10,18,.6) with 2px blur. Inner form padding 20px 22px; .modal-body grid gap 14px; label.f grid gap 5px bold; .modal-actions flex, gap 8px, margin-top 20px; ul.msgs max-height 220px scroll; .opt radio cards (border, radius 8px, checked = accent border and 10% tint).
- Table list (table.list): full width, collapsed; cells padding 11px 14px, bottom border; th 0.78rem uppercase muted 650; row hover surface-2; .row-actions flex wrap gap 6px right-aligned. At max-width 820px: hide thead, rows and cells become blocks, each cell shows its data-label as a muted prefix (except .title-cell).
Leave the other CSS markers empty.

JS FOR THIS PHASE
- Under @@JS-UTILS: $ and $$ (querySelector helpers), esc (escape & < > " ').
- Under @@JS-UI: applyTheme(theme) sets data-theme, sets the toggle text to "Light mode" in dark theme and "Dark mode" in light theme, saves to localStorage in try/catch. toast(msg, kind = 'info', ms = 3500).
- Under @@JS-INIT: an init function that applies the current theme, wires #themeToggle, and puts a placeholder in #app: a card saying "Skeleton ready. Next phases add features." plus three sample buttons (primary, default, danger) and three badges, so the styling can be checked. Footer text: "IR Scoping v3.0.0".

CHECK
Opening the file shows the dark top bar and sample card; the theme toggle switches all colours and is remembered after reload; no console errors.

## STEP 02A: Question bank, part 1

STEP 02A: Question bank, part 1 (lists, common sections, closing sections, common report checks). Return code blocks only.

The DATA section at the end of this file is the ONLY source of questions, Ask lines, report checks and answer lists. Copy them exactly; do not reword, reorder, add or drop anything, and do not convert them to JavaScript.

1. Replace the empty placeholder <script type="application/json" id="question-bank">{}</script> with a JSON object of this exact shape (every value is the JSON block of that DATA FILE):
{
  "lists": <json/questionnaire/lists.json>,
  "common": <json/questionnaire/common-sections.json>,
  "closing": <json/questionnaire/closing-sections.json>,
  "reportChecksCommon": <json/questionnaire/report-checks-common.json>,
  "types": {}
}
Write each value on one line (compact JSON) to keep the file small. If any text contains "</", write it as "<\/".

2. Under @@JS-CONFIG-QUESTIONS paste this loader exactly as written:
/* ---------------- Question bank (embedded JSON) ---------------- */
const BANK = JSON.parse(document.getElementById('question-bank').textContent);
const LISTS = BANK.lists;

const INCIDENT_TYPES = LISTS.incidentTypeOrder.map(id => ({ id, label: BANK.types[id].incidentType.label }));
const PHASES = LISTS.phases;
const SEVERITIES = LISTS.severities;
const COUNTRIES = LISTS.countries;
const DATA_TYPES = LISTS.dataTypes;
const TIMELINE_GROUPS = LISTS.timelineGroups;
const TIMELINE_EVENT_TYPES = LISTS.timelineEventTypes;
const IOC_TYPES = LISTS.iocTypes;
const IOC_CONFIDENCE = LISTS.iocConfidence;
const ASSET_STATUSES = LISTS.assetStatuses;
const ATTACK_TACTICS = LISTS.attackTactics;
const ATTACK_TECHNIQUES = LISTS.attackTechniques;

/* Shared lists a question can use with "optionsFrom". */
const OPTION_SOURCES = {
  PHASES: () => PHASES.map(p => ({ value: p.id, label: p.label })),
  INCIDENT_TYPES: () => INCIDENT_TYPES.map(t => ({ value: t.id, label: t.label })),
  SEVERITIES: () => SEVERITIES,
  COUNTRIES: () => COUNTRIES,
  DATA_TYPES: () => DATA_TYPES,
};

function bankSections(sections) {
  return sections.map(s => ({ ...s, questions: s.questions.map(q => {
    if (!q.optionsFrom) return q;
    const { optionsFrom, ...rest } = q;
    if (!OPTION_SOURCES[optionsFrom]) console.warn(`[config] "${q.id}" uses unknown optionsFrom "${optionsFrom}"`);
    return { ...rest, options: OPTION_SOURCES[optionsFrom] ? OPTION_SOURCES[optionsFrom]() : [] };
  }) }));
}

const QUESTIONNAIRE = {
  commonSections: bankSections(BANK.common.sections),
  typeSections: Object.fromEntries(LISTS.incidentTypeOrder.map(id => [id, bankSections(BANK.types[id].sections)])),
  closingSections: bankSections(BANK.closing.sections),
};

const REPORT_CHECKS_COMMON = BANK.reportChecksCommon.reportChecks;
const REPORT_CHECKS = Object.fromEntries(LISTS.incidentTypeOrder.map(id => [id, BANK.types[id].reportChecks || []]));

3. The page will log errors until step 02B adds the incident types (BANK.types is empty); that is expected.

CHECK
The question-bank block is valid JSON (paste it into any JSON validator if unsure) and contains the four keys plus an empty "types".

## STEP 02B: Question bank, part 2 (incident types)

STEP 02B: Question bank, part 2 (incident types). Return only the changed "types" part of the question bank.

Add every json/incident-types/<id>.json DATA FILE to the question bank in v3.html, inside "types", keyed by the file's incidentType.id. If the reply would be too long, do four or five types per reply and say which are left; I will reply "continue":
"types": {
  "<id>": { "incidentType": {...}, "sections": [...], "reportChecks": [...] },
  ...
}
Copy each file exactly (compact JSON, one type per line). Do not reword, reorder, add or drop anything. Keep any types already in the bank.
Put the types in the order of lists.incidentTypeOrder in the bank.

CHECK (after the last batch)
Every id in lists.incidentTypeOrder has an entry in "types". Open v3.html: no console errors about BANK.types. In the console, QUESTIONNAIRE.typeSections has one key per incident type.

## STEP 03: Utilities, database, state, autosave, home screen

STEP 03: Utilities, database, state, autosave and the home screen. Return only code blocks per marker (contract rule 10).

@@JS-CONFIG-CORE
APP_VERSION '3.0.0', SCHEMA_VERSION 3, DB_NAME 'ir-scoping', DB_VERSION 1, STORE 'incidents', THEME_KEY 'ir-scoping-theme', AUTOSAVE_DELAY 400.
INCIDENT_TYPES, PHASES, SEVERITIES and COUNTRIES already exist (step 02 builds them from the question bank). Add TYPE_IDS = INCIDENT_TYPES.map(t => t.id) and typeLabel(id) returning the label, "Unknown (id)" or "Not set".
ALL_REGISTER_KEYS = ['timeline','attack','iocs','assets','hours'] (used by normalizeRecord even before registers exist).

@@JS-UTILS (add)
uuid (crypto.randomUUID with a getRandomValues v4 fallback), nowISO, clone (structuredClone with JSON fallback), errMsg, fmtDateTime(iso) en-GB "07 Oct 2026, 09:15", fileSlug (lowercase, non-alphanumerics to single hyphens, max 60, default "incident"), dateStamp (YYYY-MM-DD), isEmpty (undefined, null, '' or empty array), normOptions (strings become {value, label}), download(filename, content, mime): Blob, object URL, hidden <a download>, click, revoke after 1.5s.

@@JS-DB
Promise-based DB object: open() resolves true or false. On missing IndexedDB or open error, switch to an in-memory Map (mode 'memory'), keep the error in DB.error. onupgradeneeded creates the store and the updatedAt index. onversionchange closes the DB and calls showBanner("Database was upgraded in another tab. Reload this page to continue saving."). A private _tx(mode, fn) runs fn(store) and resolves with the request result on transaction complete, rejecting on error or abort. Methods getAll, get, put, putMany (single transaction), delete, each with the memory fallback (store clones).

@@JS-STATE
state = { view: 'home', activeTab: null, incidents: [], current: null, search: '', phaseFilter: 'all', dirty: false, touched: false, saveTimer: null, saveChain: Promise.resolve(), saveErrorShown: false, regEditId: null }.
newRecord({title, clientName, incidentType}) following the data model (phase 'scoping', status 'draft', empty registers, budget {hoursCap: '', warnPct: 80}, lastExportedAt null).
normalizeRecord(rec): ensure answers object and every ALL_REGISTER_KEYS array; derive phase from status if missing ('complete' becomes 'closed', else 'scoping'); sync status from phase; default budget; invalid lastExportedAt becomes null. Returns rec.
upsertLocal(rec): put a normalised clone into state.incidents.
markDirty(touch = true): set dirty (and touched if touch), status "pending", debounce flushSave by AUTOSAVE_DELAY.
commit(): set dirty and touched, return flushSave().
flushSave(): if nothing dirty return state.saveChain. Otherwise clear dirty; if touched, set updatedAt = now and clear touched; snapshot with clone; chain DB.put onto state.saveChain; on success upsertLocal and status "saved"; on failure set dirty again, status "error", toast once "Could not save: <error>. Retrying. Export to JSON if this persists.", retry after 5 seconds.
setSaveStatus(s): blank when no case is open; states pending "Unsaved changes", saving "Saving...", saved "Saved HH:MM:SS", error "Save failed", memory "Held in memory only" (used instead of saved in memory mode).
Listeners: visibilitychange (hidden) and pagehide call flushSave; beforeunload calls flushSave and, in memory mode with cases present, asks to confirm leaving.

@@JS-UI (add)
showBanner(html). openModal({title, body, buttons, onOpen}) returning Promise<{value, form}>: renders a <form method="dialog"> with h2, .modal-body and .modal-actions. Buttons: {label, value, kind, cancel, autofocus}. Submit buttons come first in the DOM and the actions row uses flex-direction row-reverse with justify-content flex-start (so Enter triggers the primary button and it still appears on the right). Cancel buttons are type="button" and close with ''. Escape cancels. confirmModal(title, message, confirmLabel = 'Confirm', danger = false) resolves true/false; when danger, autofocus Cancel and use 'danger solid' for the confirm button.

@@CSS-HOME
.home-head (flex wrap, space-between, align end, gap 16px, margin-bottom 18px), .actions (flex wrap gap 8px), .search-row (flex gap 12px, input max-width 460px), .phase-filter, .pf chips (pill buttons min-height 34px, border, surface bg, 0.88rem 600; .active = accent bg and accent-contrast text; .pf-n count 0.75rem at 75% opacity), .flags (flex wrap gap 4px, margin-top 4px), .match (small, margin-top 3px; mark = warn 40% tint).

@@JS-HOME
renderHome(): flush saves, set view 'home', clear current, load state.incidents = (await DB.getAll()).map(normalizeRecord) (toast on error). Render: h1 "Incidents", subtitle "Cases saved in this browser.", buttons "Import JSON" (#btnImport clicks #importFile), "Export all" (#btnExportAll, for now a toast "Arrives in step 12"), "New incident" (primary). Then div#backupBanner (empty for now), div#phaseFilter (empty for now), search input #search (placeholder "Search everything: case IDs, answers, events, IOCs, hosts...") with span#listCount "N of M" (typing re-renders the list after a 150 ms pause, debounced with state.searchTimer), and div#list.card. Document title "IR Scoping v3".
renderList(): filter by search (case-insensitive over title, client, type label for now), sort by updatedAt newest first. Empty states: "No incidents yet" / "Start a new case, or import a JSON export." and "No incidents match ...". Table columns (only these): Case ID (link-btn data-act open showing answers.engagement_ref, falling back to the title for older cases), Incident type, Customer, Severity, Case logged ("07 Oct 2026" style, followed by an age tag: ageDays(date) = whole calendar days from the logged date to today, shown by ageTagHTML as a small grey pill "0 days" / "1 day" / "12 days", "Future date" if the date is after today, nothing if there is no date; title "12 days since the case was logged"; CSS .age-tag: inline-block, margin-left 6px, padding 1px 8px, radius 999px, surface-3 background, text colour, 0.75rem, 650, tabular-nums), actions (kept on one line on screens wider than 820px) as 34px square icon buttons with inline SVG icons, title and aria-label: arrow "Open case", braces "Export JSON", document "Export Markdown report", bin "Delete case" (danger). Show HOME_PAGE_SIZE = 10 rows per page with a pager under the table ("Showing 11 to 20 of 34", Previous, page numbers with "…" gaps, Next; hidden when there is only one page); searching or changing the phase filter returns to page 1. Cells carry data-label. Use event delegation on #list.
New incident modal fields: Incident types (fieldset "Incident types (tick all that apply)" with one checkbox per INCIDENT_TYPES entry, name "types", one per row in a bordered scrolling list, max-height 300px; at least one required, enforced with setCustomValidity "Tick at least one incident type." on the first checkbox; the first ticked type in list order is saved as rec.incidentType, the rest as rec.extraTypes), Case ID (placeholder "Internal case / matter reference"), Case logged date (date picker, defaults to today), Customer, Geolocation (select, "Select country..." then COUNTRIES), Severity (select, "Select severity..." then SEVERITIES). The title is the Case ID on its own (never combined with other values), or "Untitled incident" when there is no Case ID. Customer is saved as clientName; Case ID, Case logged date, Geolocation and Severity are saved as answers engagement_ref, case_logged, geolocation and severity (only when filled in). Save with DB.put, upsertLocal, then openIncident(id) (for now openIncident just toasts "Opened <title>"; step 05 replaces it).
Delete: confirmModal("Delete incident?", "This permanently deletes <b>title</b> (client) from this browser. Export it first if you may need it.", "Delete", true).
JSON and Markdown buttons: toast "Arrives in step 12" for now.

@@JS-INIT (replace the step 01 placeholder)
init: apply theme, wire theme toggle and #brandHome (renderHome), await DB.open(); if false, showBanner("<strong>Warning: IndexedDB is unavailable.</strong> <error> Data is held in memory only and will be lost when this tab closes. Export to JSON before closing. Common causes: private / InPrivate window, browser storage disabled, or strict site-data settings."); else call navigator.storage.persist() ignoring errors. Footer: "IR Scoping v3.0.0 · schema 3 · storage: IndexedDB (this browser profile)" or "memory only". Then renderHome().

CHECK
Create and delete cases; they survive a browser restart; search filters the list; at 400px width the table becomes cards.

## STEP 04: Question logic

STEP 04: Question logic. Return code blocks per marker.

The questions themselves already come from the question bank (step 02): QUESTIONNAIRE.commonSections, .typeSections and .closingSections. Each question may carry ask (spoken prompt), optionsFrom (already resolved to options by bankSections), help, placeholder, suggestions, showIf, bind, row and required. Do not add question data in JavaScript.

Question schema reminder:
Question schema: { id (unique across ALL sections), label, type, options?, help?, placeholder?, suggestions?, bind?, showIf?, required?, row? }
row: consecutive questions with the same row key render side by side in one row (one equal column each).
Types: text, textarea, number, date, time, datetime, select, multiselect, yesno (Yes/No/Unknown, or the question's own options list such as ['Yes', 'No']), timezone.
bind: store on the record instead of answers ('title', 'clientName', 'phase', 'types').
showIf: { q, equals } | { q, in: [...] } | { q, notEquals } | { q, notEmpty: true }.
Section: { id, title, description, questions, group?, short? }. group overrides the default heading (e.g. 'Containment'); short is the name used in the sidebar, jump list, section heading and Summary (title is still used in the report, Markdown and progress line).

@@JS-QUESTION-LOGIC
- QINDEX: Map of every question across common, all type sections and closing sections; console.warn on duplicate ids and on showIf pointing to unknown ids.
- caseTypes(rec): [primary, ...extraTypes] without repeats or blanks; caseTypesLabel(rec): their labels joined with " + ".
- getSections(arg): arg is a case (use caseTypes) or a single type id. Returns common (group "Common") + each type's typeSections in that order (group = that type's label; a section's own group, e.g. Containment, wins) + closing (group "Scope"), each section copied with a group property. Pass the case everywhere (getSections(rec)).
- getVal(rec, q) / setVal(rec, q, v): use rec[q.bind] when bound; empty answers are deleted from answers (bound fields become ''). bind 'types' is special: getVal returns caseTypes(rec); setVal keeps rec.incidentType if it is still ticked (otherwise the first ticked type, or '') and stores the other ticked types in rec.extraTypes.
- conditionMatches(value, cond): arrays match if any element matches.
- isVisible(rec, q): true without showIf; false if the controller is hidden; otherwise conditionMatches.
- sectionStats(rec, s) => {total, answered} over visible questions; overallStats(rec) sums all sections.
- formatAnswer(q, v, rec): null when empty; selects and multiselects show option labels (multiselect joined with ", "); datetime shows "YYYY-MM-DD HH:MM" for now (step 06 adds the timezone).

CHECK
In the console: getSections('ransomware').length is 17, there are no duplicate-id warnings, and isVisible works for prior_incidents_details after setting answers.prior_incidents = 'Yes'.

## STEP 05: Case shell, questionnaire wizard, summary

STEP 05: Case shell, questionnaire wizard and summary. Return code blocks per marker.

@@JS-CONFIG-CASE (add)
PHASES ({id, label, color}; ids are stored on cases) comes from the question bank (step 02); do not redefine it. DONE_PHASES = [completed, closed] (finished: status complete, excluded from Open cases). phaseOf(id).

@@CSS-WIZARD
- .inc-head: flex wrap, space-between, gap 12px 20px, margin-bottom 16px; .meta row (flex wrap gap 8px, muted); .actions buttons.
- .phase-pick: inline-flex label "Phase" + small select (auto width, min-height 30px, padding 3px 8px, left border 4px coloured by the phase).
- .tabs: flex wrap, gap 2px, bottom border, margin-bottom 18px. .tab: borderless button, 2px transparent bottom border, muted 600, padding 10px 12px; hover text colour; .active = accent-strong text, accent bottom border. .tab-badge: pill min-width 20px, padding 0 7px, surface-3, 0.74rem 650; .warn and .danger variants with 25% tints.
- .progress-meta (flex space-between, 0.88rem muted), .progress-bar (8px pill, surface-3) with inner span gradient accent to #8a5cf6, width transition.
- .wiz-body: grid 280px 1fr, gap 20px. .sec-nav: sticky top 72px, max-height viewport minus 92px, scroll, padding 10px. .nav-group: 0.74rem uppercase muted 700. .nav-item: full-width row, label left, count right (0.76rem muted); hover surface-2; .active = accent 18% tint, accent-strong text; .done count green with a check mark prefix.
- .jump-wrap (hidden on wide screens). .sec-form padding 22px 24px; header with group label (0.78rem uppercase accent 700), h2, description muted, bottom border.
- .q-row: grid, grid-template-columns repeat(var(--cols), minmax(0, 1fr)), gap 0 16px; date inputs inside have no max-width; 1 column below 700px.
- .q (margin-bottom 20px), labels and legends 600, .help 0.85rem muted, .q.conditional (padding-left 14px, 2px left border accent 45%).
- .q .ask: flex row, gap 8px, baseline aligned, margin -1px 0 8px, 0.95rem, italic, accent-strong colour. .ask-tag: not italic, 0.68rem bold uppercase, letter-spacing 0.5px, padding 1px 6px, radius 4px, background accent mixed 16% with transparent, accent-strong text.
- Choice cards .choice-grid repeat(auto-fill, minmax(230px, 1fr)); .choice with checkbox; checked = accent border + 12% tint.
- Segmented yes/no .seg (three radio labels, min-height 38px, padding 6px 18px, checked = accent bg, accent-contrast text) and .clear-btn (underlined muted link).
- .wiz-foot (flex space-between, top border, padding-top 16px), .kbd-hint 0.8rem muted.
- Summary: .sum-sec cards (padding 16px 20px), header row (h2 1.05rem, count, Edit button), dl grid minmax(200px, 38%) 1fr with dashed row separators, dd pre-wrap, .empty italic.
- At 900px and below: single column, hide .sec-nav, show .jump-wrap. At 700px summary dl stacks.

@@JS-SHELL
- incidentHeaderHTML(): h1#incTitle; meta: badge#incType (caseTypesLabel, all of the case's types), span#incClient, badge#incTz, label.phase-pick with select#phaseSelect (options from PHASES), span#backupChip.chip; actions: "All incidents" #btnHome, "Export JSON" #btnJson, "Export Markdown" #btnMd (toast "Arrives in step 12" until then).
- updateHeader(): fills title (or "Untitled incident"), type label, client (or "No client set"), timezone badge "Timezone not set" (step 06 completes it), phase select value and its border colour, document title "<title> - IR Scoping v3". Leave the backup chip empty until step 13.
- renderShell(tab, innerHtml): sets state.activeTab, clears app.onclick, renders header + nav#tabs + div#view, wires header buttons, phase select (setPhase), tab clicks (openTab), scrolls to top.
- refreshTabs(): tabs Questionnaire (badge answered/total), Summary, then one tab per REGISTER_ORDER key (label REGISTERS[k].tab, badge from REGISTERS[k].tabBadge(rec) if defined, else the item count, hidden when 0), then Report (only if renderReport exists). Active tab gets aria-current="page".
- openTab(tab): flushSave then route to renderWizard(true), renderSummary(), renderReport() or renderRegister(tab).
- setPhase(id): set phase and status, commit(), updateHeader(), toast "Phase set to X."
- REPLACE FUNCTION openIncident(id): load, normalizeRecord, clamp currentStep, set state.current, status saved, renderWizard(true).

@@JS-WIZARD
- renderWizard(focusFirst): renderShell('wizard', ...) with progress area, left sidebar nav#secNav, jump select (optgroups by group), and form#secForm (novalidate) containing the section header, the questions (via renderQuestions(rec, questions): consecutive questions sharing a row key are wrapped in div.q-row with style --cols set to their count; others render alone) and the footer (Back disabled on the first section; Next, or "Review summary" on the last), and the hint "Enter in a single-line field: next section. Ctrl + Enter: next from anywhere."
- refreshChrome(): rebuild sidebar (group headings, "answered/total" counts, done and active states), jump select, progress text "Section 3 of 10: <title>" and "X of Y questions answered (Z%)", progress bar width, and refreshTabs().
- renderField(rec, q): wrapper div.q[data-qid] (hidden when not visible, .conditional when it has showIf). Label for single inputs; fieldset + legend for multiselect and yesno. Directly under the label, when q.ask exists, show p.ask: a small tag span.ask-tag "ASK" followed by the spoken prompt. Then the optional help paragraph linked with aria-describedby. Controls: textarea rows 4; number (min 0, step any); date; time; datetime (datetime-local); select with "Select..." first, plus a "(not in current list)" option for unknown stored values; multiselect checkbox cards; yesno segmented radios with a Clear button shown when answered; text with optional datalist from q.suggestions; timezone renders a plain select for now (step 06 completes it).
- readField(wrap, q) for each type (multiselect returns array, number returns Number or '').
- Input and change events on the form: setVal, markDirty(), updateHeader when the question is bound, toggle conditional visibility in place (do not re-render; focus must stay), refreshChrome. Clear button resets the radios.
- Keyboard: Enter in single-line inputs (text, number, date, time, datetime-local) without a datalist goes Next; Ctrl or Cmd + Enter anywhere in the wizard goes Next (global keydown, ignored when the modal is open).
- goToStep(i): set currentStep, markDirty(false) (navigation does not change updatedAt), flushSave, renderWizard(true). goNext(): last section goes to renderSummary().
- Focus the first visible field only when matchMedia('(pointer: fine)') matches.

@@JS-SUMMARY
renderSummary(): renderShell('summary', ...): a muted line "X of Y questions answered. Created <date>, last updated <date>." then one .sum-sec card per section: "<title> (<group>)", "answered/total", Edit button (data-step) that calls goToStep; definition list of visible questions with formatAnswer or "Not answered". Set app.onclick after renderShell to handle the Edit buttons.

CHECK
New incident opens the questionnaire; conditional questions appear without losing focus; Enter moves on; refresh keeps the answers and section; sidebar counts and tab badge update live; Summary Edit jumps to the right section; the phase select persists.

## STEP 06: Incident timezone and UTC

STEP 06: Incident timezone and UTC conversion. Return code blocks per marker and any replaced functions.

@@JS-CONFIG-CORE (add) TZ_QUESTION_ID = 'discovery_tz'.
@@JS-TIMEZONE
- TIMEZONES: ['UTC', ...Intl.supportedValuesOf('timeZone')] de-duplicated, with a short fallback list (UTC, Europe/London, Europe/Dublin, Europe/Paris, Europe/Berlin, Europe/Helsinki, America/New_York, America/Chicago, America/Denver, America/Los_Angeles, Asia/Dubai, Asia/Kolkata, Asia/Singapore, Asia/Tokyo, Australia/Sydney).
- BROWSER_TZ from Intl.DateTimeFormat().resolvedOptions().timeZone.
- incidentTz(rec), isValidTz(tz) (try new Intl.DateTimeFormat with timeZone).
- tzOffsetMs(ms, tz) via formatToParts (hourCycle h23); zonedToUtc(local, tz): parse "YYYY-MM-DDTHH:MM", guess UTC, subtract the offset, then re-check the offset at the result and correct (two passes, correct across DST). Return null when invalid.
- fmtUtc(date) "YYYY-MM-DD HH:MM UTC"; fmtOffset(ms) "UTC+01:00".
- utcHint(rec, value): no timezone: "Set the incident timezone (Case summary) to see UTC."; invalid: "Unrecognised timezone "X". Pick one from the list to see UTC."; empty value: "Enter in Europe/London."; else "Europe/London (UTC+01:00) = 2026-10-07 08:30 UTC".
- fmtLocal("2026-10-07T09:30") => "07 Oct 2026 09:30" and fmtDate("2026-10-07") => "07 Oct 2026" using string parsing only (never a Date object, to avoid shifting).
- nowLocalString(tz) "YYYY-MM-DDTHH:MM" for now in that zone (browser zone if unset or invalid); todayLocal(tz); utcOf(rec, local) returns a Date or null.

REPLACE FUNCTION renderField: datetime renders a .dt-row with the input and span.utc-hint[data-utc-for] showing utcHint. timezone renders a select ("Select timezone..." then all TIMEZONES, plus "X (not a recognised timezone)" for unknown stored values) and a small button "Use browser timezone (<zone>)" that sets the select and fires change.
REPLACE the form input handler so that changing the timezone question or any datetime refreshes every UTC hint in the section and updates the header timezone badge.
REPLACE FUNCTION formatAnswer: timezone shows "Europe/London (currently UTC+01:00)"; datetime shows "2026-10-07 09:30 Europe/London (2026-10-07 08:30 UTC)" or "2026-10-07 09:30 (timezone not set)".
REPLACE FUNCTION updateHeader so #incTz shows incidentTz(rec) or "Timezone not set".
CSS (under @@CSS-WIZARD): .dt-row flex wrap gap 8px 14px align centre; .dt-row select max-width 340px; .utc-hint 0.88rem muted tabular-nums; date/time inputs max-width 280px.

CHECK
Europe/London: 2026-07-01 09:30 => 08:30 UTC; 2026-12-01 09:30 => 09:30 UTC. America/New_York 2026-10-07 09:30 => 13:30 UTC. Asia/Kolkata 09:30 => 04:00 UTC.

## STEP 07: Register engine, case timeline, chart and PNG

STEP 07: Generic register engine, the case timeline register, the SVG timeline chart and PNG export. Return code blocks per marker.

REGISTER DEFINITION SHAPE
{ tab, title, singular, intro, fields: [...], sort(a, b), columns: [...], chart?, panel?(rec), wirePanel?(el), normalize?(item, rec) => {item} or {error}, dedupeKey?(item), merge?(a, b), rowActions?: [{id, label, show(item), apply(item)}], rowClass?(item, rec), toolbar?: [{id, label, run(rec)}], tabBadge?(rec) => {text, warn}, afterChange?(rec, before) }
Field: { id, label, type: text|textarea|number|date|datetime|select, required, default: 'now'|'today'|value, options or groups(rec) (optgroups), suggestions(rec) (datalist), placeholder, wide, step, onChange(value, form, rec) }
Column: { label, text(item, rec), html?(item, rec), exportText?(item, rec), cls? }  (text for search and Markdown, html for screen, exportText for sharing such as defanged values)

@@JS-CONFIG-CASE (add)
TIMELINE_GROUPS ({ groupId: {label, color} }) and TIMELINE_EVENT_TYPES ([{id, label, group}]) come from the question bank (step 02); do not redefine them.
TIMELINE_EXPORT_PALETTE: bg #ffffff, surface #f6f8fb, border #d3dbe6, text #17202b, muted #5b6878, line #c3cdd9.

@@JS-REGISTER-HELPERS
whenText(rec, local) "07 Oct 2026 09:30 (2026-10-07 08:30 UTC)"; whenHTML (UTC on a second muted line); dot(color) small coloured circle span; tintBadge(label, color); optOf(options, value); byDatetime sorter; tlType(id); tlGroup(id).

@@JS-REGISTERS: entry timeline
tab "Case timeline", title "Case events", singular "event", intro "Case milestones: access and evidence requested or received, analysis, updates and reports shared."
fields: datetime (label "Date", type date, required, default today; the field id stays "datetime"); type (Event type, required, select with optgroups per TIMELINE_GROUPS); details (Details, textarea, wide, placeholder "e.g. Memory images for DC01 and FS02 requested from client IT").
sort byDatetime. columns: Date ("07 Oct 2026", nowrap), Event (dot + label), Details (pre-wrap). normalizeRecord trims any older "YYYY-MM-DDTHH:MM" timeline values to the date, and import accepts them the same way.
chart: { title "Case timeline", file "case-timeline", build(rec) => { events: [{datetime, title: event label, tag: group label, color: group colour, details}], legend: groups used } }.

@@CSS-REGISTERS
.reg-intro (muted, max-width 900px, margin-bottom 14px); .tl-note (warn notice: padding 10px 14px, radius 8px, warn 50% border, warn 10% background); .tl-layout (grid gap 18px); .tl-form (padding 18px 20px); .tl-fields (grid minmax(280px, auto) and minmax(220px, 1fr), gap 14px 18px; .f grid gap 6px; labels 600; .tl-desc spans both columns; textarea min-height 64px; datetime input max-width 260px); .tl-form-actions (flex gap 8px, margin-top 14px); .form-error (danger 600); .req (danger asterisk); .reg-panel (padding 16px 18px); .reg-toolbar (flex wrap space-between, padding 14px 16px 8px, h2 1.05rem); .tl-chart-card (padding 12px, overflow-x auto); .tl-chart svg (block, width 100%, height auto, max-width 960px, centred); .tl-dot (10px circle, margin-right 8px); .table-scroll (overflow-x auto); td.pre (pre-wrap); td.nowrap; td.mono (monospace 0.86rem, break-all); td.num (right, tabular). At 700px .tl-fields becomes one column.

@@JS-REGISTER-VIEW
renderRegister(key): view 'reg:<key>', renderShell(key, ...) with: intro; timezone notice when the register has datetime fields and no incident timezone ("No incident timezone set, so times are stored as entered and no UTC is shown. Set it in the Questionnaire under Case summary."); panel card #regPanel if panel; form#regForm (novalidate) with heading "Add <singular>", fields grid, p#regError, buttons "Add <singular>" (submit), "Cancel edit" (hidden), hint "Ctrl + Enter to save"; chart card if chart (heading + "Export PNG" #regPng + div#regChart role img); table card with title and count, toolbar buttons, div#regTable.
Field rendering: required asterisk; datetime fields show "(<tz>)" after the label, a "Now" button (data-now) and a live UTC hint (data-utc) under the input; selects start with "Select..."; text fields get a datalist from suggestions(rec).
Behaviour: defaults on reset ('now' and 'today' in the incident timezone); input/change refresh the UTC hint and call field.onChange; Ctrl+Enter submits. Submit: read values (trim, Number for number), check required ("Required: A, B."), normalize, dedupe (merge with toast "Merged with existing <singular>." or error "This <singular> is already in the register. Edit the existing entry instead."), add (id, createdAt) or update (updatedAt), remove empty fields, commit(), afterChange, reset form, refreshRegister(). Edit fills the form and switches labels to "Edit <singular>" / "Update <singular>". Delete asks confirmModal. Row quick actions apply then commit. Table: columns with data-label, rowClass, actions (quick actions, Edit, Delete). Empty state "Nothing recorded yet. Add the first <singular> above."
refreshRegister(): panel, chart, table, refreshTabs(). On-screen chart uses the current theme tokens read with getComputedStyle; REPLACE FUNCTION applyTheme to re-render the chart when a register view is open.

@@JS-CHART
buildTimelineSVG(rec, palette, {events, legend}, heading) returns {svg, width, height}:
- Width 960, padding 32. Title 20px bold wrapped; sub line 13px muted "<heading> · <client> · <type> · Timezone: <tz or not set>"; legend circles r 6 with 12px 600 muted labels, wrapping; 1px separator.
- Each event: local datetime 13px weight 650 right-aligned at x 220, UTC 11.5px muted beneath. Date-only events (case timeline) show "07 Oct 2026" with no UTC line, and a zero gap between them reads "same day". Card from x 280 to the right padding: rect rx 8, surface fill, border stroke; 5px colour bar; title 14px bold wrapped (leave room for the tag); tag uppercase 10.5px bold in the event colour, right-aligned; details 13px wrapped, 18px line height; min height 48; 30px gap.
- Vertical 2px line at x 250 from first to last dot; dots r 8 in the event colour with a 3px stroke in the background colour.
- Gap labels between events ("+1h 30m", "+1d 5h", "+3d", "same time") 11px muted, right-aligned at x 234, centred in the gap. Use real UTC differences when the timezone is valid, else wall-clock.
- Footer 11px muted "5 events · Span 6d 5h · Generated <date>".
- Measure text with a hidden canvas 2D context using the same font stack; wrap by words; split long unbroken strings (hashes, paths) by characters.
exportChartPNG(rec, key): build with TIMELINE_EXPORT_PALETTE, load as data:image/svg+xml image, draw on a canvas at 2x (1x if 2x height exceeds 16000px), toBlob PNG, download "ir-<file>-<title slug>-<date>.png", toast "<title> exported as PNG.".

CHECK
Case timeline tab appears; adding, editing and deleting events works; the chart updates live and follows the theme; Export PNG downloads a clean light image.

## STEP 08: Incident timeline (MITRE ATT&CK)

STEP 08: Incident timeline register. Return code blocks per marker.

@@JS-CONFIG-CASE (add)
ATTACK_TACTICS ([{id, label, color}]) and ATTACK_TECHNIQUES ([[id, name, tactic id]]) come from the question bank (step 02); do not redefine them.

@@JS-REGISTER-HELPERS (add): tactic(id) lookup (fallback grey); autoTactic(value, form): if the value starts with a known technique id and the tactic select is empty, set it; assetNames(kind) returning a suggestions function over rec.assets names.

@@JS-REGISTERS: entry attack
tab "Incident timeline", title "Threat actor activity", singular "activity", intro "Threat actor activity reconstructed from evidence, mapped to MITRE ATT&CK tactics. Host and account suggestions come from the Assets register."
fields: datetime (required, no default); tactic (ATT&CK tactic, required, options "Initial Access (TA0001)" style); technique (text, placeholder "e.g. T1021.001 Remote Desktop Protocol", suggestions "T1566.002 Spearphishing Link" style, onChange autoTactic); host (suggestions assetNames('host')); account (suggestions assetNames('account')); artefact (label "Source artefact", placeholder "e.g. Security.evtx 4624, $MFT, UAL, EDR telemetry"); description (textarea, wide).
sort byDatetime. columns: When, Tactic (dot + label), Technique, Host, Account, Source, Description.
chart: title "Incident timeline", file "incident-timeline"; event title = technique or tactic label, tag = tactic label, colour = tactic colour, details = description plus a line "Host: X  |  Account: Y  |  Source: Z" (only parts that exist); legend = tactics used, in tactic order.

VIEW OPTIONS (chart.views: true on this register only)
Add view buttons (same .pf chip style, group "Timeline view") before Export PNG in the chart card toolbar: Vertical, Alternating, Swimlanes, ATT&CK board. Remember the choice in localStorage key "ir-scoping-tl-view" (default vertical); chartSVG(rec, palette, key) returns the selected view (or buildTimelineSVG for registers without views) and is used both on screen and by Export PNG, whose file name includes the view id. All views: width 960, shared header (case title, "<heading> · client · all incident types · Timezone"), footer "N events · Span X · Generated <date>", tactic colours only on dots, bars and pills (text in text colours), long technique names break after a hyphen.
- Vertical: grouped by day with a date pill on the spine ("Tue 29 Sep 2026") plus "N events · <gap> after the previous event"; local time (14px bold) and UTC (11px muted) right-aligned left of a 2px spine; dot r 7 in the tactic colour with a 3px background ring; card from x 204: surface fill, border, 4px coloured left edge, "#n" top right, technique title 14.5px bold, a row of chips (tactic pill: 18% tint of the tactic colour with a coloured dot and text in the text colour; Host / Account / Source chips: bordered, muted text, truncated with an ellipsis), then the description.
- Alternating: centre spine, cards 396px wide alternating left and right (staggered so they never overlap), each card also shows its date and time; coloured connector from spine to card; day pills on the spine.
- Swimlanes: one 44px lane per tactic used (ATT&CK order, alternate lanes tinted), lane label with dot and count, time axis with clean ticks (15 min to 30 days, at most about 7), dots at true time positions joined by a faint polyline in event order, sequence number above each dot (below if crowded), an SVG <title> tooltip per dot, and a numbered two-column key below.
- ATT&CK board: one column per tactic used, up to 5 per row, 4px coloured top bar, tactic name and count, cards in time order with a numbered ring badge, date and time, technique and Host / Account / Source lines.

CHECK
Typing "T1566.002 Spearphishing Link" with no tactic selects Initial Access; the chart is coloured by tactic; all four views render and export to PNG; the chosen view survives a reload.

## STEP 09: IOC register and CSV export

STEP 09: IOC register. Return code blocks per marker.

@@JS-METRICS (add) statTile(num, label, sub, level) helper returning the .stat markup (used by register panels and the report).

@@JS-CONFIG-CASE: IOC_TYPES ([{value, label}]) and IOC_CONFIDENCE come from the question bank (step 02); do not redefine them.

@@JS-REGISTER-HELPERS (add)
- refang(s): trim; leading hxxp to http (keep case); [.] (.) {.} [dot] to "."; [:] to ":"; [@] [at] to "@"; [/] to "/".
- defang(type, v): ip and domain: dots to [.]; url: leading http to hxxp and dots to [.]; email: @ to [@] and dots to [.]; others unchanged.
- detectIocType(v) on the refanged value: 32/40/64 hex = md5/sha1/sha256; IPv4 dotted quad or IPv6 (hex and colons, more than two parts) = ip; scheme:// = url; x@y.z = email; ends with a common file extension (exe, dll, ps1, psm1, bat, cmd, js, jse, vbs, vbe, hta, lnk, msi, iso, img, zip, 7z, rar, gz, tar, docm, xlsm, doc, xls, ppt, pdf, txt, dat, tmp, scr, sys, bin, py, sh, jar) or contains a slash or backslash = filename; else a domain pattern = domain; else ''.
- normalizeIoc(item): refang; lowercase for domain, email and hashes; validate: hashes exact hex length ("SHA256 must be 64 hexadecimal characters."), IPv4 octets 0 to 255 or valid IPv6 ("Not a valid IPv4 or IPv6 address."), email ("Not a valid email address."), URL with scheme ("URL must include a scheme, e.g. https://"), domain ("Not a valid domain name."), last seen not before first seen ("Last seen is before first seen."). Returns {item} or {error}.
- mergeIoc(a, b): earliest first seen, latest last seen, highest confidence (High > Medium > Low), context lines combined without duplicates.
- csvCell(value, guard): quote when containing comma, quote or newline (double the quotes); when guard is true and the text starts with = + - @ tab or CR, prefix an apostrophe.
- exportIocCSV(rec, defanged): header type,value,confidence,first_seen_local,first_seen_utc,last_seen_local,last_seen_utc,context,case; rows sorted like the register; value raw or defanged; guard every cell except the value column; CRLF; no BOM; file "ir-iocs-raw-<slug>-<date>.csv" or "ir-iocs-defanged-..."; toast "Exported N IOCs (raw)." / "(defanged)".

@@JS-REGISTERS: entry iocs
tab "IOCs", title "Indicators of compromise", singular "IOC", intro "Paste values as-is: defanged input such as hxxp or [.] is converted back, the type is detected automatically, and duplicates are merged into the existing entry."
fields: value (required, wide, placeholder "IP, domain, URL, email, hash or file name", onChange: set the type select to detectIocType(value) unless the user picked a type manually); type (select, required); confidence (select, default Medium); firstSeen (datetime); lastSeen (datetime); context (textarea, wide, placeholder "Where it was seen and what it is, e.g. C2 beacon from WS042, Cobalt Strike").
normalize normalizeIoc; dedupeKey "type|value"; merge mergeIoc.
sort by IOC_TYPES order then value. columns: Type, Value (mono, exportText defanged), Confidence, First seen, Last seen, Context (pre).
toolbar: "CSV (raw, for EDR / SIEM)" and "CSV (defanged, for sharing)".
panel: tiles "Total IOCs" and one per type that has entries; or "No IOCs recorded yet."
Track the manual type choice with a data attribute on the type select (set on its change event, cleared on form reset, set when editing).

CHECK
"hxxps://evil[.]example[.]com/x" saves as https://evil.example.com/x (URL). Adding 185.220.101[.]45 twice merges into one entry. 999.1.1.1 is rejected. Raw CSV has 185.220.101.45, defanged CSV has 185[.]220[.]101[.]45.

## STEP 10: Affected assets register

STEP 10: Assets register (hosts and accounts). Return code blocks per marker.

@@JS-CONFIG-CASE (add) ASSET_KINDS: host "Host", account "Account". ASSET_STATUSES ([{value, label, color}]) comes from the question bank (step 02); do not redefine it.
@@JS-METRICS (add) assetStats(rec) => { host: counts per status + total + compromised, account: same, privileged }. Compromised = confirmed + contained + remediated (suspected does not count). privileged = accounts with privileged "Yes" and status not suspected.

@@JS-REGISTERS: entry assets
tab "Assets", title "Affected hosts and accounts", singular "asset", intro "Hosts and accounts in scope. Confirmed, contained and remediated all count as compromised; suspected does not."
fields: kind (Type, select, required, default host); name (required, placeholder "Hostname, or account (UPN / DOMAIN\user)"); identifier (placeholder "IP address, SID or other identifier"); role (label "Role / description", placeholder "e.g. Domain controller, finance user"); privileged (label "Privileged?", select Yes, No, Unknown); status (select, required, default suspected); notes (textarea, wide).
dedupeKey "kind|lowercase name" with no merge (duplicates are blocked).
sort by kind, then status order, then name. columns: Type, Name (mono), Identifier (mono), Role, Privileged, Status (tint badge), Notes (pre).
tabBadge "compromised/total" ('' when empty).
panel: tiles Hosts compromised (sub "N suspected", danger when above 0), Accounts compromised (same), Privileged accounts compromised (danger when above 0), Contained, Remediated; then a two-column area with a small table (rows Hosts and Accounts, columns per status and Total) and the line "Scoping estimate from the questionnaire: X systems, Y user accounts." from answers affected_systems_count and affected_users_count ("?" for a missing one; "Scoping estimate: not recorded in the questionnaire." when both are missing).
CSS (@@CSS-REGISTERS): .panel-cols (grid auto-fit minmax(240px, 1fr), gap 16px, margin-top 14px); .mini-table (collapsed, 0.88rem, cells padding 4px 14px 4px 0 with bottom border; thead muted 0.78rem); .mini-h (0.85rem muted).

CHECK
Adding DC01 then dc01 as hosts is blocked; tiles and the table match the entries; the Incident timeline host field now suggests asset hosts.

## STEP 11: Time log (hours)

STEP 11: Time log register with an optional hours cap. Return code blocks per marker.

@@JS-CONFIG-CASE (add) DEFAULT_BUDGET_WARN_PCT = 80.
@@JS-METRICS (add)
- entryMinutes(e) = Number(e.minutes) or, for older entries, Math.round(Number(e.hours) * 60).
- fmtMins(m): 95 => "1h 35m", 45 => "45m", 120 => "2h". fmtHours(h) = fmtMins(h * 60). fmtH(n) rounds to 2 decimals without trailing zeros (used for the cap).
- hoursInfo(rec) => { used (hours, from the sum of entryMinutes / 60), cap, pct, warnPct, level: '' | 'warn' | 'danger' } (danger at 100% or more, warn at warnPct or more, '' when no cap).
- normalizeRecord: convert older entries that have hours but no minutes to minutes (and drop the hours field).

@@JS-REGISTERS: entry hours
tab "Hours", title "Time log", singular "entry", intro "Each time you work on the case, log the date and the minutes spent. Totals show in hours and minutes. Optionally set an hours cap (e.g. the insurer panel limit) to get a warning as you approach it."
fields (only these two): date (required, default today); minutes (label "Time spent (minutes)", number, step 1, required, placeholder "e.g. 45").
normalize: minutes between 1 and 1440, rounded to whole minutes ("Time spent must be between 1 and 1440 minutes.").
sort by date newest first. columns: Date ("07 Oct 2026"), Time spent (fmtMins, right-aligned).
tabBadge: '' when nothing logged and no cap; "11h 30m / 10h" with a cap or "11h 30m" without; warn = level.
panel: inputs "Hours cap" (#budgetCap, min 0, step 0.5, placeholder "No cap") and "Warn at (% of cap)" (#budgetWarn, 1 to 100); then div#hoursMeter. wirePanel: on input update rec.budget, markDirty(), refresh only #hoursMeter and the tabs (do not re-render the inputs).
hoursMeterHTML(rec): with a cap: 12px meter (.meter, green fill; .warn and .danger variants) and text "11h 30m of 10h used (115%). 1h 30m over cap." (over cap in danger) or "... 2h 15m remaining."; without a cap: "11h 30m logged. No cap set." No per-person or per-activity breakdown.
afterChange(rec, before): toast "Time at 85% of the 10h cap." (warn) when crossing the warning level and "Hours cap exceeded: 11h 30m of 10h used." (error) when crossing 100%.
CSS: .budget-row (flex wrap, gap 14px, align end; labels grid gap 4px 600 0.88rem; inputs width 150px); .meter (12px pill, surface-3, margin 14px 0 6px; span success, width transition; .warn span warn; .danger span danger); .danger-text.

CHECK
With a 10h cap, logging 360 then 150 minutes warns at 85%; 180 more warns that the cap is exceeded; 0 or 2000 minutes is rejected; totals read like "11h 30m".

## STEP 12: JSON export / import and Markdown report

STEP 12: Exports and import. Return code blocks per marker and replace the temporary "Arrives in step 12" handlers.

@@JS-EXPORT
- buildExport(records): JSON string, 2-space indent: { schemaVersion: 3, app: 'ir-scoping', appVersion: '3.0.0', exportedAt, count, incidents }.
- markExported(ids): chained on state.saveChain; for each id read from DB, set lastExportedAt = now WITHOUT changing updatedAt, put back, upsertLocal; also set it on state.current; then updateHeader and renderList when on home.
- exportIncidentsJSON(records, filename): flushSave, re-read each record from the DB, download, markExported, toast "Exported N incident(s) to JSON.".
- exportAllJSON(): all records, file "ir-scoping-all-<date>.json" (toast "Nothing to export yet." when empty).
- colExport(column, item, rec) = exportText or text.
- toMarkdown(rec): "# IR Case Report: <title>", bullets Client, Incident type (caseTypesLabel), Phase, Incident timezone ("<tz or Not set> (date/times are local to this zone, UTC in brackets)"), Created, Last updated, Questions answered "X of Y", Record ID in backticks. Then "## <section>" for each questionnaire section with bold label lines (two trailing spaces) and answers or "_Not answered_" (hidden conditional questions omitted; multi-line answers keep line breaks). Then "## <tab>" for each non-empty register in REGISTER_ORDER: one bullet per item made of "**Column:** value" parts joined by " | " (IOCs defanged via exportText); the Hours section starts with "11h 30m logged against a cap of 10h (115%)." Footer "***" and "_Generated <date> by IR Scoping v3.0.0 (schema 3)._".
- exportMarkdown(rec): flush, re-read, download "ir-scoping-<slug>-<date>.md" (text/markdown), toast.
- sanitizeRegister(key, arr): keep only defined fields; numbers coerced (drop non-finite); other values must be strings (numbers and booleans converted); datetimes must match YYYY-MM-DDTHH:MM (cut to 16 chars); dates must match YYYY-MM-DD; drop items missing required fields; keep valid createdAt/updatedAt; returns {items, dropped}.
- sanitizeRecord(r, idx): errors for non-objects, missing id or incidentType, non-object answers; keep string, number, boolean or string-array answers; build the record, sanitise every register, normalizeRecord; warnings for unknown incident type, dropped answers and dropped register entries.
- validateImport(data): fatal if not an object, schemaVersion not a number, or incidents not an array; warning when schemaVersion is newer than 3; duplicate ids within the file keep the later one with a warning. Older schema 1 and 2 files must import (missing registers become empty, phase from status).
- mergeRecords(existing, incoming): the newer (by updatedAt) wins on top-level fields; answers combined (newer wins on conflicts); earliest createdAt; updatedAt = now; keep the local lastExportedAt; each register combined by item id (newer wins), then duplicate IOCs merged with mergeIoc; normalizeRecord.
- handleImportFile(file): parse errors and fatal or empty results show an "Import failed" modal listing messages. Otherwise a modal "Import incidents": counts, notes (skipped and warnings), and when ids already exist a radio choice "Merge" (default; "Combine answers and register entries. Where both have the same item, the more recently updated version wins.") or "Skip" ("Leave existing incidents untouched; only import new ones."). Apply with DB.putMany; toast "Import complete: X added, Y merged, Z skipped, N invalid."; refresh home.
- #importFile change handler: read the file, reset the input value so the same file can be imported again.
Wire: home "Export all", row JSON and Markdown buttons, and the case header "Export JSON" and "Export Markdown".

CHECK
Export one case and all cases; re-import with Merge creates no duplicates; a file with schemaVersion 1 and one incident {id, title, incidentType: 'malware', status: 'complete', answers: {}} imports as phase Closed; a broken file shows a clear error.

## STEP 13: Case phases, full search, backup reminder

STEP 13: Home screen upgrades. Return code blocks per marker and replaced functions.

@@JS-CONFIG-CASE (add) BACKUP_WARN_DAYS = 3.
@@JS-METRICS (add)
- relTime(ms): "just now", "Xm ago", "Xh ago", "Xd ago".
- backupInfo(rec): changedSince = never exported or updatedAt later than lastExportedAt; reference = lastExportedAt or createdAt; stale = changedSince and reference older than BACKUP_WARN_DAYS days; text "Last JSON export 2d ago" (+ ", changed since") or "Never exported to JSON".

REPLACE FUNCTION renderList:
- #backupBanner: when any case is stale, a .notice.warn "Backup reminder: N case(s) have changes not exported to JSON in over 3 days." with button #bkExport "Export all now" (calls exportAllJSON).
- #phaseFilter chips: All, Open cases (not completed or closed), then each phase, each with a count over all cases; clicking sets state.phaseFilter and re-renders; aria-pressed on the active chip.
- Filter by phase, then search with matchRecord; sort newest first; "N of M" counter.
- Keep the table columns from step 03 (Case ID, Incident type, Customer, Severity, Case logged, actions); do not add phase, flags or date columns to the table. When a search match was inside the case, show a line under the Case ID: "<where>: ...text with <mark>match</mark>..." (40 characters either side).
New helpers (@@JS-HOME):
- searchFields(rec): list of {where, text}: every visible non-bound question answer (formatAnswer, labelled with the question label) and every field of every register item (select values as labels, labelled with the register tab name).
- matchRecord(rec, term): search the term and its refanged form, case-insensitive; a match in title, Case ID, customer, type label, geolocation, severity or phase label returns {where: null}; otherwise the first field match with its index; null when no match.

REPLACE FUNCTION updateHeader: also fill #backupChip with backupInfo text ("Backup due: " prefix and .warn when stale).

CHECK
Phase chips filter and count correctly; searching adm_jsmith, evil[.]example, T1021 or an IOC value finds the case and shows where; a case created over 3 days ago and never exported shows "Backup due" and the banner; "Export all now" clears it without changing the case's last updated time.

## STEP 14: Print-ready report

STEP 14: Report tab and print styles. Return code blocks per marker.

@@JS-REPORT
renderReport(): renderShell('report', ...) with a toolbar (.report-toolbar): muted text "Print-ready view of the whole case. Use Print / Save as PDF and choose "Save as PDF" as the destination. IOCs are defanged." and primary button "Print / Save as PDF" (flushSave then window.print()); then article.report with reportHTML(rec).
reportHTML(rec):
- Header: kicker "INCIDENT RESPONSE CASE REPORT"; h1 title; metadata list: Client, Incident type, Phase, Timezone ("<tz> (local times shown, UTC in brackets)" or "Not set"), Created, Last updated, Generated, Record ID.
- "Key figures" stat tiles: Hosts compromised (sub "N suspected"), Accounts compromised, Privileged accounts compromised, IOCs, Incident timeline events, Time logged ("11h 30m", sub "of 10h cap (115%)" or "No cap set"), Questions answered ("X/Y").
- "Scoping questionnaire": answered questions only, h3 per section and a definition list; then "N unanswered questions omitted."
- One section per register in REGISTER_ORDER rendered as a table (headers = column labels; cells use exportText if present, else html, else escaped text). Hours starts with "11h 30m logged against a cap of 10h (115%).". Empty registers show "None recorded.".
Make sure refreshTabs shows the Report tab and openTab routes to it.

@@CSS-REPORT
.report-toolbar (flex wrap space-between, gap 12px, margin-bottom 14px). .report: override the theme tokens to the light values (surface #ffffff, surface-2 #f6f8fb, surface-3 #e9eef5, border #d3dbe6, text #17202b, muted #5b6878, accent #1d64d8, warn #a5710a, danger #c7323f, success #1e8a55), color-scheme light, white background, radius 12px, padding 36px 40px, max-width 1040px, centred. .rep-kicker (uppercase, letter-spacing 0.8px, 0.75rem 700 accent). h1 1.6rem. .rep-meta (grid max-content 1fr, gap 3px 18px, 0.9rem; dt muted). .rep-sec (margin-top 28px; h2 1.15rem with a 2px solid text-colour bottom border; h3 0.98rem). .rep-table (full width, collapsed, 0.84rem; cells padding 6px 8px with bottom border, top aligned; th surface-2, 0.72rem uppercase muted; td.pre pre-wrap). .rep-qa (grid minmax(180px, 36%) 1fr, 0.88rem, dotted row borders, dt muted, dd pre-wrap). At 700px: smaller padding and stacked Q/A.

@@CSS-PRINT
@media print: @page margin 14mm; force the light token values on :root and [data-theme]; print-color-adjust exact; white body, 12px; hide .topbar, .banner, .tabs, .inc-head, .report-toolbar, .app-foot, .toasts, all .btn, .wiz-foot and .sec-nav; main without padding or max-width; .report without padding, radius or max-width; .stat-grid 4 columns; avoid breaks inside table rows, Q/A pairs and stat tiles; avoid breaks after h2 and h3; table header rows repeat on every page.

CHECK
The Report tab looks like a clean white document in both themes. Print preview shows only the report, IOCs defanged, tables continuing across pages with repeated headers.

## STEP 15: Analytics screen

STEP 15: Analytics screen. Return code blocks per marker (put the JS under @@JS-REPORT, after the report code, and the CSS under @@CSS-REPORT).

PURPOSE
Month-on-month view across all cases: how many cases, which severities, which incident types. Opened from a new "Analytics" button in the home header (before "Import JSON"); "All incidents" goes back.

COUNTING
caseMonth(rec): "YYYY-MM" from answers.case_logged, else from createdAt. analyticsMonths(range, cases): a continuous list of months (empty months included) for the selected range: Last 6 months, Last 12 months (default, kept in state.analyticsRange), This year, All time (first case month to the later of now and the last case month).

COLOURS (validated categorical palette; never generate more hues)
CSS tokens on .viz. Dark (default): s1 #3987e5, s2 #d95926, s3 #199e70, s4 #c98500, s5 #d55181, s6 #008300, s7 #9085e9, s-other #5f6570. Light ([data-theme="light"] .viz): s1 #2a78d6, s2 #eb6834, s3 #1baf7a, s4 #eda100, s5 #e87ba4, s6 #008300, s7 #4a3aa7, s-other #b0aea6.
Severities take slots 1 to 5 in SEVERITIES order, "Not set" takes s-other. Incident types: the 7 types with the most cases ALL TIME take slots 1 to 7 (so a date-range change never repaints a type); the rest fold into "All other types" (s-other, legend shows how many types and lists them on hover). Text never uses series colours.

LAYOUT (top to bottom)
1. Title "Analytics" with the counting rule as a muted subtitle, and the range chips (same .pf chip style as the home phase filter) in one row above everything they scope.
2. Stat tiles: Cases in period (sub "Nov 2025 to Oct 2026"), Cases in <this month> (sub "+N vs <last month>", no colour), Most common severity (sub "N of M cases"), Most common incident type (value = count, sub = type).
3. Card "Cases per month by severity": legend (10px rounded swatch + label), stacked column chart, then <details> "View as table" (Month, one column per series, Total).
4. Card "Cases per month by incident type": same pattern.
5. Card "Cases by incident type in this period": horizontal bars, one series in s1, sorted by count, label left, value at the bar tip, 14px bars with a 4px rounded end, square at the start.
Empty period: a card "No cases in this period" with a hint.

STACKED COLUMN CHART (SVG, width = container width, min 320, height 280; margins left 40, right 12, top 22, bottom 30)
- Clean y ticks (1, 2 or 5 x 10^n, about 4 ticks), solid 1px gridlines in the border token, baseline in muted, tick labels 11px muted tabular figures.
- Columns at most 24px wide (60% of the band), segments stacked bottom-up with a 2px gap between segments, square at the baseline, 4px rounded top on the top segment only. Month total above each column (11px, 600, text colour). X labels "Oct 26", thinned so they never overlap.
- One transparent hit target per month covering the full band (tabindex 0, aria-label "Oct 2026: 13 cases"). Hover or focus highlights the band (surface-2) and shows one tooltip with the total and month, then EVERY series in legend order (short coloured line key, value strong, label muted; zero rows dimmed). Build tooltip, legend and table content with textContent, not innerHTML. Place the tooltip beside the column (right if it fits, else left).
- Re-render on window resize (debounced) while the Analytics screen is open.

CHECK
With cases across several months: columns and totals match "View as table"; tooltips list every series; switching ranges keeps each type's colour; both themes readable; no horizontal scrolling at 400px.

## STEP 16: Incident types screen

STEP 16: Incident types screen. Return code blocks per marker (put the JS under @@JS-REPORT, after the Analytics code, and the CSS under @@CSS-REPORT).

PURPOSE
A read-only reference of every incident type and its questions. Opened from an "Incident types" button in the home header (before "Analytics"); "All incidents" goes back.

BUILD
- QTYPE_LABELS: text Text, textarea Long text, number Number, date Date, time Time, datetime Date and time, select Dropdown, multiselect Checkboxes, yesno Yes / No, timezone Timezone.
- renderIncidentTypes(): flushSave, state.view 'types', clear current case; title "Incident types", muted subtitle "<N> incident types, <M> type-specific questions. Open a type to see its sections and questions. The common sections (Case summary, Incident Details and so on) apply to every type."; buttons "Expand all" (toggles to "Collapse all", opens / closes every card) and "All incidents".
- One <details class="card it-type"> per INCIDENT_TYPES entry. Summary: label (650 weight), id in <code> (muted, 0.8rem), "N sections, N questions" (muted small).
- Body: per section a block with a top border: h3 title, muted description, then an <ol> of questions. Each: bold label, a .type-more pill with QTYPE_LABELS[type], "(conditional)" muted when it has showIf, the Ask line (same .ask / .ask-tag style as the wizard, from q.ask), and options joined with " · " (muted small).
- document.title "Incident types - IR Scoping v3".

CHECK
Every type is listed with correct counts; new types added to the config appear without other changes; no horizontal scrolling at 400px.

## STEP 17: Report checklist

STEP 17: Report checklist tab. Return code blocks per marker (config under @@JS-CONFIG-CASE, JS under @@JS-REPORT before renderReport, CSS under @@CSS-REPORT).

PURPOSE
Yes / no / N/A checks to confirm the report is complete before it goes out. Stored in rec.checks = { checkId: 'Yes' | 'No' | 'N/A' }.

CONFIG
CHECK_ANSWERS = ['Yes', 'No', 'N/A'] under @@JS-CONFIG-CASE. REPORT_CHECKS_COMMON and REPORT_CHECKS already come from the question bank (step 02); do not retype the checks.

DATA
newRecord adds checks: {}. normalizeRecord: checks must be a plain object; drop values not in CHECK_ANSWERS. sanitizeRecord copies r.checks when it is an object. mergeRecords: checks = Object.assign({}, older.checks, newer.checks).

BUILD
- checkGroups(rec): [{label "All incidents", items REPORT_CHECKS_COMMON}, then one group per caseTypes(rec) entry with REPORT_CHECKS[t] (label typeLabel(t))], skipping empty groups.
- checkStats(rec): total, yes, na, no, open, gaps (items answered No), pct = round(yes / (total - na) * 100), 0 when nothing applies.
- Tab "Report checklist" (id 'checks') before Report; badge "<pct>%", warn (amber) when any answer is No. openTab('checks') calls renderChecks().
- renderChecks(): a summary card (#chkHead: big "<pct>% complete", "Yes n · No n · N/A n · Not answered n. Completeness is Yes divided by all checks except N/A.", and a red-tinted "Gaps (answered No)" list), a muted intro line, then one card per group with each check as a row: label on the left, a .seg radio group Yes / No / N/A (No checked = danger background, N/A checked = surface-3) and a "Clear" link (space reserved when hidden). Changing or clearing updates rec.checks, the summary card and the tab badge, then markDirty().
- Report tab: inside .report-toolbar (so it never prints) a full-width .chk-note: "Report checklist: <pct>% complete." then "<n> gap(s): <labels joined by '; '>", or "<n> checks not answered yet.", or "No gaps."
- toMarkdown: before the footer, "## Report checklist (internal)", a line with the percentage and counts, then per group a bold group label and "- <label> **<answer or Not answered>**" lines.

CHECK
Answering Yes, No and N/A updates the percentage correctly; gaps list the No items; answers survive reload, JSON export / import and appear in the Markdown export; no horizontal scrolling at 400px.

## STEP 18: Final review against acceptance tests

STEP 18: Review the whole current v3.html (I will paste it if you no longer have it) against these acceptance tests. For each test, say pass or fail. For every fail, give the fix as a REPLACE FUNCTION or a marker block. Do not rewrite unrelated code.

1. Opens with no console errors (missing favicon is fine).
2. New Ransomware case with Case ID "IR-TEST-1" gets the title "IR-TEST-1" (not combined with other values) and opens on Case summary.
3. Answering Yes to "Any related or prior incidents in the last 12 months?" reveals the prior incident details question without losing focus; reload keeps the answer and section.
4. Europe/London 2026-07-01 09:30 shows 08:30 UTC; 2026-12-01 09:30 shows 09:30 UTC; America/New_York 2026-10-07 09:30 shows 13:30 UTC.
5. Case timeline events are date-only: the table shows "07 Oct 2026", and the chart shows day gaps ("+3d") and "same day".
6. IOC refanging, auto type, merging of duplicates, rejection of 999.1.1.1, raw versus defanged CSV.
7. Duplicate asset names (any case) are blocked; compromised and privileged counts are right.
8. Technique T1566.002 auto-selects Initial Access; host suggestions come from assets; PNG exports are non-empty.
9. Hours warning at 85% and cap exceeded toasts; 30h rejected.
10. Phase changes persist; phase chips filter and count; Open cases excludes closed.
11. Search finds text inside answers and every register, including defanged terms.
12. Backup reminder appears for stale cases and clears after export without changing last updated.
13. JSON export and re-import with Merge creates no duplicates; schema 1 files import; invalid entries are reported.
14. Markdown includes every section, non-empty registers, defanged IOCs and the hours summary.
15. Report prints cleanly in light colours.
16. Theme toggle changes everything, including the on-screen chart, and is remembered.
17. Works at 400px width and with the keyboard only; focus is always visible.
18. Every customer-facing question shows its ASK spoken prompt under the heading; internal fields (incident title, Case ID, case logged date, severity) show none; no [config] warnings in the console.
20. Home > Incident types lists every incident type in json/questionnaire/lists.json with the section, question and report-check counts from its JSON file.
19. Analytics opens from the home screen; monthly columns, totals and the table views agree; each incident type keeps its colour across date ranges.
Also confirm: no em dash characters, no alert/confirm/prompt, all user text escaped, no network requests.
Finish with a short list of assumptions and how to add a new incident type (a new json/incident-types/<id>.json, its id in lists.json incidentTypeOrder, then UPDATE), a new register field and a new IOC type (lists.json iocTypes, then UPDATE).

## UPDATE: Update the question bank after editing JSON (use any time)

UPDATE: Update the question bank from edited JSON files. Return only the changed parts of the question-bank block.

The DATA section of this file is the source of truth. Compare it with the <script type="application/json" id="question-bank"> block in v3.html and update the block to match it exactly:
- json/questionnaire/lists.json -> "lists"; json/questionnaire/common-sections.json -> "common"; json/questionnaire/closing-sections.json -> "closing"; json/questionnaire/report-checks-common.json -> "reportChecksCommon"; json/incident-types/<id>.json -> "types"."<id>".
- Copy everything exactly, compact JSON, one top-level key (or one type) per line. Write "</" as "<\/".
- A new incident type needs its file under "types" AND its id in lists.incidentTypeOrder. A removed type: delete it from both.
- Do not change any JavaScript unless a file uses a new "optionsFrom" name (then add it to OPTION_SOURCES) or a new question property the code does not support (then tell me first).

Rules: never rename an existing id (question, section, check, phase, list value); question ids must stay unique across all files; showIf must point at an existing question; no em dash characters. Warn me about any of these problems instead of guessing.

Reply with: (1) what changed (added, removed, edited), (2) the replaced lines of the question-bank block, (3) a short test list.

***

## DATA

The question data. Each block is valid JSON and maps to the question bank as described in STEP 02A, STEP 02B and UPDATE.

### DATA FILE: json/questionnaire/lists.json

```json
{
  "incidentTypeOrder": ["ransomware", "bec", "malware", "exfiltration", "ad", "entra", "entra_priv", "entra_sp", "entra_mi", "entra_oauth", "device_code", "hybrid", "exchange", "sharepoint", "webserver", "azure"],
  "phases": [{"id": "scoping", "label": "Scoping call", "color": "#3b82f6"}, {"id": "collection", "label": "Evidence collection", "color": "#d97706"}, {"id": "analysis", "label": "Analysis", "color": "#8b5cf6"}, {"id": "reporting", "label": "Reporting", "color": "#0d9488"}, {"id": "completed", "label": "Completed", "color": "#16a34a"}, {"id": "closed", "label": "Closed", "color": "#6b7280"}],
  "severities": ["Sev1", "SevA", "SevB247", "SevB", "SevC"],
  "countries": ["Multiple countries", "Afghanistan", "Albania", "Algeria", "Andorra", "Angola", "Antigua and Barbuda", "Argentina", "Armenia", "Australia", "Austria", "Azerbaijan", "Bahamas", "Bahrain", "Bangladesh", "Barbados", "Belarus", "Belgium", "Belize", "Benin", "Bermuda", "Bhutan", "Bolivia", "Bosnia and Herzegovina", "Botswana", "Brazil", "Brunei", "Bulgaria", "Burkina Faso", "Burundi", "Cabo Verde", "Cambodia", "Cameroon", "Canada", "Cayman Islands", "Central African Republic", "Chad", "Chile", "China", "Colombia", "Comoros", "Congo", "Costa Rica", "Cote d'Ivoire", "Croatia", "Cuba", "Cyprus", "Czechia", "Democratic Republic of the Congo", "Denmark", "Djibouti", "Dominica", "Dominican Republic", "Ecuador", "Egypt", "El Salvador", "Equatorial Guinea", "Eritrea", "Estonia", "Eswatini", "Ethiopia", "Fiji", "Finland", "France", "Gabon", "Gambia", "Georgia", "Germany", "Ghana", "Gibraltar", "Greece", "Grenada", "Guatemala", "Guernsey", "Guinea", "Guinea-Bissau", "Guyana", "Haiti", "Honduras", "Hong Kong", "Hungary", "Iceland", "India", "Indonesia", "Iran", "Iraq", "Ireland", "Isle of Man", "Israel", "Italy", "Jamaica", "Japan", "Jersey", "Jordan", "Kazakhstan", "Kenya", "Kiribati", "Kosovo", "Kuwait", "Kyrgyzstan", "Laos", "Latvia", "Lebanon", "Lesotho", "Liberia", "Libya", "Liechtenstein", "Lithuania", "Luxembourg", "Macao", "Madagascar", "Malawi", "Malaysia", "Maldives", "Mali", "Malta", "Marshall Islands", "Mauritania", "Mauritius", "Mexico", "Micronesia", "Moldova", "Monaco", "Mongolia", "Montenegro", "Morocco", "Mozambique", "Myanmar", "Namibia", "Nauru", "Nepal", "Netherlands", "New Zealand", "Nicaragua", "Niger", "Nigeria", "North Korea", "North Macedonia", "Norway", "Oman", "Pakistan", "Palau", "Palestine", "Panama", "Papua New Guinea", "Paraguay", "Peru", "Philippines", "Poland", "Portugal", "Puerto Rico", "Qatar", "Romania", "Russia", "Rwanda", "Saint Kitts and Nevis", "Saint Lucia", "Saint Vincent and the Grenadines", "Samoa", "San Marino", "Sao Tome and Principe", "Saudi Arabia", "Senegal", "Serbia", "Seychelles", "Sierra Leone", "Singapore", "Slovakia", "Slovenia", "Solomon Islands", "Somalia", "South Africa", "South Korea", "South Sudan", "Spain", "Sri Lanka", "Sudan", "Suriname", "Sweden", "Switzerland", "Syria", "Taiwan", "Tajikistan", "Tanzania", "Thailand", "Timor-Leste", "Togo", "Tonga", "Trinidad and Tobago", "Tunisia", "Turkey", "Turkmenistan", "Tuvalu", "Uganda", "Ukraine", "United Arab Emirates", "United Kingdom", "United States", "Uruguay", "Uzbekistan", "Vanuatu", "Vatican City", "Venezuela", "Vietnam", "Yemen", "Zambia", "Zimbabwe"],
  "dataTypes": ["Personal data (PII)", "Special category / health data (PHI)", "Payment card data (PCI)", "Credentials / secrets", "Intellectual property", "Financial records", "Customer data", "Employee / HR data", "Legal / privileged material", "None identified", "Unknown"],
  "timelineGroups": {"engagement": {"label": "Engagement", "color": "#3b82f6"}, "evidence": {"label": "Evidence", "color": "#d97706"}, "analysis": {"label": "Analysis", "color": "#8b5cf6"}, "containment": {"label": "Containment", "color": "#dc2626"}, "reporting": {"label": "Reporting", "color": "#16a34a"}, "other": {"label": "Other", "color": "#6b7280"}},
  "timelineEventTypes": [
    {"id": "engagement_start", "label": "Engagement started", "group": "engagement"},
    {"id": "scoping_call", "label": "Scoping call", "group": "engagement"},
    {"id": "client_meeting", "label": "Client meeting / status call", "group": "engagement"},
    {"id": "access_requested", "label": "Access requested", "group": "evidence"},
    {"id": "access_granted", "label": "Access granted", "group": "evidence"},
    {"id": "evidence_requested", "label": "Evidence requested", "group": "evidence"},
    {"id": "evidence_received", "label": "Evidence received", "group": "evidence"},
    {"id": "collection_started", "label": "Collection started", "group": "evidence"},
    {"id": "collection_done", "label": "Collection complete", "group": "evidence"},
    {"id": "analysis_started", "label": "Analysis started", "group": "analysis"},
    {"id": "analysis_done", "label": "Analysis complete", "group": "analysis"},
    {"id": "key_finding", "label": "Key finding", "group": "analysis"},
    {"id": "containment_action", "label": "Containment / remediation action", "group": "containment"},
    {"id": "interim_shared", "label": "Interim update shared", "group": "reporting"},
    {"id": "draft_report", "label": "Draft report shared", "group": "reporting"},
    {"id": "final_report", "label": "Final report shared", "group": "reporting"},
    {"id": "engagement_closed", "label": "Engagement closed", "group": "reporting"},
    {"id": "other", "label": "Other", "group": "other"}
  ],
  "iocTypes": [
    {"value": "ip", "label": "IP address"},
    {"value": "domain", "label": "Domain"},
    {"value": "url", "label": "URL"},
    {"value": "email", "label": "Email address"},
    {"value": "md5", "label": "MD5"},
    {"value": "sha1", "label": "SHA1"},
    {"value": "sha256", "label": "SHA256"},
    {"value": "filename", "label": "File name / path"},
    {"value": "other", "label": "Other"}
  ],
  "iocConfidence": ["High", "Medium", "Low"],
  "assetStatuses": [{"value": "suspected", "label": "Suspected", "color": "#d97706"}, {"value": "confirmed", "label": "Confirmed", "color": "#dc2626"}, {"value": "contained", "label": "Contained", "color": "#8b5cf6"}, {"value": "remediated", "label": "Remediated", "color": "#16a34a"}],
  "attackTactics": [
    {"id": "TA0043", "label": "Reconnaissance", "color": "#64748b"},
    {"id": "TA0042", "label": "Resource Development", "color": "#78716c"},
    {"id": "TA0001", "label": "Initial Access", "color": "#dc2626"},
    {"id": "TA0002", "label": "Execution", "color": "#ea580c"},
    {"id": "TA0003", "label": "Persistence", "color": "#d97706"},
    {"id": "TA0004", "label": "Privilege Escalation", "color": "#a16207"},
    {"id": "TA0005", "label": "Defense Evasion", "color": "#65a30d"},
    {"id": "TA0006", "label": "Credential Access", "color": "#16a34a"},
    {"id": "TA0007", "label": "Discovery", "color": "#0d9488"},
    {"id": "TA0008", "label": "Lateral Movement", "color": "#0891b2"},
    {"id": "TA0009", "label": "Collection", "color": "#2563eb"},
    {"id": "TA0011", "label": "Command and Control", "color": "#4f46e5"},
    {"id": "TA0010", "label": "Exfiltration", "color": "#9333ea"},
    {"id": "TA0040", "label": "Impact", "color": "#db2777"}
  ],
  "attackTechniques": [
    ["T1566", "Phishing", "TA0001"],
    ["T1566.001", "Spearphishing Attachment", "TA0001"],
    ["T1566.002", "Spearphishing Link", "TA0001"],
    ["T1190", "Exploit Public-Facing Application", "TA0001"],
    ["T1133", "External Remote Services", "TA0001"],
    ["T1078", "Valid Accounts", "TA0001"],
    ["T1199", "Trusted Relationship", "TA0001"],
    ["T1189", "Drive-by Compromise", "TA0001"],
    ["T1059", "Command and Scripting Interpreter", "TA0002"],
    ["T1059.001", "PowerShell", "TA0002"],
    ["T1059.003", "Windows Command Shell", "TA0002"],
    ["T1047", "Windows Management Instrumentation", "TA0002"],
    ["T1204", "User Execution", "TA0002"],
    ["T1569.002", "Service Execution", "TA0002"],
    ["T1053.005", "Scheduled Task", "TA0003"],
    ["T1543.003", "Windows Service", "TA0003"],
    ["T1547.001", "Registry Run Keys / Startup Folder", "TA0003"],
    ["T1136", "Create Account", "TA0003"],
    ["T1098", "Account Manipulation", "TA0003"],
    ["T1505.003", "Web Shell", "TA0003"],
    ["T1068", "Exploitation for Privilege Escalation", "TA0004"],
    ["T1548.002", "Bypass User Account Control", "TA0004"],
    ["T1134", "Access Token Manipulation", "TA0004"],
    ["T1562.001", "Disable or Modify Tools", "TA0005"],
    ["T1070.001", "Clear Windows Event Logs", "TA0005"],
    ["T1070.004", "File Deletion", "TA0005"],
    ["T1027", "Obfuscated Files or Information", "TA0005"],
    ["T1036", "Masquerading", "TA0005"],
    ["T1218", "System Binary Proxy Execution", "TA0005"],
    ["T1564.008", "Email Hiding Rules", "TA0005"],
    ["T1003.001", "LSASS Memory", "TA0006"],
    ["T1003.003", "NTDS", "TA0006"],
    ["T1003.006", "DCSync", "TA0006"],
    ["T1110", "Brute Force", "TA0006"],
    ["T1558.003", "Kerberoasting", "TA0006"],
    ["T1555", "Credentials from Password Stores", "TA0006"],
    ["T1557", "Adversary-in-the-Middle", "TA0006"],
    ["T1621", "Multi-Factor Authentication Request Generation", "TA0006"],
    ["T1539", "Steal Web Session Cookie", "TA0006"],
    ["T1087", "Account Discovery", "TA0007"],
    ["T1018", "Remote System Discovery", "TA0007"],
    ["T1046", "Network Service Discovery", "TA0007"],
    ["T1482", "Domain Trust Discovery", "TA0007"],
    ["T1083", "File and Directory Discovery", "TA0007"],
    ["T1021.001", "Remote Desktop Protocol", "TA0008"],
    ["T1021.002", "SMB/Windows Admin Shares", "TA0008"],
    ["T1021.006", "Windows Remote Management", "TA0008"],
    ["T1570", "Lateral Tool Transfer", "TA0008"],
    ["T1550.002", "Pass the Hash", "TA0008"],
    ["T1560", "Archive Collected Data", "TA0009"],
    ["T1114", "Email Collection", "TA0009"],
    ["T1114.003", "Email Forwarding Rule", "TA0009"],
    ["T1005", "Data from Local System", "TA0009"],
    ["T1039", "Data from Network Shared Drive", "TA0009"],
    ["T1071.001", "Web Protocols", "TA0011"],
    ["T1219", "Remote Access Software", "TA0011"],
    ["T1572", "Protocol Tunneling", "TA0011"],
    ["T1090", "Proxy", "TA0011"],
    ["T1105", "Ingress Tool Transfer", "TA0011"],
    ["T1041", "Exfiltration Over C2 Channel", "TA0010"],
    ["T1567.002", "Exfiltration to Cloud Storage", "TA0010"],
    ["T1048", "Exfiltration Over Alternative Protocol", "TA0010"],
    ["T1486", "Data Encrypted for Impact", "TA0040"],
    ["T1490", "Inhibit System Recovery", "TA0040"],
    ["T1489", "Service Stop", "TA0040"],
    ["T1485", "Data Destruction", "TA0040"],
    ["T1657", "Financial Theft", "TA0040"]
  ]
}
```

### DATA FILE: json/questionnaire/common-sections.json

```json
{
  "sections": [
    {
      "id": "engagement",
      "title": "Case summary",
      "description": "Core engagement details. Title, client and incident type are also shown on the incident list.",
      "questions": [
        {"id": "_phase", "bind": "phase", "label": "Case phase", "type": "select", "required": true, "row": "top", "optionsFrom": "PHASES"},
        {"id": "sf", "label": "SF", "type": "yesno", "options": ["Yes", "No"], "row": "top"},
        {"id": "case_logged", "label": "Case logged date", "type": "date", "row": "top"},
        {"id": "engagement_ref", "label": "Case ID", "type": "text", "placeholder": "Internal case / matter reference", "row": "ref"},
        {"id": "severity", "label": "Severity", "type": "select", "optionsFrom": "SEVERITIES", "row": "ref"},
        {"id": "engaged_via", "label": "Engaged via", "type": "select", "options": ["Client directly", "Partner / CSP", "Other"], "row": "origin"},
        {"id": "geolocation", "label": "Geolocation (country)", "type": "select", "optionsFrom": "COUNTRIES", "row": "origin"},
        {"id": "discovery_tz", "label": "Incident timezone", "type": "timezone", "help": "All date/time answers in this incident are recorded in this timezone and also shown in UTC.", "ask": "Which time zone are you working in, so I get the times right?"},
        {"id": "_clientName", "bind": "clientName", "label": "Client name", "type": "text", "ask": "Can I just confirm the full company name?"},
        {"id": "_title", "bind": "title", "label": "Incident title", "type": "text", "placeholder": "e.g. ACME Corp ransomware Oct 2026"},
        {"id": "tenant_id", "label": "Tenant ID", "type": "text", "placeholder": "Microsoft Entra tenant ID (GUID)", "help": "Found in the Entra admin centre under Overview, or in the Microsoft 365 admin centre.", "ask": "Could you give me your Microsoft 365 tenant ID?"},
        {"id": "subscription_ids", "label": "Azure subscription ID(s)", "type": "textarea", "placeholder": "One per line", "ask": "If you're on Azure, which subscription IDs are involved?"},
        {"id": "entra_licence", "label": "Entra ID licence", "type": "multiselect", "options": ["Entra ID Free", "Entra ID P1", "Entra ID P2", "Entra ID Governance", "Entra Suite", "Unknown"], "ask": "What Entra licences have you got? P1, P2?"},
        {"id": "m365_licence", "label": "Microsoft 365 / Office 365 licence", "type": "multiselect", "options": ["Microsoft 365 E3", "Microsoft 365 E5", "Office 365 E1", "Office 365 E3", "Office 365 E5", "Microsoft 365 Business Basic", "Microsoft 365 Business Standard", "Microsoft 365 Business Premium", "Microsoft 365 Frontline (F1 / F3)", "Microsoft 365 Education (A3 / A5)", "Microsoft 365 Government (G3 / G5)", "Microsoft 365 E5 Security add-on", "Microsoft 365 E5 Compliance add-on", "Unknown"], "ask": "And which Microsoft 365 licences are you on? E3, E5, Business Premium?"}
      ]
    },
    {
      "id": "discovery",
      "title": "Incident Details",
      "description": "When and how the incident came to light.",
      "questions": [
        {"id": "_types", "bind": "types", "label": "Incident types", "type": "multiselect", "optionsFrom": "INCIDENT_TYPES", "help": "Tick every type this case covers; each adds its own question sections. Answers for a type you untick are kept but hidden.", "ask": "In a sentence, what do you think has happened? Is anything else caught up in it, say Entra or email?"},
        {"id": "discovery_dt", "label": "Date of discovery", "type": "date", "ask": "When did you first notice something was wrong?"},
        {"id": "earliest_activity_dt", "label": "Earliest known suspicious activity (if known)", "type": "date", "ask": "Do you know when it might have actually started?"},
        {"id": "detection_source", "label": "How was it detected?", "type": "multiselect", "options": ["EDR / AV alert", "SIEM alert", "MDR / SOC notification", "User report", "IT noticed outage / performance issue", "Ransom note / extortion message", "Third-party notification (customer, supplier)", "Law enforcement / NCSC notification", "Bank / financial anomaly", "Threat intel / dark web monitoring", "Audit / internal review", "Other"], "ask": "How did you find out? Was it an alert, or did someone spot it?"},
        {"id": "detection_details", "label": "What the customer thinks happened", "type": "textarea", "help": "The customer's own account of events, in their words.", "ask": "In your own words, what do you think has happened?"},
        {"id": "activity_ongoing", "label": "Is malicious activity believed to be ongoing?", "type": "yesno", "ask": "Do you think they're still in, or has it gone quiet?"},
        {"id": "prior_incidents", "label": "Any related or prior incidents in the last 12 months?", "type": "yesno", "ask": "Have you had anything similar in the last year or so?"},
        {"id": "prior_incidents_details", "label": "Prior incident details", "type": "textarea", "showIf": {"q": "prior_incidents", "equals": "Yes"}, "ask": "What happened that time?"}
      ]
    },
    {
      "id": "environment",
      "title": "CX Environment",
      "description": "The customer's estate and security tooling, before we get into what is affected.",
      "questions": [
        {"id": "total_endpoints", "label": "Total endpoints (workstations / laptops)", "type": "number", "ask": "How many laptops and desktops have you got in total?"},
        {"id": "total_servers", "label": "Total servers (physical and virtual)", "type": "number", "ask": "And how many servers, roughly?"},
        {"id": "env_platforms", "label": "Platforms in the environment", "type": "multiselect", "options": ["On-prem Active Directory", "Entra ID (Azure AD)", "Hybrid identity (AD Connect)", "Microsoft 365", "Google Workspace", "AWS", "Azure", "GCP", "VMware ESXi / vCenter", "Hyper-V", "Citrix / VDI", "OT / ICS", "Mainframe"], "ask": "What does the setup look like? On-prem AD, Microsoft 365, Azure?"},
        {"id": "env_os", "label": "Operating systems in use", "type": "multiselect", "options": ["Windows", "Linux", "macOS", "ESXi", "Network appliances", "Mobile"], "ask": "Is it mostly Windows, or have you got Linux and Macs as well?"},
        {"id": "env_remote_access", "label": "Remote access in use", "type": "multiselect", "options": ["VPN", "RDP / RD Gateway", "Citrix / VDI", "Zero trust / ZTNA", "Third-party remote tools (e.g. AnyDesk, TeamViewer)", "None", "Unknown"], "ask": "How do people connect in remotely? VPN, Citrix, RDP?"},
        {"id": "env_perimeter", "label": "Firewall / VPN vendor", "type": "text", "placeholder": "e.g. Fortinet, Palo Alto, Cisco", "ask": "What firewall or VPN kit are you using?"},
        {"id": "edr_solution", "label": "EDR / AV solution", "type": "multiselect", "help": "Tick all that apply. If they only have the antivirus built into Windows, tick Microsoft Defender Antivirus only.", "options": ["Microsoft Defender for Endpoint", "Microsoft Defender XDR", "Microsoft Defender Antivirus only", "CrowdStrike Falcon", "SentinelOne Singularity", "Palo Alto Cortex XDR", "Sophos Intercept X", "Trend Micro Vision One", "VMware Carbon Black", "WithSecure Elements", "ESET PROTECT", "Bitdefender GravityZone", "Symantec Endpoint (Broadcom)", "Trellix Endpoint", "Other", "None"], "ask": "What do you use for endpoint protection? Defender, CrowdStrike, something else?"},
        {"id": "edr_product", "label": "EDR / AV product(s) deployed (earlier free-text answer)", "type": "text", "showIf": {"q": "edr_product", "notEmpty": true}},
        {"id": "edr_coverage", "label": "Estimated EDR coverage", "type": "select", "options": ["95-100%", "75-94%", "50-74%", "Under 50%", "No EDR", "Unknown"], "ask": "Is that on everything, or are there gaps?"},
        {"id": "siem_solution", "label": "SIEM / log platform", "type": "multiselect", "help": "Tick all that apply.", "options": ["Microsoft Sentinel", "Microsoft Log Analytics (no Sentinel)", "Splunk", "IBM QRadar", "Elastic Security", "Google Security Operations (Chronicle)", "Exabeam / LogRhythm", "Rapid7 InsightIDR", "Sumo Logic", "Datadog", "MSSP-managed SIEM", "Other", "None"], "ask": "Have you got a SIEM, or somewhere the logs end up? Sentinel, Splunk?"},
        {"id": "siem_product", "label": "SIEM / log platform (earlier free-text answer)", "type": "text", "showIf": {"q": "siem_product", "notEmpty": true}},
        {"id": "graph_api_logging", "label": "Is Microsoft Graph activity logging enabled?", "type": "yesno", "help": "MicrosoftGraphActivityLogs, sent through an Entra ID diagnostic setting to Log Analytics / Sentinel, storage or Event Hub. Off by default; needs Entra ID P1 or P2.", "ask": "Do you have Graph API activity logging switched on in Entra? It's not on by default."},
        {"id": "en_log_export", "label": "Are Entra sign-in and audit logs exported?", "type": "select", "options": ["To Sentinel / Log Analytics", "To another SIEM", "Storage account only", "Not exported (portal retention only)", "Unknown"], "help": "Without export, sign-in and audit logs are usually kept 7 days (free) or 30 days (P1 / P2).", "ask": "Do your Entra sign-in logs go into Sentinel or a SIEM?"},
        {"id": "en_ual", "label": "Is the Unified Audit Log (Purview audit) enabled?", "type": "yesno", "ask": "Is auditing switched on in Purview?"},
        {"id": "en_audit_tier", "label": "Purview audit tier", "type": "select", "options": ["Audit (Standard)", "Audit (Premium)", "Unknown"], "help": "Affects how far back audit records go.", "ask": "Is that standard or premium auditing?"},
        {"id": "en_xdr", "label": "Microsoft security tooling in use", "type": "multiselect", "options": ["Defender XDR portal", "Defender for Endpoint", "Defender for Identity", "Defender for Office 365", "Defender for Cloud Apps", "Entra ID Protection", "Sentinel", "None"], "ask": "Which Microsoft security tools are you using?"},
        {"id": "msp", "label": "MSP / MSSP involvement (name, scope of service)", "type": "text", "ask": "Is there an MSP or a SOC looking after you?"}
      ]
    },
    {
      "id": "affected",
      "title": "Affected Systems",
      "description": "What is known or suspected to be affected so far.",
      "questions": [
        {"id": "affected_systems_count", "label": "Number of systems known or suspected to be affected", "type": "number", "ask": "Roughly how many machines do you think are affected?"},
        {"id": "affected_system_types", "label": "Types of systems affected", "type": "multiselect", "options": ["User workstations / laptops", "File servers", "Domain controllers", "Application / database servers", "Email servers (Exchange)", "Web servers", "Hypervisors (ESXi / Hyper-V)", "Backup systems", "Cloud workloads", "Network devices (firewall / VPN)", "OT / ICS", "Unknown"], "ask": "What sort of systems are affected? Laptops, servers, domain controllers?"},
        {"id": "affected_hosts_list", "label": "Known affected hostnames / IPs", "type": "textarea", "placeholder": "One per line", "ask": "Have you got a list of the affected machine names or IPs?"},
        {"id": "affected_users_count", "label": "Number of user accounts known or suspected to be affected", "type": "number", "ask": "And roughly how many user accounts?"},
        {"id": "affected_privileged", "label": "Are any privileged or admin accounts affected?", "type": "yesno", "ask": "Are any admin accounts caught up in this?"},
        {"id": "affected_cloud", "label": "Are cloud tenants or subscriptions affected (Microsoft 365, Azure, other)?", "type": "yesno", "ask": "Is anything in the cloud affected, like 365 or Azure?"},
        {"id": "affected_third_party", "label": "Are any third-party or supplier-managed systems involved?", "type": "yesno", "ask": "Are any systems run by a supplier involved?"},
        {"id": "sites_affected", "label": "Sites, regions or business units affected", "type": "textarea", "ask": "Is it just the one site, or are other offices or countries affected too?"}
      ]
    },
    {
      "id": "impact",
      "title": "Business impact",
      "description": "Operational and data impact as currently understood.",
      "questions": [
        {"id": "impact_level", "label": "Overall business impact", "type": "select", "options": ["Critical: core operations halted", "High: significant degradation", "Moderate: limited disruption", "Low: no noticeable disruption", "Unknown"], "ask": "How badly is the business affected right now?"},
        {"id": "impacted_functions", "label": "Business functions impacted", "type": "multiselect", "options": ["Production / manufacturing", "Customer-facing services", "Finance / payments", "Payroll / HR", "Email / collaboration", "Logistics / supply chain", "Clinical / patient care", "Website / e-commerce", "Internal IT only", "None"], "ask": "Which parts of the business are feeling it?"},
        {"id": "critical_systems_down", "label": "Critical systems currently unavailable", "type": "textarea", "ask": "Is anything critical down at the moment?"},
        {"id": "data_at_risk", "label": "Data types potentially at risk", "type": "multiselect", "optionsFrom": "DATA_TYPES", "ask": "What sort of data could be involved? Personal data, financial, anything sensitive?"},
        {"id": "records_estimate", "label": "Estimated number of individuals / records affected", "type": "number", "ask": "Any idea how many people or records that might cover?"},
        {"id": "impact_cost", "label": "Estimated cost of disruption (per day or total, with currency)", "type": "text", "ask": "Have you got a feel for what this is costing you?"},
        {"id": "media_attention", "label": "Is there media or public attention?", "type": "yesno", "ask": "Has the press or anyone outside picked up on it yet?"}
      ]
    },
    {
      "id": "evidence",
      "title": "Evidence",
      "description": "What evidence exists, how long it will survive, and how we can get it.",
      "questions": [
        {"id": "evidence_preserved", "label": "Has any evidence been preserved?", "type": "yesno", "ask": "Have you kept hold of any evidence so far?"},
        {"id": "evidence_types", "label": "Evidence preserved so far", "type": "multiselect", "showIf": {"q": "evidence_preserved", "equals": "Yes"}, "options": ["Memory images", "Full disk images", "Triage collections (KAPE / Velociraptor / CyLR)", "VM snapshots", "EDR telemetry export", "Windows event logs", "Firewall logs", "Proxy / web gateway logs", "VPN logs", "DNS logs", "NetFlow", "M365 Unified Audit Log", "Entra sign-in / audit logs", "Cloud audit logs (CloudTrail / Activity Log)", "Email samples", "Malware samples"], "ask": "What have you got? Disk images, logs, triage collections?"},
        {"id": "log_retention", "label": "Retention of key log sources", "type": "textarea", "help": "e.g. firewall 30 days, EDR 7 days, SIEM 90 days. Flag anything about to roll over.", "ask": "How long do you keep your logs? Is anything about to roll over?"},
        {"id": "chain_of_custody", "label": "Is chain of custody being recorded?", "type": "yesno", "ask": "Is anyone keeping a record of who's handled the evidence?"},
        {"id": "evidence_location", "label": "Where is preserved evidence stored?", "type": "text", "ask": "Where is that evidence stored at the moment?"},
        {"id": "remote_collection", "label": "Can we deploy collection tooling remotely (EDR / Velociraptor)?", "type": "yesno", "ask": "Could we push a collection tool out remotely, or would it be hands-on?"},
        {"id": "access_requirements", "label": "Access requirements (accounts, VPN, jump host, approvals)", "type": "textarea", "ask": "What would we need to get access? Accounts, VPN, approvals?"}
      ]
    },
    {
      "id": "containment",
      "title": "Containment So Far",
      "group": "Containment",
      "short": "Overview",
      "description": "The overall picture. The next sections go through containment area by area: endpoints, Active Directory, Entra ID, email, Azure, network and applications.",
      "questions": [
        {"id": "containment_actions", "label": "Actions already taken (earlier answer)", "type": "multiselect", "options": ["Hosts isolated (network / EDR)", "Internet egress blocked or restricted", "VPN / remote access disabled", "Accounts disabled", "Password resets (users)", "Password resets (privileged / service accounts)", "krbtgt reset", "Sessions / tokens revoked", "MFA re-registered", "IOCs blocked at firewall / proxy", "Systems powered off", "Backups isolated", "External access (RDP, etc.) closed", "None yet", "Other"], "showIf": {"q": "containment_actions", "notEmpty": true}, "ask": "What have you done so far to contain it?"},
        {"id": "containment_details", "label": "Containment details and timings", "type": "textarea", "ask": "Roughly when did you do those things?"},
        {"id": "systems_rebuilt", "label": "Have any affected systems been wiped, reimaged or restored?", "type": "yesno", "ask": "Have you wiped, rebuilt or restored anything yet?"},
        {"id": "systems_rebuilt_details", "label": "Which systems, and was anything captured first?", "type": "textarea", "showIf": {"q": "systems_rebuilt", "equals": "Yes"}, "ask": "Which ones, and did you grab any images or logs first?"},
        {"id": "ta_access_removed", "label": "Is the threat actor believed to be locked out?", "type": "yesno", "ask": "Are you confident they've been kicked out?"},
        {"id": "third_party_ir", "label": "Has any other IR firm or MSP performed work so far?", "type": "yesno", "ask": "Has anyone else, like another IR firm or your MSP, already done some work on this?"},
        {"id": "third_party_ir_details", "label": "Who, what was done, and are their outputs available?", "type": "textarea", "showIf": {"q": "third_party_ir", "equals": "Yes"}, "ask": "What did they do, and can we see what they found?"},
        {"id": "containment_evidence", "label": "Was evidence preserved before containment?", "type": "yesno", "help": "For example logs exported, memory or disk captured, mailbox content held, before resets, wipes, deletions or reimaging.", "ask": "Did anyone save the logs or take images before cleaning things up?"},
        {"id": "containment_pending", "label": "Containment still planned or blocked", "type": "textarea", "ask": "Is there anything you want to do but haven't been able to yet?"}
      ]
    },
    {
      "id": "ct_endpoint",
      "title": "Containment: Endpoints & servers",
      "group": "Containment",
      "short": "Endpoints & servers",
      "description": "Workstations, servers, hypervisors and backups.",
      "questions": [
        {"id": "ct_ep_actions", "label": "Actions taken on endpoints and servers", "type": "multiselect", "options": ["Isolated in Defender for Endpoint (device isolation)", "Isolated by another EDR (e.g. CrowdStrike, SentinelOne)", "Network cable / Wi-Fi disconnected", "Powered off or suspended", "App execution restricted (Defender)", "Defender full or offline scan run", "Malicious files quarantined", "File hashes, IPs or URLs added as Defender custom indicators", "Remote access tools removed (e.g. AnyDesk, TeamViewer, ScreenConnect)", "Persistence removed (services, scheduled tasks, run keys)", "Local admin passwords rotated (LAPS)", "Removable media blocked", "Hypervisors / vCenter isolated", "Backup servers isolated and backups protected", "Reimaged or rebuilt", "None yet"], "ask": "What have you done with the machines? Isolated them in Defender, pulled the network, wiped any?"},
        {"id": "ct_ep_details", "label": "Which hosts, what was done, who and when", "type": "textarea", "ask": "Which machines, and when was that done?"}
      ]
    },
    {
      "id": "ct_ad",
      "title": "Containment: Active Directory",
      "group": "Containment",
      "short": "Active Directory",
      "description": "On-prem accounts, domain controllers, Group Policy, PKI and hybrid identity servers.",
      "questions": [
        {"id": "ct_ad_accounts", "label": "Accounts and credentials", "type": "multiselect", "options": ["Compromised user accounts disabled", "Compromised user passwords reset", "Privileged (Tier 0) account passwords reset", "Service account passwords reset", "krbtgt reset once", "krbtgt reset twice", "Rogue accounts disabled or deleted", "Rogue group memberships removed", "Computer accounts disabled or reset", "Defender for Identity response actions used (e.g. disable user)", "None yet"], "ask": "In AD, what have you done with accounts? Disabled them, reset passwords, done krbtgt?"},
        {"id": "ct_ad_infra", "label": "Infrastructure and hybrid identity", "type": "multiselect", "options": ["Malicious GPOs unlinked or removed", "Domain controllers isolated or firewalled", "Admin logon restricted to PAWs / Tier 0 hosts", "AD CS templates disabled / rogue certificates revoked", "Entra Connect / Cloud Sync server isolated", "AD DS connector and sync account passwords reset", "ADFS token-signing certificate rolled twice", "Seamless SSO (AZUREADSSOACC) key rolled over", "PTA agents rebuilt", "None yet"], "ask": "Anything on the infrastructure side? Group Policy, the DCs, Entra Connect or ADFS?"},
        {"id": "ct_ad_details", "label": "Active Directory containment details (what, who, when)", "type": "textarea", "ask": "Who did that in AD, and when?"}
      ]
    },
    {
      "id": "ct_entra",
      "title": "Containment: Entra ID & Microsoft 365",
      "group": "Containment",
      "short": "Entra ID & Microsoft 365",
      "description": "Cloud identities, admin roles, apps and tenant settings.",
      "questions": [
        {"id": "ct_en_accounts", "label": "User and admin accounts", "type": "multiselect", "options": ["Passwords reset", "Sessions revoked (Revoke sessions)", "Accounts blocked from sign-in", "Attacker MFA / Temporary Access Pass methods removed", "Users confirmed compromised in ID Protection", "Registered devices disabled or removed", "Admin role assignments removed", "PIM eligibilities removed", "Rogue users and guests removed", "Break-glass accounts checked and credentials rotated", "None yet"], "ask": "In Entra, what have you done with the accounts? Reset them, revoked sessions, removed MFA methods?"},
        {"id": "ct_en_tenant", "label": "Tenant, apps and workload identities", "type": "multiselect", "options": ["Conditional Access tightened (block, compliant device, locations)", "Attacker IPs blocked (named locations)", "Authentication methods policy tightened (e.g. SMS, Temporary Access Pass)", "Malicious app registrations and secrets removed", "Service principal secrets, certificates or federated credentials rotated", "Service principals disabled", "OAuth consent grants revoked", "User consent restricted / admin consent workflow enabled", "Conditional Access for workload identities applied", "Rogue federated domains or federation settings removed", "Entra Connect sync paused", "Cross-tenant access / partner (GDAP) access removed", "None yet"], "ask": "Any tenant-wide changes? Conditional Access, app consents, service principals?"},
        {"id": "ct_en_details", "label": "Entra ID and Microsoft 365 containment details (what, who, when)", "type": "textarea", "ask": "Who made those changes, and when?"}
      ]
    },
    {
      "id": "ct_email",
      "title": "Containment: Email",
      "group": "Containment",
      "short": "Email",
      "description": "Exchange Online or Exchange Server mail flow and mailboxes.",
      "questions": [
        {"id": "ct_mail_actions", "label": "Actions taken on email", "type": "multiselect", "options": ["Inbox rules removed", "External forwarding removed", "Outbound auto-forwarding blocked (anti-spam policy)", "Mailbox delegates and permissions removed", "Phishing emails purged (Threat Explorer, ZAP or content search)", "Senders, domains or URLs blocked (Tenant Allow/Block List)", "Malicious transport rules or connectors removed", "Mailbox export requests removed", "Recipients of attacker emails warned", "None yet"], "ask": "On the email side, have you cleared rules, pulled the phishing emails, blocked the sender?"},
        {"id": "ct_mail_details", "label": "Email containment details (what, who, when)", "type": "textarea", "ask": "When was that done, and by whom?"}
      ]
    },
    {
      "id": "ct_azure",
      "title": "Containment: Azure",
      "group": "Containment",
      "short": "Azure",
      "description": "Subscriptions, resources and workload identities.",
      "questions": [
        {"id": "ct_az_actions", "label": "Actions taken in Azure", "type": "multiselect", "options": ["RBAC role assignments removed", "Elevated access (root scope) removed", "Managed identity role assignments removed / identity detached", "Storage account keys rotated (invalidates key-based SAS)", "Key Vault secrets, keys or certificates rotated", "Malicious resources stopped or deleted (after snapshot)", "Public network access / NSGs locked down", "Resource locks applied", "Azure Policy deny assignments added", "App Service publishing credentials reset / basic auth disabled", "Defender for Cloud recommendations applied", "Support case raised with Microsoft", "None yet"], "ask": "In Azure, have you pulled any roles, rotated keys or shut anything down?"},
        {"id": "ct_az_details", "label": "Azure containment details (what, who, when)", "type": "textarea", "ask": "Who did that in Azure, and when?"}
      ]
    },
    {
      "id": "ct_network",
      "title": "Containment: Network & perimeter",
      "group": "Containment",
      "short": "Network & perimeter",
      "description": "Internet access, VPN, firewalls and third-party connections.",
      "questions": [
        {"id": "ct_net_actions", "label": "Actions taken on the network", "type": "multiselect", "options": ["Internet access cut", "Sites or network segments disconnected", "VPN disabled or restricted", "VPN / firewall patched or hardened", "Exposed RDP / remote access closed", "C2 IPs and domains blocked (firewall, proxy, DNS)", "Exfiltration destinations / cloud storage blocked", "Unsanctioned cloud apps blocked (Defender for Cloud Apps)", "Third-party / supplier access disabled", "Geo-blocking applied", "Firewall / VPN admin interfaces removed from the internet", "None yet"], "ask": "On the network, have you cut internet access, turned off the VPN, blocked any IPs?"},
        {"id": "ct_net_details", "label": "Network containment details (what, who, when)", "type": "textarea", "ask": "When did that happen, and who did it?"}
      ]
    },
    {
      "id": "ct_app",
      "title": "Containment: Applications & web services",
      "group": "Containment",
      "short": "Applications & web services",
      "description": "Web apps, SharePoint, Exchange Server and other line-of-business applications.",
      "questions": [
        {"id": "ct_app_actions", "label": "Actions taken on applications", "type": "multiselect", "options": ["Application or site taken offline / maintenance page", "WAF rules or virtual patching applied", "Vulnerability patched", "Web shells removed", "App credentials, API keys or connection strings rotated", "App pool / service account passwords reset", "Rebuilt from known-good source", "SharePoint machine keys rotated and IIS restarted", "AMSI enabled and full scan run (SharePoint / Exchange)", "Exchange emergency mitigations or latest security update applied", "Exchange HealthChecker / Microsoft Safety Scanner run", "Sharing links removed / external sharing restricted", "None yet"], "ask": "Have you done anything with the applications? Taken the site down, patched it, rotated keys?"},
        {"id": "ct_app_details", "label": "Application containment details (what, who, when)", "type": "textarea", "ask": "When was that, and who did it?"}
      ]
    }
  ]
}
```

### DATA FILE: json/questionnaire/closing-sections.json

```json
{
  "sections": [
    {
      "id": "scope",
      "title": "Objectives & scope",
      "description": "What the client needs from us, so scope and estimate are right first time.",
      "questions": [
        {"id": "objectives", "label": "Key questions the client needs answered", "type": "textarea", "help": "e.g. How did they get in? Was data taken? Are they still in?", "ask": "What do you most need to know from us?"},
        {"id": "services", "label": "Services requested", "type": "multiselect", "options": ["Forensic investigation", "Threat hunting / compromise assessment", "Containment & eradication support", "Recovery support", "Threat actor communications support", "Dark web monitoring", "Regulatory / legal reporting support", "Executive briefing"], "ask": "What would you like us to help with?"},
        {"id": "deliverables", "label": "Expected deliverables", "type": "multiselect", "options": ["Verbal briefings", "Daily status updates", "Interim findings", "Full technical report", "Executive summary", "IOC list", "Remediation recommendations"], "ask": "What do you need from us at the end? A report, regular updates?"},
        {"id": "next_steps", "label": "Agreed next steps and owners", "type": "textarea", "ask": "So, just to agree the next steps, who's doing what?"},
        {"id": "scoping_notes", "label": "Other scoping notes", "type": "textarea", "ask": "Is there anything else we should know?"}
      ]
    }
  ]
}
```

### DATA FILE: json/questionnaire/report-checks-common.json

```json
{
  "reportChecks": [
    {"id": "common__initial_access", "label": "Has the initial access vector been identified (or stated as undetermined, with the reason)?"},
    {"id": "common__attacker_window", "label": "Have the first and the most recent attacker activity been established?"},
    {"id": "common__patient_zero", "label": "Has the first compromised host or account been identified?"},
    {"id": "common__scope_hosts", "label": "Has the full list of affected hosts been confirmed?"},
    {"id": "common__scope_accounts", "label": "Has the full list of affected accounts been confirmed?"},
    {"id": "common__timeline", "label": "Is the incident timeline complete, from initial access to containment?"},
    {"id": "common__persistence", "label": "Have persistence mechanisms been checked for and documented?"},
    {"id": "common__lateral", "label": "Has lateral movement been assessed?"},
    {"id": "common__data_access", "label": "Has data access or exfiltration been assessed?"},
    {"id": "common__lockout", "label": "Has it been confirmed that the attacker no longer has access?"},
    {"id": "common__containment_validated", "label": "Have the client's containment actions been reviewed and validated?"},
    {"id": "common__iocs", "label": "Have all IOCs been recorded?"},
    {"id": "common__attack_mapping", "label": "Have the attacker techniques been mapped to MITRE ATT&CK?"},
    {"id": "common__evidence_gaps", "label": "Are evidence sources, retention limits and gaps documented?"},
    {"id": "common__root_cause", "label": "Is the root cause clearly stated?"},
    {"id": "common__recommendations", "label": "Are prioritised recommendations included?"},
    {"id": "common__exec_summary", "label": "Is there an executive summary written for a non-technical reader?"}
  ]
}
```

### DATA FILE: json/incident-types/ransomware.json

```json
{
  "incidentType": {"id": "ransomware", "label": "Ransomware"},
  "sections": [
    {
      "id": "rw_actor",
      "title": "Ransom note & threat actor",
      "description": "Threat actor identification and extortion details.",
      "questions": [
        {"id": "rw_note_found", "label": "Has a ransom note been found?", "type": "yesno", "ask": "Have you found a ransom note?"},
        {"id": "rw_note_details", "label": "Ransom note filename, contents or contact method", "type": "textarea", "showIf": {"q": "rw_note_found", "equals": "Yes"}, "ask": "What does it say, and how do they want you to get in touch?"},
        {"id": "rw_variant", "label": "Suspected variant / group", "type": "text", "placeholder": "e.g. Akira, LockBit, Play", "ask": "Does the note say who they are, or have you got a name for the group?"},
        {"id": "rw_extension", "label": "Encrypted file extension", "type": "text", "ask": "What extension are the encrypted files showing?"},
        {"id": "rw_ta_contact", "label": "Has anyone contacted the threat actor or visited the negotiation portal?", "type": "yesno", "ask": "Has anyone been in touch with them or visited their site yet?"},
        {"id": "rw_negotiator", "label": "Is a negotiator engaged? (firm and contact)", "type": "text", "showIf": {"q": "rw_ta_contact", "equals": "Yes"}, "ask": "Have you got a negotiator involved?"},
        {"id": "rw_demand", "label": "Ransom demand (amount and currency)", "type": "text", "ask": "How much are they asking for?"},
        {"id": "rw_deadline", "label": "Threat actor deadline", "type": "datetime", "ask": "Have they given you a deadline?"},
        {"id": "rw_exfil_claimed", "label": "Does the threat actor claim data theft?", "type": "yesno", "ask": "Are they saying they've taken data as well?"},
        {"id": "rw_leak_site", "label": "Is the client listed on a leak site?", "type": "yesno", "ask": "Have you shown up on their leak site yet?"},
        {"id": "rw_payment_stance", "label": "Client position on payment", "type": "select", "options": ["Will not pay", "Considering", "Undecided", "Awaiting legal / insurer advice"], "ask": "Where are you on paying, at this stage?"},
        {
          "id": "rw_sanctions",
          "label": "Has sanctions screening (OFSI / OFAC) been considered?",
          "type": "yesno",
          "showIf": {"q": "rw_payment_stance", "in": ["Considering", "Undecided", "Awaiting legal / insurer advice"]},
          "ask": "Has anyone looked at sanctions checks, just in case payment comes up?"
        }
      ]
    },
    {
      "id": "rw_scope",
      "title": "Encryption scope & recovery",
      "description": "Blast radius, identity infrastructure and recoverability.",
      "questions": [
        {"id": "rw_encryption_start", "label": "When did encryption start (first observed)?", "type": "datetime", "ask": "When did the encryption kick off, as far as you can tell?"},
        {"id": "rw_encrypted_types", "label": "System types encrypted", "type": "multiselect", "options": ["Workstations", "File servers", "Database servers", "Application servers", "Domain controllers", "ESXi / hypervisors", "NAS / SAN", "Backup servers", "Cloud VMs", "OT / ICS"], "ask": "What's been encrypted? Laptops, servers, file shares?"},
        {"id": "rw_encrypted_count", "label": "Approximate number of encrypted systems", "type": "number", "ask": "Roughly how many machines are encrypted?"},
        {"id": "rw_dc_impact", "label": "Are domain controllers encrypted or compromised?", "type": "yesno", "ask": "Have the domain controllers been hit?"},
        {
          "id": "rw_dc_details",
          "label": "DC details (how many, which sites, any healthy DCs left)",
          "type": "textarea",
          "showIf": {"q": "rw_dc_impact", "in": ["Yes", "Unknown"]},
          "ask": "How many DCs have you got, and are any still healthy?"
        },
        {"id": "rw_hypervisor", "label": "Were hypervisors / vCenter targeted?", "type": "yesno", "ask": "Did they go after VMware or Hyper-V as well?"},
        {"id": "rw_backup_status", "label": "Backup status", "type": "select", "options": ["Intact, offline / immutable", "Intact but online / reachable", "Partially affected", "Encrypted or deleted", "No backups", "Unknown"], "ask": "What state are your backups in?"},
        {"id": "rw_backup_product", "label": "Backup product and storage location", "type": "text", "ask": "What do you use for backups, and where are they kept?"},
        {"id": "rw_last_good_backup", "label": "Last known good backup", "type": "datetime", "ask": "When was your last good backup?"},
        {"id": "rw_backup_tampering", "label": "Evidence of backup tampering (deleted jobs, VSS deletion, console access)?", "type": "yesno", "ask": "Any sign they tried to delete or tamper with the backups?"},
        {"id": "rw_initial_access", "label": "Suspected initial access vector", "type": "multiselect", "options": ["VPN / firewall vulnerability", "Exposed RDP", "Phishing", "Valid credentials (no MFA)", "Third-party / supplier access", "Unpatched public-facing application", "Unknown"], "ask": "Any idea how they got in?"},
        {"id": "rw_deploy_method", "label": "How was the ransomware deployed?", "type": "multiselect", "options": ["Group Policy (scheduled task / startup script)", "PsExec / remote services", "SCCM / MECM", "Intune", "RMM tool", "WMI / PowerShell remoting", "Manually via RDP", "ESXi shell / SSH", "Unknown"], "ask": "Do you know how it got pushed out? Group Policy, PsExec, SCCM?"},
        {"id": "rw_defender_tamper", "label": "Was Microsoft Defender disabled or tampered with before encryption?", "type": "yesno", "help": "Check Defender for Endpoint for tamper protection alerts, and whether tamper protection was on.", "ask": "Did they switch off Defender, or try to, before it kicked off?"},
        {"id": "rw_cloud_impact", "label": "Are Microsoft 365 or Azure resources also affected?", "type": "multiselect", "options": ["OneDrive / SharePoint files encrypted via sync", "Azure VMs encrypted", "Azure Backup / Recovery Services vault tampered with", "Entra ID accounts compromised", "None known", "Unknown"], "ask": "Has anything in the cloud been hit as well? OneDrive, Azure VMs, backups?"},
        {"id": "rw_decryptor", "label": "Has a public decryptor been checked (e.g. No More Ransom)?", "type": "yesno", "ask": "Has anyone checked whether there's a free decryptor out there?"},
        {"id": "rw_recovery_priorities", "label": "Recovery priorities (systems in order)", "type": "textarea", "ask": "What do you need back up and running first?"}
      ]
    }
  ],
  "reportChecks": [
    {"id": "ransomware__variant", "label": "Has the ransomware variant or threat actor been identified?"},
    {"id": "ransomware__encryption_start", "label": "Has the time encryption started been established?"},
    {"id": "ransomware__deploy_method", "label": "Has the deployment method been identified (GPO, PsExec, SCCM, RMM, ESXi shell)?"},
    {"id": "ransomware__encrypted_list", "label": "Is there a complete list of encrypted systems?"},
    {"id": "ransomware__domain_privilege", "label": "Has it been established how domain-level privilege was obtained?"},
    {"id": "ransomware__tool_tampering", "label": "Has security tool tampering been checked (Defender disabled, EDR removed)?"},
    {"id": "ransomware__backups", "label": "Has backup deletion or tampering been assessed?"},
    {"id": "ransomware__theft_claim", "label": "Has the threat actor's data theft claim been tested against evidence (e.g. rclone, MEGA, egress volumes)?"},
    {"id": "ransomware__leak_site", "label": "Has the leak site been checked for the client?"},
    {"id": "ransomware__dwell_time", "label": "Has the dwell time from initial access to encryption been calculated?"},
    {"id": "ransomware__clean_restore", "label": "Have restored systems been confirmed clean before reconnection?"}
  ]
}
```

### DATA FILE: json/incident-types/bec.json

```json
{
  "incidentType": {"id": "bec", "label": "Business Email Compromise"},
  "sections": [
    {
      "id": "bec_tenant",
      "title": "Mail platform & identity",
      "description": "Tenant, logging and identity controls.",
      "questions": [
        {"id": "bec_platform", "label": "Mail platform", "type": "select", "options": ["Microsoft 365 / Exchange Online", "Exchange on-prem", "Hybrid Exchange", "Google Workspace", "Other"], "ask": "Is your email on Microsoft 365, or on-prem Exchange?"},
        {
          "id": "bec_tenant",
          "label": "Primary email domain(s)",
          "type": "text",
          "showIf": {"q": "bec_platform", "in": ["Microsoft 365 / Exchange Online", "Hybrid Exchange"]},
          "placeholder": "e.g. contoso.com",
          "ask": "What's your main email domain?"
        },
        {"id": "bec_mailitemsaccessed", "label": "Is MailItemsAccessed auditing available for the affected mailboxes?", "type": "yesno", "help": "MailItemsAccessed shows which emails were read; check the mailbox audit settings for the affected users.", "ask": "Can we see which emails were actually opened? Is MailItemsAccessed being logged?"},
        {"id": "bec_admin_access", "label": "Can we be granted read-only admin roles (Global Reader, Security Reader)?", "type": "yesno", "ask": "Could you give us read-only admin access so we can have a look?"},
        {"id": "bec_affected_count", "label": "Number of affected mailboxes", "type": "number", "ask": "How many mailboxes do you think are affected?"},
        {"id": "bec_affected_mailboxes", "label": "Affected mailboxes / accounts", "type": "textarea", "placeholder": "One per line", "ask": "Which mailboxes are they?"},
        {"id": "bec_privileged", "label": "Are any affected accounts privileged (admin roles)?", "type": "yesno", "ask": "Are any of those admin accounts?"},
        {"id": "bec_mfa_status", "label": "MFA status for affected users", "type": "select", "options": ["Enforced for all users", "Enforced for some users", "Not enabled", "Unknown"], "ask": "Is MFA switched on for those users?"},
        {
          "id": "bec_mfa_methods",
          "label": "MFA methods in use",
          "type": "multiselect",
          "showIf": {"q": "bec_mfa_status", "in": ["Enforced for all users", "Enforced for some users"]},
          "options": ["SMS / voice", "Authenticator push", "Push with number matching", "TOTP code", "FIDO2 / passkey", "Windows Hello"],
          "ask": "What kind of MFA do they use? App notifications, text messages?"
        },
        {"id": "bec_ca", "label": "Are Conditional Access policies in place?", "type": "yesno", "ask": "Do you use Conditional Access?"},
        {"id": "bec_legacy_auth", "label": "Is legacy authentication blocked?", "type": "yesno", "ask": "Is legacy authentication blocked?"},
        {"id": "bec_aitm", "label": "Is AiTM phishing or token theft suspected?", "type": "yesno", "ask": "Do you think it was a phishing link that stole their session?"}
      ]
    },
    {
      "id": "bec_activity",
      "title": "Mailbox activity & financial loss",
      "description": "Attacker actions in the mailbox and any resulting fraud.",
      "questions": [
        {"id": "bec_rules", "label": "Suspicious inbox rules found?", "type": "yesno", "ask": "Have you spotted any odd inbox rules?"},
        {"id": "bec_rules_details", "label": "Inbox rule details (names, conditions, actions)", "type": "textarea", "showIf": {"q": "bec_rules", "equals": "Yes"}, "ask": "What do the rules do?"},
        {"id": "bec_forwarding", "label": "External forwarding configured?", "type": "yesno", "ask": "Is any mail being forwarded outside the company?"},
        {"id": "bec_oauth", "label": "Suspicious OAuth app consents or new app registrations?", "type": "yesno", "ask": "Have any strange apps been given access to the mailbox?"},
        {"id": "bec_mfa_added", "label": "New MFA methods or devices registered by the attacker?", "type": "yesno", "ask": "Have you seen any new phones or MFA methods added?"},
        {"id": "bec_phish_sent", "label": "Was the mailbox used to send phishing (internal or external)?", "type": "yesno", "ask": "Did they send emails from the account?"},
        {"id": "bec_phish_count", "label": "Approximate number of recipients", "type": "number", "showIf": {"q": "bec_phish_sent", "equals": "Yes"}, "ask": "Roughly how many people got those emails?"},
        {"id": "bec_mailbox_perms", "label": "Were mailbox permissions or delegates added (FullAccess, SendAs, folder permissions)?", "type": "yesno", "ask": "Has anyone been given access to the mailbox, or send-as rights?"},
        {"id": "bec_ediscovery", "label": "Any sign of eDiscovery or content searches run by the attacker?", "type": "yesno", "ask": "Any sign they ran searches across mailboxes using eDiscovery?"},
        {"id": "bec_file_access", "label": "Evidence of SharePoint / OneDrive / Drive access by the attacker?", "type": "yesno", "ask": "Any sign they went into SharePoint or OneDrive?"},
        {"id": "bec_fraud", "label": "Fraudulent payment or bank detail change?", "type": "yesno", "ask": "Has any money gone out, or have bank details been changed?"},
        {"id": "bec_loss", "label": "Financial loss (amount and currency)", "type": "text", "showIf": {"q": "bec_fraud", "equals": "Yes"}, "ask": "How much are we talking about?"},
        {"id": "bec_payment_date", "label": "Date of fraudulent payment", "type": "date", "showIf": {"q": "bec_fraud", "equals": "Yes"}, "ask": "When did the payment go out?"},
        {"id": "bec_bank_recall", "label": "Has the bank been contacted to recall funds?", "type": "yesno", "showIf": {"q": "bec_fraud", "equals": "Yes"}, "ask": "Have you been on to the bank to try and get it back?"},
        {"id": "bec_recall_status", "label": "Recall status / bank reference", "type": "text", "showIf": {"q": "bec_bank_recall", "equals": "Yes"}, "ask": "What did the bank say?"},
        {"id": "bec_third_parties", "label": "Suppliers or customers impersonated or targeted", "type": "textarea", "ask": "Were any suppliers or customers impersonated or contacted?"},
        {"id": "bec_remediated", "label": "Have sessions been revoked and passwords reset for affected accounts?", "type": "yesno", "ask": "Have you reset the passwords and signed them out everywhere?"}
      ]
    }
  ],
  "reportChecks": [
    {"id": "bec__first_signin", "label": "Has the first malicious sign-in been identified, with IP and location?"},
    {"id": "bec__aitm", "label": "Has AiTM phishing or token theft been confirmed or ruled out?"},
    {"id": "bec__inbox_rules", "label": "Have inbox rules and forwarding been reviewed on every affected mailbox?"},
    {"id": "bec__mailitems", "label": "Has MailItemsAccessed (or equivalent) been reviewed to show which emails were read?"},
    {"id": "bec__sent_mail", "label": "Have emails sent by the attacker been identified?"},
    {"id": "bec__recipients", "label": "Have external recipients of attacker emails been identified?"},
    {"id": "bec__mfa_changes", "label": "Have MFA method changes on the affected accounts been checked?"},
    {"id": "bec__oauth", "label": "Have OAuth consents and app registrations been checked?"},
    {"id": "bec__files", "label": "Has SharePoint / OneDrive access by the attacker been assessed?"},
    {"id": "bec__other_mailboxes", "label": "Have other mailboxes been checked for the same IPs and indicators?"},
    {"id": "bec__fraud", "label": "Is any fraudulent payment or bank detail change documented, with amounts and dates?"}
  ]
}
```

### DATA FILE: json/incident-types/malware.json

```json
{
  "incidentType": {"id": "malware", "label": "Malware / Endpoint Compromise"},
  "sections": [
    {
      "id": "mw_details",
      "title": "Malware details",
      "description": "What was found and where.",
      "questions": [
        {"id": "mw_family", "label": "Suspected malware family / tooling", "type": "text", "placeholder": "e.g. Qakbot, Cobalt Strike, infostealer", "ask": "Do you know what the malware is?"},
        {"id": "mw_detection_name", "label": "Detection name(s) from security tooling", "type": "text", "ask": "What did your security tool call it?"},
        {"id": "mw_sample", "label": "Is a sample available?", "type": "yesno", "ask": "Have you got a copy of the file?"},
        {"id": "mw_iocs", "label": "Known IOCs (hashes, file paths, domains, IPs)", "type": "textarea", "ask": "Have you got any hashes, domains or IPs you can share?"},
        {"id": "mw_delivery", "label": "Suspected delivery vector", "type": "multiselect", "options": ["Email attachment", "Malicious link", "Drive-by / SEO poisoning / malvertising", "Removable media", "Software supply chain", "Exploit of public-facing service", "Cracked / pirated software", "Unknown"], "ask": "How do you think it got onto the machine?"},
        {"id": "mw_host_count", "label": "Number of hosts affected", "type": "number", "ask": "How many machines have got it?"},
        {"id": "mw_host_roles", "label": "Roles of affected hosts", "type": "multiselect", "options": ["User workstations", "Privileged admin workstations", "Servers", "Domain controllers", "Web servers", "Developer machines", "Cloud workloads"], "ask": "What kind of machines are they? User laptops, servers?"},
        {"id": "mw_hosts", "label": "Affected hostnames", "type": "textarea", "placeholder": "One per line", "ask": "Could you send me the machine names?"},
        {"id": "mw_patient_zero", "label": "Suspected patient zero (host / user)", "type": "text", "ask": "Do you know which machine or user it started with?"},
        {"id": "mw_mde_onboarded", "label": "Are the affected hosts onboarded to Defender for Endpoint?", "type": "select", "options": ["All affected hosts", "Some", "None", "Unknown"], "ask": "Are those machines in Defender for Endpoint?"},
        {"id": "mw_av_mode", "label": "Microsoft Defender Antivirus mode on affected hosts", "type": "select", "options": ["Active", "Passive (third-party AV primary)", "Disabled", "Unknown"], "ask": "Is Defender antivirus actually active on them, or running alongside something else?"},
        {"id": "mw_asr", "label": "Are attack surface reduction (ASR) rules enabled?", "type": "select", "options": ["Block mode", "Audit mode", "Not configured", "Unknown"], "ask": "Have you got ASR rules switched on?"},
        {"id": "mw_inv_package", "label": "Can a Defender investigation package or Live Response collection be taken?", "type": "yesno", "ask": "Could we pull an investigation package from Defender for the affected machines?"}
      ]
    },
    {
      "id": "mw_behaviour",
      "title": "Behaviour & spread",
      "description": "Post-compromise activity and current host state.",
      "questions": [
        {"id": "mw_edr_result", "label": "Security tooling outcome", "type": "select", "options": ["Blocked", "Detected, not blocked", "Not detected", "No EDR on host", "Unknown"], "ask": "Did your security tool block it, or just flag it?"},
        {"id": "mw_c2", "label": "Command and control traffic observed?", "type": "yesno", "ask": "Have you seen it talking out to the internet?"},
        {"id": "mw_c2_details", "label": "C2 indicators and timeframe", "type": "textarea", "showIf": {"q": "mw_c2", "equals": "Yes"}, "ask": "Where was it talking to, and when?"},
        {"id": "mw_persistence", "label": "Persistence mechanisms identified", "type": "multiselect", "options": ["Services", "Scheduled tasks", "Run keys", "WMI subscriptions", "Startup folder", "Remote access tools (AnyDesk, etc.)", "None identified", "Unknown"], "ask": "Have you found how it's staying on the machine?"},
        {"id": "mw_cred_access", "label": "Indicators of credential access (LSASS dumping, browser credential theft)?", "type": "yesno", "ask": "Any sign it went after passwords?"},
        {"id": "mw_lateral", "label": "Indicators of lateral movement?", "type": "yesno", "ask": "Has it spread to other machines?"},
        {"id": "mw_lateral_details", "label": "Lateral movement details (protocols, accounts, hosts)", "type": "textarea", "showIf": {"q": "mw_lateral", "equals": "Yes"}, "ask": "How did it move, and which accounts did it use?"},
        {"id": "mw_host_state", "label": "Current state of affected hosts", "type": "select", "options": ["Running, connected", "Running, isolated", "Powered off", "Reimaged", "Mixed"], "ask": "Are those machines still running, isolated or switched off?"},
        {"id": "mw_user_impact", "label": "Were affected users privileged, or did they access sensitive data?", "type": "yesno", "ask": "Were the users admins, or do they handle sensitive data?"}
      ]
    }
  ],
  "reportChecks": [
    {"id": "malware__family", "label": "Has the malware family or tooling been identified?"},
    {"id": "malware__sample", "label": "Has a sample been analysed, or its hashes checked against threat intelligence?"},
    {"id": "malware__delivery", "label": "Has the delivery vector been identified?"},
    {"id": "malware__execution", "label": "Has the initial execution (process tree) been documented?"},
    {"id": "malware__c2", "label": "Has the C2 infrastructure been identified?"},
    {"id": "malware__cred_access", "label": "Has credential access been checked (LSASS, browser credentials)?"},
    {"id": "malware__sweep", "label": "Have all hosts been swept for the same IOCs (e.g. Defender advanced hunting)?"},
    {"id": "malware__detection_gap", "label": "Has it been explained why security tooling did or did not stop it?"},
    {"id": "malware__clean", "label": "Have affected hosts been confirmed clean or rebuilt?"}
  ]
}
```

### DATA FILE: json/incident-types/exfiltration.json

```json
{
  "incidentType": {"id": "exfiltration", "label": "Data Exfiltration"},
  "sections": [
    {
      "id": "dx_data",
      "title": "Data & indicators of theft",
      "description": "What is believed to have left the environment and why we think so.",
      "questions": [
        {"id": "dx_how_known", "label": "Why is exfiltration suspected?", "type": "multiselect", "options": ["Threat actor claim / leak site", "Data found online", "Extortion email", "DLP alert", "Unusual egress volume", "Third-party notification", "Forensic finding (e.g. rclone, archives)", "Other"], "ask": "What makes you think data has been taken?"},
        {"id": "dx_claim_details", "label": "Claim / alert details", "type": "textarea", "ask": "What exactly have they said, or what did the alert show?"},
        {"id": "dx_data_types", "label": "Data types believed exfiltrated", "type": "multiselect", "options": ["Personal data (PII)", "Special category / health data (PHI)", "Payment card data (PCI)", "Credentials / secrets", "Intellectual property", "Financial records", "Customer data", "Employee / HR data", "Legal / privileged material", "None identified", "Unknown"], "ask": "What kind of data do you think it is?"},
        {"id": "dx_source", "label": "Source repositories (file shares, databases, SaaS)", "type": "textarea", "ask": "Where was that data kept?"},
        {"id": "dx_m365_source", "label": "Was data taken from Microsoft 365?", "type": "multiselect", "options": ["SharePoint Online", "OneDrive", "Exchange Online mailboxes", "Teams", "Not Microsoft 365", "Unknown"], "help": "Purview audit events such as FileDownloaded, FileSyncDownloadedFull and MailItemsAccessed show what was taken.", "ask": "Did any of it come out of SharePoint, OneDrive or email?"},
        {"id": "dx_purview", "label": "Microsoft Purview controls on the data", "type": "multiselect", "options": ["Sensitivity labels", "DLP policies", "Insider Risk Management", "None", "Unknown"], "ask": "Do you use sensitivity labels or DLP on that data?"},
        {"id": "dx_volume", "label": "Estimated volume (e.g. 120 GB, 40k files)", "type": "text", "ask": "Any idea how much data?"},
        {"id": "dx_records", "label": "Estimated number of data subjects / records", "type": "number", "ask": "Roughly how many people or records?"},
        {"id": "dx_proof", "label": "Has the threat actor provided proof (file tree, samples)?", "type": "yesno", "ask": "Have they sent you any proof, like a file list or samples?"},
        {"id": "dx_proof_validated", "label": "Has the proof been validated as genuine client data?", "type": "yesno", "showIf": {"q": "dx_proof", "equals": "Yes"}, "ask": "Have you checked whether it's genuinely your data?"}
      ]
    },
    {
      "id": "dx_channel",
      "title": "Exfiltration channel & timeline",
      "description": "How and when data moved, and what telemetry covers it.",
      "questions": [
        {"id": "dx_channels", "label": "Suspected exfiltration channel", "type": "multiselect", "options": ["Cloud storage (MEGA, Dropbox, etc.)", "rclone / similar sync tool", "FTP / SFTP", "HTTP(S) upload", "Email", "C2 channel", "Removable media", "Personal cloud / SaaS account", "Unknown"], "ask": "Any idea how it was taken out?"},
        {"id": "dx_tools", "label": "Tools or destinations identified", "type": "text", "ask": "Have you found any tools, or where it was sent?"},
        {"id": "dx_window_start", "label": "Exfiltration window start", "type": "datetime", "ask": "When do you think it started?"},
        {"id": "dx_window_end", "label": "Exfiltration window end", "type": "datetime", "ask": "And when did it stop?"},
        {"id": "dx_egress_logs", "label": "Firewall / proxy logs available covering the window?", "type": "yesno", "ask": "Do your firewall or proxy logs go back that far?"},
        {"id": "dx_netflow", "label": "NetFlow or bandwidth data available?", "type": "yesno", "ask": "Have you got any network traffic or bandwidth data?"},
        {"id": "dx_cloud_logs", "label": "Cloud / SaaS audit logs available covering the window?", "type": "yesno", "ask": "Do the cloud audit logs cover that period?"},
        {"id": "dx_mde_network", "label": "Do Defender for Endpoint network events cover the window (e.g. rclone, cloud upload)?", "type": "yesno", "help": "Advanced hunting DeviceNetworkEvents is kept for 30 days unless exported.", "ask": "Would Defender show the uploads from the machines involved?"},
        {"id": "dx_extortion", "label": "Is there an extortion demand?", "type": "yesno", "ask": "Are they asking for money?"},
        {"id": "dx_extortion_details", "label": "Extortion details (amount, deadline, contact)", "type": "textarea", "showIf": {"q": "dx_extortion", "equals": "Yes"}, "ask": "How much, by when, and how do they want you to get in touch?"},
        {"id": "dx_affected_parties", "label": "Third parties whose data may be involved", "type": "textarea", "ask": "Could any customers' or partners' data be in there?"}
      ]
    }
  ],
  "reportChecks": [
    {"id": "exfiltration__data_types", "label": "Have the data types taken been identified?"},
    {"id": "exfiltration__sources", "label": "Have the source systems or repositories been identified?"},
    {"id": "exfiltration__volume", "label": "Has the volume been estimated?"},
    {"id": "exfiltration__channel", "label": "Has the exfiltration channel or tool been identified?"},
    {"id": "exfiltration__destination", "label": "Has the destination been identified (IP, domain, cloud account)?"},
    {"id": "exfiltration__window", "label": "Has the exfiltration time window been established?"},
    {"id": "exfiltration__m365_audit", "label": "Have Microsoft 365 audit logs been reviewed (e.g. FileDownloaded, FileSyncDownloadedFull)?"},
    {"id": "exfiltration__proof", "label": "Has any threat actor proof been validated against client data?"},
    {"id": "exfiltration__data_subjects", "label": "Has the number of affected data subjects been estimated for notification decisions?"}
  ]
}
```

### DATA FILE: json/incident-types/ad.json

```json
{
  "incidentType": {"id": "ad", "label": "Active Directory compromise"},
  "sections": [
    {
      "id": "ad_scope",
      "title": "Domain & forest scope",
      "description": "Size and structure of the directory, and how far the attacker is believed to have got.",
      "questions": [
        {"id": "ad_forests", "label": "Number of forests", "type": "number", "ask": "How many AD forests have you got?"},
        {"id": "ad_domains", "label": "Number of domains", "type": "number", "ask": "And how many domains?"},
        {"id": "ad_trusts", "label": "Trusts in place", "type": "multiselect", "options": ["Parent / child", "Forest trusts (internal)", "External trusts to third parties", "Trusts with acquired companies", "None", "Unknown"], "ask": "Have you got trusts with any other domains or companies?"},
        {"id": "ad_dc_count", "label": "Number of domain controllers", "type": "number", "ask": "How many domain controllers?"},
        {"id": "ad_dc_os", "label": "Domain controller OS versions", "type": "multiselect", "options": ["Server 2012 R2 or older", "Server 2016", "Server 2019", "Server 2022", "Server 2025", "Unknown"], "ask": "What Windows Server versions are the DCs on?"},
        {"id": "ad_dc_sites", "label": "Sites / regions hosting domain controllers", "type": "textarea", "ask": "Where are the DCs? Which sites or countries?"},
        {"id": "ad_priv_compromise", "label": "Privileged access believed obtained", "type": "multiselect", "options": ["Domain Admins / Enterprise Admins", "Built-in Administrator", "Tier 0 service accounts", "Backup Operators / Server Operators", "Synced accounts with privileged Entra roles", "None confirmed", "Unknown"], "ask": "Do you think they got Domain Admin, or something similar?"},
        {"id": "ad_indicators", "label": "Indicators observed so far", "type": "multiselect", "options": ["DCSync / replication from a non-DC", "NTDS.dit access or copy", "Kerberoasting", "Golden / silver ticket suspected", "New privileged accounts or group changes", "GPO created or modified", "AdminSDHolder / ACL changes", "AD CS certificate abuse", "LSASS access on DCs", "None yet", "Unknown"], "ask": "What have you seen so far that points to AD being compromised?"},
        {"id": "ad_gpo_abuse", "label": "Have GPOs been used to deploy software, scripts or scheduled tasks?", "type": "yesno", "ask": "Have they used Group Policy to push anything out?"},
        {"id": "ad_gpo_details", "label": "Which GPOs, and what they deployed", "type": "textarea", "showIf": {"q": "ad_gpo_abuse", "equals": "Yes"}, "ask": "Which policies, and what did they push?"},
        {"id": "ad_adcs", "label": "Is Active Directory Certificate Services (AD CS) deployed?", "type": "yesno", "ask": "Do you run your own certificate authority, AD CS?"},
        {"id": "ad_adcs_abuse", "label": "Any suspicion of certificate template abuse or rogue certificates?", "type": "yesno", "showIf": {"q": "ad_adcs", "equals": "Yes"}, "ask": "Any concern they've misused it to issue certificates?"}
      ]
    },
    {
      "id": "ad_controls",
      "title": "AD controls, logging & recovery",
      "description": "What telemetry exists, and how recoverable the directory is.",
      "questions": [
        {"id": "ad_krbtgt_reset", "label": "Date krbtgt was last reset (if known)", "type": "date", "ask": "Do you know when the krbtgt password was last reset?"},
        {"id": "ad_tiering", "label": "Admin tiering model", "type": "select", "options": ["Enforced tiering / PAWs", "Partial", "None", "Unknown"], "ask": "Do your admins use separate accounts or admin workstations for the DCs?"},
        {"id": "ad_laps", "label": "Local admin password management", "type": "select", "options": ["Windows LAPS", "Legacy LAPS", "None (shared local admin passwords)", "Unknown"], "ask": "Are local admin passwords unique on each machine, with LAPS?"},
        {"id": "ad_mdi", "label": "Is Microsoft Defender for Identity deployed on all DCs?", "type": "yesno", "ask": "Have you got Defender for Identity on the DCs?"},
        {"id": "ad_mdi_alerts", "label": "Has Defender for Identity raised alerts for this activity?", "type": "yesno", "showIf": {"q": "ad_mdi", "equals": "Yes"}, "ask": "Has Defender for Identity flagged anything?"},
        {"id": "ad_protected_users", "label": "Are admin accounts in the Protected Users group?", "type": "yesno", "ask": "Are your admins in the Protected Users group?"},
        {"id": "ad_audit_policy", "label": "Is advanced audit policy configured on DCs (e.g. 4662, 4728, 4769, 5136)?", "type": "yesno", "ask": "Is detailed auditing switched on for the DCs?"},
        {"id": "ad_dc_log_retention", "label": "Security log retention on DCs, and is it forwarded to a SIEM?", "type": "text", "ask": "How far back do the DC security logs go, and do they go to a SIEM?"},
        {"id": "ad_backup", "label": "AD backup status", "type": "select", "options": ["Recent offline system state backups", "Online backups only", "Backups possibly compromised", "No AD backups", "Unknown"], "ask": "Have you got AD backups you'd trust?"},
        {"id": "ad_backup_date", "label": "Last known good AD backup", "type": "date", "ask": "When's the last good one from?"},
        {"id": "ad_forest_recovery", "label": "Has a forest recovery plan been tested?", "type": "yesno", "ask": "Have you ever tested recovering the forest?"},
        {"id": "ad_tier0_assets", "label": "Other Tier 0 systems possibly affected", "type": "multiselect", "options": ["Entra Connect / Cloud Sync servers", "ADFS servers", "AD CS / PKI servers", "Backup servers", "Virtualisation management (vCenter / Hyper-V / SCVMM)", "SCCM / MECM", "PAM / password vault", "None known"], "ask": "Could they have reached other key servers, like Entra Connect, ADFS or the backup servers?"},
        {"id": "ad_resets_done", "label": "Credential actions already taken (earlier answer)", "type": "multiselect", "options": ["krbtgt reset once", "krbtgt reset twice", "Privileged account resets", "Service account resets", "Computer account resets", "None yet"], "showIf": {"q": "ad_resets_done", "notEmpty": true}}
      ]
    }
  ],
  "reportChecks": [
    {"id": "ad__dcsync", "label": "Have DCSync and NTDS.dit theft been checked for?"},
    {"id": "ad__priv_groups", "label": "Have privileged group membership changes been reviewed?"},
    {"id": "ad__gpo", "label": "Have Group Policy changes been reviewed?"},
    {"id": "ad__kerberos", "label": "Have Kerberoasting, AS-REP roasting and forged tickets been checked for?"},
    {"id": "ad__adcs", "label": "Has AD CS abuse been checked (templates, rogue certificates)?"},
    {"id": "ad__tier0", "label": "Have other Tier 0 systems been checked (Entra Connect, ADFS, backups, virtualisation)?"},
    {"id": "ad__dc_logs", "label": "Have DC security logs been reviewed (e.g. 4662, 4728, 4769, 5136)?"},
    {"id": "ad__mdi", "label": "Have Defender for Identity alerts been reviewed?"},
    {"id": "ad__krbtgt", "label": "Is the krbtgt reset status documented and recommended where needed?"},
    {"id": "ad__trusts", "label": "Have trusts been reviewed for abuse?"}
  ]
}
```

### DATA FILE: json/incident-types/entra.json

```json
{
  "incidentType": {"id": "entra", "label": "Entra ID user compromise (standard users)"},
  "sections": [
    {
      "id": "en_tenant",
      "title": "Tenant",
      "description": "Tenant ID and licences are in Case summary; Microsoft 365 logging and tooling are in CX Environment.",
      "questions": [{"id": "en_tenant_id", "label": "Primary domain(s)", "type": "text", "placeholder": "e.g. contoso.com", "ask": "What's your main email domain?"}, {"id": "en_tenants", "label": "Number of tenants in scope", "type": "number", "ask": "Is it just the one tenant?"}]
    },
    {
      "id": "en_access",
      "title": "Compromised users & access",
      "description": "Which users, how the attacker got in, and what they reached. For admin accounts, also add the privileged account type.",
      "questions": [
        {"id": "en_accounts_count", "label": "Number of compromised accounts", "type": "number", "ask": "How many accounts do you think are compromised?"},
        {"id": "en_accounts", "label": "Compromised accounts (UPNs)", "type": "textarea", "placeholder": "One per line", "ask": "Could you list them for me?"},
        {"id": "en_account_type", "label": "Who are the affected users?", "type": "multiselect", "options": ["Standard users", "Executives / VIPs", "Finance / payroll", "Shared or generic accounts", "Guests (B2B)", "Synced from on-prem AD", "Cloud-only", "Unknown"], "ask": "Who are they? Anyone senior, in finance, or with a shared mailbox?"},
        {"id": "en_roles", "label": "Privileged roles held by compromised accounts", "type": "multiselect", "options": ["Global Administrator", "Privileged Role Administrator", "Privileged Authentication Administrator", "Application / Cloud Application Administrator", "Exchange Administrator", "SharePoint Administrator", "Intune Administrator", "User / Helpdesk Administrator", "Security Administrator", "Azure subscription Owner", "None"], "help": "If any do, also add \"Entra ID privileged account compromise (admins)\" as an additional incident type.", "ask": "Are any of them admins? Global Admin, for example?"},
        {"id": "en_technique", "label": "Suspected access technique", "type": "multiselect", "options": ["AiTM phishing / token theft", "Password spray", "Credential stuffing", "MFA fatigue / push bombing", "Device code phishing", "OAuth consent phishing", "Help desk social engineering", "Stolen session cookie / PRT from an endpoint", "Leaked credentials", "Unknown"], "ask": "Any idea how they got in? Phishing, a password spray, something else?"},
        {"id": "en_mfa", "label": "MFA coverage", "type": "select", "options": ["Enforced for all users", "Gaps for some users or apps", "Not enforced", "Unknown"], "ask": "Is MFA on for everyone, or are there gaps?"},
        {"id": "en_mfa_methods", "label": "MFA methods registered on the affected accounts", "type": "multiselect", "options": ["Authenticator push / number matching", "SMS / voice call", "FIDO2 / passkey", "Windows Hello for Business", "Hardware OATH token", "None", "Unknown"], "ask": "How do they do MFA? Authenticator app, text messages?"},
        {"id": "en_first_bad", "label": "First suspicious sign-in", "type": "date", "ask": "When was the first dodgy sign-in?"},
        {"id": "en_ips", "label": "Suspicious IPs, locations or user agents", "type": "textarea", "ask": "Any odd IPs or countries in the sign-in logs?"},
        {"id": "en_token", "label": "Is token or session theft suspected (AiTM, stolen cookie or PRT)?", "type": "yesno", "ask": "Could it be a stolen session rather than the password? An AiTM phish, say?"},
        {"id": "en_noninteractive", "label": "Have the non-interactive sign-in logs been reviewed?", "type": "yesno", "help": "Token replay often shows only in non-interactive sign-ins.", "ask": "Have you looked at the non-interactive sign-ins as well?"},
        {"id": "en_risk", "label": "Entra ID Protection user risk", "type": "select", "options": ["High", "Medium", "Low", "None", "Not licensed (needs Entra ID P2)", "Unknown"], "ask": "Has ID Protection flagged them as risky?"},
        {"id": "en_user_changes", "label": "Changes the attacker made to the accounts", "type": "multiselect", "options": ["New MFA method registered", "Password changed / self-service reset used", "New device registered or joined", "Inbox rules or forwarding", "App consented by the user", "Profile details changed (phone, alternate email)", "None found", "Unknown"], "ask": "What did they change? A new MFA method, a new device, inbox rules?"},
        {"id": "en_containment", "label": "Containment already done (earlier answer)", "type": "multiselect", "options": ["Passwords reset", "Sessions / refresh tokens revoked", "MFA methods reviewed", "Malicious apps removed", "Role assignments removed", "Conditional Access tightened", "None yet"], "showIf": {"q": "en_containment", "notEmpty": true}},
        {"id": "en_data_access", "label": "Data potentially accessed", "type": "multiselect", "options": ["Exchange Online mail", "SharePoint / OneDrive files", "Teams chats", "Azure resources", "Other SaaS apps via SSO", "Unknown"], "ask": "What could they have got to? Email, SharePoint, Azure?"}
      ]
    }
  ],
  "reportChecks": [
    {"id": "entra__signin_review", "label": "Have interactive and non-interactive sign-in logs been reviewed for every affected user?"},
    {"id": "entra__first_signin", "label": "Has the first malicious sign-in been identified, with IP and location?"},
    {"id": "entra__technique", "label": "Has the access technique been confirmed (AiTM, password spray, device code, consent phishing)?"},
    {"id": "entra__mfa_changes", "label": "Have MFA methods registered by the attacker been identified?"},
    {"id": "entra__devices", "label": "Have devices registered by the attacker been identified?"},
    {"id": "entra__revocation", "label": "Has session and refresh token revocation been confirmed?"},
    {"id": "entra__data_access", "label": "Have mailbox and file access been reviewed in the audit log?"},
    {"id": "entra__risk", "label": "Have ID Protection risk detections been reviewed?"},
    {"id": "entra__same_ips", "label": "Have other users been checked for sign-ins from the same IPs?"}
  ]
}
```

### DATA FILE: json/incident-types/entra_priv.json

```json
{
  "incidentType": {"id": "entra_priv", "label": "Entra ID privileged account compromise (admins)"},
  "sections": [
    {
      "id": "pa_accounts",
      "title": "Privileged accounts & roles",
      "description": "Which admin accounts, what roles they hold, and how admin access is protected.",
      "questions": [
        {"id": "pa_accounts", "label": "Compromised admin accounts (UPNs)", "type": "textarea", "ask": "Which admin accounts are we talking about?"},
        {"id": "pa_roles", "label": "Roles held by the compromised accounts", "type": "multiselect", "options": ["Global Administrator", "Privileged Role Administrator", "Privileged Authentication Administrator", "Security Administrator", "Conditional Access Administrator", "Application / Cloud Application Administrator", "Hybrid Identity Administrator", "Exchange Administrator", "SharePoint Administrator", "Intune Administrator", "User / Helpdesk Administrator", "Azure Owner / User Access Administrator", "Unknown"], "ask": "What roles have they got? Global Admin?"},
        {"id": "pa_account_model", "label": "How are admin accounts set up?", "type": "select", "options": ["Dedicated cloud-only admin accounts", "Dedicated accounts synced from on-prem AD", "Same account used for email and admin", "Mixed", "Unknown"], "ask": "Are your admin accounts separate cloud-only accounts, or people's normal accounts?"},
        {"id": "pa_mfa_strength", "label": "MFA required for admins", "type": "select", "options": ["Phishing-resistant (FIDO2, passkey, WHfB, certificate) via authentication strength", "Standard MFA", "Not enforced", "Unknown"], "ask": "What MFA do admins have to use? Anything phishing-resistant?"},
        {"id": "en_pim", "label": "Is Privileged Identity Management (PIM) used for admin roles?", "type": "yesno", "ask": "Do you use PIM for admin roles?"},
        {"id": "pa_pim_activation", "label": "Did the attacker activate PIM roles or approve PIM requests?", "type": "yesno", "showIf": {"q": "en_pim", "equals": "Yes"}, "ask": "Did they activate any PIM roles?"},
        {"id": "pa_paw", "label": "Do admins use dedicated privileged access workstations?", "type": "yesno", "ask": "Do admins work from separate admin machines?"},
        {"id": "en_breakglass", "label": "Are break-glass accounts in place and confirmed unaffected?", "type": "yesno", "ask": "Have you got break-glass accounts, and are they safe?"}
      ]
    },
    {
      "id": "pa_changes",
      "title": "Tenant changes by the attacker",
      "description": "What the attacker changed with admin rights; these are often persistence.",
      "questions": [
        {"id": "en_changes", "label": "Tenant changes made by the attacker", "type": "multiselect", "options": ["MFA methods registered", "New users or guests created", "Role assignments added", "App registrations, secrets or certificates added", "OAuth consent grants", "Federated domain or federation settings changed", "Cross-tenant access settings changed", "Mailbox rules / forwarding", "Intune scripts, apps or device wipes", "None identified", "Unknown"], "ask": "What have they changed? New users, apps, MFA methods?"},
        {"id": "en_changes_details", "label": "Details of changes (objects, times, actors)", "type": "textarea", "ask": "Can you tell me what changed, and when?"},
        {"id": "en_ca", "label": "Did the attacker change Conditional Access policies or named locations?", "type": "yesno", "ask": "Have they touched any Conditional Access policies?"},
        {"id": "pa_auth_policy", "label": "Were authentication methods policies changed (e.g. Temporary Access Pass enabled, SMS allowed)?", "type": "yesno", "ask": "Have they changed which sign-in methods are allowed?"},
        {"id": "pa_elevated_access", "label": "Did a Global Administrator turn on \"Access management for Azure resources\" (elevated access)?", "type": "yesno", "help": "This grants User Access Administrator at the root scope of every subscription.", "ask": "Did anyone use Global Admin to give themselves access to all your Azure subscriptions?"},
        {"id": "pa_cross_tenant", "label": "Were cross-tenant access settings or partner relationships (GDAP) changed?", "type": "yesno", "ask": "Any changes to which other tenants or partners can get in?"},
        {"id": "pa_exchange_admin", "label": "Were Exchange admin actions used (transport rules, mailbox permissions, eDiscovery exports)?", "type": "yesno", "ask": "Did they do anything in Exchange admin? Mailbox permissions, eDiscovery searches?"},
        {"id": "pa_audit_tamper", "label": "Any sign of audit tampering (Purview audit turned off, diagnostic settings removed, mailbox audit bypass)?", "type": "yesno", "ask": "Any sign they tried to turn off logging?"}
      ]
    }
  ],
  "reportChecks": [
    {"id": "entra_priv__audit_review", "label": "Have Entra audit logs been reviewed for every action by the compromised admins?"},
    {"id": "entra_priv__roles", "label": "Have role assignment and PIM changes been identified?"},
    {"id": "entra_priv__objects", "label": "Have users, guests and apps created by the attacker been identified?"},
    {"id": "entra_priv__policies", "label": "Have Conditional Access and authentication methods policy changes been reviewed?"},
    {"id": "entra_priv__federation", "label": "Have domain and federation changes been checked?"},
    {"id": "entra_priv__elevated", "label": "Has Azure elevated access (root scope) been checked?"},
    {"id": "entra_priv__exchange_admin", "label": "Have Exchange admin and eDiscovery actions been reviewed?"},
    {"id": "entra_priv__audit_tamper", "label": "Has audit tampering been checked for?"},
    {"id": "entra_priv__breakglass", "label": "Have break-glass accounts been reviewed?"},
    {"id": "entra_priv__persistence_removed", "label": "Has removal of every attacker persistence item been confirmed?"}
  ]
}
```

### DATA FILE: json/incident-types/entra_sp.json

```json
{
  "incidentType": {"id": "entra_sp", "label": "Entra service principal / app registration compromise"},
  "sections": [
    {
      "id": "spn_app",
      "title": "Application & credentials",
      "description": "Which application, what it can do, and how it authenticates.",
      "questions": [
        {"id": "spn_app_names", "label": "Affected application(s): display name, application (client) ID, object ID", "type": "textarea", "ask": "Which app is it? Have you got the name or the application ID?"},
        {"id": "spn_app_type", "label": "Application type", "type": "multiselect", "options": ["Single-tenant app registration (built in-house)", "Multi-tenant app from a vendor (enterprise app)", "Microsoft first-party app", "Unknown"], "ask": "Is it an app you built yourselves, or one from a vendor?"},
        {"id": "spn_credential_type", "label": "How does the app authenticate?", "type": "multiselect", "options": ["Client secret", "Certificate", "Federated credential (workload identity federation)", "Unknown"], "ask": "Does it sign in with a secret, a certificate or federated credentials?"},
        {"id": "spn_cred_storage", "label": "Where were the credentials stored?", "type": "multiselect", "options": ["Azure Key Vault", "Source code / repository", "CI/CD pipeline variables (GitHub, Azure DevOps)", "Config files on a server", "Developer machines", "Unknown"], "ask": "Where was that secret kept? Key Vault, in the code, a pipeline?"},
        {"id": "spn_permissions", "label": "Permissions held by the app", "type": "multiselect", "options": ["Microsoft Graph application permissions (e.g. Mail.Read, Directory.ReadWrite.All)", "Microsoft Graph delegated permissions", "RoleManagement.ReadWrite.Directory or AppRoleAssignment.ReadWrite.All", "Exchange Online full_access_as_app", "SharePoint Sites.FullControl.All", "Entra directory roles assigned to the service principal", "Azure RBAC roles on subscriptions / resources", "Unknown"], "ask": "What can the app do? Has it got Graph permissions like reading all mail?"},
        {"id": "spn_permissions_details", "label": "Permission details (exact scopes and role assignments)", "type": "textarea"},
        {"id": "spn_owners", "label": "App and service principal owners", "type": "textarea", "help": "Owners can add credentials, so a compromised owner account is a common way in.", "ask": "Who owns the app? Are any user accounts listed as owners?"}
      ]
    },
    {
      "id": "spn_activity",
      "title": "Service principal activity & evidence",
      "description": "How the app was abused and what evidence is available.",
      "questions": [
        {"id": "spn_cred_added", "label": "Were new secrets or certificates added to the app or service principal?", "type": "yesno", "ask": "Has anyone added a new secret or certificate to it recently?"},
        {"id": "spn_cred_added_details", "label": "Added credential details (key IDs, times, who added them)", "type": "textarea", "showIf": {"q": "spn_cred_added", "equals": "Yes"}},
        {"id": "spn_perm_changes", "label": "Were permissions, consents or role assignments added?", "type": "yesno", "ask": "Have its permissions changed? Any new admin consents?"},
        {"id": "spn_audit", "label": "Do the Entra audit logs show credential or permission changes on the app?", "type": "yesno", "help": "Look for \"Update application - Certificates and secrets management\", \"Add service principal credentials\" and \"Add app role assignment to service principal\".", "ask": "Can you see who added the secret in the audit logs?"},
        {"id": "spn_vendor", "label": "If it is a vendor app, has the vendor been told?", "type": "yesno", "ask": "If it's a supplier's app, have you let them know?"},
        {"id": "spn_signin_logs", "label": "Are service principal sign-in logs available?", "type": "select", "options": ["Exported to Sentinel / Log Analytics", "Exported to another SIEM", "Portal only (30 days)", "Unknown"], "ask": "Are the service principal sign-in logs going anywhere, or just the portal?"},
        {"id": "spn_source_ips", "label": "Suspicious sign-in source IPs / locations", "type": "textarea", "ask": "Have you spotted it signing in from odd IPs or countries?"},
        {"id": "spn_actions", "label": "Suspected actions taken with the app", "type": "multiselect", "options": ["Mail read or exported via Graph", "Files downloaded from SharePoint / OneDrive", "Users, groups or roles changed", "Other apps or credentials created", "Azure resources accessed or changed", "Data exfiltration", "Unknown"], "ask": "Any idea what they used it for? Reading mail, pulling files?"},
        {"id": "spn_workload_ca", "label": "Is Conditional Access for workload identities in use?", "type": "yesno", "help": "Needs Microsoft Entra Workload ID Premium.", "ask": "Have you got any Conditional Access on your service principals?"}
      ]
    }
  ],
  "reportChecks": [
    {"id": "entra_sp__cred_changes", "label": "Have added secrets, certificates and federated credentials been identified, with who and when?"},
    {"id": "entra_sp__leak_source", "label": "Has it been established how the credential was obtained or leaked?"},
    {"id": "entra_sp__sp_signins", "label": "Have service principal sign-in logs been reviewed?"},
    {"id": "entra_sp__source_ips", "label": "Have the attacker source IPs been identified?"},
    {"id": "entra_sp__graph_activity", "label": "Has Graph activity by the app been reviewed (MicrosoftGraphActivityLogs)?"},
    {"id": "entra_sp__permissions", "label": "Are the permissions at the time of compromise documented?"},
    {"id": "entra_sp__data_access", "label": "Has data accessed through the app been assessed?"},
    {"id": "entra_sp__owners", "label": "Have the app owners been reviewed?"},
    {"id": "entra_sp__vendor", "label": "If it is a vendor app, has the vendor been informed?"}
  ]
}
```

### DATA FILE: json/incident-types/entra_mi.json

```json
{
  "incidentType": {"id": "entra_mi", "label": "Azure managed identity compromise"},
  "sections": [
    {
      "id": "mi_identity",
      "title": "Managed identity & access",
      "description": "Which managed identity, what it is attached to, and what it can reach.",
      "questions": [
        {"id": "mi_names", "label": "Affected managed identities (name, client ID, object ID)", "type": "textarea", "ask": "Which identity is it? Have you got the name or client ID?"},
        {"id": "mi_type", "label": "Managed identity type", "type": "multiselect", "options": ["System-assigned", "User-assigned", "Unknown"], "ask": "Is it system-assigned or user-assigned?"},
        {"id": "mi_hosts", "label": "Resources the identity is attached to", "type": "multiselect", "options": ["Virtual machine / scale set", "App Service / Function App", "Azure Kubernetes Service (AKS)", "Container Apps / Container Instances", "Logic App", "Automation account", "Data Factory / Synapse", "Azure Arc server", "Unknown"], "ask": "What's it attached to? A VM, a function app, AKS?"},
        {"id": "mi_host_details", "label": "Host resource details (names, subscriptions, resource groups)", "type": "textarea"},
        {"id": "mi_rbac", "label": "Azure RBAC roles held", "type": "multiselect", "options": ["Owner", "Contributor", "User Access Administrator", "Key Vault access (secrets, keys, certificates)", "Storage data access (Blob / File)", "Virtual Machine Contributor / Run Command", "Reader only", "Unknown"], "ask": "What can it do in Azure? Is it Owner or Contributor anywhere?"},
        {"id": "mi_scope", "label": "Scope of its role assignments", "type": "select", "options": ["Single resource", "Resource group", "Subscription", "Management group / tenant root", "Unknown"], "ask": "How far do those permissions reach? One resource, or the whole subscription?"},
        {"id": "mi_graph_perms", "label": "Does the identity have Microsoft Graph permissions or Entra roles?", "type": "yesno", "ask": "Has it got any Graph permissions or Entra roles as well?"}
      ]
    },
    {
      "id": "mi_activity",
      "title": "Managed identity abuse & evidence",
      "description": "How the token was obtained and what was done with it.",
      "questions": [
        {"id": "mi_abuse_path", "label": "How was the identity abused?", "type": "multiselect", "options": ["Code execution on the host (e.g. compromised web app or VM)", "Token requested from the instance metadata service (IMDS)", "Token stolen from the app or container environment", "VM Run Command or extensions", "Misconfigured CI/CD or automation runbook", "Unknown"], "ask": "How do you think they got at it? Code running on the VM or the app?"},
        {"id": "mi_host_compromised", "label": "Is the host resource itself confirmed compromised?", "type": "yesno", "ask": "Is the VM or app itself compromised, as far as you know?"},
        {"id": "mi_actions", "label": "Suspected actions taken with the identity", "type": "multiselect", "options": ["Key Vault secrets read", "Storage data read or downloaded", "Role assignments added", "Resources created (e.g. VMs, crypto mining)", "Resources deleted or encrypted", "Other identities or credentials created", "Unknown"], "ask": "What did they do with it? Pull secrets from Key Vault, spin up resources?"},
        {"id": "mi_activity_log", "label": "Is the Azure Activity Log exported?", "type": "select", "options": ["To Sentinel / Log Analytics", "To another SIEM", "Storage account", "Not exported (90-day portal retention)", "Unknown"], "ask": "Are your Azure activity logs going into Log Analytics or Sentinel?"},
        {"id": "mi_signin_logs", "label": "Are managed identity sign-in logs exported?", "type": "yesno", "ask": "And the managed identity sign-in logs from Entra, are they kept anywhere?"},
        {"id": "mi_diag_logs", "label": "Resource diagnostic logs enabled", "type": "multiselect", "options": ["Key Vault audit logs", "Storage account logs", "AKS audit logs", "App Service logs", "None", "Unknown"], "ask": "Have you got diagnostic logging on Key Vault and storage?"}
      ]
    }
  ],
  "reportChecks": [{"id": "entra_mi__host_path", "label": "Has the path to the host resource (VM, app, container) been established?"}, {"id": "entra_mi__token_use", "label": "Has token use been evidenced (managed identity sign-in logs)?"}, {"id": "entra_mi__activity_log", "label": "Have Azure Activity Log actions by the identity been reviewed?"}, {"id": "entra_mi__data_plane", "label": "Have Key Vault and storage data-plane logs been reviewed?"}, {"id": "entra_mi__rbac", "label": "Are the RBAC roles and their scope documented?"}, {"id": "entra_mi__new_identities", "label": "Have identities or credentials created with it been checked for?"}, {"id": "entra_mi__graph", "label": "Has Graph or Entra activity by the identity been checked?"}, {"id": "entra_mi__roles_removed", "label": "Has removal of unnecessary role assignments been confirmed?"}]
}
```

### DATA FILE: json/incident-types/entra_oauth.json

```json
{
  "incidentType": {"id": "entra_oauth", "label": "Malicious OAuth app / consent grant"},
  "sections": [
    {
      "id": "oa_app",
      "title": "Malicious app & consent",
      "description": "Which app was consented to, by whom, and with what permissions.",
      "questions": [
        {"id": "oa_app_names", "label": "Malicious app(s): display name, application ID, publisher", "type": "textarea", "ask": "What's the app called? Do you know the app ID or the publisher?"},
        {"id": "oa_consent_type", "label": "Type of consent", "type": "select", "options": ["User consent (delegated)", "Admin consent (tenant-wide)", "Both", "Unknown"], "ask": "Did a user consent to it, or an admin for the whole tenant?"},
        {"id": "oa_users_consented", "label": "Number of users who consented", "type": "number", "ask": "How many people clicked through and consented?"},
        {"id": "oa_scopes", "label": "Permissions granted", "type": "multiselect", "options": ["Mail.Read / Mail.ReadWrite", "Mail.Send", "Files.Read.All / Files.ReadWrite.All", "offline_access", "User.Read.All / Directory.Read.All", "Contacts / Calendars", "Unknown"], "ask": "What did it ask for? Mail, files, offline access?"},
        {"id": "oa_publisher", "label": "Publisher verification", "type": "select", "options": ["Verified publisher", "Unverified", "Unknown"]},
        {"id": "oa_consent_policy", "label": "User consent setting in the tenant", "type": "select", "options": ["Users can consent to any app", "Users can consent to verified publishers / low-risk permissions only", "User consent disabled (admin consent workflow)", "Unknown"], "ask": "Can your users consent to apps themselves, or does it need an admin?"}
      ]
    },
    {
      "id": "oa_activity",
      "title": "App activity & evidence",
      "description": "How consent was obtained and what the app did.",
      "questions": [
        {"id": "oa_lure", "label": "How was the consent obtained?", "type": "select", "options": ["Phishing link to a consent page", "Device code phishing", "Compromised admin account", "Unknown"], "ask": "How did they get people to consent? A phishing link?"},
        {"id": "oa_actions", "label": "Suspected actions taken by the app", "type": "multiselect", "options": ["Mail read", "Mail sent", "Files downloaded", "Inbox rules created", "Unknown"], "ask": "Do you know what it did? Read mail, send mail, download files?"},
        {"id": "oa_audit", "label": "Are the Entra audit logs showing the consent available?", "type": "yesno", "help": "Look for \"Consent to application\" and \"Add delegated permission grant\" events.", "ask": "Can you still see the consent in the audit logs?"},
        {"id": "oa_app_governance", "label": "Has Defender for Cloud Apps app governance flagged the app?", "type": "yesno", "ask": "Has app governance in Defender flagged it?"},
        {"id": "oa_still_present", "label": "Is the app still in the tenant?", "type": "yesno", "ask": "Is the app still there, or has it been removed?"}
      ]
    }
  ],
  "reportChecks": [{"id": "entra_oauth__app_identified", "label": "Has the app been identified (app ID, publisher, reply URLs)?"}, {"id": "entra_oauth__users", "label": "Have all users who consented been identified?"}, {"id": "entra_oauth__scopes", "label": "Are the permissions granted documented?"}, {"id": "entra_oauth__consent_events", "label": "Have the consent events been found in the audit log?"}, {"id": "entra_oauth__graph_activity", "label": "Has Graph activity by the app been reviewed?"}, {"id": "entra_oauth__data_access", "label": "Has data accessed by the app been assessed?"}, {"id": "entra_oauth__revoked", "label": "Has revocation of the grants and disabling of the app been confirmed?"}, {"id": "entra_oauth__reported", "label": "Has the app been reported to Microsoft?"}]
}
```

### DATA FILE: json/incident-types/device_code.json

```json
{
  "incidentType": {"id": "device_code", "label": "Device code phishing"},
  "sections": [
    {
      "id": "dc_lure",
      "title": "Lure & targeted users",
      "description": "How the device code lure reached people, and who entered a code.",
      "questions": [
        {"id": "dc_lure_channel", "label": "How the lure was delivered", "type": "multiselect", "options": ["Email", "Teams chat or call", "Signal / WhatsApp / other messaging app", "Phone call", "Fake meeting invite", "QR code", "Unknown"], "ask": "How did the message reach them? Email, Teams, WhatsApp?"},
        {"id": "dc_pretext", "label": "Pretext used", "type": "text", "placeholder": "e.g. Teams meeting invite, document share, IT support", "ask": "What was the story? A meeting invite, a shared document, IT support?"},
        {"id": "dc_sender", "label": "Lure source (sender address, external Teams tenant, phone number)", "type": "textarea", "ask": "Where did it come from? An email address, an outside Teams tenant?"},
        {"id": "dc_recipients", "label": "Number of users who received the lure", "type": "number", "ask": "Roughly how many people got the message?"},
        {"id": "dc_entered_count", "label": "Number of users who entered a code", "type": "number", "ask": "How many actually typed a code in?"},
        {"id": "dc_entered_users", "label": "Users who entered a code (UPNs)", "type": "textarea", "ask": "Which users were they?"},
        {"id": "dc_privileged", "label": "Are any of those users admins or executives?", "type": "yesno", "help": "If so, also add \"Entra ID privileged account compromise (admins)\" as an incident type.", "ask": "Are any of them admins or senior people?"}
      ]
    },
    {
      "id": "dc_signins",
      "title": "Device code sign-ins & token use",
      "description": "What the sign-in logs show, and what the attacker did with the tokens.",
      "questions": [
        {"id": "dc_signin_evidence", "label": "Have sign-ins using the device code flow been found?", "type": "yesno", "help": "Entra sign-in logs: filter Authentication protocol = Device code. In Sentinel or Log Analytics, SigninLogs where AuthenticationProtocol == \"deviceCode\".", "ask": "Can you see sign-ins in Entra where the protocol shows as device code?"},
        {"id": "dc_first_signin", "label": "First device code sign-in", "type": "date", "ask": "When was the first one?"},
        {"id": "dc_app", "label": "Application the attacker got a token for", "type": "multiselect", "options": ["Microsoft Authentication Broker", "Microsoft Office / Teams", "Microsoft Graph", "Azure CLI / Azure PowerShell", "Other", "Unknown"], "help": "The Microsoft Authentication Broker is often used to go on to register a device and get a Primary Refresh Token.", "ask": "Do the logs show which app the code was for? Teams, Office, the Authentication Broker?"},
        {"id": "dc_ips", "label": "Attacker IPs, locations or user agents", "type": "textarea", "ask": "Any IPs or countries that stand out in those sign-ins?"},
        {"id": "dc_device_reg", "label": "Did the attacker register or join a device in Entra (e.g. to get a Primary Refresh Token)?", "type": "yesno", "ask": "Did a new device get registered against any of those accounts?"},
        {"id": "dc_token_reuse", "label": "Is the attacker still using refresh tokens (non-interactive sign-ins after the event)?", "type": "yesno", "ask": "Are there still non-interactive sign-ins from them after the event?"},
        {"id": "dc_activity", "label": "Attacker activity after sign-in", "type": "multiselect", "options": ["Mailboxes searched (e.g. for passwords, admin, credentials)", "Mail read or exported via Graph", "Internal phishing sent from compromised accounts", "Teams messages sent", "SharePoint / OneDrive files accessed", "MFA methods added", "Azure resources accessed", "Unknown"], "ask": "What did they do once they were in? Read email, send more phishing?"},
        {"id": "dc_ca_block", "label": "Is device code flow blocked by Conditional Access (authentication flows)?", "type": "select", "options": ["Blocked for all users", "Blocked with exceptions", "Not blocked", "Unknown"], "ask": "Have you got a Conditional Access policy that blocks device code sign-in?"},
        {"id": "dc_legit_use", "label": "Is device code flow used legitimately in the tenant (e.g. meeting room devices, CLI tools)?", "type": "yesno", "ask": "Does anything in your business actually use device code sign-in, like meeting room kit?"}
      ]
    }
  ],
  "reportChecks": [
    {"id": "device_code__users", "label": "Have all users who entered a code been identified?"},
    {"id": "device_code__first_signin", "label": "Has the first device code sign-in been identified, with source IPs?"},
    {"id": "device_code__app", "label": "Has the app the token was issued for been identified?"},
    {"id": "device_code__device_reg", "label": "Has device registration or Primary Refresh Token use been checked?"},
    {"id": "device_code__refresh_reuse", "label": "Has refresh token reuse in non-interactive sign-ins been checked?"},
    {"id": "device_code__mail_access", "label": "Has MailItemsAccessed been reviewed for the affected users?"},
    {"id": "device_code__internal_phish", "label": "Has internal phishing from compromised accounts been checked?"},
    {"id": "device_code__lure_scope", "label": "Have all recipients of the lure been identified?"},
    {"id": "device_code__ca_block", "label": "Has a Conditional Access block on device code flow been recommended or confirmed?"}
  ]
}
```

### DATA FILE: json/incident-types/hybrid.json

```json
{
  "incidentType": {"id": "hybrid", "label": "Hybrid identity compromise (Entra Connect / ADFS)"},
  "sections": [
    {
      "id": "hy_arch",
      "title": "Hybrid identity architecture",
      "description": "How on-prem AD and Entra ID are joined, which decides the blast radius in both directions.",
      "questions": [
        {"id": "hy_auth", "label": "Authentication method", "type": "select", "options": ["Password hash sync (PHS)", "Pass-through authentication (PTA)", "Federation with ADFS", "Federation with a third-party IdP", "Mixed / staged rollout", "Unknown"], "ask": "How do users sign in to the cloud? Password hash sync, pass-through, or ADFS?"},
        {"id": "hy_sync_tool", "label": "Sync tooling", "type": "multiselect", "options": ["Entra Connect Sync", "Entra Cloud Sync", "Unknown"], "ask": "Do you use Entra Connect or Cloud Sync?"},
        {"id": "hy_connect_servers", "label": "Entra Connect / Cloud Sync servers (names, OS, location)", "type": "textarea", "ask": "Which servers run it, and where are they?"},
        {"id": "hy_connect_tier0", "label": "Are Entra Connect servers protected as Tier 0?", "type": "yesno", "ask": "Is that server locked down as tightly as a domain controller?"},
        {"id": "hy_seamless_sso", "label": "Is Seamless SSO enabled?", "type": "yesno", "ask": "Is Seamless SSO turned on?"},
        {"id": "hy_sso_rollover", "label": "Last Seamless SSO (AZUREADSSOACC) key rollover", "type": "date", "showIf": {"q": "hy_seamless_sso", "equals": "Yes"}, "ask": "Do you know when that key was last rolled over?"},
        {
          "id": "hy_adfs_count",
          "label": "Number of ADFS and WAP servers",
          "type": "number",
          "showIf": {"q": "hy_auth", "in": ["Federation with ADFS", "Mixed / staged rollout"]},
          "ask": "How many ADFS and proxy servers have you got?"
        },
        {
          "id": "hy_pta_agents",
          "label": "Number of PTA agents",
          "type": "number",
          "showIf": {"q": "hy_auth", "in": ["Pass-through authentication (PTA)", "Mixed / staged rollout"]},
          "ask": "How many pass-through agents are there?"
        },
        {"id": "hy_writeback", "label": "Writeback features enabled", "type": "multiselect", "options": ["Password writeback", "Group writeback", "Device writeback", "None", "Unknown"], "ask": "Have you got password writeback, or any other writeback, turned on?"},
        {"id": "hy_synced_admins", "label": "Do any synced (on-prem) accounts hold privileged Entra roles?", "type": "yesno", "help": "Privileged cloud roles should normally be held by cloud-only accounts.", "ask": "Are any of your cloud admins synced from on-prem accounts?"},
        {"id": "hy_mdi_sensors", "label": "Are Defender for Identity sensors on the Entra Connect, ADFS and AD CS servers?", "type": "yesno", "ask": "Has Defender for Identity got sensors on the Connect and ADFS servers too?"}
      ]
    },
    {
      "id": "hy_compromise",
      "title": "Hybrid compromise indicators",
      "description": "Signs the attacker crossed between on-prem and cloud.",
      "questions": [
        {"id": "hy_direction", "label": "Suspected direction of movement", "type": "select", "options": ["On-prem to cloud", "Cloud to on-prem", "Both", "Unknown"], "ask": "Did it start on-prem and move to the cloud, or the other way round?"},
        {"id": "hy_connect_compromised", "label": "Is an Entra Connect / Cloud Sync server suspected compromised?", "type": "yesno", "ask": "Any chance the Entra Connect server itself was compromised?"},
        {"id": "hy_connector_accounts", "label": "Any suspicious use of sync / connector accounts (e.g. MSOL_ or Sync_ accounts)?", "type": "yesno", "ask": "Have you seen the sync accounts doing anything odd?"},
        {
          "id": "hy_adfs_compromised",
          "label": "Is an ADFS server suspected compromised?",
          "type": "yesno",
          "showIf": {"q": "hy_auth", "in": ["Federation with ADFS", "Mixed / staged rollout"]},
          "ask": "Do you think an ADFS server was compromised?"
        },
        {
          "id": "hy_adfs_cert",
          "label": "Any concern the ADFS token-signing certificate was exported (Golden SAML)?",
          "type": "yesno",
          "showIf": {"q": "hy_adfs_compromised", "in": ["Yes", "Unknown"]},
          "ask": "Any worry the token-signing certificate was taken?"
        },
        {
          "id": "hy_pta_compromised",
          "label": "Is a PTA agent server suspected compromised?",
          "type": "yesno",
          "showIf": {"q": "hy_auth", "in": ["Pass-through authentication (PTA)", "Mixed / staged rollout"]},
          "ask": "Any chance a pass-through agent server was compromised?"
        },
        {"id": "hy_federation_changes", "label": "Any new federated domains or federation trust changes in Entra ID?", "type": "yesno", "ask": "Have you seen any new domains or federation changes in Entra?"},
        {"id": "hy_indicators", "label": "Other indicators and timings", "type": "textarea", "ask": "Anything else you've noticed, and roughly when?"}
      ]
    }
  ],
  "reportChecks": [
    {"id": "hybrid__auth_method", "label": "Is the hybrid authentication method documented?"},
    {"id": "hybrid__connect_server", "label": "Has Entra Connect / Cloud Sync server compromise been assessed?"},
    {"id": "hybrid__connector_accounts", "label": "Has use of sync and connector accounts been reviewed?"},
    {"id": "hybrid__golden_saml", "label": "Has ADFS token-signing certificate theft (Golden SAML) been checked?"},
    {"id": "hybrid__federation_changes", "label": "Have federation trust and domain changes been checked?"},
    {"id": "hybrid__seamless_sso", "label": "Is the Seamless SSO key status checked?"},
    {"id": "hybrid__pta", "label": "Has PTA agent compromise been assessed?"},
    {"id": "hybrid__direction", "label": "Has the direction of movement (on-prem to cloud or back) been established?"},
    {"id": "hybrid__synced_admins", "label": "Have synced accounts with privileged Entra roles been reviewed?"}
  ]
}
```

### DATA FILE: json/incident-types/exchange.json

```json
{
  "incidentType": {"id": "exchange", "label": "Exchange Server (on-prem) compromise"},
  "sections": [
    {
      "id": "ex_server",
      "title": "Exchange estate & exposure",
      "description": "Versions, exposure and the likely entry point.",
      "questions": [
        {"id": "ex_versions", "label": "Exchange versions in use", "type": "multiselect", "options": ["Exchange 2013 or older", "Exchange 2016", "Exchange 2019", "Exchange Server Subscription Edition", "Unknown"], "ask": "What version of Exchange are you on?"},
        {"id": "ex_cu", "label": "Cumulative update / security update level", "type": "text", "ask": "Do you know which update it's on?"},
        {"id": "ex_server_count", "label": "Number of Exchange servers", "type": "number", "ask": "How many Exchange servers have you got?"},
        {"id": "ex_exposed", "label": "Internet-exposed services", "type": "multiselect", "options": ["OWA", "ECP / EAC", "EWS", "ActiveSync", "Autodiscover", "MAPI / RPC over HTTP", "SMTP", "None directly (behind reverse proxy)", "Unknown"], "ask": "Which bits are open to the internet? OWA, ActiveSync?"},
        {"id": "ex_hybrid", "label": "Is Exchange in a hybrid configuration with Exchange Online?", "type": "yesno", "ask": "Is it set up in hybrid with Exchange Online?"},
        {"id": "ex_emergency_mitigation", "label": "Is the Exchange Emergency Mitigation service enabled?", "type": "yesno", "ask": "Is the Emergency Mitigation service switched on?"},
        {"id": "ex_vuln", "label": "Suspected vulnerability or exploit (e.g. ProxyShell, ProxyNotShell)", "type": "text", "ask": "Do you think they came in through a known Exchange vulnerability?"},
        {"id": "ex_edr", "label": "Is EDR deployed on the Exchange servers?", "type": "yesno", "ask": "Is your EDR running on the Exchange servers?"}
      ]
    },
    {
      "id": "ex_activity",
      "title": "Exchange compromise activity",
      "description": "What the attacker did on or through Exchange.",
      "questions": [
        {"id": "ex_webshells", "label": "Have web shells been found (e.g. in OWA, ECP or aspnet_client folders)?", "type": "yesno", "ask": "Have you found any web shells?"},
        {"id": "ex_webshell_details", "label": "Web shell paths, hashes and timestamps", "type": "textarea", "showIf": {"q": "ex_webshells", "equals": "Yes"}, "ask": "Where were they, and when were they created?"},
        {"id": "ex_iis_logs", "label": "IIS log retention on the Exchange servers", "type": "text", "ask": "How far back do the IIS logs go on those servers?"},
        {"id": "ex_mailbox_export", "label": "Evidence of mailbox export requests or bulk mailbox access?", "type": "yesno", "ask": "Any sign they exported mailboxes or pulled a lot of mail?"},
        {"id": "ex_transport_rules", "label": "Suspicious transport rules or connectors?", "type": "yesno", "ask": "Any odd transport rules or connectors?"},
        {"id": "ex_ad_perms", "label": "Any concern the attacker used Exchange's AD permissions to escalate (e.g. Exchange Windows Permissions)?", "type": "yesno", "ask": "Any sign they used Exchange's permissions to get further into AD?"},
        {"id": "ex_mailboxes_accessed", "label": "Mailboxes known or suspected to be accessed", "type": "textarea", "ask": "Do you know which mailboxes they got into?"},
        {"id": "ex_lateral", "label": "Evidence of lateral movement from the Exchange servers?", "type": "yesno", "ask": "Did they move from Exchange onto other servers?"},
        {"id": "ex_isolated", "label": "Have the Exchange servers been isolated or taken offline?", "type": "yesno", "ask": "Have you taken the Exchange servers offline or isolated them?"}
      ]
    }
  ],
  "reportChecks": [
    {"id": "exchange__vuln", "label": "Has the exploited vulnerability been identified?"},
    {"id": "exchange__first_exploit", "label": "Has the first exploitation time been established?"},
    {"id": "exchange__webshells", "label": "Have web shells been searched for on every Exchange server?"},
    {"id": "exchange__iis_logs", "label": "Have IIS and HttpProxy logs been reviewed for exploitation?"},
    {"id": "exchange__mailbox_access", "label": "Have mailbox exports and mailbox access been identified?"},
    {"id": "exchange__transport", "label": "Have transport rules and connectors been reviewed?"},
    {"id": "exchange__ad_perms", "label": "Has abuse of Exchange's AD permissions been checked?"},
    {"id": "exchange__lateral", "label": "Has movement from the Exchange servers been assessed?"},
    {"id": "exchange__patch", "label": "Is the current patch level confirmed?"}
  ]
}
```

### DATA FILE: json/incident-types/sharepoint.json

```json
{
  "incidentType": {"id": "sharepoint", "label": "SharePoint compromise (Server / Online)"},
  "sections": [
    {
      "id": "sp_platform",
      "title": "SharePoint platform & exposure",
      "description": "Which SharePoint is involved and how exposed it is.",
      "questions": [
        {"id": "sp_deployment", "label": "SharePoint deployment", "type": "select", "options": ["SharePoint Server (on-prem)", "SharePoint Online", "Both / hybrid"], "ask": "Is it SharePoint Online, on-prem SharePoint Server, or both?"},
        {
          "id": "sp_version",
          "label": "SharePoint Server version",
          "type": "select",
          "options": ["2013 or older", "2016", "2019", "Subscription Edition", "Unknown"],
          "showIf": {"q": "sp_deployment", "in": ["SharePoint Server (on-prem)", "Both / hybrid"]},
          "ask": "Which version of SharePoint Server is it?"
        },
        {
          "id": "sp_patch",
          "label": "Patch level / last security update installed",
          "type": "text",
          "showIf": {"q": "sp_deployment", "in": ["SharePoint Server (on-prem)", "Both / hybrid"]},
          "ask": "When was it last patched?"
        },
        {
          "id": "sp_internet",
          "label": "Is the SharePoint Server farm internet-facing?",
          "type": "yesno",
          "showIf": {"q": "sp_deployment", "in": ["SharePoint Server (on-prem)", "Both / hybrid"]},
          "ask": "Can the farm be reached from the internet?"
        },
        {
          "id": "sp_vuln",
          "label": "Suspected vulnerability (e.g. ToolShell, CVE-2025-53770)",
          "type": "text",
          "showIf": {"q": "sp_deployment", "in": ["SharePoint Server (on-prem)", "Both / hybrid"]},
          "ask": "Do you think it was a known SharePoint vulnerability, like ToolShell?"
        },
        {
          "id": "sp_amsi",
          "label": "Is AMSI integration enabled with Defender (or another AV) on the farm?",
          "type": "yesno",
          "showIf": {"q": "sp_deployment", "in": ["SharePoint Server (on-prem)", "Both / hybrid"]},
          "ask": "Is AMSI switched on for SharePoint?"
        },
        {
          "id": "sp_machinekeys",
          "label": "Have the ASP.NET machine keys been rotated since the suspected compromise?",
          "type": "yesno",
          "showIf": {"q": "sp_deployment", "in": ["SharePoint Server (on-prem)", "Both / hybrid"]},
          "help": "Stolen machine keys can allow continued access even after patching.",
          "ask": "Have you rotated the machine keys since this happened?"
        },
        {
          "id": "sp_farm_servers",
          "label": "Farm servers and roles",
          "type": "textarea",
          "showIf": {"q": "sp_deployment", "in": ["SharePoint Server (on-prem)", "Both / hybrid"]},
          "ask": "Which servers are in the farm?"
        },
        {
          "id": "sp_service_accounts",
          "label": "Do SharePoint farm or service accounts hold high AD privileges?",
          "type": "yesno",
          "showIf": {"q": "sp_deployment", "in": ["SharePoint Server (on-prem)", "Both / hybrid"]},
          "ask": "Do the SharePoint service accounts have high AD rights?"
        }
      ]
    },
    {
      "id": "sp_data",
      "title": "SharePoint data access & exposure",
      "description": "What content may have been reached, and what logs can show it.",
      "questions": [
        {
          "id": "sp_webshells",
          "label": "Web shells or unexpected .aspx files found on the farm?",
          "type": "yesno",
          "showIf": {"q": "sp_deployment", "in": ["SharePoint Server (on-prem)", "Both / hybrid"]},
          "ask": "Have you found any web shells or odd .aspx files?"
        },
        {"id": "sp_sites", "label": "Sites, libraries or folders believed to be accessed", "type": "textarea", "ask": "Which sites or libraries do you think they got into?"},
        {"id": "sp_sensitive", "label": "Sensitive content held in the affected sites", "type": "multiselect", "options": ["Personal data (PII)", "Special category / health data (PHI)", "Payment card data (PCI)", "Credentials / secrets", "Intellectual property", "Financial records", "Customer data", "Employee / HR data", "Legal / privileged material", "None identified", "Unknown"], "ask": "What kind of data is on those sites?"},
        {"id": "sp_mass_download", "label": "Evidence of bulk downloads or library sync?", "type": "yesno", "ask": "Any sign of bulk downloads or syncing?"},
        {
          "id": "sp_sharing",
          "label": "Suspicious sharing links (anyone links, external guests) created?",
          "type": "yesno",
          "showIf": {"q": "sp_deployment", "in": ["SharePoint Online", "Both / hybrid"]},
          "ask": "Have you seen any new sharing links or external guests?"
        },
        {
          "id": "sp_external_sharing",
          "label": "Tenant external sharing setting",
          "type": "select",
          "options": ["Anyone", "New and existing guests", "Existing guests only", "Only people in your organisation", "Unknown"],
          "showIf": {"q": "sp_deployment", "in": ["SharePoint Online", "Both / hybrid"]},
          "ask": "How is external sharing set up? Are 'anyone' links allowed?"
        },
        {"id": "sp_audit", "label": "Audit logs available", "type": "select", "options": ["Purview audit (UAL) for SharePoint Online", "ULS / IIS logs on-prem", "Both", "Neither", "Unknown"], "ask": "What logs have you got for SharePoint?"},
        {"id": "sp_labels", "label": "Are sensitivity labels or DLP applied to the affected content?", "type": "yesno", "ask": "Do you use sensitivity labels or DLP on that content?"}
      ]
    }
  ],
  "reportChecks": [{"id": "sharepoint__vuln", "label": "Has the exploited vulnerability been identified?"}, {"id": "sharepoint__webshells", "label": "Have web shells and unexpected .aspx files been searched for?"}, {"id": "sharepoint__machine_keys", "label": "Has machine key theft been assessed?"}, {"id": "sharepoint__logs", "label": "Have ULS, IIS or Purview audit logs been reviewed?"}, {"id": "sharepoint__sites", "label": "Have the sites and libraries accessed been identified?"}, {"id": "sharepoint__downloads", "label": "Have bulk downloads and sync been assessed?"}, {"id": "sharepoint__sharing", "label": "Have sharing links and external access been reviewed?"}, {"id": "sharepoint__post_exploit", "label": "Has post-exploitation activity on the farm servers been assessed?"}]
}
```

### DATA FILE: json/incident-types/webserver.json

```json
{
  "incidentType": {"id": "webserver", "label": "Web server / IIS compromise"},
  "sections": [
    {
      "id": "ws_app",
      "title": "Web application & hosting",
      "description": "What the application is, where it runs and what it can reach.",
      "questions": [
        {"id": "ws_stack", "label": "Hosting stack", "type": "multiselect", "options": ["IIS / ASP.NET", "IIS / classic ASP", "Azure App Service", "Apache", "nginx", "Tomcat / Java", "PHP", "Node.js", "Other"], "ask": "What's the website running on? IIS, Apache, something else?"},
        {"id": "ws_app_type", "label": "Application type", "type": "select", "options": ["Custom in-house application", "Commercial product / portal", "CMS (e.g. WordPress, Umbraco, Sitecore)", "ERP / CRM web front end", "API", "Other"], "ask": "What kind of application is it? In-house, off the shelf, a CMS?"},
        {"id": "ws_urls", "label": "Affected URLs / hostnames", "type": "textarea", "ask": "Which sites or URLs are we talking about?"},
        {"id": "ws_hosting", "label": "Where it is hosted", "type": "select", "options": ["On-prem DMZ", "On-prem internal (published via reverse proxy)", "Azure VM", "Azure App Service", "Other cloud", "Third-party hosting"], "ask": "Where's it hosted? On-prem, Azure, a hosting company?"},
        {"id": "ws_appservice_access", "label": "If Azure App Service: were Kudu / SCM, FTP or deployment credentials used?", "type": "yesno", "showIf": {"q": "ws_hosting", "equals": "Azure App Service"}, "ask": "Did they get in through Kudu, FTP or the deployment credentials?"},
        {"id": "ws_waf", "label": "Is a WAF, CDN or reverse proxy in front (e.g. Front Door, Application Gateway)?", "type": "yesno", "ask": "Is there a WAF or reverse proxy in front of it?"},
        {"id": "ws_domain_joined", "label": "Is the web server domain-joined?", "type": "yesno", "ask": "Is the server joined to the domain?"},
        {"id": "ws_app_identity", "label": "Application pool / service identity and its privileges", "type": "text", "ask": "What account does the app run as, and what can it get to?"},
        {"id": "ws_backend", "label": "Back-end systems it connects to (databases, APIs, file shares)", "type": "textarea", "ask": "What does it connect to behind the scenes? Databases, file shares?"},
        {"id": "ws_secrets", "label": "Are credentials or connection strings stored in config files (e.g. web.config)?", "type": "yesno", "ask": "Are there passwords or connection strings in the config files?"},
        {"id": "ws_source", "label": "Is source code and a known-good build available for comparison?", "type": "yesno", "ask": "Have you got the source code, or a clean copy to compare against?"}
      ]
    },
    {
      "id": "ws_activity",
      "title": "Web compromise indicators",
      "description": "What was seen, and what the logs can still show.",
      "questions": [
        {"id": "ws_how_found", "label": "What was observed", "type": "multiselect", "options": ["Web shell", "Defacement", "Unexpected files or scheduled tasks", "Suspicious child processes from w3wp.exe", "Data exposure / leak", "Redirects or injected scripts (e.g. card skimming)", "Crypto mining", "WAF / EDR alert", "Other"], "ask": "What did you actually see? A web shell, defacement, an alert?"},
        {"id": "ws_webshell_details", "label": "Web shell or file details (paths, hashes, timestamps)", "type": "textarea", "ask": "Where was it, and when was it created?"},
        {"id": "ws_vuln", "label": "Suspected vulnerability or entry point", "type": "text", "ask": "Any idea how they got in?"},
        {"id": "ws_logs", "label": "Web server log retention", "type": "select", "options": ["More than 90 days", "30 to 90 days", "Less than 30 days", "Logging disabled", "Unknown"], "ask": "How far back do the web logs go?"},
        {"id": "ws_xff", "label": "Do the logs capture the real client IP (X-Forwarded-For) behind the proxy?", "type": "yesno", "ask": "Do the logs show the real visitor IP, or just the proxy?"},
        {"id": "ws_db_access", "label": "Evidence the database or back-end data was accessed?", "type": "yesno", "ask": "Any sign they got into the database?"},
        {"id": "ws_lateral", "label": "Evidence of movement from the web server into the internal network?", "type": "yesno", "ask": "Did they get from the web server into the internal network?"},
        {"id": "ws_personal_data", "label": "Does the application process personal or payment data?", "type": "yesno", "ask": "Does the site handle personal data or card payments?"}
      ]
    }
  ],
  "reportChecks": [{"id": "webserver__entry_point", "label": "Has the entry point or vulnerability been identified?"}, {"id": "webserver__webshells", "label": "Have web shells and unexpected files been searched for?"}, {"id": "webserver__logs", "label": "Have web logs been reviewed, with the real client IP?"}, {"id": "webserver__child_procs", "label": "Have suspicious child processes of the web server been reviewed (e.g. w3wp.exe)?"}, {"id": "webserver__secrets", "label": "Has exposure of secrets in config files been assessed?"}, {"id": "webserver__backend", "label": "Has back-end and database access been assessed?"}, {"id": "webserver__pivot", "label": "Has movement into the internal network been assessed?"}, {"id": "webserver__integrity", "label": "Has the site been compared with a known-good source?"}]
}
```

### DATA FILE: json/incident-types/azure.json

```json
{
  "incidentType": {"id": "azure", "label": "Azure subscription / workload compromise"},
  "sections": [
    {
      "id": "az_estate",
      "title": "Azure estate & logging",
      "description": "What Azure can still tell us. Subscription IDs are in Case summary.",
      "questions": [
        {"id": "az_sub_count", "label": "Total subscriptions in the tenant", "type": "number", "ask": "How many subscriptions have you got in total?"},
        {"id": "az_mgmt_groups", "label": "Are management groups and Azure Policy in use?", "type": "yesno", "ask": "Do you use management groups and Azure Policy?"},
        {"id": "az_resources", "label": "Resource types involved", "type": "multiselect", "options": ["Virtual machines", "AKS / containers", "App Service / Functions", "Storage accounts", "Key Vault", "Azure SQL / Cosmos DB", "Virtual networks / NSGs", "Azure DevOps / pipelines", "Logic Apps / Automation accounts", "Other"], "ask": "What kind of resources are involved? VMs, storage, Key Vault?"},
        {"id": "az_activity_log", "label": "Activity Log retention", "type": "select", "options": ["Exported to Log Analytics / Sentinel", "Exported to storage / event hub", "Default 90 days only", "Unknown"], "ask": "Do your Azure activity logs go into Log Analytics or Sentinel?"},
        {"id": "az_diag", "label": "Are diagnostic settings enabled for key resources (Key Vault, storage, NSG flow logs)?", "type": "yesno", "ask": "Have you got diagnostic logging on for things like Key Vault and storage?"},
        {"id": "az_defender", "label": "Is Defender for Cloud enabled with paid plans on the affected subscriptions?", "type": "yesno", "ask": "Is Defender for Cloud switched on, with the paid plans?"}
      ]
    },
    {
      "id": "az_activity",
      "title": "Azure compromise activity",
      "description": "Which identities were used and what changed in the subscriptions.",
      "questions": [
        {"id": "az_detection", "label": "How it was spotted", "type": "multiselect", "options": ["Defender for Cloud alert", "Unexpected cost / billing spike", "Crypto mining", "Resources created or deleted", "Data exposure report", "Leaked secret found", "Other"], "ask": "How did you spot it? An alert, a jump in the bill?"},
        {"id": "az_identity", "label": "Identities used by the attacker", "type": "multiselect", "options": ["User account", "Service principal / app registration", "Managed identity", "Storage account key or SAS token", "Azure DevOps / GitHub service connection", "Unknown"], "ask": "Which accounts or identities did they use?"},
        {"id": "az_role_changes", "label": "Were RBAC role assignments added (e.g. Owner, Contributor, User Access Administrator)?", "type": "yesno", "ask": "Have they given themselves any new roles, like Owner?"},
        {"id": "az_vm_actions", "label": "VM actions observed", "type": "multiselect", "options": ["Run Command", "Custom Script Extension", "Password reset via VM Access extension", "Disk snapshot / export", "New VMs created", "None", "Unknown"], "ask": "Did they run commands on any VMs, or create new ones?"},
        {"id": "az_data", "label": "Was data in storage, databases or Key Vault accessed or exfiltrated?", "type": "yesno", "ask": "Any sign they got to data in storage or databases?"},
        {"id": "az_keyvault", "label": "Were Key Vault secrets, keys or certificates read?", "type": "yesno", "ask": "Did they read anything from Key Vault?"},
        {"id": "az_storage_keys", "label": "Were storage account keys listed or SAS tokens generated?", "type": "yesno", "help": "Look for listKeys operations in the Activity Log.", "ask": "Did they pull storage account keys or make SAS links?"},
        {"id": "az_network", "label": "Were NSG, firewall or public network access settings changed?", "type": "yesno", "ask": "Have they changed any firewall or network settings?"},
        {"id": "az_destructive", "label": "Were resources, backups or Recovery Services vaults deleted?", "type": "yesno", "ask": "Have they deleted anything, including backups?"},
        {"id": "az_backup", "label": "Azure Backup protection", "type": "select", "options": ["Soft delete and immutability enabled", "Soft delete only", "Not configured", "Unknown"], "ask": "Are your Azure backups protected with soft delete or immutability?"},
        {"id": "az_cost", "label": "Billing impact so far (amount and currency)", "type": "text", "ask": "Has it hit your bill? Roughly how much?"}
      ]
    }
  ],
  "reportChecks": [
    {"id": "azure__identities", "label": "Have the identities used by the attacker been identified?"},
    {"id": "azure__activity_log", "label": "Has the Azure Activity Log been reviewed for the whole window?"},
    {"id": "azure__rbac_changes", "label": "Have role assignment changes been identified?"},
    {"id": "azure__resources", "label": "Have resources created, changed or deleted been identified?"},
    {"id": "azure__data_plane", "label": "Has data-plane access been assessed (storage, Key Vault, databases)?"},
    {"id": "azure__keys", "label": "Have storage key listing and SAS creation been checked?"},
    {"id": "azure__network", "label": "Have network exposure changes been checked?"},
    {"id": "azure__defender", "label": "Have Defender for Cloud alerts been reviewed?"},
    {"id": "azure__cost", "label": "Is the cost impact documented?"}
  ]
}
```
