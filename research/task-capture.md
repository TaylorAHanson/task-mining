# Task capture options, privacy norms and landing path

Research for issue #4. Public sources only. Accessed 2026-10-28 unless a date is given. General orientation, not legal advice.

Scope note: the repo is public, so everything here is generic. Vendor product behaviour is described from vendor docs. Volume figures are back-of-envelope estimates with stated assumptions, not measurements.

## 1. Capture options

Ordered from least to most intrusive in terms of what reaches the data store.

| Option | What it sees | Where it runs | Notes |
|---|---|---|---|
| **A. Browser extension (content script, Manifest V3)** | DOM of pages the user visits: form fields, URLs, page text, click targets | Browser (Chrome, Edge) | Limited to the browser. Chrome Web Store treats content-script data as user data under User Data and Limited Use policies, even when processed locally. Requires in-product disclosure and affirmative consent. [1][2] |
| **B. OS accessibility / UI-automation APIs** | Structured UI tree: app, window, control names, roles, values, focus/selection events | Desktop agent | Windows: Microsoft UI Automation exposes `IsPassword` and blocks `Value` reads on password fields. [3] macOS: `AXUIElement` needs Accessibility trust, which is separate from Screen Recording. [4] Most structured, least pixel data; good for a "what app / which control / what action" event model. |
| **C. Input-event hooks (keyboard/mouse)** | Raw keystrokes and clicks, global | Desktop agent | Highest risk: keystroke capture is the classic covert-monitoring pattern. Treat as out of scope for the default design; if ever needed, restrict to click and shortcut events and never log character content. |
| **D. Screen recording + vision model** | Pixels: everything on screen, including unrelated personal or third-party content | Desktop agent, then server-side or on-device model | Broadest coverage, broadest privacy exposure. macOS requires separate screen-capture authorization. [4] Sample rather than stream; redact before anything leaves the device. |
| **E. Vendor task-mining recorder (UiPath, Microsoft Power Automate, Celonis-class tools)** | Vendor-defined mix of screenshots, text entries, action descriptions | Desktop recorder, then vendor cloud or tenant | Benchmark for what "commercial task mining" captures. See section 2. |
| **F. Ingest existing traces (not new capture)** | Logs already produced by systems (app logs, RPA logs, ticket/workflow events, system tables) | Existing platforms | Not new capture, but it is the lowest-risk source for Discovery and should be tried first. Covered by other tickets. |

**Recommended shape for the thin slice (for the parent to test, not decided here):** option B as the primary signal (structured app/window/control events, aggregated to a Task-level event model), with option A limited to browser tabs if browser coverage is needed, and option D kept off by default and used only in narrowly scoped, consented pilots. Raw keystrokes (C) and continuous pixel capture (D as default) should not be in the design.

## 2. What commercial task-mining tools capture and how they handle PII

### UiPath Task Mining (Automation Cloud, latest docs)
- Project privacy settings: PII masking is on by default and conceals identifiable information in screenshots and step titles. User-name/avatar anonymization is a separate option. Changes apply only to newly recorded data. [5]
- Detected PII categories include names, email, phone, address, financial identifiers, and government IDs; some are country-specific. Detection uses a cloud service, and **masking happens server-side after upload, not in the desktop client**. [6]
- Protection-of-data page covers recording controls, encryption, data-subject notification responsibilities, and deletion requests. [7]
- Raw data export is subject to the same masking settings. [8]
- Implication: vendor masking is post-upload. A design that needs pre-upload redaction must do it on-device or not upload the raw data at all.

### Microsoft Power Automate task mining
- Recordings can include screenshots, text entries and action descriptions. Microsoft explicitly says results should not be assumed anonymous or aggregate-only. Analytics show task durations, applications used, transitions and timing by recording author. [9]
- Redaction is manual: users delete screenshots and replace sensitive text with placeholders before analysis. Pause/resume/delete-step controls exist. [10][11]
- Access is controlled through Dataverse roles; administrators can access all processes in an environment. [12]
- Implication: vendor tooling relies on user-operated recording and manual redaction. Not a model for silent or always-on capture.

### Common pattern across vendors
1. Screenshots and typed text are the sensitive part.
2. Masking is either server-side after upload (UiPath) or manual (Microsoft).
3. Recording is user-initiated, with pause controls.
4. Role-based access to recordings is the main control.

## 3. Legal and organisational norms (orientation only, not legal advice)

### 3.1 EU/EEA: GDPR
- **Lawful basis.** Article 88 GDPR does not supply a standalone basis for monitoring. Employers still need an Article 6 basis, and national employment rules under Article 88 must contain specific safeguards. The CJEU confirmed in *C-65/23 (K GmbH), 19 Dec 2024* that collective agreements do not displace GDPR requirements, and that Article 88 rules must comply with Articles 5, 6(1) and 9. [13][14]
- **Consent.** Usually unsuitable for employees because of the power imbalance. [15]
- **Legitimate interests (6(1)(f))** are possibly available for security-type monitoring, but require necessity and balancing. [15]
- **EDPB Guidelines 3/2019 on video devices** (adopted Jan 2020, corrected Feb 2020) give the reasonable-expectation and less-intrusive-alternatives tests that are reused in employment analysis. Note: this search did not find a 2025 replacement. [16]
- **DPIA (Article 35).** Required for high-risk processing; employee monitoring of this kind would typically trigger it. Treat a DPIA as the default first deliverable.

### 3.2 Works councils and co-determination
- **Germany, BetrVG §87(1) Nr. 6.** Co-determination applies when a technical device is *objectively suitable* to monitor behaviour or performance. Actual use or monitoring intent is not required. Live transmission without storage can also trigger it. [17]
- Federal Labour Court, 1 ABR 16/23 (16 Jul 2024): a headset system triggered co-determination because supervisors could listen in, even without recording. [17]
- Practical consequence: any desktop agent deployed to German employees will almost certainly need a works agreement (Betriebsvereinbarung) before rollout. The agreement should fix purposes, data fields, permitted evaluations, access rights, retention, limits on rankings or profiling, and the procedure for feature extensions. [17]
- Other EU states have analogous consultation rules (for example France's works council consultation); not researched in depth here.

### 3.3 Enforcement signal
- **CNIL, Amazon France Logistique (decision of 27 Dec 2023, announced 23 Jan 2024).** Fined €32M for intrusive scanner-based productivity indicators and over-long retention of detailed data (31 days for all staff). Data minimisation and aggregation were the central findings. [18]
- On appeal, the Conseil d'État (23 Dec 2025, decision 492830) rejected the Article 6 findings on the three scanner indicators and reduced the fine to €15M. The lesson on retention and granularity still stands as a design signal; the Article 6 point was reversed. [18]

### 3.4 EU AI Act (Regulation (EU) 2024/1689)
- **Article 5(1)(f) prohibition.** Using AI to infer emotions in the workplace is prohibited, except for medical or safety reasons. Notice to workers does not cure a prohibited use. Example in Commission guidance: webcam or voice analysis to infer call-centre employees' anger. [19]
- **Annex III, point 4(b).** High-risk category covers AI intended to "monitor and evaluate the performance and behaviour of persons" in employment. Commission draft guidance reads "monitor" and "evaluate" as alternatives. Purely administrative or operational monitoring needs a case-by-case check. [20]
- **Article 26(7).** Before putting a high-risk system into use at work, deployers must inform workers' representatives and affected workers. [19]
- **Timing.** The AI Omnibus (in force 27 Jul 2026 per the Commission summary) moves Annex III high-risk obligations, including Article 26, to **2 Dec 2027**. Article 5 prohibitions and Article 4 AI literacy are not postponed in the same way. Verify against the Official Journal before relying on dates. [21]
- Article 4 (AI literacy) has applied since 2 Feb 2025, amended July 2026, and covers deployers generally. [21]

### 3.5 UK
- ICO, "Employment practices and data protection: monitoring workers" (under review after the Data (Use and Access) Act). Monitoring that is likely high-risk, including keystroke monitoring, needs a DPIA. Workers must be told before monitoring starts, in accessible terms covering nature, extent and purpose. Covert monitoring is exceptional and needs senior authorisation, targeted and time-limited. Notification alone does not make monitoring lawful. [22]

### 3.6 United States (high level)
- **New York Civil Rights Law §52-c** (effective 7 May 2022). Covered employers with a New York place of business must give prior notice at hire, obtain written or electronic acknowledgement, and post a notice. Applies to telephone, email and internet access or usage. Narrow exception for system maintenance or protection that does not target an individual. Penalties from $500 up to $3,000 per violation, enforced by the Attorney General. [23]
- Other states (for example Connecticut, Delaware) have electronic-monitoring notice rules; not researched here.
- Federal law is sector-specific and light on employee monitoring; state privacy laws such as CCPA/CPRA cover employee data in California. Not researched in depth.

## 4. Privacy constraints any design must meet

Derived from the sources above. Each is a requirement to test a design against, not legal advice.

1. **Purpose limitation and a documented lawful basis per jurisdiction.** Capture must be scoped to a stated purpose (for example, Process Discovery for a named process), not general productivity monitoring. Covers the EU/UK requirement and the CNIL and AI Act findings.
2. **No emotion, sentiment or wellbeing inference on people.** Article 5(1)(f). Ban it in the design, not only in policy.
3. **Works-council or equivalent consultation before deployment (EU, esp. Germany).** Design the agent so the works agreement can specify fields, evaluations and retention. The agent must not be able to silently gain new fields.
4. **Transparency before capture.** Tell workers before monitoring starts, covering nature, extent, purpose, recipients and retention. UK/EU. In-product disclosure and affirmative consent for browser extensions (Chrome policy).
5. **Data minimisation by design.**
   - Default to structured events (app, window, control role, action type) rather than pixels or text.
   - No keystroke content. No password or value reads (honour `IsPassword`/equivalent).
   - Aggregate to Task-level event models on the device before upload where possible.
6. **On-device redaction before upload.** Vendor post-upload masking (UiPath) is weaker than pre-upload. Redact or drop screenshots and typed text on the device. Screenshots, if any, must not leave the device unredacted.
7. **Pause, scope and delete controls for the user.** Microsoft's model: pause, delete a step, delete a screenshot. Also needed for data-subject access and deletion requests.
8. **Short retention for raw data.** CNIL's 31-day finding is a signal. Keep raw events short (days), keep aggregated Task/Case evidence longer, and set this in the works agreement.
9. **Role-based access.** Only the Discovery team sees identifiable events; process owners see aggregates. Individual performance use must be excluded. Connects to the Unity Catalog grant model.
10. **No individual ranking or performance scoring from capture data.** Avoids Annex III point 4(b) "evaluate" and BetrVG profiling concerns. Discovery should operate on process-level evidence.
11. **Notice and acknowledgement mechanics (NY).** Hiring notice, acknowledgement and posting if NY employees are in scope.
12. **DPIA before pilot.** Also a works-agreement draft before any employee machine is touched.
13. **Public-repo hygiene.** Not a legal constraint, but the design docs must use synthetic data only.

## 5. Landing path into Databricks

### 5.1 Zerobus Ingest (public preview status not confirmed on the pages read)
- Push-based streaming API that writes directly into Unity Catalog managed Delta tables; no message bus to run. [24]
- Auth: service principal by default (client ID/secret plus the "Workspace access" entitlement). Entra ID token exchange is supported via SDK; REST requires you to run the exchange yourself. [25]
- Required privileges: `USE CATALOG`, `USE SCHEMA`, and `MODIFY, SELECT` on the table. [25]
- Formats: **JSON is the only format over REST** (the page says so explicitly). Protobuf is supported by SDKs and recommended for production. Arrow via Arrow Flight is a separate guide. [25]
- REST shape: `POST $ZEROBUS_ENDPOINT/zerobus/v1/tables/<catalog>.<schema>.<table>/insert`, JSON array body, OAuth bearer token refreshed hourly. [25]
- Schema: Zerobus never auto-evolves table schemas. Mismatched records are rejected by default unless a rescue column is configured. [25]
- **Quotas (Zerobus quotas page):** 100 MB/s and 100,000 records/s per gRPC stream (benchmarked at 1 KB messages, adjustable); 10 GB/s per table; 10,000 REST requests/s; unlimited concurrent streams per workspace; max record 10 MB; 42 nesting levels; buffered data retained 28 days if unflushed. Raise via account representative. [26]
- Not stated on the pages read: batch-size limits for REST, streams per table, GA status, region list (separate page). Confirm before committing to a design.

### 5.2 Auto Loader (from object storage)
- Incremental file ingestion from cloud object storage into Delta. Schema inference and evolution via `cloudFiles.schemaEvolutionMode` (`addNewColumns`, `addNewColumnsWithTypeWidening` [Public Preview, Runtime 16.4+], `rescue`, `failOnNewColumns`, `none`). `cloudFiles.schemaLocation` stores inferred schema. [27]
- Best fit when the desktop agent writes batched files (for example, a JSON-lines file per user per hour) to object storage, a volume, or a landing zone. Simple, durable, no always-on service. Latency is batch-level (minutes), which is fine for Discovery.

### 5.3 Databricks Apps REST endpoint (custom)
- A Databricks App can host a FastAPI route (for example `/api/ingest`) that writes to Unity Catalog through a SQL warehouse or the SQL Statement Execution API (`POST /api/2.0/sql/statements`). [28]
- Token-authenticated API routes must use `/api/` paths, and callers need `CAN USE` on the app. [28]
- This is a route you implement, not a built-in ingest service. It adds app-level throughput limits and a SQL warehouse dependency. Use only if Zerobus or Auto Loader do not fit (for example, a need for custom validation before write).

### 5.4 Comparison

| Path | Latency | Operational weight | Fit |
|---|---|---|---|
| Zerobus (JSON or protobuf) | Seconds | Low (no bus, serverless) | Continuous event stream from desktop/browser agents |
| Auto Loader from object storage | Minutes (batch) | Low; agent writes files | Batched uploads; simplest durable path; works with air-gapped or throttled endpoints |
| Databricks App REST → UC | Seconds | Medium (app + warehouse) | Only if custom validation is needed |

**Recommendation to test in the thin slice:** agent batches structured events on device, uploads to Zerobus (JSON) in small batches or uses Auto Loader files as fallback. Screenshots, if ever used, go to a volume, not a Delta row.

### 5.5 Volume estimates (per user-day, assumptions stated)

These are estimates to size the design, not measurements. Assumptions: 8 active hours per user-day; event sizes around 300 bytes JSON for a structured event; screenshot size around 200 KB compressed.

| Capture tier | Assumption | Records / user-day | Bytes / user-day | Notes |
|---|---|---|---|---|
| **Structured UI events (B), aggregated** | 1 event per 2–20 s of active use, plus focus changes | ~1,500–15,000 | ~0.5–5 MB | Dominant volume is record count, not bytes |
| **Browser extension events (A)** | Tab/navigation events, no page content | ~100–1,000 | ~0.03–0.3 MB | Small |
| **Keystroke / text content (C)** | Not in design | n/a | n/a | Excluded |
| **Screenshots (D), 1 per 5 s** | ~5,760 frames per 8 h | ~5,760 objects | ~1.1 GB | Dominant storage cost; argues against pixels by default |
| **Screenshots, 1 per 60 s (sampled)** | ~480 frames per 8 h | ~480 objects | ~0.1 GB | Still heavy for a Discovery signal |

Fleet scaling (structured tier only):

| Users | Records / day (mid estimate, 5,000/user-day) | Average rate over an 8 h window | Peak (assume 10x average) | Zerobus fit |
|---|---|---|---|---|
| 100 | 0.5 M | ~17 rec/s | ~170 rec/s | Trivial vs 100k rec/s per stream |
| 1,000 | 5 M | ~174 rec/s | ~1.7k rec/s | One stream is ample; use 2 for redundancy |
| 10,000 | 50 M | ~1.7k rec/s | ~17k rec/s | Well under one stream's 100k rec/s; 2 streams for headroom |

Storage: at 5,000 records/user-day and 300 bytes each, about 1.5 MB per user-day, about 375 MB per user-year (250 working days). For 10,000 users, about 3.75 TB/year before compression. Delta compression typically reduces this substantially; treat the figure as an upper bound.

Screenshot storage at 1 fps-equivalent every 5 s is about 1.1 GB per user-day, so 1,000 users is about 1.1 TB/day. This is the reason to keep pixels off by default.

## 6. Viable capture designs (shortlist)

Each is a candidate for the Task capture agent design ticket. Privacy constraints in section 4 apply to all.

1. **Structured accessibility-event agent (Windows UIA / macOS AX).** App, window, control role, action type, timing, focus transitions. No values, no text, no pixels. Batches to Zerobus or Auto Loader. Strongest privacy profile; main gap is apps with weak accessibility trees (thin client, Citrix, some Electron/canvas apps).
2. **Browser-extension event agent.** Same event model for browser tasks, with domain-level (not URL-path or content) capture by default. Chrome Web Store disclosure and consent apply.
3. **Consented, time-boxed pixel pilot.** Screenshots at low frequency, redacted on device before upload, stored in a volume with short retention. Only for named processes, only for volunteers or with works agreement; used to validate structured-event models, not as ongoing capture.
4. **Process-owner-initiated recording (vendor-style).** The user starts and stops a recording for a known task, as in Microsoft's model. Low volume, high user control, easy to explain. Slower than passive capture but the most consentable option.

Designs explicitly not recommended: keystroke logging, always-on screen streaming, server-side-only masking of raw screenshots, any emotion or productivity scoring of individuals.

## 7. Surprises and follow-ups for the parent session

- **Zerobus REST is JSON-only.** A protobuf-only assumption in the event model would break the REST path. Event schema must be JSON-compatible.
- **Zerobus quotas are generous** (100k records/s per stream), so volume is not the constraint for structured events. Pixels are.
- **Zerobus has no stated GA status or regions on the pages read.** Needs confirmation before a design depends on it.
- **Annex III high-risk date moved to 2 Dec 2027** under the AI Omnibus. This relaxes the timeline for the Article 26 employer duties but not the Article 5 prohibition. Verify against the Official Journal; this research used the Commission summary and AI Act Service Desk pages, not the EUR-Lex text (EUR-Lex fetch returned empty).
- **German co-determination is triggered by objective suitability.** Any desktop agent for German employees needs a works agreement before pilot, even if it is not switched on for monitoring.
- **Possible new tickets:**
  - Consent and works-agreement template (feeds the privacy posture fog item).
  - Event model on-device schema (structured events only) to size against the 5,000-records-per-user-day assumption.
  - Pilot design with named process and volunteer population (synthetic data only in the repo).
- **Fog item "Task capture agent design & rollout" can graduate once:** (a) one capture option is chosen (probably design 1, with 2 optional), (b) the event schema is drafted, and (c) a DPIA-scoped pilot boundary is agreed. The privacy constraints in section 4 are sufficient as acceptance criteria for the design ticket.
- **Tickets #5–#10 check:** not reviewed (not in scope for this ticket). Parent to check whether any assume pixel capture, keystroke data, or per-individual performance outputs.

## 8. Sources

[1] Chrome for Developers, Manifest V3 requirements (Web Store program policies), https://developer.chrome.com/docs/webstore/program-policies/mv3-requirements (accessed 2026-10-28)
[2] Chrome for Developers, Limited Use policy and User Data FAQ, https://developer.chrome.com/docs/webstore/program-policies/limited-use and https://developer.chrome.com/docs/webstore/program-policies/user-data-faq (accessed 2026-10-28)
[3] Microsoft Learn, UI Automation overview and Edit control requirements, https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-uiautomationoverview ; https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-supporteditcontroltype ; https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-automation-element-propids (accessed 2026-10-28)
[4] Apple Developer, AXUIElement / AXIsProcessTrustedWithOptions / CGPreflightScreenCaptureAccess; Apple Platform Deployment, Privacy Preferences Policy Control payload, https://developer.apple.com/documentation/applicationservices/axuielement_h ; https://developer.apple.com/documentation/coregraphics/cgpreflightscreencaptureaccess%28%29 ; https://support.apple.com/guide/deployment/privacy-preferences-policy-control-payload-dep38df53c2a/web (accessed 2026-10-28)
[5] UiPath docs, Task Mining, Editing a project, https://docs.uipath.com/task-mining/automation-cloud/latest/user-guide/editing-a-project (accessed 2026-10-28)
[6] UiPath docs, Information about the collected data, https://docs.uipath.com/task-mining/automation-cloud/latest/user-guide/information-about-the-collected-data (accessed 2026-10-28)
[7] UiPath docs, Protection of personal data and privacy of the users; Security, https://docs.uipath.com/task-mining/automation-cloud/latest/user-guide/protection-of-personal-data-and-privacy-of-the-users ; https://docs.uipath.com/task-mining/automation-cloud/latest/user-guide/security (accessed 2026-10-28)
[8] UiPath docs, Raw data export, https://docs.uipath.com/task-mining/automation-cloud/latest/user-guide/raw-data-export (accessed 2026-10-28)
[9] Microsoft Learn, Power Automate task mining overview, https://learn.microsoft.com/en-us/power-automate/task-mining-overview (accessed 2026-10-28)
[10] Microsoft Learn, Protect your data (task mining), https://learn.microsoft.com/en-us/power-automate/process-advisor-protect (accessed 2026-10-28)
[11] Microsoft Learn, Prepare processes and recordings, https://learn.microsoft.com/en-us/power-automate/process-advisor-processes (accessed 2026-10-28)
[12] Microsoft Learn, Security and privacy (Process Advisor), https://learn.microsoft.com/en-us/power-automate/process-advisor-security (accessed 2026-10-28)
[13] CJEU, Case C-65/23 (K GmbH), 19 Dec 2024, https://eur-lex.europa.eu/legal-content/EN/ALL/?uri=celex%3A62023CJ0065 (summarised via search; accessed 2026-10-28)
[14] CJEU, Case C-34/21, 30 Mar 2023, https://curia.europa.eu (summarised via search; accessed 2026-10-28)
[15] EDPB Guidelines 3/2019 on processing of personal data through video devices (adopted 29 Jan 2020, corrected 26 Feb 2020), https://www.edpb.europa.eu/sites/default/files/files/file1/edpb_guidelines_201903_video_devices_en_0.pdf (accessed 2026-10-28)
[16] As [15]; no 2025 replacement located.
[17] Bundesarbeitsgericht, 1 ABR 16/23 (16 Jul 2024), https://www.bundesarbeitsgericht.de/entscheidung/1-abr-16-23/ ; BetrVG §87, https://www.gesetze-im-internet.de/betrvg/__87.html (accessed 2026-10-28)
[18] CNIL decision SAN-2023-021 (Amazon France Logistique), announced 23 Jan 2024; Conseil d'État decision 492830 (23 Dec 2025), https://www.legifrance.gouv.fr/ceta/id/CETATEXT000053163189 (accessed 2026-10-28)
[19] European Commission AI Act Service Desk, Article 5 and Article 26, https://ai-act-service-desk.ec.europa.eu/en/ai-act/article-5 ; https://ai-act-service-desk.ec.europa.eu/en/ai-act/article-26 ; Commission Guidelines on prohibited AI practices (2025), https://ai-act-service-desk.ec.europa.eu/sites/default/files/2025-08/guidelines_on_prohibited_artificial_intelligence_practices_established_by_regulation_eu_20241689_ai_act_english_ied3r5nwo50xggpcfmwckm3nuc_112367-1.PDF (accessed 2026-10-28)
[20] Regulation (EU) 2024/1689, Annex III point 4(b) and Recital 57, https://eur-lex.europa.eu/legal-content/EN/TXT/PDF/?uri=CONSIL%3AST_7536_2024_INIT ; Commission AI Act Service Desk, Employment, https://ai-act-service-desk.ec.europa.eu/en/employment-0 (accessed 2026-10-28; EUR-Lex text not fetched directly)
[21] European Commission, AI Omnibus / regulatory framework for AI, https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai ; AI Act Service Desk, Article 4 and Article 113 (accessed 2026-10-28; the Commission news page returned HTTP 403 and was not read directly)
[22] UK ICO, Employment practices and data protection: monitoring workers, https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/employment/monitoring-workers/ (accessed 2026-10-28)
[23] New York Civil Rights Law §52-C, https://www.nysenate.gov/legislation/laws/CVR/52-C%2A2 (accessed 2026-10-28)
[24] Databricks docs, Zerobus Ingest overview, https://docs.databricks.com/aws/en/ingestion/zerobus-overview (accessed 2026-10-28)
[25] Databricks docs, Use Zerobus Ingest, https://docs.databricks.com/aws/en/ingestion/zerobus-ingest/ (accessed 2026-10-28)
[26] Databricks docs, Zerobus Ingest quotas, https://docs.databricks.com/aws/en/ingestion/zerobus-quotas (accessed 2026-10-28)
[27] Databricks docs, Auto Loader schema inference and evolution, https://docs.databricks.com/aws/en/ingestion/cloud-object-storage/auto-loader/schema (accessed 2026-10-28)
[28] Databricks docs, Databricks Apps: connect via token auth, authorization, SQL warehouse resource; Statement Execution API, https://docs.databricks.com/aws/en/dev-tools/databricks-apps/connect-local ; https://docs.databricks.com/aws/en/dev-tools/databricks-apps/auth ; https://docs.databricks.com/api/statement-execution/v1/execute-statement (accessed 2026-10-28)
