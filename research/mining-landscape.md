# Process mining, task mining and agentic automation: landscape and where a lakehouse-native approach differs

## Question

"What does the process-mining, task-mining and agentic-automation landscape look like, and where would a lakehouse-native, Databricks-native approach actually differ?"

Date of research: 2026-10-28

Method note: public sources only (vendor docs and sites, licence files, standards pages, arXiv). No internal sources were used. Where a claim was only visible in a search-result snippet and I could not fetch the page, it is marked "(via search)". Where I could only find it secondhand, it is marked "(secondary)". Access date for every source is 2026-10-28.

---

## Summary

- **Celonis and Databricks now have a public, Delta Sharing-based integration.** Announced 2025-11-04. Celonis reads live Databricks data, enriches it in its Process Intelligence Graph, and feeds PI Graph context into Agent Bricks. The release describes data sharing, not native execution of Celonis engines inside Databricks [1][27].
- **UiPath has narrowed Task Mining in cloud.** Unassisted (automatic discovery) Task Mining was removed from Automation Cloud on 2025-12-01. Current Task Mining is known-task capture and analysis, and UiPath states analysts, not the AI, assess and prioritise automation candidates [2][3].
- **The commercial field has converged on "observe, discover, then generate or orchestrate agents".** Skan, Soroco, KYP.ai and Mimica all now market agent generation or agent-ready blueprints alongside desktop discovery [12][13][14][15]. ServiceNow ties Process Mining to Task Mining projects with traceability [8][9]. UiPath pairs Process Mining with Maestro orchestration [5].
- **Mining-to-automation loops are mostly vendor-internal.** The vendors above close the loop inside their own suites. Power Automate Process Mining and SAP Signavio document event-log ingestion (CSV, XES, connectors), not a documented export into a lakehouse agent runtime [7][10].
- **PM4Py is AGPL-3.0.** The repo offers a commercial licence as an alternative [16][17]. Modifying it and offering it over a network, or distributing it with a product, triggers AGPL obligations (Section 13 and Section 5(c)); see the licensing section. The permissive option (bupaR, MIT) is R-only and less scalable [20]. ProM is GPL core [21]; Apromore Core is LGPL-3.0-or-later [19].
- **Standards: XES is an IEEE standard (IEEE 1849-2023, paywalled) [25]. OCEL 2.0 is the object-centric successor with JSON, XML and SQLite exchange formats, and it can express object changes and object-to-object relations that XES cannot [22][24].**
- **LLM/agent research is active in 2025-2026, but mostly as tool-assisted analysis or agent-spec generation, not end-to-end automation from logs.** Notable: PMAx (arXiv:2603.15351, 2026-03-16), which keeps computation in classical algorithms and uses LLMs for interpretation [35]; and LLM-guided discovery with expert rules (arXiv:2510.07161, 2025-10-08) [28].

---

## Commercial landscape

Legend: "Closes loop?" = whether the vendor's public docs describe going from mined insight to an automation or agent. "Not verified" means I could not confirm it from a public primary source in this run.

| Vendor / product | What it discovers | Data sources (documented) | Closes the loop into automation? | Sources |
|---|---|---|---|---|
| **Celonis** Process Intelligence Platform | Process models and KPIs from system event data; PI Graph as a "digital twin" of operations | Live data via Databricks Delta Sharing (bidirectional) per the 2025-11-04 release; Celonis connectors generally (not enumerated in this run) | Partial: PI Graph context is fed to Databricks Agent Bricks, and agent outputs feed back into the PI Graph. Celonis's own action/orchestration layer: not verified from public docs in this run | [1][27] |
| **UiPath Process Mining** | End-to-end process flows from ERP/CRM and other system event logs | Business-system event logs (ERP, CRM); CSV via coding-agent preview | Yes, by product pairing: Process Mining hands off to Task Mining investigation, and Maestro orchestrates agents, robots, APIs and humans. Process Insights GA on 2026-10-13 flags anomalies and KPI impact | [4][5][6] |
| **UiPath Task Mining** | Desktop task recordings of a known task; AI clusters and merges traces; exports PDD or a Studio XAML skeleton | Desktop recordings | Partial. Exports a Studio XAML skeleton; analysts assess and prioritise candidates. Unassisted discovery removed from Automation Cloud on 2025-12-01 | [2][3] |
| **UiPath agentic (Maestro, coding agents)** | Not a miner. Orchestrates agents, RPA workflows, API workflows and human tasks. Coding-agent Process Mining preview (2026-09-16) turns CSV/event logs into a published app | Process-mining outputs as input | Orchestration, not discovery. Maestro is a runtime for the automations that mining identifies | [5][6] |
| **Microsoft Power Automate Process Mining** | Process maps, variants, bottlenecks, root cause, KPIs from extracted system event data; "identify opportunities for automation" is a stated benefit | "Extracting event data from your system of records" (Microsoft Learn). Connection specifics on the data-requirements page, not fetched in this run | Not verified from public docs in this run. Microsoft's page lists automation identification as a benefit, not a documented handoff to a flow | [7] |
| **ServiceNow Process Mining + Task Mining** | Process visualisation, bottlenecks, root cause, multidimensional mining. Task Mining collects and analyses workstation activity | ServiceNow platform records (process mining); desktop agent (task mining) | Partial, by documented integration: Task Mining projects can be started from process nodes or improvement opportunities, keeping traceability. Task Mining Core 3.2.2 (March 2026) adds "automation requests" | [8][9] |
| **SAP Signavio Process Intelligence** | Process models and variants from event logs | Connectors for SAP ERP (RFC), S/4HANA CDS views, SuccessFactors, ServiceNow, Jira, AWS Athena/S3, Google BigQuery, Snowflake, SAP Datasphere, HANA, MySQL, PostgreSQL, MongoDB, SQL Server, OData, Ingestion API; CSV/XES upload | Not verified as a documented automation-generation loop in this run. Data integration is the documented path | [10][11] |
| **Skan.ai** | Desktop observation via computer vision across apps, grouped into tasks, workflows, variants and handoffs; covers email, spreadsheets and legacy apps | Desktop software agent (no backend integrations, but agent install required) | Yes, as documented: identifies and prioritises automation and AI opportunities; also markets "Agents" that execute workflows grounded in operational context | [12] |
| **Soroco (Scout)** | Work graph of how work flows across teams and applications; effort and cost analysis | Desktop and application activity (per vendor site) | Yes, per vendor site: agent-ready blueprints and monitoring of human-plus-agent workflows | [13] |
| **KYP.ai** | Continuous process intelligence across people, processes and technology; capacity and ROI-ranked automation opportunities | Task mining / desktop capture (per vendor page) | Yes, per vendor page: agent code generation in current offering. Depth not verified | [14] |
| **Mimica** (Miner and Mapper) | Miner finds repetitive desktop work and ranks automation candidates; Mapper documents steps, decisions, exceptions and variants | Desktop observation | Yes, per vendor about page: markets agent generation and deployment. Depth not verified | [15] |
| Others to note | Not researched in this run (for example Apromore EE, Process.st, Minit, Automation Anywhere, Blue Prism/SS&C) | — | — | — |

Notes for comparison:
- Celonis, UiPath, Microsoft, ServiceNow and SAP are platform vendors whose mining sits next to their own automation or workflow products. Skan, Soroco, KYP.ai and Mimica are desktop-observation specialists, and their automation-generation claims are largely from their own sites (secondary to product behaviour; treat as vendor claims).
- The Celonis-Databricks integration is the only public Databricks-side mining integration found in primary sources. A LinkedIn article and a CloudFronts blog describe custom Databricks-to-process-mining pipelines (secondary) [42][43].

---

## Open source and licensing

### PM4Py

- **What it does:** Python library for process mining. The README documents discovery (for example `discover_petri_net_inductive` from an XES log), an optional `ocel` extra (jsonschema, pyarrow) for object-centric logs, and an optional `polars` extra (Polars 1.40.1 and 2.x supported; joins preserve event order, including with the streaming engine) [16].
- **Scalability:** The README does not describe a Spark or distributed backend. Polars and pyarrow are the documented accelerators [16]. Spark-scale discovery exists only in research prototypes (see below) [40][41].
- **Licence:** GNU Affero General Public License, version 3 (AGPL-3.0), dated 19 November 2007. The repo LICENSE is the FSF AGPL-3.0 text [17]. The README states the open-source version is AGPL-3.0 and that a separate commercial closed-source licence is offered [16].
- **Latest release version:** not visible on the repo page in this run (the Releases section was empty in the cached view) [16]. Check `pyproject.toml` or PyPI before pinning.

**What AGPL-3.0 means for a field asset (factual summary of the licence text; not legal advice):**
- Section 5(c): if you convey a work based on AGPL code, you must license the entire work as a whole under AGPL to anyone who receives a copy [17].
- Section 13: if you modify AGPL code and users interact with it remotely over a network, your modified version must prominently offer those users the Corresponding Source [17].
- Consequence for a field asset: (a) if the asset ships to customers, or is installed in a customer's environment, PM4Py as a dependency likely pulls the asset under AGPL conveyance terms; (b) if the asset runs as a hosted service (a Databricks App, for example) that modifies PM4Py, Section 13 applies; (c) importing PM4Py unmodified into a Databricks notebook used only internally has a different obligation profile. Which applies depends on exact use and must be decided by counsel.
- Avoid assuming "internal use only" removes obligations. Read Section 13 and Section 5 in the full text before deciding.

### Permissively or less-restrictively licensed alternatives (verified from LICENSE or DESCRIPTION files)

| Project | Licence (verified) | Notes | Source |
|---|---|---|---|
| bupaR (R, bupaverse) | MIT + file LICENSE, version 1.0.0 in DESCRIPTION | R-based; event-log class and descriptive statistics, Petri net objects, process maps | [20] |
| ProM 6 framework | GPL core; plug-ins typically LGPL (per promtools.org; version not stated on that page) | Java; GPL core is a copyleft constraint | [21] |
| Apromore Core (open-source edition) | LGPL-3.0-or-later (verified from LICENSE.txt) | Apromore Enterprise Edition is commercial | [19] (EE licence via search, not fetched) |
| Other Python OCEL / OCPM libraries | Not verified in this run | Check each LICENSE before adoption | — |

Notes:
- The bupaR MIT licence is the cleanest permissive option verified in this run, but it is R-only and not designed for Spark-scale data.
- No permissively licensed Spark-scale process-discovery implementation was verified as maintained. Research code (see the Spark papers below) has no verified licence in this run.

---

## Standards

### XES (eXtensible Event Stream)
- **IEEE 1849-2023**, "IEEE Standard for eXtensible Event Stream (XES) for Achieving Interoperability in Event Logs and Event Streams". Published 2023-09-08. Supersedes IEEE 1849-2016. Active standard. The full text is paywalled (purchase or subscription) [25].
- Defines an XML grammar, XML Schemas for XES instances and extensions, and standard extensions. The 2023 edition adds Micro, Software Event, Software Communication, Software Telemetry and Artifact Lifecycle extensions [25][26 (via search)].
- XES is flat: one case per trace, with events in a trace. It cannot natively represent an event that touches several objects of different types.

### OCEL 2.0 (Object-Centric Event Log)
- Current version: **OCEL 2.0**. The standard site presents 2.0 as the current standard and 1.0 as legacy [22]. Specification document dated 2023-10-16 (per the search snippet, the PDF itself was not parseable in this run) [23 (via search)].
- Metamodel: events have a type, timestamp and attribute values; objects have an object type and attributes that can change over time; events link to at least one object through event-to-object relationships; objects can be linked to other objects through object-to-object relationships; both relationship kinds can carry qualifiers [22].
- Formats: relational (SQLite), XML and JSON [24].
- Reference paper: Berti et al., arXiv:2403.01975, submitted 2024-03-04 [24].
- PM4Py exposes OCEL via the `ocel` extra [16].

Key differences in practice:
- OCEL 2.0 represents one event touching many objects (for example, one "approve" event linked to an order, a customer and an approver) without duplicating the event per case. XES needs a case choice and loses the other objects [22][24].
- OCEL 2.0 can record attribute changes on objects over time, which XES's case-attribute model cannot express in the same way [24].

### Event-log requirements in commercial tools (for reference)
- SAP Signavio: each uploaded event row needs a case ID, activity and end timestamp; supported uploads are CSV and XES (optionally zipped) [10][11 (via search; fetch did not confirm)].
- Microsoft Power Automate: event-log requirements are documented on a separate data-requirements page, not verified in this run [7].
- Task mining (UiPath, ServiceNow, Skan, Mimica): proprietary desktop capture formats; no open task-mining event standard was found in this run.

---

## LLM / agent-based discovery and automation research

Dates are submission or publication dates as shown by the source. Verified means the abstract or page was fetched; otherwise "(via search)".

| Paper | arXiv / DOI | Date | Finding |
|---|---|---|---|
| Norouzifar, Kourani, Dees, van der Aalst: *Integrating Domain Knowledge into Process Discovery Using LLMs* | arXiv:2510.07161 | 2025-10-08 | LLM extracts declarative rules from expert descriptions; rules steer the IMr discovery algorithm; implemented tool and real-log case study. Venue not named (under journal review) [28] (verified) |
| Rebmann, Schmidt, Glavaš, van der Aa: *On the Potential of LLMs to Solve Semantics-Aware Process Mining Tasks* | arXiv:2504.21074 | 2025-04-29 | Benchmark of five semantics-aware tasks; fine-tuning substantially beats out-of-the-box prompting [29] (via search) |
| Pyrih, Rebmann, van der Aa: *LLMs that Understand Processes: Instruction-tuning for Semantics-Aware Process Mining* | arXiv:2508.16270 | 2025-08-22 | Instruction-tuning generalises to unseen tasks including discovery; considerable gains on discovery and prediction; mixed on anomaly detection [30] (via search) |
| *IoT Miner: Intelligent Extraction of Event Logs from Sensor Data for Process Mining* | arXiv:2509.05769 | 2025-09-06 | Clustering plus LLM-generated activity labels to build event logs from sensor data (log construction, not model discovery) [31] (via search) |
| Redis, Fani Sani, Zarrin, Burattin: *Skill Learning Using Process Mining for LLM Plan Generation* | arXiv:2410.12870 | 2024-10-14 (proceedings 2025-03-28) | Discovers reusable skills from action sequences, stores them as process models, retrieves via conformance checking, uses LLMs for plan generation [32] (via search) |
| *ProAgent: From Robotic Process Automation to Agentic Process Automation* (authors not verified in this run) | arXiv:2311.10751 | 2023-11 | LLM agents build and execute workflows from human instructions, not from mined logs [33] (via search) |
| *Automating the Enterprise with Foundation Models* (ECLAIR; authors not verified in this run) | arXiv:2405.03710 | 2024-05 | Foundation-model enterprise workflow automation; discusses limits of process mining plus traditional RPA [34] (via search) |
| Antonov, Kourani, Berti, Park, van der Aalst: *PMAx: An Agentic Framework for AI-Driven Process Mining* | arXiv:2603.15351 | 2026-03-16 | Two agents: Engineer generates scripts that run classical mining algorithms locally; Analyst interprets outputs. No empirical evaluation in the abstract; privacy and accuracy are design claims [35] (verified) |
| Yang et al.: *From Event Logs to Governed Action: A BlueSky Agenda for Agentic Process Mining* | arXiv:2609.07984 | 2026-09-07 | Research agenda: event-to-action evidence for authorising or refusing agent actions; proposes governance contracts and benchmarks [36] (via search) |
| Bala, Fournier, Limonad, Metzger: *Using Process Mining to Generate AI Agents from Software Engineering Process Records* | arXiv:2607.04948 | 2026-07-06 | Mines software-repository logs for roles and constraints, then LLMs synthesise agent specs; proof of concept on one open-source project [37] (via search) |
| Amyot: *Building a Process-Modeling Tool using Agentic AI: PM4Py-UCM* | arXiv:2607.28825 | 2026-07-30 | Experience report on agent-assisted tool that mines Use Case Maps from logs [38] (via search) |

Scalability background (non-LLM, but relevant to Spark-scale):
- Berti (2019), "Increasing Scalability of Process Mining using Event Dataframes", arXiv:1907.12817: columnar storage (Parquet/ORC) and dataframe-based directly-follows computation; evaluation in Pandas, not distributed Spark [39] (via search).
- Spark-based alpha and heuristic miner: Journal of King Saud University, DOI 10.1016/j.jksuci.2021.02.008 (2022) [40] (via search).
- Cheng, van Dongen, van der Aalst: f-HMD hybrid discovery on Spark, IEEE TSC 13(2), DOI 10.1109/TSC.2019.2906203; reported ~100M events / 2.52M traces in about one minute on 16 nodes / 64 cores under their setup [41] (via search).

---

## Gaps and differentiators for a Databricks-native asset

### Credible claims (each tied to a source above)

1. **Data gravity and governance in place.** Celonis-Databricks is a public, Delta Sharing-based integration that the vendors describe as removing copy between platforms [1][27]. A Databricks-native asset can credibly say it mines governed Unity Catalog tables without moving them, which is the same architectural argument Celonis makes. Note the public Databricks docs use "OpenSharing" for the protocol page [27]; check current naming before publishing.
2. **Open event-log format is first-class.** Commercial tools mostly ingest CSV and XES with fixed connectors [10][11]. An OCEL 2.0-shaped Delta model with XES export would support object-centric analysis that flat XES cannot express [22][24].
3. **Agents on top of deterministic mining (interpretation, not invention).** PMAx's design (classical algorithms compute, LLM interprets) is the published pattern [35]. A lakehouse asset can offer the same split with lineage from Unity Catalog. The claim is "a documented pattern," not "better accuracy" (PMAx has no benchmark in its abstract).
4. **Evidence-gated automation.** The BlueSky agenda argues agent actions should be authorised by event evidence and governance contracts [36]. Lakehouse governance (Unity Catalog permissions, lineage, audit tables) is a natural substrate for this. Claim as a design direction, not a shipped feature.

### Not credible (as of this research)

- "Only Databricks can do process mining over lakehouse data." Celonis, SAP (Datasphere, Snowflake, BigQuery, Athena connectors) and others already read lakehouse-style sources [1][10].
- "Better than Celonis/UiPath/Skan at discovering automations." No public benchmark in this run supports a comparison of automation-candidate quality.
- "Native task mining." Desktop capture requires an endpoint agent. None of the vendors above run this on the lakehouse; a Databricks asset would have to ingest from a separate capture tool. Not verified that any Databricks-native desktop capture exists.
- "Spark-scale PM4Py." PM4Py's README documents no Spark backend [16]. Spark-scale discovery exists only as research code [40][41].
- "Full automation from mined logs is solved." The 2026 literature frames it as open (agenda papers, experience reports) rather than solved [36][38].

---

## Licensing constraints that should shape the build

- **Do not use PM4Py as a dependency in anything shipped or hosted without a licence decision.** AGPL-3.0 Section 5(c) and Section 13 are the constraints [17]. Options: (a) get PM4Py's commercial licence from the owner (offered per README [16]); (b) keep PM4Py out of the shipped asset and reimplement the needed algorithms; (c) isolate PM4Py in a separate, separately deployed process and use a clear interface (still needs counsel review).
- **ProM (GPL core) and Apromore Core (LGPL-3.0-or-later) are not drop-in permissive options** [19][21]. Java-based; treat as reference implementations.
- **bupaR (MIT) is the verified permissive option** [20], but it is R-only and not designed for Spark-scale data.
- **Commercial vendor formats and connectors are proprietary.** Do not embed vendor-specific connector code or schemas without checking that vendor's terms.
- **XES standard text is paywalled (IEEE)** [25]. Implementing the XES grammar from the standard text may require purchase; the OCEL 2.0 specification is publicly available [22][24].
- **Attribution and vendor claims:** Use vendor claims as claims, with dates and links, not as verified capabilities.
- **Databricks side:** Only public databricks.com docs and the Celonis public press release were used. Any Databricks product claim in the asset must cite a public Databricks page at the time of publication [27].
- Confirm the licence of every dependency (Python package, R package, Spark extension) with its LICENSE file before adoption. This research verified only the licences listed above.

---

## Implications for the canonical event model

Recommendation (not a finding; reasoning from the sources above):

- **Use OCEL 2.0 as the canonical internal model, stored as normalised Delta tables (events, objects, event-to-object relations, object-to-object relations, and attribute-change history), with XES export as a derived format.** Reasoning:
  - OCEL 2.0 can express multi-object events and changing object attributes; XES cannot without loss [22][24].
  - OCEL 2.0 publishes JSON, XML and SQLite exchange formats, which map well to Delta tables and a SQLite-compatible export path [24].
  - XES remains the format most tools (including SAP Signavio's upload path) accept, so XES export keeps the asset interoperable [10].
  - PM4Py has an `ocel` extra, so OCEL 2.0 round-trips with at least one open tool [16] (round-trip fidelity not verified in this run).
- **Do not adopt a flat, XES-only model as the canonical format.** It forces an arbitrary case choice on multi-object processes [22].
- **Task-mining events do not yet have a standard.** Keep a separate desktop-event schema, and map it to OCEL 2.0 events and objects at ingestion. No task-mining standard was found in this run.
- Keep the event model independent of any one miner's internal representation so that PM4Py, bupaR, or a commercial engine can be swapped without a data migration.

---

## Open questions / unverified claims

- Celonis's own automation or orchestration products (beyond the PI Graph feeding Agent Bricks) were not verified from public docs in this run [1].
- Microsoft Power Automate: task mining, the handoff from process mining to flows, and the data-requirements page were not verified beyond the overview [7].
- Soroco, KYP.ai, Mimica and Skan agent-generation claims were verified only from vendor landing pages (via search); depth and accuracy are not verified [12][13][14][15].
- SAP Signavio: the data-file-types page was not fetched successfully. The CSV/XES and required-field claims come from a search snippet [11 (via search)].
- The OCEL 2.0 specification PDF could not be parsed in this run; the version date (2023-10-16) comes from a search snippet, and the JSON field names were not verified [23].
- The PM4Py latest release version was not visible on the cached repo page [16].
- The AGPL-3.0 full text (gnu.org) returned HTTP 429; the Section 13 and 5(c) quotes were taken from the copy in the PM4Py repo LICENSE file, which is the FSF text [17].
- Other Python OCEL or OCPM libraries and their licences were not researched.
- Spark licence and the research code licences for [40][41] were not checked.
- Commercial pricing, customer references, and comparative accuracy were out of scope and not researched.
- Benchmarks for LLM automation generation from logs were not found beyond the papers listed above.
- The Databricks Delta Sharing docs page now uses the "OpenSharing" name [27]; check for current naming before publishing.
- Secondary sources [42][43] (CloudFronts blog, LinkedIn article) were not verified against primary Databricks documentation.

---

## Sources

1. Celonis, "Celonis partners with Databricks to power enterprise AI that continuously improves business operations" (press release). https://www.celonis.com/news/press/celonis-partners-with-databricks-to-power-enterprise-ai-that-continuously-improves-business-operations . Owner: Celonis. Published 2025-11-04. Accessed 2026-10-28.
2. UiPath docs, "Task Mining — Introduction". https://docs.uipath.com/task-mining/automation-cloud/latest/user-guide/introduction-as . Owner: UiPath. Date not shown. Accessed 2026-10-28 (via search; page not fetched directly).
3. UiPath docs, "Deprecation timeline". https://docs.uipath.com/overview/other/latest/overview/deprecation-timeline . Owner: UiPath. Last updated not shown (includes entries to 2026-09-01). Accessed 2026-10-28.
4. UiPath docs, "Process Mining release notes — October 2026". https://docs.uipath.com/process-mining/automation-cloud/latest/release-notes/october-2026 . Owner: UiPath. Dated 2026-10 (Process Insights GA on 2026-10-13). Accessed 2026-10-28.
5. UiPath docs, "Maestro — Overview". https://docs.uipath.com/maestro/automation-cloud/latest/user-guide/overview . Owner: UiPath. Date not shown. Accessed 2026-10-28 (via search).
6. UiPath docs, "Process Mining release notes — September 2026". https://docs.uipath.com/process-mining/automation-cloud/latest/release-notes/september-2026 . Owner: UiPath. Dated 2026-09 (coding-agent preview on 2026-09-16). Accessed 2026-10-28 (via search).
7. Microsoft Learn, "Overview of process mining in Power Automate". https://learn.microsoft.com/en-us/power-automate/process-mining-overview . Owner: Microsoft. ms.date 2026-08-28; updated_at 2026-09-01. Accessed 2026-10-28.
8. ServiceNow docs, "Process Mining" (product documentation; Brazil release indexed). https://www.servicenow.com/docs/r/now-intelligence/process-mining/process-mining.html . Owner: ServiceNow. Updated 2026-09-10 per search. Accessed 2026-10-28 (via search).
9. ServiceNow docs, "Task Mining Core release notes". https://www.servicenow.com/docs/r/store-release-notes/store-rn-nowintel-task-mining-core.html . Owner: ServiceNow. Versions 3.2.2 (2026-03) and 4.1.3 (2026-06). Accessed 2026-10-28 (via search).
10. SAP Help, "Supported Source System Types" (SAP Signavio Process Intelligence). https://help.sap.com/docs/signavio-process-intelligence/onboarding-and-data-integration-guide/supported-source-system-types . Owner: SAP. Date not shown. Accessed 2026-10-28 (via search).
11. SAP Help, "Data File Types" (SAP Signavio Process Intelligence). https://help.sap.com/docs/signavio-process-intelligence/user-guide/data-file-types . Owner: SAP. Date not shown. Accessed 2026-10-28 (via search; fetch returned only the page title).
12. Skan AI, "Automation discovery" use-case page. https://www.skan.ai/use-cases/automation-discovery . Owner: Skan AI. Date not shown. Accessed 2026-10-28 (via search).
13. Soroco, homepage (Scout). https://soroco.com/ . Owner: Soroco. Date not shown. Accessed 2026-10-28 (via search).
14. KYP.ai, "Task mining software". https://kyp.ai/task-mining-software/ . Owner: KYP.ai. Date not shown. Accessed 2026-10-28 (via search).
15. Mimica, "About us". https://www.mimica.ai/about-us . Owner: Mimica. Date not shown. Accessed 2026-10-28 (via search).
16. PM4Py, "pm4py-core" GitHub repository (README, sidebar). https://github.com/pm4py/pm4py-core . Owner: PM4Py project (Fraunhofer / RWTH community). Date not shown. Accessed 2026-10-28.
17. PM4Py, LICENSE (GNU AGPL-3.0 text). https://github.com/pm4py/pm4py-core/blob/release/LICENSE (Section 5(c) and Section 13 via https://raw.githubusercontent.com/pm4py/pm4py-core/release/LICENSE). Owner: FSF text in PM4Py repo. Text dated 19 November 2007. Accessed 2026-10-28.
18. GNU Affero General Public License v3 (canonical text). https://www.gnu.org/licenses/agpl-3.0.html . Owner: Free Software Foundation. Dated 2007-11-19. Accessed 2026-10-28 (HTTP 429 on fetch; not directly read).
19. Apromore Core, LICENSE.txt (development branch). https://github.com/apromore/ApromoreCore/blob/development/LICENSE.txt . Owner: Apromore. Date not shown. Accessed 2026-10-28. Enterprise licence: https://apromore.com/ee-licenses (via search).
20. bupaR, DESCRIPTION file (version 1.0.0; License: MIT + file LICENSE). https://raw.githubusercontent.com/bupaverse/bupaR/master/DESCRIPTION . Owner: bupaverse. Date not shown. Accessed 2026-10-28.
21. ProM Tools, home page (licence statement). https://promtools.org/ . Owner: ProM project. Date not shown. Accessed 2026-10-28.
22. OCEL standard, home page (OCEL 2.0 as current). https://www.ocel-standard.org/ . Owner: RWTH Aachen, Chair of Process and Data Science. Copyright 2024-2026. Accessed 2026-10-28.
23. OCEL 2.0 Specification (PDF). https://www.ocel-standard.org/2.0/ocel20_specification.pdf . Owner: OCEL standard authors. Dated 2023-10-16 (per search). Accessed 2026-10-28 (via search; PDF not parseable in this run).
24. Berti, Koren, Adams, Park, Knopp, Graves, Rafiei, Liß, Tacke Genannt Unterberg, Zhang, Schwanen, Pegoraro, van der Aalst, "OCEL (Object-Centric Event Log) 2.0 Specification", arXiv:2403.01975. https://arxiv.org/abs/2403.01975 . Submitted 2024-03-04. Accessed 2026-10-28.
25. IEEE Standards Association, IEEE 1849-2023 (XES). https://standards.ieee.org/ieee/1849/10907/ . Owner: IEEE. Published 2023-09-08. Accessed 2026-10-28.
26. XES Standard site. https://www.xes-standard.org/ . Owner: XES working group. Date not shown. Accessed 2026-10-28 (via search).
27. Databricks docs, Delta Sharing (OpenSharing protocol page). https://docs.databricks.com/aws/en/delta-sharing/ . Owner: Databricks. Last updated not shown. Accessed 2026-10-28.
28. Norouzifar, Kourani, Dees, van der Aalst, "Integrating Domain Knowledge into Process Discovery Using Large Language Models", arXiv:2510.07161. https://arxiv.org/abs/2510.07161 . Submitted 2025-10-08. Accessed 2026-10-28.
29. Rebmann, Schmidt, Glavaš, van der Aa, "On the Potential of Large Language Models to Solve Semantics-Aware Process Mining Tasks", arXiv:2504.21074. https://arxiv.org/abs/2504.21074 . Submitted 2025-04-29. Accessed 2026-10-28 (via search).
30. Pyrih, Rebmann, van der Aa, "LLMs that Understand Processes: Instruction-tuning for Semantics-Aware Process Mining", arXiv:2508.16270. https://arxiv.org/abs/2508.16270 . Submitted 2025-08-22. Accessed 2026-10-28 (via search).
31. "IoT Miner: Intelligent Extraction of Event Logs from Sensor Data for Process Mining", arXiv:2509.05769. https://arxiv.org/abs/2509.05769 . Submitted 2025-09-06. Accessed 2026-10-28 (via search).
32. Redis, Fani Sani, Zarrin, Burattin, "Skill Learning Using Process Mining for Large Language Model Plan Generation", arXiv:2410.12870. https://arxiv.org/abs/2410.12870 . Submitted 2024-10-14; proceedings 2025-03-28. Accessed 2026-10-28 (via search).
33. ProAgent: From Robotic Process Automation to Agentic Process Automation, arXiv:2311.10751. https://arxiv.org/abs/2311.10751 . Submitted 2023-11. Accessed 2026-10-28 (via search).
34. Automating the Enterprise with Foundation Models (ECLAIR), arXiv:2405.03710. https://arxiv.org/abs/2405.03710 . Submitted 2024-05. Accessed 2026-10-28 (via search).
35. Antonov, Kourani, Berti, Park, van der Aalst, "PMAx: An Agentic Framework for AI-Driven Process Mining", arXiv:2603.15351. https://arxiv.org/abs/2603.15351 . Submitted 2026-03-16. Accessed 2026-10-28.
36. Yang, Wu, Chu, Chen, Xu, Wen, "From Event Logs to Governed Action: A BlueSky Agenda for Agentic Process Mining", arXiv:2609.07984. https://arxiv.org/abs/2609.07984 . Submitted 2026-09-07. Accessed 2026-10-28 (via search).
37. Bala, Fournier, Limonad, Metzger, "Using Process Mining to Generate AI Agents from Software Engineering Process Records", arXiv:2607.04948. https://arxiv.org/abs/2607.04948 . Submitted 2026-07-06. Accessed 2026-10-28 (via search).
38. Amyot, "Building a Process-Modeling Tool using Agentic AI: An Experience Report on PM4Py-UCM", arXiv:2607.28825. https://arxiv.org/abs/2607.28825 . Submitted 2026-07-30. Accessed 2026-10-28 (via search).
39. Berti, "Increasing Scalability of Process Mining using Event Dataframes: How Data Structure Matters", arXiv:1907.12817. https://arxiv.org/abs/1907.12817 . Submitted 2019-07. Accessed 2026-10-28 (via search).
40. Journal of King Saud University – Computer and Information Sciences, 34(10 Part A), 8478–8489 (2022), "Processes meet Big Data: Scaling process discovery algorithms in Big Data environment". DOI 10.1016/j.jksuci.2021.02.008. https://www.sciencedirect.com/science/article/pii/S1319157821000471 . Accessed 2026-10-28 (via search).
41. Cheng, van Dongen, van der Aalst, "Scalable Discovery of Hybrid Process Models in a Cloud Computing Environment", IEEE Transactions on Services Computing 13(2), 368–380 (2020). DOI 10.1109/TSC.2019.2906203. https://publications.rwth-aachen.de/record/792461 . Accessed 2026-10-28 (via search).
42. CloudFronts, "From ERP Data to Process Mining Insights: Building an Automated Pipeline for Real-Time Process Visibility". https://www.cloudfronts.com/azure/azure-databricks/from-erp-data-to-process-mining-insights-building-an-automated-pipeline-for-real-time-process-visibility/ . Dated 2026-07-28 (secondary; blog). Accessed 2026-10-28 (via search).
43. Shreya Kar, "How Databricks Powers Process Mining Use Cases: From Operational Tables to Event Logs" (LinkedIn article). https://kr.linkedin.com/pulse/how-databricks-powers-process-mining-use-cases-from-operational-kar-xk7jf . Dated 2026-04-21 (secondary; not an official Databricks source). Accessed 2026-10-28 (via search).
