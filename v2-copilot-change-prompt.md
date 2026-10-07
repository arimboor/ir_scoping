# Copilot prompts: change the existing IR Scoping v2 app

Use these prompts when you want Copilot to modify the existing `v2.html` rather than build it from scratch.

## How to use

**Option 1: GitHub Copilot in VS Code (recommended).** Open the project folder, open Copilot Chat in **Agent** or **Edit** mode, and add `v2.html` as context (type `#file:v2.html` or use "Add context"). Paste PROMPT A, then PROMPT B with your change. Copilot edits the file directly and you review the diff.

**Option 2: Microsoft 365 Copilot (chat).** Upload or attach `v2.html`, or paste it in. The file is about 4,100 lines, so if it is too big to paste, paste PROMPT A and then only the sections that the change touches (the section map in PROMPT A tells you where things live). Copilot replies with find-and-replace edits that you apply by hand.

Workflow for every change:

1. Paste **PROMPT A** once at the start of the chat (context and rules).
2. Paste **PROMPT B** with your change filled in, or one of the ready-made requests at the end of this file.
3. Apply the edits, open `v2.html` in Chrome or Edge and test.
4. If something breaks, paste **PROMPT C** with the console error (F12).
5. Before keeping a bigger change, paste **PROMPT D** for a regression check.
6. Keep a copy of the working file before each change (for example `v2-backup-2026-10-07.html`), and export your cases to JSON from the app first ("Export all" on the home screen).

***

## PROMPT A: Context and rules (paste once per chat)

```
You are maintaining an existing single-file web app, v2.html ("IR Scoping v2"), which I have attached or pasted. It is an offline case manager for an incident response (DFIR) consultant, mainly for Microsoft-centric incidents (on-prem AD and Exchange, Entra ID, Microsoft 365, Azure). It has a scoping questionnaire per incident type with spoken prompts for the call, plus case registers (case timeline, attack timeline mapped to MITRE ATT&CK, IOCs, affected assets, time log), JSON / Markdown / CSV / PNG exports and a print-ready report. Read the file before proposing changes, and follow its existing patterns.

THE APP AS IT IS TODAY
- Home screen: phase filter chips, a search box that searches everything, a backup reminder banner, and a table with ONLY these columns: Case ID, Incident type, Customer, Severity, Case logged, then the buttons Open, Duplicate, JSON, Markdown, Delete (on one line on wide screens).
- New incident dialog: Incident type (required), Case ID, Case logged date (defaults to today), Customer, Geolocation (country dropdown from COUNTRIES), Severity (SEVERITIES: Sev1, SevA, SevB247, SevB, SevC). The case title is the Case ID on its own, never combined with other values ("Untitled incident" if empty). Customer is stored as clientName; Case ID, Case logged date, Geolocation and Severity are stored as answers engagement_ref, case_logged, geolocation, severity.
- Case tabs: Questionnaire, Summary, Case timeline, Attack timeline, IOCs, Assets, Hours, Report. The case header has a Phase dropdown (Scoping, Collection, Analysis, Reporting, Closed) and a backup status chip.
- Questionnaire sections, in order: Case summary (section id "engagement"), Discovery & detection, Environment & affected systems, Business impact, Containment actions taken, Logs & evidence preserved, then two sections for the chosen incident type, then Objectives & scope.
- Incident types (14): Ransomware, Business Email Compromise, Malware / Endpoint Compromise, Data Exfiltration, Insider Threat, Active Directory compromise, Entra ID / Microsoft 365 identity compromise, Hybrid identity compromise (Entra Connect / ADFS), Exchange Server (on-prem) compromise, SharePoint compromise (Server / Online), Web server / IIS compromise, Azure subscription / workload compromise, Help desk / Teams social engineering, Other (always last).
- Spoken prompts: every customer-facing question shows an "ASK" line under its heading, taken from the ASK object (keyed by question id): a short, natural question in plain British English for verbal use on the call, not scripted or formal. Internal fields (incident title, Case ID, case logged date, severity) have none.
- Date handling: "Date of discovery" and "Earliest known suspicious activity" are date-only. The Case timeline is date-only (its field id is still "datetime" for compatibility); the chart shows day gaps ("+3d", "same day"). The Attack timeline keeps date and time, shown in the incident timezone with UTC.
- Hours tab ("Time log"): each entry is only a Date and Time spent (minutes). Totals display as hours and minutes ("11h 30m") via entryMinutes / fmtMins / fmtHours. An optional hours cap with a warning percentage is kept in rec.budget.

REMOVED ON PURPOSE (do not bring back unless I ask)
- Questionnaire sections: Points of contact, Regulatory & notification, Cyber insurance & broker, Law enforcement.
- Questions: client industry, organisation size, headquarters country, legal privilege and outside counsel, on-site presence and location, urgency, budget or hours constraints.
- Registers: Requests to client, Decisions log. Hours fields for person, activity and description.
- Home table columns: Geolocation, Phase, Last updated, and the warning flags under the Case ID.
Older saved cases may still hold data for these in the record; it is simply not shown. Do not delete it.

HOW THE FILE IS ORGANISED (script sections, in order)
1. CONFIGURATION: APP_VERSION, SCHEMA_VERSION, DB names, TZ_QUESTION_ID, INCIDENT_TYPES, TIMELINE_GROUPS, TIMELINE_EVENT_TYPES, TIMELINE_EXPORT_PALETTE, SEVERITIES, COUNTRIES, DATA_TYPES, QUESTIONNAIRE (commonSections, typeSections, closingSections), ASK, then the case config: PHASES, BACKUP_WARN_DAYS, DEFAULT_BUDGET_WARN_PCT, IOC_TYPES, IOC_CONFIDENCE, ASSET_KINDS, ASSET_STATUSES, ATTACK_TACTICS, ATTACK_TECHNIQUES, TIMEZONES.
2. CONFIG INDEX & QUESTION LOGIC: QINDEX (warns about duplicate ids, unknown showIf targets and ASK keys without a question), getSections, getVal/setVal, isVisible (showIf), stats, timezone helpers (incidentTz, zonedToUtc, utcHint, fmtUtc), formatAnswer.
3. UTILITIES: $, $$, esc, uuid, nowISO, clone, errMsg, fmtDateTime, fileSlug, dateStamp, download.
4. DATABASE LAYER: DB object over IndexedDB ("ir-scoping", store "incidents", keyPath "id") with an in-memory fallback.
5. STATE & AUTOSAVE: state, newRecord, normalizeRecord (also upgrades older data: date-only questions, date-only case timeline, hours to minutes), upsertLocal, markDirty(touch), commit(), flushSave(), setSaveStatus.
6. UI HELPERS: applyTheme, showBanner, toast, openModal / confirmModal (<dialog>), fmtLocal, fmtDate, nowLocalString, todayLocal, utcOf, whenText / whenHTML, tintBadge, optOf, phaseOf, phaseBadge.
7. CASE METRICS: assetStats, entryMinutes, fmtMins, fmtHours, hoursInfo, backupInfo, caseFlags.
8. REGISTERS: helpers (refang, defang, detectIocType, normalizeIoc, mergeIoc, csvCell), the REGISTERS object (entries timeline, attack, iocs, assets, hours; each has tab, title, singular, intro, fields, sort, columns, and optional chart, panel, wirePanel, normalize, dedupeKey, merge, rowActions, rowClass, toolbar, tabBadge, afterChange), REGISTER_ORDER, statTile, hoursMeterHTML.
9. RENDERING: HOME: renderHome, searchFields, matchRecord, snippetHTML, renderList, onListClick, newIncidentFlow, duplicateIncident, openIncident.
10. RENDERING: CASE SHELL, WIZARD, SUMMARY: incidentHeaderHTML, updateHeader, renderShell, refreshTabs, openTab, setPhase, renderWizard, refreshChrome, renderField (shows the ASK line), readField, goToStep, renderSummary.
11. RENDERING: REGISTER VIEWS: generic renderRegister (form, table, panel, chart) and its handlers.
12. TIMELINE CHART AND PNG EXPORT: eventMs, fmtGap, buildTimelineSVG, exportChartPNG.
13. PRINT-READY REPORT: renderReport, reportHTML, reportTable.
14. EXPORT / IMPORT: buildExport, markExported, exportIncidentsJSON, exportAllJSON, exportIocCSV, toMarkdown, exportMarkdown, sanitizeRegister, sanitizeRecord, validateImport, mergeRecords, handleImportFile.
15. INIT.
CSS is grouped into theme tokens (dark default, light override via data-theme on <html>), base, controls, home, wizard (including .ask and .ask-tag), summary, modal and toasts, timeline/registers, v2 shell (tabs, phases, flags), registers (stats, meter), and report plus @media print.

KEY CONCEPTS
- Config-driven: questions are objects in QUESTIONNAIRE (id, label, type, options, help, placeholder, suggestions, bind, showIf, optional ask). Question types: text, textarea, number, date, time, datetime, select, multiselect, yesno, timezone. Registers are entries in REGISTERS; register field types: text, textarea, number, date, datetime, select.
- Case record: { id, title, clientName, incidentType, phase, status, createdAt, updatedAt, currentStep, answers, budget, lastExportedAt, timeline, attack, iocs, assets, hours }. Datetimes are "YYYY-MM-DDTHH:MM" strings in the incident timezone (answer discovery_tz); dates are "YYYY-MM-DD".
- Saving: markDirty() for typed changes (debounced), commit() for immediate content saves, markDirty(false) for navigation only (does not bump updatedAt).
- Theme colours are CSS custom properties; never hard-code colours in components, use the tokens.
- To remove a question: delete it from QUESTIONNAIRE and delete its ASK entry. If other questions depend on it via showIf, remove or rewire those too. Stored answers stay in old cases.

RULES FOR EVERY CHANGE
1. Make the smallest change that does what I ask. Do not refactor, rename or "improve" anything I did not ask for.
2. Keep it a single offline file: no libraries, CDNs, web fonts or network calls. Vanilla JS and inline CSS only.
3. Follow the existing style: config objects for data, the shared helpers above, esc() for every piece of user text that goes into HTML or SVG, section banner comments, similar comment density.
4. Never use alert(), confirm() or prompt(); use openModal / confirmModal and toast.
5. Never use the em dash character in UI text or generated output.
6. Protect existing data: never rename or remove stored ids or keys (question ids, register keys, field ids, option values) because saved cases and JSON exports use them. If you must rename something on screen, change only the label.
7. If you add a new field to the case record or a new register, also update everything that must know about it: newRecord, normalizeRecord, sanitizeRecord / sanitizeRegister, mergeRecords, toMarkdown, reportHTML, search (searchFields), and bump SCHEMA_VERSION only if old files would be read incorrectly. Old JSON files must still import.
8. Keep both themes working and keep the layout usable at 400px width and with the keyboard.
9. Every new customer-facing question needs an ASK entry: a short, natural, spoken question in plain British English (e.g. "Roughly how many machines do you think are affected?"), not a scripted or formal sentence. Internal fields get none.
10. Do not re-add anything listed under REMOVED ON PURPOSE unless I explicitly ask for it.
11. If my request is ambiguous or would need a large redesign, ask me one short question first instead of guessing.

HOW TO REPLY
- If you can edit the file directly (agent / edit mode), make the edits and then summarise them.
- Otherwise reply with numbered edits. For each edit give: the section and function name, a FIND block containing the exact existing code (short but unique), and a REPLACE WITH block, or INSERT AFTER for new code. Give whole functions when a function changes substantially. Never output the whole file.
- Finish with: a short list of what changed, any data or export impact, and 3 to 6 test steps I can follow in the browser.

Reply "Ready for changes" and wait for my request.
```

***

## PROMPT B: Change request (template)

Copy, fill in, paste. Delete lines you do not need.

```
CHANGE REQUEST

What I want:
<describe the change in one or two sentences>

Where it appears:
<screen or tab, e.g. Questionnaire > Containment actions taken, New incident dialog, home table, IOCs tab, Hours tab, report, Markdown export>

Details and behaviour:
- <field names, labels, options, defaults, validation, sort order, colours>
- <spoken ASK line for any new question, or "write one in the same style">
- <what happens on save / edit / delete>
- <keyboard or mobile considerations if any>

Should it appear in exports?
<JSON (always), Markdown yes/no, report yes/no, CSV yes/no, search yes/no>

Done when:
- <test 1>
- <test 2>

Follow the rules from PROMPT A.
```

***

## PROMPT C: Something broke (paste with the error)

```
After applying your last change, something is wrong.

What I did: <steps>
What I expected: <expected>
What happened: <actual>
Console error (F12 > Console), copied exactly:
<paste the red error lines, including file and line numbers>

Find the cause in v2.html and give the smallest fix in the same FIND / REPLACE WITH format. Do not change unrelated code. If you need to see a specific part of the file, tell me which function or section to paste.
```

***

## PROMPT D: Regression check before I keep a change

```
Review the change you just made against the existing behaviour of v2.html. Check each item and say OK or PROBLEM (with a fix):

1. No console errors or [config] warnings on load, on the home screen, in every tab of a case, and in the report.
2. Existing cases still open, and older JSON exports (schema 1, 2 and 3) still import.
3. Autosave still works: typing, then reloading, keeps the data; navigation alone does not change "Last updated".
4. JSON export / import round trip keeps the new data; Merge does not create duplicates.
5. Markdown export, the report and search include the new data where they should; IOCs stay defanged in shared outputs.
6. The home table still shows only Case ID, Incident type, Customer, Severity, Case logged and the buttons; the New incident dialog still has its six fields.
7. Every customer-facing question still shows its ASK line; nothing from REMOVED ON PURPOSE has come back.
8. Both themes look right, including the timeline charts and PNG export.
9. Works at 400px width and with the keyboard only; focus is visible.
10. All user text is escaped with esc(); no alert/confirm/prompt; no em dash characters; no network calls.
```

***

## Ready-made change requests

Paste one of these after PROMPT A, edited to taste.

### 1. Add a question to an existing section

```
CHANGE REQUEST
Add a question to the "Containment actions taken" common section, directly after "Is the threat actor believed to be locked out?":
- id: containment_owner, type: text, label: "Who is leading containment on the client side?"
Then add a conditional question right after it:
- id: containment_blockers, type: textarea, label: "Blockers or approvals needed for containment", shown only if ta_access_removed is "No" or "Unknown".
Add ASK entries for both, e.g. "Who's leading containment on your side?" and "Is anything holding up containment, like approvals?". They must appear in the summary, Markdown and report automatically through the existing questionnaire logic. Do not change any other question.
Follow the rules from PROMPT A.
```

### 2. Remove questions or a whole section

```
CHANGE REQUEST
In the "Business impact" section, remove these questions: "Estimated cost of disruption" (impact_cost) and "Is there media or public attention?" (media_attention).
- Delete them from QUESTIONNAIRE and delete their ASK entries.
- Check no other question uses them in showIf.
- Keep any answers already saved in old cases (do not touch stored data).
They must disappear from the questionnaire, summary, report, Markdown and search. Nothing else changes.
Follow the rules from PROMPT A.
```

### 3. Reword spoken prompts

```
CHANGE REQUEST
Change these ASK lines only (same natural, spoken British English style):
- detection_source: "How did you first get wind of it? An alert, or did someone notice something?"
- affected_systems_count: "Ballpark, how many machines are we talking about?"
Do not change question labels or any other prompt.
Follow the rules from PROMPT A.
```

### 4. Add a new incident type

```
CHANGE REQUEST
Add a new incident type "Intune / device management abuse" with id "intune", placed before "Other" in INCIDENT_TYPES.
Add QUESTIONNAIRE.typeSections.intune with two sections, question ids prefixed "in_":
   a) in_estate "Intune estate & admin access": number of managed devices (number), platforms managed (multiselect: Windows, macOS, iOS / iPadOS, Android), Intune admin accounts compromised (yesno), roles involved (multiselect: Intune Administrator, Global Administrator, Custom RBAC role, Unknown), multi-admin approval enabled (yesno), Intune audit logs exported (yesno).
   b) in_activity "Device management activity": scripts or remediations pushed (yesno), details (textarea, if yes), apps deployed (yesno), device wipes or retires (yesno), number of devices affected (number), compliance or Conditional Access policies changed (yesno), devices enrolled by the attacker (yesno).
Add an ASK entry for every new question in natural, spoken British English.
Existing types and their answers must be unaffected.
Follow the rules from PROMPT A.
```

### 5. Add an option to the New incident dialog lists

```
CHANGE REQUEST
Add "SevD" to SEVERITIES (after SevC), and add "Channel Islands" to COUNTRIES in alphabetical order.
Both the New incident dialog and the Case summary questions read from these lists, so only the config changes. Existing cases are unaffected.
Follow the rules from PROMPT A.
```

### 6. Show one more column on the home table

```
CHANGE REQUEST
Add a "Phase" column to the home table after "Case logged", showing the phase as the existing coloured badge (phaseBadge). Keep all other columns and the buttons as they are, and keep the buttons on one line on wide screens.
Follow the rules from PROMPT A.
```

### 7. Add a field to an existing register

```
CHANGE REQUEST
In the Assets register, add a "Contained on" date field (id containedOn, type date, optional) after Status.
- When an asset's status is changed to Contained and Contained on is empty, set it to today (incident timezone).
- Show it as a column "Contained on" after Status, formatted like the other dates ("07 Oct 2026").
- Include it in Markdown, the report and search through the existing column/field logic, and make sure import keeps it.
Follow the rules from PROMPT A.
```

### 8. Add a new register: Evidence and chain of custody

```
CHANGE REQUEST
Add a new register "Evidence" (key "evidence") using the existing REGISTERS engine, placed between Attack timeline and IOCs in REGISTER_ORDER.
Fields:
- itemId (text, required, label "Evidence ID", placeholder "e.g. EV-001")
- description (text, required, wide, label "Description", placeholder "e.g. Memory image of DC01")
- evType (select, required, label "Type": Memory image, Disk image, Triage collection, Logs, Email / mailbox export, Cloud audit export, Malware sample, Screenshot / document, Other)
- source (text, label "Source host / system")
- collectedBy (text, label "Collected by")
- collectedOn (date, label "Collected on", default today)
- hash (text, label "SHA256", validated as 64 hex characters when not empty, stored lowercase, monospace column)
- storage (text, label "Storage location")
- custody (textarea, wide, label "Chain of custody / transfers", placeholder "One line per hand-over: date, from, to, reason")
- notes (textarea, wide)
Rules: dedupeKey on lowercase itemId (block duplicates). Sort by itemId. Columns: Evidence ID (mono), Description, Type, Source, Collected on ("07 Oct 2026"), SHA256 (mono), Storage, Custody (pre).
Panel: stat tiles for total items and count per type (non-zero only).
Tab badge: item count. Toolbar: "CSV export" with columns evidence_id, description, type, source, collected_by, collected_on, sha256, storage, custody, notes (formula-guard free-text cells using csvCell).
Also: add "evidence" to newRecord / normalizeRecord defaults, import sanitising and merge (by id), Markdown, report (its own section) and search. Old JSON files without "evidence" must still import. Bump SCHEMA_VERSION to 4 and accept older versions.
Follow the rules from PROMPT A.
```

### 9. Add an IOC type

```
CHANGE REQUEST
Add an IOC type "JA3 / JA4 fingerprint" (value "ja3") and "Registry key" (value "registry") to IOC_TYPES.
- detectIocType: do not auto-detect these (user picks them manually), but make sure a 32-character hex value still defaults to MD5.
- defang: leave both unchanged.
- No extra validation for registry; for ja3 accept 32 hex characters or a JA4-style string with underscores.
Existing IOCs must be unaffected.
Follow the rules from PROMPT A.
```

### 10. Add ATT&CK techniques to the suggestions

```
CHANGE REQUEST
Add these entries to ATTACK_TECHNIQUES (format [id, name, tactic id]); do not change anything else:
- T1556.006 Multi-Factor Authentication (Credential Access, TA0006)
- T1484.002 Trust Modification (Defense Evasion, TA0005)
- T1537 Transfer Data to Cloud Account (Exfiltration, TA0010)
Keep the list grouped by tactic in the existing order.
Follow the rules from PROMPT A.
```

### 11. Change the look

```
CHANGE REQUEST
Change the accent colour from blue to teal in both themes:
- Dark: accent #2dd4bf, accent-strong #5eead4, accent-contrast #042f2e, focus #5eead4.
- Light: accent #0f766e, accent-strong #115e59, accent-contrast #ffffff, focus #0f766e.
Also change the brand mark and progress bar gradient end colour from #8a5cf6 to #22d3ee.
Only change theme tokens and those two gradients; do not hard-code colours anywhere else. The report and PNG export stay light as they are.
Follow the rules from PROMPT A.
```

### 12. Adjust settings

```
CHANGE REQUEST
Change BACKUP_WARN_DAYS from 3 to 2, and DEFAULT_BUDGET_WARN_PCT from 80 to 75. Update any user-facing text that mentions these numbers so it reads from the constants instead of hard-coded values.
Follow the rules from PROMPT A.
```
