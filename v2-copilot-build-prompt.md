# IR Scoping v2: phased build prompts for Microsoft Copilot

Build the app in 16 small phases. Each phase is one copy-paste prompt.

## How to use

1. Open a new Copilot chat. Paste **PHASE 0** and wait for "Ready".
2. Paste **PHASE 1**. Copilot returns the complete starter `v2.html`. Save it and open it in Chrome or Edge to check it.
3. For every later phase: paste the prompt. Copilot returns code blocks, each labelled with a **marker** (for example `// @@JS-DB`). In `v2.html`, find that marker line and paste the block **directly below it**. Keep the marker line itself, because later phases paste below the same markers.
4. If a phase says "REPLACE FUNCTION X", delete the old function and paste the new one in the same place.
5. After each phase, open the file, run the "CHECK" list, and fix anything before moving on. If something breaks, paste the error from the browser console (F12) back to Copilot.
6. **If the chat gets long or Copilot loses track**: start a new chat, paste PHASE 0, then paste your current `v2.html` with the line "This is the current file. Continue from the next phase.", then paste the next phase.
7. If an answer is cut off, reply: "Continue exactly where you stopped, inside the same code block."

Phase list:

- 0 Kick-off and contract (no code)
- 1 Design system and skeleton (full file)
- 2 Utilities, database, state, autosave, home screen
- 3 Question engine and common questions
- 4 Incident-type and closing questions
- 5 Case shell, questionnaire wizard, summary
- 6 Incident timezone and UTC
- 7 Register engine, engagement timeline, chart and PNG export
- 8 Attack timeline (MITRE ATT&CK)
- 9 Requests to client
- 10 IOC register and CSV export
- 11 Affected assets register
- 12 Decision log and hours budget
- 13 JSON export / import and Markdown report
- 14 Case phases, home flags, full search, backup reminder
- 15 Print-ready report
- 16 Final review against acceptance tests

***

## PHASE 0: Kick-off and contract

```
We are going to build a single-file web app called "IR Scoping v2" in 16 phases. Do not write any code in this message. Read the contract below, keep it for the whole conversation, and reply only with "Ready".

WHAT IT IS
An offline case manager for an incident response (DFIR) consultant: pick an incident type, walk through a scoping questionnaire, then manage the case (engagement timeline, attack timeline mapped to MITRE ATT&CK, requests to the client, IOCs, affected hosts and accounts, decision log, hours budget) and print a report.

HARD RULES (apply to every phase)
1. One file, v2.html, inline CSS and vanilla JavaScript. No frameworks, libraries, CDNs, web fonts or network calls.
2. Must run by double-clicking the file (file:// URL) in current Chrome and Edge.
3. Case data is stored in IndexedDB only. localStorage is used only for the theme preference.
4. Never lose typed data: debounced autosave, and flush saves when the tab is hidden or closed.
5. Config-driven: questions, lists, statuses and colours live in config objects, not in logic.
6. 'use strict', const/let, small functions, clear comments, no clutter.
7. Escape all user text before putting it into HTML or SVG, using esc().
8. Never use the em dash character in UI text or generated output.
9. Never use alert(), confirm() or prompt(). Use the custom <dialog> modal.
10. In every phase after phase 1, return ONLY new or changed code, as separate code blocks. Start each block with a comment naming the marker it goes under, for example "// PASTE BELOW: // @@JS-DB" or "/* PASTE BELOW: @@CSS-HOME */". If an existing function must change, say "REPLACE FUNCTION name" and give the whole new function. Do not repeat unchanged code.

MARKERS (created in phase 1, never removed)
CSS: @@CSS-BASE, @@CSS-HOME, @@CSS-WIZARD, @@CSS-REGISTERS, @@CSS-REPORT, @@CSS-PRINT
JS, in this order: @@JS-CONFIG-CORE, @@JS-CONFIG-QUESTIONS, @@JS-CONFIG-CASE, @@JS-UTILS, @@JS-DB, @@JS-STATE, @@JS-UI, @@JS-QUESTION-LOGIC, @@JS-TIMEZONE, @@JS-METRICS, @@JS-REGISTER-HELPERS, @@JS-REGISTERS (inside the REGISTERS object literal), @@JS-HOME, @@JS-SHELL, @@JS-WIZARD, @@JS-SUMMARY, @@JS-REGISTER-VIEW, @@JS-CHART, @@JS-REPORT, @@JS-EXPORT, @@JS-INIT

DATA MODEL (one IndexedDB record per case; database "ir-scoping", version 1, store "incidents", keyPath "id", index "updatedAt")
{
  id (UUID), title, clientName, incidentType,
  phase: 'scoping' | 'collection' | 'analysis' | 'reporting' | 'closed',
  status: 'draft' | 'complete'   (kept in sync: closed means complete),
  createdAt, updatedAt (ISO strings), currentStep (wizard section index),
  answers: { questionId: string | number | string[] },
  budget: { hoursCap: number or '', warnPct: number (default 80) },
  lastExportedAt: ISO string or null (set only by JSON export),
  timeline: [], attack: [], requests: [], iocs: [], assets: [], decisions: [], hours: []
}
Register items always have id and createdAt, plus updatedAt after an edit. Datetimes are "YYYY-MM-DDTHH:MM" strings in the incident timezone; dates are "YYYY-MM-DD".

SHARED NAMES (use exactly these so phases fit together)
Utilities: $, $$, esc, uuid, nowISO, clone, errMsg, fmtDateTime, fileSlug, dateStamp, download(filename, content, mime), isEmpty, normOptions
DB: DB.open, DB.getAll, DB.get, DB.put, DB.putMany, DB.delete, DB.mode ('idb' or 'memory')
State: state, markDirty(touch = true), commit(), flushSave(), setSaveStatus, normalizeRecord, newRecord, upsertLocal
UI: applyTheme, showBanner, toast(msg, kind, ms), openModal({title, body, buttons, onOpen}), confirmModal(title, message, label, danger)
Questions: QUESTIONNAIRE, QINDEX, getSections(type), getVal, setVal, isVisible, sectionStats, overallStats, formatAnswer(q, v, rec), typeLabel
Timezone: incidentTz, isValidTz, zonedToUtc, utcHint, fmtUtc, fmtLocal, fmtDate, nowLocalString, todayLocal, utcOf
Views: renderHome, renderList, openIncident, renderShell(tab, html), refreshTabs, openTab, updateHeader, renderWizard, renderSummary, renderRegister(key), renderReport
Registers: REGISTERS (object keyed by register key), REGISTER_ORDER
Chart/export: buildTimelineSVG, exportChartPNG, exportIncidentsJSON, exportAllJSON, toMarkdown, exportMarkdown

Reply only "Ready".
```

***

## PHASE 1: Design system and skeleton

```
PHASE 1: Design system and skeleton. Return the COMPLETE v2.html for this phase only.

Create the full page structure with all markers from the contract, the complete design system CSS, and a working top bar with a theme toggle. JS sections that later phases fill should contain just their marker comment (plus anything listed below).

PAGE STRUCTURE
<head>: charset, viewport, <title>IR Scoping v2</title>, a tiny inline script that reads localStorage key "ir-scoping-theme" (in try/catch) and sets data-theme on <html> before first paint, then <style>.
<body>:
- header.topbar: brand button #brandHome (26px rounded square mark with "IR", gradient 135deg from accent to #8a5cf6, white bold text; then "IR Scoping" and a small accent "v2"), spacer, span#saveStatus (aria-live polite), ghost button #themeToggle.
- div#dbBanner.banner (role alert, hidden)
- main#app (tabindex -1)
- footer#appFoot.app-foot
- dialog#modal.modal
- div#toasts.toasts (aria-live polite)
- input#importFile type file, accept ".json,application/json", hidden
- <script> with 'use strict' and the JS markers in contract order. In the @@JS-REGISTERS area write: const REGISTERS = { // @@JS-REGISTERS  }; and below it: const REGISTER_ORDER = ['timeline','attack','requests','iocs','assets','decisions','hours'].filter(k => REGISTERS[k]);

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
- Under @@JS-INIT: an init function that applies the current theme, wires #themeToggle, and puts a placeholder in #app: a card saying "Skeleton ready. Next phases add features." plus three sample buttons (primary, default, danger) and three badges, so the styling can be checked. Footer text: "IR Scoping v2.0.0".

CHECK
Opening the file shows the dark top bar and sample card; the theme toggle switches all colours and is remembered after reload; no console errors.
```

***

## PHASE 2: Utilities, database, state, autosave, home screen

```
PHASE 2: Utilities, database, state, autosave and the home screen. Return only code blocks per marker (contract rule 10).

@@JS-CONFIG-CORE
APP_VERSION '2.0.0', SCHEMA_VERSION 3, DB_NAME 'ir-scoping', DB_VERSION 1, STORE 'incidents', THEME_KEY 'ir-scoping-theme', AUTOSAVE_DELAY 400.
INCIDENT_TYPES: ransomware "Ransomware", bec "Business Email Compromise", malware "Malware / Endpoint Compromise", exfiltration "Data Exfiltration", insider "Insider Threat", other "Other". TYPE_IDS, and typeLabel(id) returning the label, "Unknown (id)" or "Not set".
ALL_REGISTER_KEYS = ['timeline','attack','requests','iocs','assets','decisions','hours'] (used by normalizeRecord even before registers exist).

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
renderHome(): flush saves, set view 'home', clear current, load state.incidents = (await DB.getAll()).map(normalizeRecord) (toast on error). Render: h1 "Incidents", subtitle "Cases saved in this browser.", buttons "Import JSON" (#btnImport clicks #importFile), "Export all" (#btnExportAll, for now a toast "Arrives in phase 13"), "New incident" (primary). Then div#backupBanner (empty for now), div#phaseFilter (empty for now), search input #search (placeholder "Search everything: titles, answers, events, IOCs, hosts, requests...") with span#listCount "N of M", and div#list.card. Document title "IR Scoping v2".
renderList(): filter by search (case-insensitive over title, client, type label for now), sort by updatedAt newest first. Empty states: "No incidents yet" / "Start a new case, or import a JSON export." and "No incidents match ...". Table columns: Title (link-btn data-act open), Type, Client, Phase (plain text for now), Last updated, actions: Open, Duplicate, JSON, Markdown, Delete (danger). Cells carry data-label. Use event delegation on #list.
New incident modal: Incident type select (required, "Select incident type..."), Client name, Incident title (placeholder "Defaults to client, type and date"); default title = [client, type label, dateStamp()] joined with " - ". Save with DB.put, upsertLocal, then openIncident(id) (for now openIncident just toasts "Opened <title>"; phase 5 replaces it).
Duplicate: deep copy, new id, title "Copy of ...", phase scoping, status draft, new timestamps, currentStep 0, lastExportedAt null.
Delete: confirmModal("Delete incident?", "This permanently deletes <b>title</b> (client) from this browser. Export it first if you may need it.", "Delete", true).
JSON and Markdown buttons: toast "Arrives in phase 13" for now.

@@JS-INIT (replace the phase 1 placeholder)
init: apply theme, wire theme toggle and #brandHome (renderHome), await DB.open(); if false, showBanner("<strong>Warning: IndexedDB is unavailable.</strong> <error> Data is held in memory only and will be lost when this tab closes. Export to JSON before closing. Common causes: private / InPrivate window, browser storage disabled, or strict site-data settings."); else call navigator.storage.persist() ignoring errors. Footer: "IR Scoping v2.0.0 · schema 3 · storage: IndexedDB (this browser profile)" or "memory only". Then renderHome().

CHECK
Create, duplicate and delete cases; they survive a browser restart; search filters the list; at 400px width the table becomes cards.
```

***

## PHASE 3: Question engine and common questions

```
PHASE 3: Question engine and common questions. Return code blocks per marker.

@@JS-CONFIG-QUESTIONS
Create const DATA_TYPES = [Personal data (PII), Special category / health data (PHI), Payment card data (PCI), Credentials / secrets, Intellectual property, Financial records, Customer data, Employee / HR data, Legal / privileged material, None identified, Unknown] and const QUESTIONNAIRE = { commonSections: [...], typeSections: {}, closingSections: [] } (phase 4 fills typeSections and closingSections).
Put a comment at the top explaining the schema below so the user can edit questions.

Question schema: { id (unique across ALL sections), label, type, options?, help?, placeholder?, suggestions?, bind?, showIf? }
Types: text, textarea, number, date, time, datetime, select, multiselect, yesno (Yes/No/Unknown), timezone.
bind: store on the record instead of answers ('title', 'clientName', 'incidentType').
showIf: { q, equals } | { q, in: [...] } | { q, notEquals } | { q, notEmpty: true }.
Section: { id, title, description, questions }.

Notation below: id | type | label | options | condition | help. "yn" means yesno.

engagement "Engagement & client" (desc "Core engagement details. Title, client and incident type are also shown on the incident list.")
_title | text, bind title | Incident title | placeholder "e.g. ACME Corp ransomware Oct 2026"
_clientName | text, bind clientName | Client name
_incidentType | select, bind incidentType | Incident type | INCIDENT_TYPES as {value: id, label} | help "Changing this switches the type-specific sections. Answers already given for another type are kept but hidden."
discovery_tz | timezone | Incident timezone | help "All date/time answers in this incident are recorded in this timezone and also shown in UTC."
client_industry | select | Client industry / sector | Financial services, Insurance, Healthcare, Pharma / life sciences, Manufacturing, Retail / e-commerce, Technology / SaaS, Professional services / legal, Education, Public sector / government, Energy / utilities, Telecoms, Transport / logistics, Charity / non-profit, Other
client_size | select | Organisation size (employees) | 1-49, 50-249, 250-999, 1,000-4,999, 5,000-19,999, 20,000+
client_hq | text | Headquarters country and other operating countries
engagement_ref | text | Internal case / matter reference
engaged_via | select | Engaged via | Client directly, Outside counsel, Cyber insurer / panel, Broker, MSSP / partner, Other
privilege | yn | Is the engagement under legal privilege? | help "If yes, confirm labelling and distribution requirements for all work product."
counsel_details | text | Outside counsel firm and lead contact | if privilege = Yes

contacts "Points of contact" (desc "Who we work with day to day, and how we talk to them safely.")
poc_name | text | Primary contact name
poc_role | text | Primary contact role | placeholder "e.g. CISO, Head of IT"
poc_email | text | Primary contact email
poc_phone | text | Primary contact phone
poc_tech | text | Technical lead (name, role, contact)
poc_exec | text | Executive sponsor / decision maker
comms_compromised | yn | Is corporate email or chat believed to be compromised?
comms_channel | select | Agreed communication channel | Corporate email, Corporate Teams / Slack, Phone only, Out-of-band (Signal / WhatsApp), Separate clean tenant / bridge, Other
comms_oob_details | textarea | Out-of-band channel details | if comms_compromised in Yes, Unknown | help "Assume the threat actor may be reading corporate comms until proven otherwise."
poc_availability | textarea | Availability, working hours and escalation path

discovery "Discovery & detection" (desc "When and how the incident came to light.")
discovery_dt | datetime | Date and time of discovery
earliest_activity_dt | datetime | Earliest known suspicious activity (if known)
detection_source | multiselect | How was it detected? | EDR / AV alert, SIEM alert, MDR / SOC notification, User report, IT noticed outage / performance issue, Ransom note / extortion message, Third-party notification (customer, supplier), Law enforcement / NCSC notification, Bank / financial anomaly, Threat intel / dark web monitoring, Audit / internal review, Other
detection_details | textarea | Detection narrative | help "Who noticed what, when, and what happened next."
activity_ongoing | yn | Is malicious activity believed to be ongoing?
prior_incidents | yn | Any related or prior incidents in the last 12 months?
prior_incidents_details | textarea | Prior incident details | if prior_incidents = Yes

environment "Environment & affected systems" (desc "Size and shape of the estate, and what is known to be affected.")
affected_systems_count | number | Number of systems known or suspected to be affected
affected_users_count | number | Number of user accounts known or suspected to be affected
total_endpoints | number | Total endpoints (workstations / laptops)
total_servers | number | Total servers (physical and virtual)
env_platforms | multiselect | Platforms in the environment | On-prem Active Directory, Entra ID (Azure AD), Hybrid identity (AD Connect), Microsoft 365, Google Workspace, AWS, Azure, GCP, VMware ESXi / vCenter, Hyper-V, Citrix / VDI, OT / ICS, Mainframe
env_os | multiselect | Operating systems in scope | Windows, Linux, macOS, ESXi, Network appliances, Mobile
edr_product | text | EDR / AV product(s) deployed
edr_coverage | select | Estimated EDR coverage | 95-100%, 75-94%, 50-74%, Under 50%, No EDR, Unknown
siem_product | text | SIEM / log platform
msp | text | MSP / MSSP involvement (name, scope of service)
sites_affected | textarea | Sites, regions or business units affected

impact "Business impact" (desc "Operational and data impact as currently understood.")
impact_level | select | Overall business impact | Critical: core operations halted, High: significant degradation, Moderate: limited disruption, Low: no noticeable disruption, Unknown
impacted_functions | multiselect | Business functions impacted | Production / manufacturing, Customer-facing services, Finance / payments, Payroll / HR, Email / collaboration, Logistics / supply chain, Clinical / patient care, Website / e-commerce, Internal IT only, None
critical_systems_down | textarea | Critical systems currently unavailable
data_at_risk | multiselect | Data types potentially at risk | DATA_TYPES
records_estimate | number | Estimated number of individuals / records affected
impact_cost | text | Estimated cost of disruption (per day or total, with currency)
media_attention | yn | Is there media or public attention?

regulatory "Regulatory & notification" (desc "Notification obligations and clocks that may already be running.")
reg_frameworks | multiselect | Potentially applicable regimes | UK GDPR / DPA 2018 (ICO), EU GDPR, NIS Regulations / NIS2, DORA, FCA / PRA, PCI DSS, HIPAA, SEC cyber disclosure (8-K), US state breach laws, Sector regulator (other), None identified, Unknown
dpo_engaged | yn | Has the DPO / privacy team been engaged?
reg_notified | yn | Has any regulator been notified?
reg_notified_details | textarea | Regulator(s), date notified and reference | if reg_notified = Yes
reg_deadline | datetime | Earliest notification deadline | help "e.g. 72 hours from awareness for UK/EU GDPR."
contract_notify | yn | Contractual notification obligations to customers or partners?
contract_notify_details | textarea | Contractual obligations details | if contract_notify in Yes, Unknown

insurance "Cyber insurance & broker" (desc "Policy, panel and approval requirements.")
insured | yn | Does the client hold cyber insurance?
insurer | text | Insurer | if insured = Yes
policy_number | text | Policy number | if insured = Yes
broker | text | Broker (firm and contact) | if insured = Yes
insurer_notified | yn | Has the insurer been notified? | if insured = Yes
claim_ref | text | Claim reference | if insurer_notified = Yes
panel_requirements | textarea | Panel / pre-approval requirements (rates, scope caps, reporting) | if insured = Yes

law_enforcement "Law enforcement" (desc "Current or planned law enforcement and government agency involvement.")
le_involved | yn | Is law enforcement involved?
le_agencies | multiselect | Agencies involved | Action Fraud / NFIB, Regional Organised Crime Unit (ROCU), NCA, Local police, NCSC, FBI, CISA / US Secret Service, Europol / national police (EU), Other | if le_involved = Yes
le_reference | text | Crime / case reference and officer contact | if le_involved = Yes
le_planned | select | Does the client intend to report? | Yes, No, Undecided, Awaiting legal advice | if le_involved in No, Unknown
le_constraints | textarea | Any constraints from law enforcement (e.g. evidence handling, do-not-alert) | if le_involved = Yes

containment "Containment actions taken" (desc "What has already been done. Note anything that may have destroyed or altered evidence.")
containment_actions | multiselect | Actions already taken | Hosts isolated (network / EDR), Internet egress blocked or restricted, VPN / remote access disabled, Accounts disabled, Password resets (users), Password resets (privileged / service accounts), krbtgt reset, Sessions / tokens revoked, MFA re-registered, IOCs blocked at firewall / proxy, Systems powered off, Backups isolated, External access (RDP, etc.) closed, None yet, Other
containment_details | textarea | Containment details and timings
systems_rebuilt | yn | Have any affected systems been wiped, reimaged or restored?
systems_rebuilt_details | textarea | Which systems, and was anything captured first? | if systems_rebuilt = Yes
ta_access_removed | yn | Is the threat actor believed to be locked out?
third_party_ir | yn | Has any other IR firm or MSP performed work so far?
third_party_ir_details | textarea | Who, what was done, and are their outputs available? | if third_party_ir = Yes

evidence "Logs & evidence preserved" (desc "What evidence exists, how long it will survive, and how we can get it.")
evidence_preserved | yn | Has any evidence been preserved?
evidence_types | multiselect | Evidence preserved so far | Memory images, Full disk images, Triage collections (KAPE / Velociraptor / CyLR), VM snapshots, EDR telemetry export, Windows event logs, Firewall logs, Proxy / web gateway logs, VPN logs, DNS logs, NetFlow, M365 Unified Audit Log, Entra sign-in / audit logs, Cloud audit logs (CloudTrail / Activity Log), Email samples, Malware samples | if evidence_preserved = Yes
log_retention | textarea | Retention of key log sources | help "e.g. firewall 30 days, EDR 7 days, SIEM 90 days. Flag anything about to roll over."
chain_of_custody | yn | Is chain of custody being recorded?
evidence_location | text | Where is preserved evidence stored?
remote_collection | yn | Can we deploy collection tooling remotely (EDR / Velociraptor)?
access_requirements | textarea | Access requirements (accounts, VPN, jump host, approvals)

@@JS-QUESTION-LOGIC
- QINDEX: Map of every question across common, all type sections and closing sections; console.warn on duplicate ids and on showIf pointing to unknown ids.
- getSections(type): common (group "Common") + typeSections[type] (group = type label) + closing (group "Scope"), each section copied with a group property.
- getVal(rec, q) / setVal(rec, q, v): use rec[q.bind] when bound; empty answers are deleted from answers (bound fields become '').
- conditionMatches(value, cond): arrays match if any element matches.
- isVisible(rec, q): true without showIf; false if the controller is hidden; otherwise conditionMatches.
- sectionStats(rec, s) => {total, answered} over visible questions; overallStats(rec) sums all sections.
- formatAnswer(q, v, rec): null when empty; incidentType shows the type label; selects and multiselects show option labels (multiselect joined with ", "); datetime shows "YYYY-MM-DD HH:MM" for now (phase 6 adds the timezone).

CHECK
In the console: getSections('ransomware').length is 10 for now, there are no duplicate-id warnings, and isVisible works for counsel_details after setting answers.privilege = 'Yes'.
```

***

## PHASE 4: Incident-type and closing questions

```
PHASE 4: Incident-type and closing questions. Return one code block to paste under @@JS-CONFIG-QUESTIONS, directly after the QUESTIONNAIRE object, that assigns QUESTIONNAIRE.typeSections = {...} and QUESTIONNAIRE.closingSections = [...]. Use the same schema and notation as phase 3. (QINDEX is built later in @@JS-QUESTION-LOGIC, so it will include these.)

RANSOMWARE (key ransomware)
rw_actor "Ransom note & threat actor" (desc "Threat actor identification and extortion details."): rw_note_found yn "Has a ransom note been found?"; rw_note_details textarea "Ransom note filename, contents or contact method" (if rw_note_found = Yes); rw_variant text "Suspected variant / group" (placeholder "e.g. Akira, LockBit, Play"); rw_extension text "Encrypted file extension"; rw_ta_contact yn "Has anyone contacted the threat actor or visited the negotiation portal?"; rw_negotiator text "Is a negotiator engaged? (firm and contact)" (if rw_ta_contact = Yes); rw_demand text "Ransom demand (amount and currency)"; rw_deadline datetime "Threat actor deadline"; rw_exfil_claimed yn "Does the threat actor claim data theft?"; rw_leak_site yn "Is the client listed on a leak site?"; rw_payment_stance select "Client position on payment" (Will not pay, Considering, Undecided, Awaiting legal / insurer advice); rw_sanctions yn "Has sanctions screening (OFSI / OFAC) been considered?" (if rw_payment_stance in Considering, Undecided, Awaiting legal / insurer advice).
rw_scope "Encryption scope & recovery" (desc "Blast radius, identity infrastructure and recoverability."): rw_encryption_start datetime "When did encryption start (first observed)?"; rw_encrypted_types multiselect "System types encrypted" (Workstations, File servers, Database servers, Application servers, Domain controllers, ESXi / hypervisors, NAS / SAN, Backup servers, Cloud VMs, OT / ICS); rw_encrypted_count number "Approximate number of encrypted systems"; rw_dc_impact yn "Are domain controllers encrypted or compromised?"; rw_dc_details textarea "DC details (how many, which sites, any healthy DCs left)" (if rw_dc_impact in Yes, Unknown); rw_hypervisor yn "Were hypervisors / vCenter targeted?"; rw_backup_status select "Backup status" (Intact, offline / immutable; Intact but online / reachable; Partially affected; Encrypted or deleted; No backups; Unknown); rw_backup_product text "Backup product and storage location"; rw_last_good_backup datetime "Last known good backup"; rw_backup_tampering yn "Evidence of backup tampering (deleted jobs, VSS deletion, console access)?"; rw_initial_access multiselect "Suspected initial access vector" (VPN / firewall vulnerability, Exposed RDP, Phishing, Valid credentials (no MFA), Third-party / supplier access, Unpatched public-facing application, Unknown); rw_decryptor yn "Has a public decryptor been checked (e.g. No More Ransom)?"; rw_recovery_priorities textarea "Recovery priorities (systems in order)".

BUSINESS EMAIL COMPROMISE (key bec)
bec_tenant "Mail platform & identity" (desc "Tenant, logging and identity controls."): bec_platform select "Mail platform" (Microsoft 365 / Exchange Online, Exchange on-prem, Hybrid Exchange, Google Workspace, Other); bec_tenant text "M365 / Entra tenant ID and primary domain" (if bec_platform in Microsoft 365 / Exchange Online, Hybrid Exchange); bec_licence select "Licence level" (E5 / A5 / G5, E3 / A3 / G3, Business Premium, Business Standard / Basic, Google Workspace Enterprise, Google Workspace Business, Mixed, Unknown; help "Drives log availability (e.g. MailItemsAccessed, retention)."); bec_ual yn "Is the Unified Audit Log enabled?" (same condition as bec_tenant); bec_admin_access yn "Can we be granted read-only admin roles (Global Reader, Security Reader)?"; bec_affected_count number "Number of affected mailboxes"; bec_affected_mailboxes textarea "Affected mailboxes / accounts" (placeholder "One per line"); bec_privileged yn "Are any affected accounts privileged (admin roles)?"; bec_mfa_status select "MFA status for affected users" (Enforced for all users, Enforced for some users, Not enabled, Unknown); bec_mfa_methods multiselect "MFA methods in use" (SMS / voice, Authenticator push, Push with number matching, TOTP code, FIDO2 / passkey, Windows Hello; if bec_mfa_status in Enforced for all users, Enforced for some users); bec_ca yn "Are Conditional Access policies in place?"; bec_legacy_auth yn "Is legacy authentication blocked?"; bec_aitm yn "Is AiTM phishing or token theft suspected?".
bec_activity "Mailbox activity & financial loss" (desc "Attacker actions in the mailbox and any resulting fraud."): bec_rules yn "Suspicious inbox rules found?"; bec_rules_details textarea "Inbox rule details (names, conditions, actions)" (if bec_rules = Yes); bec_forwarding yn "External forwarding configured?"; bec_oauth yn "Suspicious OAuth app consents or new app registrations?"; bec_mfa_added yn "New MFA methods or devices registered by the attacker?"; bec_phish_sent yn "Was the mailbox used to send phishing (internal or external)?"; bec_phish_count number "Approximate number of recipients" (if bec_phish_sent = Yes); bec_file_access yn "Evidence of SharePoint / OneDrive / Drive access by the attacker?"; bec_fraud yn "Fraudulent payment or bank detail change?"; bec_loss text "Financial loss (amount and currency)" (if bec_fraud = Yes); bec_payment_date date "Date of fraudulent payment" (if bec_fraud = Yes); bec_bank_recall yn "Has the bank been contacted to recall funds?" (if bec_fraud = Yes); bec_recall_status text "Recall status / bank reference" (if bec_bank_recall = Yes); bec_third_parties textarea "Suppliers or customers impersonated or targeted"; bec_remediated yn "Have sessions been revoked and passwords reset for affected accounts?".

MALWARE / ENDPOINT (key malware)
mw_details "Malware details" (desc "What was found and where."): mw_family text "Suspected malware family / tooling" (placeholder "e.g. Qakbot, Cobalt Strike, infostealer"); mw_detection_name text "Detection name(s) from security tooling"; mw_sample yn "Is a sample available?"; mw_iocs textarea "Known IOCs (hashes, file paths, domains, IPs)"; mw_delivery multiselect "Suspected delivery vector" (Email attachment, Malicious link, Drive-by / SEO poisoning / malvertising, Removable media, Software supply chain, Exploit of public-facing service, Cracked / pirated software, Unknown); mw_host_count number "Number of hosts affected"; mw_host_roles multiselect "Roles of affected hosts" (User workstations, Privileged admin workstations, Servers, Domain controllers, Web servers, Developer machines, Cloud workloads); mw_hosts textarea "Affected hostnames" (placeholder "One per line"); mw_patient_zero text "Suspected patient zero (host / user)".
mw_behaviour "Behaviour & spread" (desc "Post-compromise activity and current host state."): mw_edr_result select "Security tooling outcome" (Blocked, Detected, not blocked, Not detected, No EDR on host, Unknown); mw_c2 yn "Command and control traffic observed?"; mw_c2_details textarea "C2 indicators and timeframe" (if mw_c2 = Yes); mw_persistence multiselect "Persistence mechanisms identified" (Services, Scheduled tasks, Run keys, WMI subscriptions, Startup folder, Remote access tools (AnyDesk, etc.), None identified, Unknown); mw_cred_access yn "Indicators of credential access (LSASS dumping, browser credential theft)?"; mw_lateral yn "Indicators of lateral movement?"; mw_lateral_details textarea "Lateral movement details (protocols, accounts, hosts)" (if mw_lateral = Yes); mw_host_state select "Current state of affected hosts" (Running, connected; Running, isolated; Powered off; Reimaged; Mixed); mw_user_impact yn "Were affected users privileged, or did they access sensitive data?".

DATA EXFILTRATION (key exfiltration)
dx_data "Data & indicators of theft" (desc "What is believed to have left the environment and why we think so."): dx_how_known multiselect "Why is exfiltration suspected?" (Threat actor claim / leak site, Data found online, Extortion email, DLP alert, Unusual egress volume, Third-party notification, Forensic finding (e.g. rclone, archives), Other); dx_claim_details textarea "Claim / alert details"; dx_data_types multiselect "Data types believed exfiltrated" (DATA_TYPES); dx_source textarea "Source repositories (file shares, databases, SaaS)"; dx_volume text "Estimated volume (e.g. 120 GB, 40k files)"; dx_records number "Estimated number of data subjects / records"; dx_proof yn "Has the threat actor provided proof (file tree, samples)?"; dx_proof_validated yn "Has the proof been validated as genuine client data?" (if dx_proof = Yes).
dx_channel "Exfiltration channel & timeline" (desc "How and when data moved, and what telemetry covers it."): dx_channels multiselect "Suspected exfiltration channel" (Cloud storage (MEGA, Dropbox, etc.), rclone / similar sync tool, FTP / SFTP, HTTP(S) upload, Email, C2 channel, Removable media, Personal cloud / SaaS account, Unknown); dx_tools text "Tools or destinations identified"; dx_window_start datetime "Exfiltration window start"; dx_window_end datetime "Exfiltration window end"; dx_egress_logs yn "Firewall / proxy logs available covering the window?"; dx_netflow yn "NetFlow or bandwidth data available?"; dx_cloud_logs yn "Cloud / SaaS audit logs available covering the window?"; dx_extortion yn "Is there an extortion demand?"; dx_extortion_details textarea "Extortion details (amount, deadline, contact)" (if dx_extortion = Yes); dx_affected_parties textarea "Third parties whose data may be involved".

INSIDER THREAT (key insider)
it_subject "Subject & allegation" (desc "Handle with care: confirm need-to-know and HR / legal oversight before any action."): it_subject_role text "Subject role / department (avoid names unless required)"; it_subject_status select "Subject employment status" (Current employee, Serving notice, Departed, Contractor, Third-party / supplier staff); it_leave_date date "Leaving / departure date" (if it_subject_status in Serving notice, Departed); it_allegation select "Nature of allegation" (Data theft, Sabotage, Fraud, Unauthorised access, Policy violation, Harassment / misconduct (digital evidence), Other); it_allegation_details textarea "Allegation summary and trigger"; it_privileged yn "Does the subject hold privileged or admin access?"; it_subject_aware yn "Is the subject aware of the investigation?"; it_covert yn "Must the investigation remain covert?"; it_hr yn "Is HR engaged?"; it_legal yn "Is legal (internal or external) engaged?".
it_evidence "Devices, evidence & constraints" (desc "Sources available and legal constraints on collection."): it_devices multiselect "Devices and accounts in scope" (Corporate laptop, Corporate desktop, Corporate mobile, Personal device (BYOD), Removable media, Corporate email, Corporate cloud storage, Collaboration / chat, Physical access records); it_devices_secured yn "Have relevant devices been secured?"; it_access_revoked yn "Has the subject's access been revoked or restricted?"; it_monitoring_policy yn "Is there an acceptable use / monitoring policy the subject was notified of?"; it_jurisdiction text "Employment jurisdiction(s)"; it_works_council yn "Works council or union consultation required?"; it_dlp yn "DLP / CASB / UEBA data available?"; it_outcome select "Anticipated outcome" (Disciplinary, Civil litigation, Criminal referral, Undecided); it_evidential_standard yn "Is the work expected to support expert witness / court proceedings?".

OTHER (key other)
ot_details "Incident details" (desc "For incidents that do not fit the other categories."): ot_category select "Closest category" (Cloud account / tenant compromise, Web application compromise, Edge device exploitation (VPN, firewall), DDoS, Supply chain / third-party compromise, OT / ICS incident, Credential stuffing / account takeover, Lost or stolen device, Unknown, Other); ot_description textarea "Description of the incident"; ot_assets textarea "Key assets involved"; ot_indicators textarea "Known indicators or observables"; ot_vendor yn "Is a third-party vendor or supplier involved?"; ot_vendor_details textarea "Vendor details and their engagement so far" (if ot_vendor = Yes).

CLOSING SECTION (all types)
scope "Objectives & scope" (desc "What the client needs from us, so scope and estimate are right first time."): objectives textarea "Key questions the client needs answered" (help "e.g. How did they get in? Was data taken? Are they still in?"); services multiselect "Services requested" (Forensic investigation, Threat hunting / compromise assessment, Containment & eradication support, Recovery support, Threat actor communications support, Dark web monitoring, Regulatory / legal reporting support, Executive briefing); deliverables multiselect "Expected deliverables" (Verbal briefings, Daily status updates, Interim findings, Full technical report, Executive summary, IOC list, Remediation recommendations); onsite yn "Is on-site presence required?"; onsite_location text "On-site location(s)" (if onsite = Yes); urgency select "Urgency" (Immediate (start today), Within 24 hours, Within the week, Planned); budget text "Budget or hours constraints"; next_steps textarea "Agreed next steps and owners"; scoping_notes textarea "Other scoping notes".

CHECK
getSections('ransomware').length is 13; getSections('other').length is 12; no duplicate-id warnings.
```

***

## PHASE 5: Case shell, questionnaire wizard, summary

```
PHASE 5: Case shell, questionnaire wizard and summary. Return code blocks per marker.

@@JS-CONFIG-CASE (add)
PHASES: scoping "Scoping" #3b82f6, collection "Collection" #d97706, analysis "Analysis" #8b5cf6, reporting "Reporting" #0d9488, closed "Closed" #6b7280. phaseOf(id).

@@CSS-WIZARD
- .inc-head: flex wrap, space-between, gap 12px 20px, margin-bottom 16px; .meta row (flex wrap gap 8px, muted); .actions buttons.
- .phase-pick: inline-flex label "Phase" + small select (auto width, min-height 30px, padding 3px 8px, left border 4px coloured by the phase).
- .tabs: flex wrap, gap 2px, bottom border, margin-bottom 18px. .tab: borderless button, 2px transparent bottom border, muted 600, padding 10px 12px; hover text colour; .active = accent-strong text, accent bottom border. .tab-badge: pill min-width 20px, padding 0 7px, surface-3, 0.74rem 650; .warn and .danger variants with 25% tints.
- .progress-meta (flex space-between, 0.88rem muted), .progress-bar (8px pill, surface-3) with inner span gradient accent to #8a5cf6, width transition.
- .wiz-body: grid 280px 1fr, gap 20px. .sec-nav: sticky top 72px, max-height viewport minus 92px, scroll, padding 10px. .nav-group: 0.74rem uppercase muted 700. .nav-item: full-width row, label left, count right (0.76rem muted); hover surface-2; .active = accent 18% tint, accent-strong text; .done count green with a check mark prefix.
- .jump-wrap (hidden on wide screens). .sec-form padding 22px 24px; header with group label (0.78rem uppercase accent 700), h2, description muted, bottom border.
- .q (margin-bottom 20px), labels and legends 600, .help 0.85rem muted, .q.conditional (padding-left 14px, 2px left border accent 45%).
- Choice cards .choice-grid repeat(auto-fill, minmax(230px, 1fr)); .choice with checkbox; checked = accent border + 12% tint.
- Segmented yes/no .seg (three radio labels, min-height 38px, padding 6px 18px, checked = accent bg, accent-contrast text) and .clear-btn (underlined muted link).
- .wiz-foot (flex space-between, top border, padding-top 16px), .kbd-hint 0.8rem muted.
- Summary: .sum-sec cards (padding 16px 20px), header row (h2 1.05rem, count, Edit button), dl grid minmax(200px, 38%) 1fr with dashed row separators, dd pre-wrap, .empty italic.
- At 900px and below: single column, hide .sec-nav, show .jump-wrap. At 700px summary dl stacks.

@@JS-SHELL
- incidentHeaderHTML(): h1#incTitle; meta: badge#incType, span#incClient, badge#incTz, label.phase-pick with select#phaseSelect (options from PHASES), span#backupChip.chip; actions: "All incidents" #btnHome, "Export JSON" #btnJson, "Export Markdown" #btnMd (toast "Arrives in phase 13" until then).
- updateHeader(): fills title (or "Untitled incident"), type label, client (or "No client set"), timezone badge "Timezone not set" (phase 6 completes it), phase select value and its border colour, document title "<title> - IR Scoping v2". Leave the backup chip empty until phase 14.
- renderShell(tab, innerHtml): sets state.activeTab, clears app.onclick, renders header + nav#tabs + div#view, wires header buttons, phase select (setPhase), tab clicks (openTab), scrolls to top.
- refreshTabs(): tabs Questionnaire (badge answered/total), Summary, then one tab per REGISTER_ORDER key (label REGISTERS[k].tab, badge from REGISTERS[k].tabBadge(rec) if defined, else the item count, hidden when 0), then Report (only if renderReport exists). Active tab gets aria-current="page".
- openTab(tab): flushSave then route to renderWizard(true), renderSummary(), renderReport() or renderRegister(tab).
- setPhase(id): set phase and status, commit(), updateHeader(), toast "Phase set to X."
- REPLACE FUNCTION openIncident(id): load, normalizeRecord, clamp currentStep, set state.current, status saved, renderWizard(true).

@@JS-WIZARD
- renderWizard(focusFirst): renderShell('wizard', ...) with progress area, left sidebar nav#secNav, jump select (optgroups by group), and form#secForm (novalidate) containing the section header, the questions and the footer (Back disabled on the first section; Next, or "Review summary" on the last), and the hint "Enter in a single-line field: next section. Ctrl + Enter: next from anywhere."
- refreshChrome(): rebuild sidebar (group headings, "answered/total" counts, done and active states), jump select, progress text "Section 3 of 13: <title>" and "X of Y questions answered (Z%)", progress bar width, and refreshTabs().
- renderField(rec, q): wrapper div.q[data-qid] (hidden when not visible, .conditional when it has showIf). Label for single inputs; fieldset + legend for multiselect and yesno. Optional help paragraph linked with aria-describedby. Controls: textarea rows 4; number (min 0, step any); date; time; datetime (datetime-local); select with "Select..." first, plus a "(not in current list)" option for unknown stored values; multiselect checkbox cards; yesno segmented radios with a Clear button shown when answered; text with optional datalist from q.suggestions; timezone renders a plain select for now (phase 6 completes it).
- readField(wrap, q) for each type (multiselect returns array, number returns Number or '').
- Input and change events on the form: setVal, markDirty(), updateHeader when the question is bound, toggle conditional visibility in place (do not re-render; focus must stay), refreshChrome. Clear button resets the radios.
- Keyboard: Enter in single-line inputs (text, number, date, time, datetime-local) without a datalist goes Next; Ctrl or Cmd + Enter anywhere in the wizard goes Next (global keydown, ignored when the modal is open).
- goToStep(i): set currentStep, markDirty(false) (navigation does not change updatedAt), flushSave, renderWizard(true). goNext(): last section goes to renderSummary().
- Focus the first visible field only when matchMedia('(pointer: fine)') matches.

@@JS-SUMMARY
renderSummary(): renderShell('summary', ...): a muted line "X of Y questions answered. Created <date>, last updated <date>." then one .sum-sec card per section: "<title> (<group>)", "answered/total", Edit button (data-step) that calls goToStep; definition list of visible questions with formatAnswer or "Not answered". Set app.onclick after renderShell to handle the Edit buttons.

CHECK
New incident opens the questionnaire; conditional questions appear without losing focus; Enter moves on; refresh keeps the answers and section; sidebar counts and tab badge update live; Summary Edit jumps to the right section; the phase select persists.
```

***

## PHASE 6: Incident timezone and UTC

```
PHASE 6: Incident timezone and UTC conversion. Return code blocks per marker and any replaced functions.

@@JS-CONFIG-CORE (add) TZ_QUESTION_ID = 'discovery_tz'.
@@JS-TIMEZONE
- TIMEZONES: ['UTC', ...Intl.supportedValuesOf('timeZone')] de-duplicated, with a short fallback list (UTC, Europe/London, Europe/Dublin, Europe/Paris, Europe/Berlin, Europe/Helsinki, America/New_York, America/Chicago, America/Denver, America/Los_Angeles, Asia/Dubai, Asia/Kolkata, Asia/Singapore, Asia/Tokyo, Australia/Sydney).
- BROWSER_TZ from Intl.DateTimeFormat().resolvedOptions().timeZone.
- incidentTz(rec), isValidTz(tz) (try new Intl.DateTimeFormat with timeZone).
- tzOffsetMs(ms, tz) via formatToParts (hourCycle h23); zonedToUtc(local, tz): parse "YYYY-MM-DDTHH:MM", guess UTC, subtract the offset, then re-check the offset at the result and correct (two passes, correct across DST). Return null when invalid.
- fmtUtc(date) "YYYY-MM-DD HH:MM UTC"; fmtOffset(ms) "UTC+01:00".
- utcHint(rec, value): no timezone: "Set the incident timezone (Engagement & client) to see UTC."; invalid: "Unrecognised timezone "X". Pick one from the list to see UTC."; empty value: "Enter in Europe/London."; else "Europe/London (UTC+01:00) = 2026-10-07 08:30 UTC".
- fmtLocal("2026-10-07T09:30") => "07 Oct 2026 09:30" and fmtDate("2026-10-07") => "07 Oct 2026" using string parsing only (never a Date object, to avoid shifting).
- nowLocalString(tz) "YYYY-MM-DDTHH:MM" for now in that zone (browser zone if unset or invalid); todayLocal(tz); utcOf(rec, local) returns a Date or null.

REPLACE FUNCTION renderField: datetime renders a .dt-row with the input and span.utc-hint[data-utc-for] showing utcHint. timezone renders a select ("Select timezone..." then all TIMEZONES, plus "X (not a recognised timezone)" for unknown stored values) and a small button "Use browser timezone (<zone>)" that sets the select and fires change.
REPLACE the form input handler so that changing the timezone question or any datetime refreshes every UTC hint in the section and updates the header timezone badge.
REPLACE FUNCTION formatAnswer: timezone shows "Europe/London (currently UTC+01:00)"; datetime shows "2026-10-07 09:30 Europe/London (2026-10-07 08:30 UTC)" or "2026-10-07 09:30 (timezone not set)".
REPLACE FUNCTION updateHeader so #incTz shows incidentTz(rec) or "Timezone not set".
CSS (under @@CSS-WIZARD): .dt-row flex wrap gap 8px 14px align centre; .dt-row select max-width 340px; .utc-hint 0.88rem muted tabular-nums; date/time inputs max-width 280px.

CHECK
Europe/London: 2026-07-01 09:30 => 08:30 UTC; 2026-12-01 09:30 => 09:30 UTC. America/New_York 2026-10-07 09:30 => 13:30 UTC. Asia/Kolkata 09:30 => 04:00 UTC.
```

***

## PHASE 7: Register engine, engagement timeline, chart and PNG

```
PHASE 7: Generic register engine, the engagement timeline register, the SVG timeline chart and PNG export. Return code blocks per marker.

REGISTER DEFINITION SHAPE
{ tab, title, singular, intro, fields: [...], sort(a, b), columns: [...], chart?, panel?(rec), wirePanel?(el), normalize?(item, rec) => {item} or {error}, dedupeKey?(item), merge?(a, b), rowActions?: [{id, label, show(item), apply(item)}], rowClass?(item, rec), toolbar?: [{id, label, run(rec)}], tabBadge?(rec) => {text, warn}, afterChange?(rec, before) }
Field: { id, label, type: text|textarea|number|date|datetime|select, required, default: 'now'|'today'|value, options or groups(rec) (optgroups), suggestions(rec) (datalist), placeholder, wide, step, onChange(value, form, rec) }
Column: { label, text(item, rec), html?(item, rec), exportText?(item, rec), cls? }  (text for search and Markdown, html for screen, exportText for sharing such as defanged values)

@@JS-CONFIG-CASE (add)
TIMELINE_GROUPS: engagement "Engagement" #3b82f6, evidence "Evidence" #d97706, analysis "Analysis" #8b5cf6, containment "Containment" #dc2626, reporting "Reporting" #16a34a, other "Other" #6b7280.
TIMELINE_EVENT_TYPES (id, label, group): engagement_start "Engagement started" engagement; scoping_call "Scoping call" engagement; client_meeting "Client meeting / status call" engagement; access_requested "Access requested" evidence; access_granted "Access granted" evidence; evidence_requested "Evidence requested" evidence; evidence_received "Evidence received" evidence; collection_started "Collection started" evidence; collection_done "Collection complete" evidence; analysis_started "Analysis started" analysis; analysis_done "Analysis complete" analysis; key_finding "Key finding" analysis; containment_action "Containment / remediation action" containment; interim_shared "Interim update shared" reporting; draft_report "Draft report shared" reporting; final_report "Final report shared" reporting; engagement_closed "Engagement closed" reporting; other "Other" other.
TIMELINE_EXPORT_PALETTE: bg #ffffff, surface #f6f8fb, border #d3dbe6, text #17202b, muted #5b6878, line #c3cdd9.

@@JS-REGISTER-HELPERS
whenText(rec, local) "07 Oct 2026 09:30 (2026-10-07 08:30 UTC)"; whenHTML (UTC on a second muted line); dot(color) small coloured circle span; tintBadge(label, color); optOf(options, value); byDatetime sorter; tlType(id); tlGroup(id).

@@JS-REGISTERS: entry timeline
tab "Engagement timeline", title "Engagement events", singular "event", intro "Engagement milestones: access and evidence requested or received, analysis, updates and reports shared."
fields: datetime (Date and time, required, default now); type (Event type, required, select with optgroups per TIMELINE_GROUPS); details (Details, textarea, wide, placeholder "e.g. Memory images for DC01 and FS02 requested from client IT").
sort byDatetime. columns: When (whenText / whenHTML, nowrap), Event (dot + label), Details (pre-wrap).
chart: { title "Engagement timeline", file "engagement-timeline", build(rec) => { events: [{datetime, title: event label, tag: group label, color: group colour, details}], legend: groups used } }.

@@CSS-REGISTERS
.reg-intro (muted, max-width 900px, margin-bottom 14px); .tl-note (warn notice: padding 10px 14px, radius 8px, warn 50% border, warn 10% background); .tl-layout (grid gap 18px); .tl-form (padding 18px 20px); .tl-fields (grid minmax(280px, auto) and minmax(220px, 1fr), gap 14px 18px; .f grid gap 6px; labels 600; .tl-desc spans both columns; textarea min-height 64px; datetime input max-width 260px); .tl-form-actions (flex gap 8px, margin-top 14px); .form-error (danger 600); .req (danger asterisk); .reg-panel (padding 16px 18px); .reg-toolbar (flex wrap space-between, padding 14px 16px 8px, h2 1.05rem); .tl-chart-card (padding 12px, overflow-x auto); .tl-chart svg (block, width 100%, height auto, max-width 960px, centred); .tl-dot (10px circle, margin-right 8px); .table-scroll (overflow-x auto); td.pre (pre-wrap); td.nowrap; td.mono (monospace 0.86rem, break-all); td.num (right, tabular); tr.overdue td (danger 8% tint). At 700px .tl-fields becomes one column.

@@JS-REGISTER-VIEW
renderRegister(key): view 'reg:<key>', renderShell(key, ...) with: intro; timezone notice when the register has datetime fields and no incident timezone ("No incident timezone set, so times are stored as entered and no UTC is shown. Set it in the Questionnaire under Engagement & client."); panel card #regPanel if panel; form#regForm (novalidate) with heading "Add <singular>", fields grid, p#regError, buttons "Add <singular>" (submit), "Cancel edit" (hidden), hint "Ctrl + Enter to save"; chart card if chart (heading + "Export PNG" #regPng + div#regChart role img); table card with title and count, toolbar buttons, div#regTable.
Field rendering: required asterisk; datetime fields show "(<tz>)" after the label, a "Now" button (data-now) and a live UTC hint (data-utc) under the input; selects start with "Select..."; text fields get a datalist from suggestions(rec).
Behaviour: defaults on reset ('now' and 'today' in the incident timezone); input/change refresh the UTC hint and call field.onChange; Ctrl+Enter submits. Submit: read values (trim, Number for number), check required ("Required: A, B."), normalize, dedupe (merge with toast "Merged with existing <singular>." or error "This <singular> is already in the register. Edit the existing entry instead."), add (id, createdAt) or update (updatedAt), remove empty fields, commit(), afterChange, reset form, refreshRegister(). Edit fills the form and switches labels to "Edit <singular>" / "Update <singular>". Delete asks confirmModal. Row quick actions apply then commit. Table: columns with data-label, rowClass, actions (quick actions, Edit, Delete). Empty state "Nothing recorded yet. Add the first <singular> above."
refreshRegister(): panel, chart, table, refreshTabs(). On-screen chart uses the current theme tokens read with getComputedStyle; REPLACE FUNCTION applyTheme to re-render the chart when a register view is open.

@@JS-CHART
buildTimelineSVG(rec, palette, {events, legend}, heading) returns {svg, width, height}:
- Width 960, padding 32. Title 20px bold wrapped; sub line 13px muted "<heading> · <client> · <type> · Timezone: <tz or not set>"; legend circles r 6 with 12px 600 muted labels, wrapping; 1px separator.
- Each event: local datetime 13px weight 650 right-aligned at x 220, UTC 11.5px muted beneath. Card from x 280 to the right padding: rect rx 8, surface fill, border stroke; 5px colour bar; title 14px bold wrapped (leave room for the tag); tag uppercase 10.5px bold in the event colour, right-aligned; details 13px wrapped, 18px line height; min height 48; 30px gap.
- Vertical 2px line at x 250 from first to last dot; dots r 8 in the event colour with a 3px stroke in the background colour.
- Gap labels between events ("+1h 30m", "+1d 5h", "+3d", "same time") 11px muted, right-aligned at x 234, centred in the gap. Use real UTC differences when the timezone is valid, else wall-clock.
- Footer 11px muted "5 events · Span 6d 5h · Generated <date>".
- Measure text with a hidden canvas 2D context using the same font stack; wrap by words; split long unbroken strings (hashes, paths) by characters.
exportChartPNG(rec, key): build with TIMELINE_EXPORT_PALETTE, load as data:image/svg+xml image, draw on a canvas at 2x (1x if 2x height exceeds 16000px), toBlob PNG, download "ir-<file>-<title slug>-<date>.png", toast "<title> exported as PNG.".

CHECK
Engagement timeline tab appears; adding, editing and deleting events works; the chart updates live and follows the theme; Export PNG downloads a clean light image.
```

***

## PHASE 8: Attack timeline (MITRE ATT&CK)

```
PHASE 8: Attack timeline register. Return code blocks per marker.

@@JS-CONFIG-CASE (add)
ATTACK_TACTICS (id, label, colour): TA0043 Reconnaissance #64748b; TA0042 Resource Development #78716c; TA0001 Initial Access #dc2626; TA0002 Execution #ea580c; TA0003 Persistence #d97706; TA0004 Privilege Escalation #a16207; TA0005 Defense Evasion #65a30d; TA0006 Credential Access #16a34a; TA0007 Discovery #0d9488; TA0008 Lateral Movement #0891b2; TA0009 Collection #2563eb; TA0011 Command and Control #4f46e5; TA0010 Exfiltration #9333ea; TA0040 Impact #db2777.
ATTACK_TECHNIQUES as [id, name, tactic]:
TA0001: T1566 Phishing; T1566.001 Spearphishing Attachment; T1566.002 Spearphishing Link; T1190 Exploit Public-Facing Application; T1133 External Remote Services; T1078 Valid Accounts; T1199 Trusted Relationship; T1189 Drive-by Compromise.
TA0002: T1059 Command and Scripting Interpreter; T1059.001 PowerShell; T1059.003 Windows Command Shell; T1047 Windows Management Instrumentation; T1204 User Execution; T1569.002 Service Execution.
TA0003: T1053.005 Scheduled Task; T1543.003 Windows Service; T1547.001 Registry Run Keys / Startup Folder; T1136 Create Account; T1098 Account Manipulation; T1505.003 Web Shell.
TA0004: T1068 Exploitation for Privilege Escalation; T1548.002 Bypass User Account Control; T1134 Access Token Manipulation.
TA0005: T1562.001 Disable or Modify Tools; T1070.001 Clear Windows Event Logs; T1070.004 File Deletion; T1027 Obfuscated Files or Information; T1036 Masquerading; T1218 System Binary Proxy Execution; T1564.008 Email Hiding Rules.
TA0006: T1003.001 LSASS Memory; T1003.003 NTDS; T1003.006 DCSync; T1110 Brute Force; T1558.003 Kerberoasting; T1555 Credentials from Password Stores; T1557 Adversary-in-the-Middle; T1621 Multi-Factor Authentication Request Generation; T1539 Steal Web Session Cookie.
TA0007: T1087 Account Discovery; T1018 Remote System Discovery; T1046 Network Service Discovery; T1482 Domain Trust Discovery; T1083 File and Directory Discovery.
TA0008: T1021.001 Remote Desktop Protocol; T1021.002 SMB/Windows Admin Shares; T1021.006 Windows Remote Management; T1570 Lateral Tool Transfer; T1550.002 Pass the Hash.
TA0009: T1560 Archive Collected Data; T1114 Email Collection; T1114.003 Email Forwarding Rule; T1005 Data from Local System; T1039 Data from Network Shared Drive.
TA0011: T1071.001 Web Protocols; T1219 Remote Access Software; T1572 Protocol Tunneling; T1090 Proxy; T1105 Ingress Tool Transfer.
TA0010: T1041 Exfiltration Over C2 Channel; T1567.002 Exfiltration to Cloud Storage; T1048 Exfiltration Over Alternative Protocol.
TA0040: T1486 Data Encrypted for Impact; T1490 Inhibit System Recovery; T1489 Service Stop; T1485 Data Destruction; T1657 Financial Theft.

@@JS-REGISTER-HELPERS (add): tactic(id) lookup (fallback grey); autoTactic(value, form): if the value starts with a known technique id and the tactic select is empty, set it; assetNames(kind) returning a suggestions function over rec.assets names.

@@JS-REGISTERS: entry attack
tab "Attack timeline", title "Threat actor activity", singular "activity", intro "Threat actor activity reconstructed from evidence, mapped to MITRE ATT&CK tactics. Host and account suggestions come from the Assets register."
fields: datetime (required, no default); tactic (ATT&CK tactic, required, options "Initial Access (TA0001)" style); technique (text, placeholder "e.g. T1021.001 Remote Desktop Protocol", suggestions "T1566.002 Spearphishing Link" style, onChange autoTactic); host (suggestions assetNames('host')); account (suggestions assetNames('account')); artefact (label "Source artefact", placeholder "e.g. Security.evtx 4624, $MFT, UAL, EDR telemetry"); description (textarea, wide).
sort byDatetime. columns: When, Tactic (dot + label), Technique, Host, Account, Source, Description.
chart: title "Attack timeline", file "attack-timeline"; event title = technique or tactic label, tag = tactic label, colour = tactic colour, details = description plus a line "Host: X  |  Account: Y  |  Source: Z" (only parts that exist); legend = tactics used, in tactic order.

CHECK
Typing "T1566.002 Spearphishing Link" with no tactic selects Initial Access; the chart is coloured by tactic; PNG export works.
```

***

## PHASE 9: Requests to client

```
PHASE 9: Requests register. Return code blocks per marker.

@@JS-CONFIG-CASE (add) REQUEST_STATUSES: open "Open" #3b82f6, chased "Chased" #d97706, received "Received" #16a34a, not_available "Not available" #6b7280.
@@JS-METRICS (add) requestStats(rec) => { by: counts per status, active: open + chased, overdue: items with status open or chased and a due date earlier than todayLocal(incident tz), today }. statTile(num, label, sub, level) helper returning the .stat markup.

@@JS-REGISTERS: entry requests
tab "Requests", title "Requests to client", singular "request", intro "Everything we have asked the client for. Open or chased requests past their due date are flagged as overdue."
fields: description (label "Request", textarea, required, wide, placeholder "e.g. Firewall logs for 01-07 Oct from both edge firewalls"); owner (placeholder "Who at the client is responsible"); raised (date, required, default today); due (date); status (select, required, default open); notes (textarea, wide).
sort: open/chased first, then due date ascending with empty last, then raised.
rowClass: 'overdue' for overdue items.
columns: Request (pre), Owner, Raised (fmtDate), Due (fmtDate plus a red "Overdue" flag when overdue), Status (tint badge in the status colour), Notes (pre).
rowActions: "Chased" (show when open, sets chased), "Received" (show when open or chased, sets received).
tabBadge: '' when none active; "N open" or "N open, M overdue" with warn 'danger' when overdue.
panel: stat tiles per status plus "Overdue" (danger level when above 0).

CHECK
A request due yesterday and still open is highlighted and flagged; the tab shows "1 open, 1 overdue"; quick actions update status.
```

***

## PHASE 10: IOC register and CSV export

```
PHASE 10: IOC register. Return code blocks per marker.

@@JS-CONFIG-CASE (add) IOC_TYPES: ip "IP address", domain "Domain", url "URL", email "Email address", md5 "MD5", sha1 "SHA1", sha256 "SHA256", filename "File name / path", other "Other". IOC_CONFIDENCE: High, Medium, Low.

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
```

***

## PHASE 11: Affected assets register

```
PHASE 11: Assets register (hosts and accounts). Return code blocks per marker.

@@JS-CONFIG-CASE (add) ASSET_KINDS: host "Host", account "Account". ASSET_STATUSES: suspected "Suspected" #d97706, confirmed "Confirmed" #dc2626, contained "Contained" #8b5cf6, remediated "Remediated" #16a34a.
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
Adding DC01 then dc01 as hosts is blocked; tiles and the table match the entries; the Attack timeline host field now suggests asset hosts.
```

***

## PHASE 12: Decision log and hours budget

```
PHASE 12: Decision log and hours register with a budget cap. Return code blocks per marker.

@@JS-CONFIG-CASE (add) DECISION_KINDS: Decision, Approval, Instruction from client, Advice given, Communication, Other. HOUR_ACTIVITIES: Scoping, Evidence collection, Analysis, Containment / remediation support, Reporting, Meetings / comms, Project management, Travel, Other. DEFAULT_BUDGET_WARN_PCT = 80.
@@JS-METRICS (add) hoursInfo(rec) => { used, cap, pct, warnPct, level: '' | 'warn' | 'danger' } (danger at 100% or more, warn at warnPct or more, '' when no cap). fmtH(n) rounds to 2 decimals without trailing zeros.

@@JS-REGISTERS: entry decisions
tab "Decisions", title "Decision and communications log", singular "entry", intro "Who decided or approved what, and when. Entries show when they were last edited, to support a defensible record."
fields: datetime (required, default now); kind (Type, select, required, default Decision); who (label "Made / approved by", required, placeholder "e.g. Client CISO (J. Smith)"); summary (required, wide, placeholder "e.g. Approved isolation of FS02"); details (label "Rationale, participants, channel", textarea, wide).
sort byDatetime. columns: When, Type, By, Summary (html adds a muted "Edited <date time>" line when updatedAt exists), Details (pre).

@@JS-REGISTERS: entry hours
tab "Hours", title "Hours log", singular "entry", intro "Time booked against this case. Set the hours cap (e.g. the insurer panel limit) to get warnings as you approach it."
fields: date (required, default today); person (required, suggestions = people already used in this case); hours (number, step 0.25, required); activity (select, required); description (wide).
normalize: hours must be more than 0 and no more than 24 ("Hours must be more than 0 and no more than 24.").
sort by date newest first. columns: Date, Person, Hours (num), Activity, Description.
tabBadge: '' when nothing logged and no cap; "11.5/10h" with a cap or "11.5h" without; warn = level.
panel: inputs "Hours cap" (#budgetCap, min 0, step 0.5, placeholder "No cap") and "Warn at (% of cap)" (#budgetWarn, 1 to 100); then div#hoursMeter. wirePanel: on input update rec.budget, markDirty(), refresh only #hoursMeter and the tabs (do not re-render the inputs).
hoursMeterHTML(rec): with a cap: 12px meter (.meter, green fill; .warn and .danger variants), text "<used>h of <cap>h used (<pct>%). <remaining>h remaining." or "... <over>h over cap." (danger); without a cap: "<used>h logged. No cap set."; then two mini tables "By activity" and "By person" sorted by hours.
afterChange(rec, before): toast "Hours at 85% of the 10h cap." (warn) when crossing the warning level and "Hours cap exceeded: 11.5 of 10h used." (error) when crossing 100%.
CSS: .budget-row (flex wrap, gap 14px, align end; labels grid gap 4px 600 0.88rem; inputs width 150px); .meter (12px pill, surface-3, margin 14px 0 6px; span success, width transition; .warn span warn; .danger span danger); .danger-text.

CHECK
Edit a decision and see "Edited ..." appear. With a 10h cap, adding 6h then 2.5h warns at 85%; adding 3h more warns that the cap is exceeded; 30h is rejected; the tab badge colours follow.
```

***

## PHASE 13: JSON export / import and Markdown report

```
PHASE 13: Exports and import. Return code blocks per marker and replace the temporary "Arrives in phase 13" handlers.

@@JS-EXPORT
- buildExport(records): JSON string, 2-space indent: { schemaVersion: 3, app: 'ir-scoping', appVersion: '2.0.0', exportedAt, count, incidents }.
- markExported(ids): chained on state.saveChain; for each id read from DB, set lastExportedAt = now WITHOUT changing updatedAt, put back, upsertLocal; also set it on state.current; then updateHeader and renderList when on home.
- exportIncidentsJSON(records, filename): flushSave, re-read each record from the DB, download, markExported, toast "Exported N incident(s) to JSON.".
- exportAllJSON(): all records, file "ir-scoping-all-<date>.json" (toast "Nothing to export yet." when empty).
- colExport(column, item, rec) = exportText or text.
- toMarkdown(rec): "# IR Case Report: <title>", bullets Client, Incident type, Phase, Incident timezone ("<tz or Not set> (date/times are local to this zone, UTC in brackets)"), Created, Last updated, Questions answered "X of Y", Record ID in backticks. Then "## <section>" for each questionnaire section with bold label lines (two trailing spaces) and answers or "_Not answered_" (hidden conditional questions omitted; multi-line answers keep line breaks). Then "## <tab>" for each non-empty register in REGISTER_ORDER: one bullet per item made of "**Column:** value" parts joined by " | " (IOCs defanged via exportText); the Hours section starts with "Xh logged against a cap of Yh (Z%)." Footer "***" and "_Generated <date> by IR Scoping v2.0.0 (schema 3)._".
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
```

***

## PHASE 14: Case phases, home flags, full search, backup reminder

```
PHASE 14: Home screen upgrades. Return code blocks per marker and replaced functions.

@@JS-CONFIG-CASE (add) BACKUP_WARN_DAYS = 3. phaseBadge(id) = tint badge in the phase colour.
@@JS-METRICS (add)
- relTime(ms): "just now", "Xm ago", "Xh ago", "Xd ago".
- backupInfo(rec): changedSince = never exported or updatedAt later than lastExportedAt; reference = lastExportedAt or createdAt; stale = changedSince and reference older than BACKUP_WARN_DAYS days; text "Last JSON export 2d ago" (+ ", changed since") or "Never exported to JSON".
- caseFlags(rec): "N overdue request(s)" (danger), "Hours at X% of cap" (hours level), "Backup due" (warn).

REPLACE FUNCTION renderList:
- #backupBanner: when any case is stale, a .notice.warn "Backup reminder: N case(s) have changes not exported to JSON in over 3 days." with button #bkExport "Export all now" (calls exportAllJSON).
- #phaseFilter chips: All, Open cases (not closed), then each phase, each with a count over all cases; clicking sets state.phaseFilter and re-renders; aria-pressed on the active chip.
- Filter by phase, then search with matchRecord; sort newest first; "N of M" counter.
- Phase column shows phaseBadge. Under each title: flags from caseFlags, and when the match was inside the case, a line "<where>: ...text with <mark>match</mark>..." (40 characters either side).
New helpers (@@JS-HOME):
- searchFields(rec): list of {where, text}: every visible non-bound question answer (formatAnswer, labelled with the question label) and every field of every register item (select values as labels, labelled with the register tab name).
- matchRecord(rec, term): search the term and its refanged form, case-insensitive; a match in title, client, type label or phase label returns {where: null}; otherwise the first field match with its index; null when no match.

REPLACE FUNCTION updateHeader: also fill #backupChip with backupInfo text ("Backup due: " prefix and .warn when stale).

CHECK
Phase chips filter and count correctly; searching adm_jsmith, evil[.]example, T1021 or a request description finds the case and shows where; a case created over 3 days ago and never exported shows "Backup due" and the banner; "Export all now" clears it without changing the case's last updated time.
```

***

## PHASE 15: Print-ready report

```
PHASE 15: Report tab and print styles. Return code blocks per marker.

@@JS-REPORT
renderReport(): renderShell('report', ...) with a toolbar (.report-toolbar): muted text "Print-ready view of the whole case. Use Print / Save as PDF and choose "Save as PDF" as the destination. IOCs are defanged." and primary button "Print / Save as PDF" (flushSave then window.print()); then article.report with reportHTML(rec).
reportHTML(rec):
- Header: kicker "INCIDENT RESPONSE CASE REPORT"; h1 title; metadata list: Client, Incident type, Phase, Timezone ("<tz> (local times shown, UTC in brackets)" or "Not set"), Created, Last updated, Generated, Record ID.
- "Key figures" stat tiles: Hosts compromised (sub "N suspected"), Accounts compromised, Privileged accounts compromised, IOCs, Attack timeline events, Open requests (sub "N overdue"), Hours used ("11.5h", sub "of 10h cap (115%)" or "No cap set"), Questions answered ("X/Y").
- "Scoping questionnaire": answered questions only, h3 per section and a definition list; then "N unanswered questions omitted."
- One section per register in REGISTER_ORDER rendered as a table (headers = column labels; cells use exportText if present, else html, else escaped text). Hours starts with "Xh logged against a cap of Yh (Z%).". Empty registers show "None recorded.".
Make sure refreshTabs shows the Report tab and openTab routes to it.

@@CSS-REPORT
.report-toolbar (flex wrap space-between, gap 12px, margin-bottom 14px). .report: override the theme tokens to the light values (surface #ffffff, surface-2 #f6f8fb, surface-3 #e9eef5, border #d3dbe6, text #17202b, muted #5b6878, accent #1d64d8, warn #a5710a, danger #c7323f, success #1e8a55), color-scheme light, white background, radius 12px, padding 36px 40px, max-width 1040px, centred. .rep-kicker (uppercase, letter-spacing 0.8px, 0.75rem 700 accent). h1 1.6rem. .rep-meta (grid max-content 1fr, gap 3px 18px, 0.9rem; dt muted). .rep-sec (margin-top 28px; h2 1.15rem with a 2px solid text-colour bottom border; h3 0.98rem). .rep-table (full width, collapsed, 0.84rem; cells padding 6px 8px with bottom border, top aligned; th surface-2, 0.72rem uppercase muted; td.pre pre-wrap). .rep-qa (grid minmax(180px, 36%) 1fr, 0.88rem, dotted row borders, dt muted, dd pre-wrap). At 700px: smaller padding and stacked Q/A.

@@CSS-PRINT
@media print: @page margin 14mm; force the light token values on :root and [data-theme]; print-color-adjust exact; white body, 12px; hide .topbar, .banner, .tabs, .inc-head, .report-toolbar, .app-foot, .toasts, all .btn, .wiz-foot and .sec-nav; main without padding or max-width; .report without padding, radius or max-width; .stat-grid 4 columns; avoid breaks inside table rows, Q/A pairs and stat tiles; avoid breaks after h2 and h3; table header rows repeat on every page.

CHECK
The Report tab looks like a clean white document in both themes. Print preview shows only the report, IOCs defanged, tables continuing across pages with repeated headers.
```

***

## PHASE 16: Final review against acceptance tests

```
PHASE 16: Review the whole current v2.html (I will paste it if you no longer have it) against these acceptance tests. For each test, say pass or fail. For every fail, give the fix as a REPLACE FUNCTION or a marker block. Do not rewrite unrelated code.

1. Opens with no console errors (missing favicon is fine).
2. New Ransomware case for ACME gets the title "ACME - Ransomware - <today>" and opens on Engagement & client.
3. Answering Yes to legal privilege reveals the counsel question without losing focus; reload keeps the answer and section.
4. Europe/London 2026-07-01 09:30 shows 08:30 UTC; 2026-12-01 09:30 shows 09:30 UTC; America/New_York 2026-10-07 09:30 shows 13:30 UTC.
5. An open request due in the past is flagged overdue in the table, the tab ("N open, 1 overdue") and the home list.
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
Also confirm: no em dash characters, no alert/confirm/prompt, all user text escaped, no network requests.
Finish with a short list of assumptions and how to add a new incident type, a new register field and a new IOC type.
```
