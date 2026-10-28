# Connector coverage and agent-building path

Issue: [#13](https://github.com/TaylorAHanson/task-mining/issues/13). Map: [#1](https://github.com/TaylorAHanson/task-mining/issues/1). Prior research: [databricks-building-blocks](https://github.com/TaylorAHanson/task-mining/blob/research/databricks-building-blocks/research/databricks-building-blocks.md) (issue [#2](https://github.com/TaylorAHanson/task-mining/issues/2)). Vocabulary: [GLOSSARY.md](https://github.com/TaylorAHanson/task-mining/blob/main/GLOSSARY.md).

Snapshot date: 2026-10-28. Public sources only: docs.databricks.com (AWS pages and release notes), with a few public search results flagged where a primary page was not read. No internal sources were used.

## Method and confidence

- Release status comes from the status wording on the Databricks release-notes page that announced each connector (dated), or from the status label on the connector's own docs page. Where neither gives a label, this note says **not stated**.
- The Lakeflow Connect overview page says connectors "are in various release states" and gives no per-connector status. The `release-types` subpage returned 404 in this snapshot, so release notes are the status source of record here.
- Release notes are staged. Databricks says an account "might not be updated until a week or more after the initial release date" ([2026/october](https://docs.databricks.com/aws/en/release-notes/product/2026/october)). Treat the date as the announcement date.
- Items marked **(search)** were found by a public web search and the cited Databricks page was not fully read in this session. Re-check before relying on them.
- Historical status is kept where it matters, because a connector can move from Beta to GA between announcements. The latest label wins.

## Summary

- **Connector coverage is broad and mostly GA for the systems that matter most for this asset.** GA managed connectors exist for ITSM (ServiceNow), CRM (Salesforce, Microsoft Dynamics 365, HubSpot), ERP and finance (NetSuite, Dynamics 365), HR reporting (Workday Reports), file and collaboration (SharePoint, Google Drive, Confluence), and support (Zendesk Support). Workday HCM and Jira are still Beta.
- **Gaps for the Scenario's role-named systems:** no managed Lakeflow Connect connector found for SAP ERP or SAP SuccessFactors, ADP, or UKG. SAP Business Data Cloud is GA but is a zero-copy sharing mechanism, not Lakeflow Connect ingestion. Slack and Microsoft Teams message ingestion has no managed GA connector found. Oracle Fusion ERP is not established.
- **Agent path:** the recommended path for new code-first agents is **Agent Bricks CLI** (Beta) → **DurableAgentServer** → **Agent Runtime** on Databricks Apps. Agent Runtime's own status is **not stated**. The CLI is Beta, so the agent Automation kind is anchored on a Beta entry point with a not-stated runtime. **Knowledge Assistant, Supervisor Agent, the Supervisor API, and the Model Serving `ResponsesAgent` / `agents.deploy()` path are legacy** and must not anchor new Automations.
- **Managed MCP:** Unity Gateway is GA. The Genie One MCP is GA. Workspace-level managed MCP endpoints are Public Preview and bypass Unity Gateway. Most external-service MCPs (`system.ai.slack`, `system.ai.github`, `system.ai.atlassian`, and others) have no status label on the managed-MCP page.

## Connector matrix by system category

Status columns show the latest verified label and its announcement date. "Not found" means no managed Lakeflow Connect connector was found in the release notes or docs read. It does not prove that none exists in a private preview.

### ITSM

| System | Status | Date and source | Notes |
|---|---|---|---|
| ServiceNow | **GA** | GA on Aug 8, 2025 ([2025/august](https://docs.databricks.com/aws/en/release-notes/product/2025/august)). Public Preview Mar 10, 2025 ([2025/march](https://docs.databricks.com/aws/en/release-notes/product/2025/march)). Connector page: [servicenow](https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/servicenow) (no status label on page) | Incremental ingestion, SCD2, schema evolution, UC governance. Max 250 tables per pipeline. Limits page does not mention ITSM tables (incident, change, problem) specifically. |
| PagerDuty | Beta | Aug 4, 2026 ([2026/august](https://docs.databricks.com/aws/en/release-notes/product/2026/august)) | Incident-response adjacent. |
| Jira | Beta | Beta on Dec 10, 2025 ([2025/december](https://docs.databricks.com/aws/en/release-notes/product/2025/december)). No GA entry found in the 2026 release notes read (Apr–Oct 2026). | Browser OAuth (U2M) only, so it cannot be created programmatically (per the overview page). Jira Service Management not checked. |
| Zendesk Support | **GA** | Apr 28, 2026 ([2026/april](https://docs.databricks.com/aws/en/release-notes/product/2026/april)) | Browser OAuth (U2M) only per the overview page. |

### CRM

| System | Status | Date and source | Notes |
|---|---|---|---|
| Salesforce (ingestion connector) | **GA** | GA on Apr 2, 2025 ([2025/april](https://docs.databricks.com/aws/en/release-notes/product/2025/april)). Mutual TLS auth GA Aug 20, 2026 ([2026/august](https://docs.databricks.com/aws/en/release-notes/product/2026/august)). Row filtering GA July 2, 2026 covers SaaS connectors including Salesforce ([2026/july](https://docs.databricks.com/aws/en/release-notes/product/2026/july)). | Lakeflow Connect Salesforce ingestion. Salesforce Data Cloud File Sharing GA Aug 22, 2025. Salesforce Marketing Cloud **Beta** June 18, 2026 ([2026/june](https://docs.databricks.com/aws/en/release-notes/product/2026/june)). |
| Microsoft Dynamics 365 | **GA** | Public Preview Dec 10, 2025 ([2025/december](https://docs.databricks.com/aws/en/release-notes/product/2025/december)). GA June 26, 2026 ([2026/june](https://docs.databricks.com/aws/en/release-notes/product/2026/june)). | Also ERP-adjacent. Route via Azure Synapse Link per a public search result (**search**); not read on a primary page. |
| HubSpot | **GA** | GA May 11, 2026 for the managed connector ([2026/may](https://docs.databricks.com/aws/en/release-notes/product/2026/may)). Search summary says Marketing Hub. | Browser OAuth (U2M) only per the overview page. |
| Zoho CRM | Beta | Oct 12, 2026 ([2026/october](https://docs.databricks.com/aws/en/release-notes/product/2026/october)) | |

### ERP and finance

| System | Status | Date and source | Notes |
|---|---|---|---|
| Oracle NetSuite | **GA** | Public Preview Dec 10, 2025 ([2025/december](https://docs.databricks.com/aws/en/release-notes/product/2025/december)). GA Aug 4, 2026 ([2026/august](https://docs.databricks.com/aws/en/release-notes/product/2026/august)). | Strongest GA ERP candidate in the public release notes. |
| Microsoft Dynamics 365 | **GA** | See CRM row | Covers ERP and CRM per search summary (**search**). |
| Oracle database (query-based) | **GA** | Query-based connectors (Oracle, Teradata, SQL Server, MySQL, MariaDB, PostgreSQL, and Lakehouse Federation sources) GA May 29, 2026 ([2026/may](https://docs.databricks.com/aws/en/release-notes/product/2026/may)). Public Preview Apr 15, 2026 ([2026/april](https://docs.databricks.com/aws/en/release-notes/product/2026/april)). | Database-level, not application-aware. |
| Oracle database (integrated CDC) | Not verified | Search summary says Beta, with workspace enablement (**search**). Primary Oracle connector page returned 404 in this session. | Oracle E-Business Suite would be reached only through the database. Oracle Fusion Cloud ERP is **not established** as supported. |
| SAP Business Data Cloud | **GA** (not Lakeflow Connect) | GA Sep 10, 2025 (**search**, citing [2025/september](https://docs.databricks.com/aws/en/release-notes/product/2025/september)) | Zero-copy sharing. Databricks distinguishes it from Lakeflow Connect ingestion. |
| SAP ERP / S/4HANA, SAP SuccessFactors | **Not found** | No managed Lakeflow Connect connector found in the release notes or overview read (**search** for SuccessFactors). | Do not use SAP BDC GA as evidence for a direct SAP ERP or SuccessFactors connector. |
| QuickBooks Online | Beta | Oct 26, 2026 ([2026/october](https://docs.databricks.com/aws/en/release-notes/product/2026/october)) | |
| Zoho Books | Beta | June 8, 2026 ([2026/june](https://docs.databricks.com/aws/en/release-notes/product/2026/june)) | |
| PayPal | Beta | Oct 24, 2026 ([2026/october](https://docs.databricks.com/aws/en/release-notes/product/2026/october)) | |
| Anaplan | Beta | Sep 9, 2026 ([2026/september](https://docs.databricks.com/aws/en/release-notes/product/2026/september)) | |
| MySQL integrated CDC pipeline | **GA** | Oct 15, 2026 ([2026/october](https://docs.databricks.com/aws/en/release-notes/product/2026/october)). Beta Jul 1, 2026. | Database CDC, single pipeline, no separate gateway. |

### HR

| System | Status | Date and source | Notes |
|---|---|---|---|
| Workday Reports | **GA** | GA Apr 2, 2025, with Salesforce ([2025/april](https://docs.databricks.com/aws/en/release-notes/product/2025/april)). Incremental ingestion Beta June 18, 2026 ([2026/june](https://docs.databricks.com/aws/en/release-notes/product/2026/june)). | Custom reports. |
| Workday HCM | **Beta** | Connector page: "This feature is in Beta." ([workday-hcm-overview](https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/workday-hcm-overview)). Covers workers, positions, payroll, and talent records. | Separate from Workday Reports. |
| Workday Activity Logging | Beta | Sep 28, 2026 ([2026/september](https://docs.databricks.com/aws/en/release-notes/product/2026/september)) | Audit logs. |
| ADP, UKG | **Not found** | No managed connector in the release notes or SaaS overview (**search**). | |

### Collaboration and file

| System | Status | Date and source | Notes |
|---|---|---|---|
| SharePoint (managed and standard) | **GA** | GA Aug 4, 2026 ([2026/august](https://docs.databricks.com/aws/en/release-notes/product/2026/august)). Customizable SharePoint Beta Dec 10, 2025. | SharePoint Lists ingestion and `FILE`-type ingestion remain **Beta** (**search**, citing [sharepoint-pipeline](https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/sharepoint-pipeline)). Structured file ingestion Beta June 5, 2026. |
| Google Drive (managed and standard) | **GA** | GA Aug 4, 2026 ([2026/august](https://docs.databricks.com/aws/en/release-notes/product/2026/august)). Beta May 27, 2026 ([2026/may](https://docs.databricks.com/aws/en/release-notes/product/2026/may)). | |
| Confluence | **GA** | GA Apr 25, 2026 ([2026/april](https://docs.databricks.com/aws/en/release-notes/product/2026/april)). Beta Dec 16, 2025. | Browser OAuth (U2M) only per the overview page. |
| Outlook | Beta | May 5, 2026 ([2026/may](https://docs.databricks.com/aws/en/release-notes/product/2026/may)) | |
| Gmail | Beta | Aug 13, 2026 ([2026/august](https://docs.databricks.com/aws/en/release-notes/product/2026/august)) | |
| Notion | Beta | Aug 10, 2026 ([2026/august](https://docs.databricks.com/aws/en/release-notes/product/2026/august)) | |
| Asana | Beta | Oct 20, 2026 ([2026/october](https://docs.databricks.com/aws/en/release-notes/product/2026/october)) | |
| Smartsheet, Monday.com | Beta | May 5, 2026; June 8, 2026 | |
| Slack (access and integration logs) | Beta | June 2026 (**search**, Azure mirror of release notes) | Logs only. No managed GA Slack message-ingestion connector found. |
| Microsoft Teams | **Not managed** | Community connector only (**search**, GitHub README in `databrickslabs/lakeflow-community-connectors`). Community connectors framework Beta, May 1, 2026 ([2026/may](https://docs.databricks.com/aws/en/release-notes/product/2026/may)). | Channels, messages, and replies via community connector. Chats not supported. Not Databricks-maintained, no SLA. |
| GitHub | Beta | May 5, 2026 ([2026/may](https://docs.databricks.com/aws/en/release-notes/product/2026/may)) | |
| Azure DevOps, GitLab | Beta | Oct 8 and Oct 1, 2026 ([2026/october](https://docs.databricks.com/aws/en/release-notes/product/2026/october)) | |

### Database and streaming (for evidence sources)

| System | Status | Date and source |
|---|---|---|
| SQL Server (ingestion connector) | **GA** | GA Aug 26, 2025 ([2025/august](https://docs.databricks.com/aws/en/release-notes/product/2025/august)) |
| PostgreSQL (CDC) | Public Preview Dec 16, 2025 ([2025/december](https://docs.databricks.com/aws/en/release-notes/product/2025/december)); status after that not checked | |
| RabbitMQ (streaming) | Beta | June 2, 2026 ([2026/june](https://docs.databricks.com/aws/en/release-notes/product/2026/june)) |

## Recommended agent-building path

Recommended for new code-first agents, per the Agent Bricks docs (updated Oct 13, 2026) and the legacy-offerings page:

| Step | Component | Release status | Source |
|---|---|---|---|
| 1. Scaffold, run locally, deploy | **Agent Bricks CLI** (`agentbricks init / dev / deploy`) | **Beta**: "This feature is in Beta. No workspace setting is required to enable it." | [agent-bricks-cli](https://docs.databricks.com/aws/en/agents/custom-agents/agent-bricks-cli) |
| 2. Agent loop server | **DurableAgentServer** (Agent Server) | Not stated | [agents concepts](https://docs.databricks.com/aws/en/agents/concepts/) |
| 3. Hosting | **Agent Runtime** (serverless Databricks Apps, authenticated endpoint) | Not stated | [agents concepts](https://docs.databricks.com/aws/en/agents/concepts/) |
| 4. Memory and sessions | Agent memory and sessions (Lakebase-backed) | Managed agent memory Beta June 23, 2026; agent memory pipelines Beta Oct 12, 2026 ([2026/june](https://docs.databricks.com/aws/en/release-notes/product/2026/june), [2026/october](https://docs.databricks.com/aws/en/release-notes/product/2026/october)) | release notes |
| 5. Tools and MCP | Managed MCP servers, Unity Gateway | See MCP table below | [managed-mcp](https://docs.databricks.com/aws/en/generative-ai/mcp/managed-mcp) |
| 6. Model access | Unity Gateway model access | Unity Gateway **GA** Aug 4, 2026 (for the gateway; some service policies and agent services remain Beta per the same entry) | [2026/august](https://docs.databricks.com/aws/en/release-notes/product/2026/august) |
| 7. Tracing and evaluation | MLflow Tracing; evaluate and improve with LLM judges and scorers | Not stated on the agents concepts page. Unified trace table for AI activity Beta Aug 21, 2026. | [agents concepts](https://docs.databricks.com/aws/en/agents/concepts/), [2026/august](https://docs.databricks.com/aws/en/release-notes/product/2026/august) |
| Low-code alternative | **Genie Agents** (formerly Genie Spaces, renamed July 8, 2026) | Not stated on the page. Genie Code Beta/PP entries exist separately. | [2026/july](https://docs.databricks.com/aws/en/release-notes/product/2026/july) |
| Frameworks | Any framework or harness (the agents page names no specific framework) | n/a | [agents concepts](https://docs.databricks.com/aws/en/agents/concepts/) |

Components not to anchor on for new agents:

| Component | Status | Source |
|---|---|---|
| Knowledge Assistant | **Legacy**. Replace with Genie Agents for low-code. | [agents/legacy](https://docs.databricks.com/aws/en/agents/legacy/) (updated Oct 13, 2026); release notes Oct 12, 2026 |
| Supervisor Agent | **Legacy**. Genie Agents for low-code; Agent Bricks CLI for multi-agent code. | same |
| Supervisor API | **Legacy and deprecated** | [agents/legacy](https://docs.databricks.com/aws/en/agents/legacy/); 2026/april release note Apr 20, 2026 |
| Mosaic AI Agent Framework `ResponsesAgent` + `agents.deploy()` (Model Serving) | **Legacy** ("Custom Agent on Model Serving"). Migration guide exists. | [agents/legacy](https://docs.databricks.com/aws/en/agents/legacy/) |
| MLflow `AgentServer` on Databricks Apps ("legacy agent server") | **Legacy** | [agents/legacy](https://docs.databricks.com/aws/en/agents/legacy/) |
| Mosaic AI Agent Framework as a named product | **Not stated.** The `/generative-ai/agent-framework/` path 404'd in the prior research, and no current page gives its status. | prior research |
| Omnigent (managed multi-harness coordination layer) | **Beta**. Requires Omnigent workspace preview and a Unity Gateway region. Sandbox use needs a further preview and region. | [omnigent](https://docs.databricks.com/aws/en/omnigent/) (read in this session) |
| Model Serving (endpoints) | Labels not stated in prior research or this session | prior research |

## Managed MCP and Unity Gateway

| Item | Status | Source and date |
|---|---|---|
| Unity Gateway | **GA** | Aug 4, 2026 ([2026/august](https://docs.databricks.com/aws/en/release-notes/product/2026/august)) |
| `system.ai.genie_one_mcp` (Genie One MCP) | **GA** | [managed-mcp](https://docs.databricks.com/aws/en/generative-ai/mcp/managed-mcp) |
| `system.ai.dbsql`, `system.ai.web_search`, `system.ai.sandbox` | **Beta** | [managed-mcp](https://docs.databricks.com/aws/en/generative-ai/mcp/managed-mcp) |
| `system.ai.slack`, `system.ai.github`, `system.ai.atlassian`, `system.ai.google_drive`, `system.ai.google_calendar`, `system.ai.gmail`, `system.ai.microsoft_365` | **Not stated** on the managed-MCP page | [managed-mcp](https://docs.databricks.com/aws/en/generative-ai/mcp/managed-mcp) |
| Databricks-managed MCP connectors on Unity Gateway | Beta (heading) | Aug 6, 2026 ([2026/august](https://docs.databricks.com/aws/en/release-notes/product/2026/august)) |
| Managed workspace MCP endpoints (Genie Agent, AI Search, Databricks SQL legacy, UC functions) | **Public Preview**. These bypass Unity Gateway, so they have no MCP-level grants, policies, or tracing. | [managed-mcp](https://docs.databricks.com/aws/en/generative-ai/mcp/managed-mcp) |
| Default OAuth apps for external AI agents (connecting Claude, Codex, ChatGPT clients to Databricks MCP) | **GA** | Oct 27, 2026 ([2026/october](https://docs.databricks.com/aws/en/release-notes/product/2026/october)) |
| AI Gateway governs MCP servers | Beta | Apr 15, 2026 ([2026/april](https://docs.databricks.com/aws/en/release-notes/product/2026/april)) |
| ABAC GRANT policies for models, model services, and MCP services | **GA** | Aug 31, 2026 ([2026/august](https://docs.databricks.com/aws/en/release-notes/product/2026/august)). Other ABAC GRANT types for Unity Gateway resources Beta Aug 11, 2026. |
| Third-party MCP servers registered through UC connections | Not stated on the managed-MCP page | [managed-mcp](https://docs.databricks.com/aws/en/generative-ai/mcp/managed-mcp) |

## Implications

**For the Automation taxonomy (#7).**
- Agent Automations should be built on Agent Bricks CLI → DurableAgentServer → Agent Runtime, not on Knowledge Assistant, Supervisor Agent, or the Model Serving `agents.deploy()` path. The CLI is Beta, and Agent Runtime's status is not stated. Shadow-mode builds should record that status risk.
- Low-code agent Automations, if the taxonomy needs them, map to Genie Agents. Genie Agents' status is not stated on the page.
- External writes: connectors are read-side for most sources, so write-side tool access goes through Unity Gateway MCP services or API calls from Jobs/Apps. Most external-service MCPs have no status label, so do not treat them as GA.
- Omnigent is Beta and region-gated. Keep it out of the first wave.

**For which role-named systems a Scenario should model (#5).** Use systems whose managed connector is GA and whose shape matches the role:

| Role in the Scenario | Recommended connector shape | Status | Avoid or caveat |
|---|---|---|---|
| ERP | NetSuite (or Dynamics 365) | NetSuite GA Aug 4, 2026; Dynamics 365 GA June 26, 2026 | SAP ERP has no found managed connector. Oracle Fusion not established. |
| CRM | Salesforce (or HubSpot Marketing Hub, or Dynamics 365) | Salesforce GA Apr 2, 2025; HubSpot GA May 11, 2026 | Zoho CRM Beta. |
| ITSM | ServiceNow | GA Aug 8, 2025 | Jira is Beta. PagerDuty is Beta. |
| HR | Workday Reports (GA) for reporting data; Workday HCM only if a Beta source is acceptable | Reports GA Apr 2, 2025; HCM Beta | ADP, UKG, SuccessFactors not found. |
| Collaboration and documents | SharePoint or Google Drive; Confluence for wiki | SharePoint GA Aug 4, 2026; Google Drive GA Aug 4, 2026; Confluence GA Apr 25, 2026 | Slack and Teams messages not GA managed. |
| Support desk | Zendesk Support | GA Apr 28, 2026 | |

Role-named Scenario systems can therefore follow NetSuite-shaped ERP, Salesforce-shaped CRM, ServiceNow-shaped ITSM, Workday Reports-shaped HR, and SharePoint/Confluence-shaped collaboration without depending on Beta connectors. Slack, Teams, SAP, ADP, and UKG should be modelled as Beta or community sources only if the Scenario explicitly accepts that.

## Corrections to the prior research

- ServiceNow and Salesforce were "not verified" in [databricks-building-blocks](https://github.com/TaylorAHanson/task-mining/blob/research/databricks-building-blocks/research/databricks-building-blocks.md). They are now verified (GA), with dates above.
- Agent Bricks status was "not stated" there. The Agent Bricks CLI is now verified **Beta**. Agent Runtime and DurableAgentServer remain not stated.
- Genie spaces are now called **Genie Agents** (July 8, 2026, [2026/july](https://docs.databricks.com/aws/en/release-notes/product/2026/july)).
- The prior note said Genie One and Genie Agents usage is "free through Jan 31, 2027." The July 15, 2026 release-note entry says "billing for Genie One and Genie Agents is paused," and the July 8 entry says Genie products "are billed following a pay-as-you-go pricing model" ([2026/july](https://docs.databricks.com/aws/en/release-notes/product/2026/july)). The two statements conflict and need reconciling by whoever owns cost questions.
- Unity Gateway is GA as of Aug 4, 2026, which the prior note did not state.

## Follow-up research (not resolved here)

1. Agent Runtime and DurableAgentServer: a release-status label on a primary page.
2. Oracle database integrated CDC (Beta per search) and Oracle Fusion: primary page not reachable here.
3. Jira status between Apr and Oct 2026 (no GA entry found; Jan–Mar 2026 not read).
4. Jira Service Management, Slack messages, and Teams managed connectors: confirm whether any private or account-level preview exists through the account team (not checkable in public docs).
5. Model Serving and Lakehouse Federation release labels (still not stated in the reads here).
6. Whether third-party MCP servers can be registered through UC connections (not stated).

## Sources

Dated release notes (docs.databricks.com/aws/en/release-notes/product/...):
- 2025/march (ServiceNow Public Preview Mar 10, 2025): https://docs.databricks.com/aws/en/release-notes/product/2025/march
- 2025/april (Salesforce and Workday Reports GA Apr 2, 2025): https://docs.databricks.com/aws/en/release-notes/product/2025/april
- 2025/august (ServiceNow GA Aug 8, 2025; SQL Server GA Aug 26, 2025): https://docs.databricks.com/aws/en/release-notes/product/2025/august
- 2025/december (NetSuite, Dynamics 365, Jira, Confluence, PostgreSQL, MySQL Public Preview or Beta, Dec 2025): https://docs.databricks.com/aws/en/release-notes/product/2025/december
- 2026/april (Zendesk GA Apr 28; Confluence GA Apr 25; query-based Public Preview Apr 15; Supervisor API deprecated Apr 20): https://docs.databricks.com/aws/en/release-notes/product/2026/april
- 2026/may (HubSpot GA May 11; query-based GA May 29; Google Drive Beta May 27; community connectors Beta May 1): https://docs.databricks.com/aws/en/release-notes/product/2026/may
- 2026/june (Dynamics 365 GA June 26; Workday Reports incremental Beta June 18; Salesforce Marketing Cloud Beta June 18; managed agent memory Beta June 23): https://docs.databricks.com/aws/en/release-notes/product/2026/june
- 2026/july (Genie Spaces renamed Genie Agents July 8; Genie One and Genie Agents billing paused July 15; row filtering GA July 2): https://docs.databricks.com/aws/en/release-notes/product/2026/july
- 2026/august (NetSuite GA Aug 4; Google Drive GA Aug 4; SharePoint GA Aug 4; Unity Gateway GA Aug 4; Salesforce mTLS GA Aug 20; managed MCP connectors Beta Aug 6): https://docs.databricks.com/aws/en/release-notes/product/2026/august
- 2026/september (Workday Activity Logging Beta; Zerobus default-storage Public Preview Sep 7): https://docs.databricks.com/aws/en/release-notes/product/2026/september
- 2026/october (Knowledge Assistant and Supervisor Agent legacy Oct 12; Agent Bricks code-first docs Oct 5; QuickBooks Beta Oct 26; default OAuth apps for MCP GA Oct 27; MySQL integrated CDC GA Oct 15): https://docs.databricks.com/aws/en/release-notes/product/2026/october

Docs pages (read in this session):
- Lakeflow Connect overview (updated Oct 9, 2026; no per-connector status): https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/
- ServiceNow connector (no status label on page; supported features and 250-table limit): https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/servicenow
- Workday HCM overview ("This feature is in Beta."): https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/workday-hcm-overview
- Agents concepts (updated Oct 13, 2026; components; no status labels): https://docs.databricks.com/aws/en/agents/concepts/
- Agents overview (no single recommended path label; no statuses): https://docs.databricks.com/aws/en/agents/
- Agent Bricks CLI ("This feature is in Beta."): https://docs.databricks.com/aws/en/agents/custom-agents/agent-bricks-cli
- Agents legacy offerings (updated Oct 13, 2026): https://docs.databricks.com/aws/en/agents/legacy/
- Managed MCP and Unity Gateway MCPs: https://docs.databricks.com/aws/en/generative-ai/mcp/managed-mcp
- Omnigent ("Omnigent is in Beta"): https://docs.databricks.com/aws/en/omnigent/

Not reachable in this session (404): `https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/release-types`; `https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/oracle-database-overview`.

Public search results used for flagged (**search**) items only; re-check the cited pages before relying on them: SAP BDC GA (Sep 10, 2025), Dynamics 365 via Azure Synapse Link, SharePoint Lists and `FILE` Beta, Teams community connector README, Slack access-logs Beta (Azure mirror), Oracle integrated CDC Beta, SuccessFactors/ADP/UKG not found.
