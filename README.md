xx
Build a single-page HTML technical wiki: "APIM 'User 1' Attack — Investigation Wiki".
Audience: security responders new to Azure API Management (APIM), ARM and SAS tokens.

DESIGN RULES
- White background, flat technical-documentation style (like Stripe or Microsoft Learn docs).
- Neutral greys, ONE muted accent colour. Red only for attacker/danger, green only for safe/fix.
- Boxes: 1px hairline borders, small radius, no shadows, no gradients, no glow, no emoji.
- Small monochrome line icons (Lucide-style inline SVG) next to node titles.
- Sticky left table of contents, numbered sections, smooth scroll.
- Top banner: "Work in progress – updated as findings are confirmed". Mark unconfirmed points with a small "verify" tag.

MOTION
- Subtle and purposeful only: diagram nodes fade in step by step as you scroll into view (IntersectionObserver).
- On flow diagrams, a small dot travels along the connector to show the request direction. Play once per scroll-in, don't loop forever.
- Respect prefers-reduced-motion. No bouncing, pulsing or glowing.

DIAGRAM TYPES
- Flow: left-to-right boxes joined by thin arrows (stack vertically on mobile).
- Sequence: actors across the top (Client, Entra, ARM, APIM instance, Direct Mgmt API) with numbered arrows between them.
- Comparison: two side-by-side columns, or a table.
- Decision tree: a question leading to branches.

SECTIONS

2. How APIM works – control plane vs data plane, using a restaurant analogy (Entra = security desk, ARM = manager's office, gateway = dining room, direct management API = unguarded back office). Diagram: three doors into APIM (ARM, Direct Mgmt API, Gateway).
3. The two tokens – Token 1 is an Entra JWT for management.azure.com. Token 2 is an APIM SAS token for user 1, signed with the instance's primary/secondary key. Sequence diagram of the two client calls, with the internal signing hop marked "hidden from client". Show the stolen-key shortcut: build the SAS offline, no Entra, no logs.
4. The attack & why rotation failed – Path A (identity via ARM: legacy GET or users/1/token), Path B (key leaked from pipeline/Terraform/laptop), what user 1 can read (named values, SP secrets, IdP secrets, backend creds, subscription keys). Explain that rotation kills tokens, not access. Add an endpoint/URL reference table.
5. ARM deep dive – what ARM is, how it works with Entra, request pipeline (authenticate, RBAC, policy, provider, Activity Log), and the fact that GET reads aren't logged.
6. User 1 vs managed identity – inward (manage APIM) vs outward (APIM calls Key Vault/Azure OpenAI). Two-direction diagram. Caveat: managed identity removes stored secrets but user 1 can still edit policies.
7. Tiers: classic vs v2 – which features exist where (direct mgmt API, Git endpoint, VNet options). Decision tree on why the tier matters. Timeline. Per-instance fill-in table.
8. Logging & blind spots – flow coloured by logged by default / must enable / blind spot. Table of log sources with retention. Starter KQL in code blocks.
9. Hardening & remediation – ordered steps: cut foothold, enforce min API version 2021-08-01, disable the direct mgmt API, rotate both keys, rotate stolen secrets. Six layers: identity, management, secrets, network, API security, logging. Show what internal VNet stops and what it doesn't (ARM stays public). Four-instance checklist.
10. Investigation playbook – probing questions tagged Customer/Microsoft/Self-check (pipelines, key locations, identity, APIM settings, logging), five hypotheses table, step-by-step log checklist (Entra interactive, non-interactive, SP/MI sign-ins, audit logs, Activity Log, timeline).
11. Key takeaways & open questions – three one-line takeaways, verify list, sources (Binary Security 2024 parts 1 and 2, Microsoft API version retirement docs).

Output a single self-contained HTML file with no external dependencies.
