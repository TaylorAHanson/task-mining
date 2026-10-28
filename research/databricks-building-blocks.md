# Databricks building blocks for Discovery and Automation

Issue: [#2](https://github.com/TaylorAHanson/task-mining/issues/2). Map: [#1](https://github.com/TaylorAHanson/task-mining/issues/1). Vocabulary: [GLOSSARY.md](https://github.com/TaylorAHanson/task-mining/blob/main/GLOSSARY.md).

Snapshot date: 2026-10-28. Public sources only (docs.databricks.com, public standards pages). No internal sources were used.

## Method and confidence

- Sources are Databricks public documentation pages (AWS docs under `docs.databricks.com/aws/en/...`) and the public OCEL 2.0 specification. Each page was read through a summarizing fetch, not by reading the raw HTML. Before any claim becomes a build decision, re-check it on the linked page.
- "Last updated" dates are the dates shown on the pages when fetched. Pages carry no per-feature GA dates unless noted.
- Where a page did not state a release status, this doc says **status not stated** rather than guessing. Those gaps are listed at the end as follow-ups.
- URLs that returned 404 during the research: `.../ingestion/lakeflow-connect/release-types`, `.../generative-ai/agent-framework/`, and the OCEL `https://www.ocel-standard.org/2.0/` HTML path (its PDF resolves). Some guessed paths redirected to other pages (for example `.../generative-ai/agent-bricks/` and `.../generative-ai/agent-framework/build-genai-apps` returned the Agent Bricks concepts content). Links below point at the pages whose content was actually read.

## Summary

- **Discovery signal exists in system tables, but it is machine-centric and lagged.** `system.query.history`, `system.lakeflow.*`, `system.access.audit`, and `system.access.table_lineage` / `column_lineage` record who ran what, when, and what read or wrote which table. None of them flags work as manual. Human vs. repetitive vs. manual has to be inferred from identity, cadence, and gaps. Records appear about an hour after the event, so these tables are batch evidence, not a live capture stream. Several are still Public Preview (audit, alerts, the MLflow system tables, others).
- **Live task capture needs a separate ingest path.** Zerobus Ingest is the right shape (push API into Unity Catalog Delta tables, records land within seconds). Its default-storage ingest is Public Preview, and the C++ and C#/.NET SDKs are Beta. It has no quota numbers on the page, so throughput and region fit must be checked on the quotas page.
- **Getting ERP/CRM/ITSM data in:** Lakeflow Connect (managed ingestion, including SaaS and database CDC connectors) is the primary route. Lakehouse Federation is read-only and therefore only helps with reads. Connector release states are not on the page fetched, so each connector's status must be checked before it is relied on.
- **Automation execution:** Jobs and Lakeflow Spark Declarative Pipelines run batch and streaming work. Agents run on Agent Runtime / Agent Bricks components. Model Serving hosts agents and models as endpoints. Apps provide human-facing UIs for process owners and approvals. Lakebase provides low-latency state. Genie is a business-user Q&A layer, not an execution engine.
- **Writing to external systems is not a federation or Genie job.** Federation is read-only. External writes go through Unity Gateway external-service MCPs (`system.ai.slack`, `system.ai.github`, `system.ai.atlassian`, and others), through UC-governed Apps/Jobs calling APIs, or through Lakeflow Connect's sync paths where they apply. Status of the external-service MCPs is not stated on the page.
- **Shadow mode is not a named primitive.** MLflow tracing and evaluation record every agent step and support judges and scorers, which is enough to compare a candidate Automation's output against the human's output. The orchestration of "run alongside, compare, hold until approved, then cut over with rollback" has to be built on Jobs / Apps / Lakebase. The docs describe no built-in equivalent.
- **Governance:** Unity Catalog covers tables, models, functions, and MCP servers through Unity Gateway. ABAC (attribute policies) is in Beta at metastore level. Workspace-level managed MCP endpoints are Public Preview and **bypass Unity Gateway** (no MCP-level grants, policies, guardrails, or tracing tables).

## Implications for the map's open decisions

These are the questions the map currently lists as open. Each implication is labeled with the ticket it affects.

1. **Canonical event model (#6).** Lineage gives many-to-many relations (one job run reads several tables and writes several others; one query touches several objects). Query history, jobs, and task runs carry run IDs and parent IDs. That is object-centric data, not a single-case log. The public standard that already models this is OCEL 2.0 (events, objects, event-to-object relations, object-to-object relations, qualifiers, dynamic attributes; JSON/XML/SQLite formats). Suggestion for #6: model the canonical event as object-centric, use OCEL 2.0 as the interchange format for process-mining algorithms, and map each Databricks source to events with explicit object references. A single `case_id` is too narrow for lineage-derived evidence. Task capture (Zerobus) events need the same object references (application, account, record) to be joinable.
2. **Automation taxonomy (#7).** Map each Automation kind to a primitive that exists and is not deprecated: batch and scheduled work → Jobs; streaming transforms and data quality → Lakeflow Spark Declarative Pipelines; reasoning/tool-using agent → Agent Runtime (Agent Bricks CLI; the page's "legacy agent builders" Knowledge Assistant and Supervisor Agent are **not recommended for new agents**, so do not use them as taxonomy anchors); human-in-the-loop approval → Apps; state → Lakebase; natural-language answers → Genie One (GA for the MCP). Also note that a single Automation can reach external systems only through connectors, Unity Gateway external-service MCPs, or API calls. Federation cannot be the write path.
3. **Discovery architecture: algorithms vs agents (#8).** The system tables are the best deterministic input for algorithmic process mining because they are complete for their covered systems (SQL warehouses, serverless notebooks/jobs, Lakeflow). But they are only 365-day, regional, lagged, and do not include application-level activity. Agents are needed for the unstructured sources and for interpreting ambiguous lineage. This supports the map's hybrid framing, with the structured-log side constrained by the retention and region limits below.
4. **Candidate model and value scoring (#9).** Billing (`system.billing.usage`, `system.billing.list_prices`) gives cost per workload. Jobs timelines give run durations and frequency. Together they supply the frequency and effort inputs for value scoring. `WORKFLOW_RUN` cost is attributed to the parent notebook, and all-purpose compute cost is only an estimate. Value scores should carry that uncertainty.
5. **Shadow mode, approval, cutover, rollback (#10).** MLflow tracing and evaluation support comparison. No platform cutover feature is documented. Approval UI would live in Apps. The shadow-mode design should not assume a managed traffic-split feature; that needs confirmation in a later ticket. Governance of the Automation itself (who approved, what was run as which identity) should use Unity Catalog and audit logs. Note that `run_as` and `run_by` are distinct fields in audit identity metadata.
6. **Reference enterprise and seeded processes (#5).** Seeded processes that depend on SaaS CRM/ITSM data depend on Lakeflow Connect connector availability, which is not verified for every SaaS connector in this doc. Pick the seeded sources after the connector status checks in the follow-ups below.
7. **Task capture and landing path (#4).** Zerobus is a reasonable landing path for high-volume capture events. Public Preview status for default-storage ingest and the Beta SDK status mean the capture design must stay within the supported SDKs (Python, Java, Rust, Go, TypeScript over gRPC; REST; OTLP; MQTT v5; Kafka-compatible API) until GA. Zerobus is documented for telemetry and clickstream use, so using it for desktop/browser capture is consistent with its stated intent. Privacy of captured data is a separate question.
8. **Privacy and governance (cross-cutting).** `statement_text` in query history is redacted by default and visible only to account admins and `databricks_pii_access` members. Audit `request_params` keys for SQL definitions are similarly masked. Column masks on `statement_text` can break queries. These defaults help the privacy posture but mean Discovery needs admin-level or PII-group access to read SQL text. That access requirement must go into the grants section of the spec.

## Building block-by-building block

Status labels are as stated on the page fetched. "Not stated" means the page gave no label.

### System tables

- **Source:** https://docs.databricks.com/aws/en/admin/system-tables/ (page fetched, labels as stated on that page)
- **Discovery gives:** the complete record of activity inside covered systems. `query.history` (SQL warehouses, serverless notebooks and jobs, Lakeflow pipelines; redacted statement text by default; `query_source` links to notebook, dashboard, alert, Genie space, or job run). `lakeflow.jobs` (SCD2 job definitions), `lakeflow.job_run_timeline` and `job_task_run_timeline` (run durations, triggers, result states). `access.audit` (user identity, service, action, request params). `access.table_lineage` and `column_lineage` (who read or wrote which table and column, via which job/notebook/query/dashboard, joinable to query history via `statement_id`). `billing.usage` and `billing.list_prices` (cost).
- **Automation gives:** the baseline cost and run data needed to value an Automation, and a way to verify an Automation's outcomes after cutover (the same tables record the replacement's runs).
- **Status:** `access` audit and lineage are Public Preview on the access schema (audit is labeled Public Preview on its page; the page lists `column_lineage` and `table_lineage` as unlabeled). `billing`, `query`, `lakeflow` jobs and pipelines, and most `compute` tables are unlabeled. Zerobus tables and `lakeflow_pipeline_events_preview` are Beta. `lineage` (the older schema) is deprecated. Configurable retention is Beta.
- **Limits:**
  - Retention: 365 days for most tables; `compute.node_timeline` 90 days; `ai_gateway.external_model_spend` 365 days (Beta). Configurable retention (Beta) extends the free period to 395 days and allows 30 to 3,650 days. Retention beyond 395 days incurs storage charges once GA.
  - Regional: `jobs` and `query.history` include only workspaces in the same cloud region as the workspace you query from. Lakebase system tables have 7-day retention and are Beta.
  - Latency: no real-time monitoring. Job and query records are typically available within about 1 hour. Newer workspaces can take longer.
  - Schema drift: new columns may be added at any time, so fixed-schema queries can break.
  - Query size: insufficiently selective queries fail on big tables.
  - Streaming: a stream falling more than 7 days behind may break due to VACUUM.
  - GovCloud: only a subset is available.
  - Privileges: account and metastore admins by default. Others need `USE CATALOG` on `system`, plus `USE SCHEMA` and `SELECT`. Metastore must be on UC Privilege Model v1.0. Read-only.
- **Source pages used:** system tables index; audit log table (`/admin/system-tables/audit-logs`); lineage tables (`/admin/system-tables/lineage`); jobs (`/admin/system-tables/jobs`); query history (`/admin/system-tables/query-history`).

### Lakeflow Connect

- **Source:** https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/ (page fetched; the release-types subpage returned 404 in this snapshot)
- **Discovery gives:** managed ingestion of ERP/CRM/ITSM-adjacent systems into the lakehouse, including SaaS connectors (Workday named as an example), database CDC (SQL Server named), and file connectors (SharePoint named). Source data becomes the evidence layer for process mining.
- **Automation gives:** the path for pulling source data that automations need to read, and for keeping it fresh.
- **Status:** the page says connectors are "in various release states" and does not list per-connector status. Salesforce, ServiceNow, and Google Analytics are **not mentioned** on the fetched page, so their availability is unverified here.
- **Limits:** some sources or tables don't support incremental ingestion. Connectors that use only browser OAuth (U2M) cannot be created programmatically (listed examples include HubSpot, Jira, Slack, Zendesk Support, Confluence, Google Ads, Meta Ads, Slack Audit Logs, TikTok Ads). Databricks may discontinue a connector if the external API changes make it impractical. Networking: SaaS via serverless egress controls; cloud DBs via Private Link or peering; on-prem via Direct Connect or ExpressRoute.

### Lakehouse Federation

- **Source:** https://docs.databricks.com/aws/en/query-federation/ (page fetched)
- **Discovery gives:** read access to external operational data without copying it. Query federation supports MySQL, PostgreSQL, Teradata, Oracle, Redshift, Salesforce Data 360, Snowflake, SQL Server, Synapse, BigQuery, Databricks. Catalog federation covers Hive metastores, Glue, Snowflake, Palantir Foundry, Workday Data Connect, and others.
- **Automation gives:** read-side input for Automations. It is **read-only** for both types.
- **Status:** not stated on the page.
- **Limits:** read-only; query federation pushes queries over JDBC and runs on both sides; catalog federation runs only on Databricks compute against storage. The page recommends Lakeflow Connect when volume or latency matters. For writes or unsupported sources, use Spark data sources.

### Zerobus Ingest

- **Source:** https://docs.databricks.com/aws/en/ingestion/zerobus-overview and https://docs.databricks.com/aws/en/ingestion/zerobus-ingest (both pages fetched; zerobus-ingest page updated Oct 20, 2026, overview updated Oct 1, 2026)
- **Discovery gives:** a push API that writes directly to UC Delta tables with no message bus. A realistic landing path for task-capture events (desktop/browser) and high-volume application events. Lakeflow Zerobus tables (`system.lakeflow.zerobus_stream`, `zerobus_ingest`) make ingest observable.
- **Automation gives:** a path for Automations to publish outcome events back into the lakehouse as well as for capture.
- **Status:** ingesting into default-storage tables is **Public Preview** (per the ingest page, updated Oct 20, 2026). The C++ and C#/.NET SDKs are **Beta**. The overview page does not state an overall release status. Zerobus system tables are Beta.
- **Limits:** quotas and region availability are not on these pages; they are on a separate "Zerobus Ingest quotas" page and an "Ingestion availability" page, which were not fetched successfully. Authentication: service principal by default (needs "Workspace access" entitlement and `USE CATALOG`, `USE SCHEMA`, `MODIFY, SELECT` on the target table). Microsoft Entra ID federation via Python SDK 1.10.0+ or Rust SDK 2.11.0+ (other interfaces need manual token exchange). OAuth tokens expire every hour. Billing: Jobs Serverless SKU.

### Agent Bricks and Mosaic AI Agent Framework

- **Source:** https://docs.databricks.com/aws/en/agents/concepts/ (page fetched, updated Oct 13, 2026). The guessed `/generative-ai/agent-bricks/` path redirected to this content. The `/generative-ai/agent-framework/` path returned 404, and no fetched page confirmed the older Agent Framework API surface (ResponsesAgent, `agents.deploy`).
- **Discovery gives:** agents that investigate data and documents and label evidence, built on Agent Runtime and MLflow tracing.
- **Automation gives:** agents as Automations (hosted on Agent Runtime, which hosts agents "built with any framework or harness" on Databricks Apps-based infrastructure at a stable, authenticated endpoint). Agent Server (`DurableAgentServer`) provides durable runs with crash recovery. AgentKit adds durable execution.
- **Components per the page:** Agent Bricks CLI, Agent Runtime, Agent Server, AgentKit, Databricks Sandbox (isolated code execution, no network egress for the `system.ai.sandbox` MCP), agent memory and sessions (Lakebase-backed), Unity Gateway, MCP servers and tools, AI Search, MLflow Tracing/evaluation/monitoring.
- **Separate products named:** Genie Agents, Genie One, Omnigent (multi-agent harness). Legacy builders Knowledge Assistant and Supervisor Agent are **no longer recommended for new agents**.
- **Status, regions, limits:** **not stated** on the Agent Bricks concepts page. Agent types listed in older material could not be verified in this snapshot.

### Model Serving

- **Source:** https://docs.databricks.com/aws/en/machine-learning/model-serving/ (page fetched, updated Oct 6, 2026)
- **Discovery gives:** hosting of classification/extraction models that turn unstructured text (emails, tickets) into structured signals. Hosts Databricks-hosted and external foundation models.
- **Automation gives:** endpoints for agents and custom models behind Automations. Serverless. Foundation Model API is pay-per-token or provisioned throughput.
- **Status:** labels not stated on the page. Claims: over 25K QPS and under 50 ms overhead latency (page claim, not independently verified).
- **Limits:** payload and rate limits and regions are on a separate page not fetched. Container build logs kept up to 30 days; metrics up to 14 days. Foundation Model API inputs and outputs may be stored up to 30 days for abuse detection. Partner retention terms apply to some externally hosted models.

### MLflow (tracing and evaluation)

- **Source:** https://docs.databricks.com/aws/en/mlflow3/genai/tracing/ (page fetched, updated Oct 8, 2026)
- **Discovery gives:** trace records of each agent step (inputs, outputs, latency, token usage, cost), storable in UC Delta tables with SQL access and standard UC permissions.
- **Automation gives:** the comparison record needed for shadow mode. Traces from the Automation can be compared against recorded human outputs or against an evaluation set. Evaluation with LLM judges, custom scorers, and expert feedback is referenced on the Agent Bricks page.
- **Status:** not stated on the tracing page. System tables `mlflow.experiments_latest`, `runs_latest`, `run_metrics_history` are Public Preview.
- **Limits:** no per-experiment limit for UC storage (the only limit mentioned). Other limits not stated.

### Managed and external MCP servers

- **Source:** https://docs.databricks.com/aws/en/generative-ai/mcp/ (page fetched; it lists managed servers as Genie One, Databricks SQL, GitHub, Slack, and does not give status) and https://docs.databricks.com/aws/en/generative-ai/mcp/managed-mcp (page fetched; this sub-page holds the Unity Gateway `system.ai` MCP table and the workspace endpoint table). The two pages are not consistent on the list of managed servers, so treat the managed-mcp page as the more specific source and re-check it before building.
- **Discovery gives:** natural-language access to governed data (Genie One MCP) and to Databricks SQL, and access to Slack, GitHub, Atlassian, Google Drive, Gmail, Calendar, Microsoft 365 via Unity Gateway external-service MCPs (`system.ai.*`). Unity Gateway MCP status:
  - `system.ai.genie_one_mcp`: **GA**
  - `system.ai.dbsql`: **Beta** (runs SQL with caller's permissions)
  - `system.ai.web_search`: **Beta** (excluded for HIPAA/BAA workspaces unless cross-Geo allowed)
  - `system.ai.sandbox`: **Beta** (no network egress)
  - External-service MCPs: status not stated. GitHub reads only public repos by default; private repos and write actions need the Databricks GitHub connector app.
- **Automation gives:** tool calls from agents to external systems (writes included for some services, subject to the connector app). Workspace-level managed endpoints (genie, ai-search, sql, unity-catalog functions) are **Public Preview** and require admin enablement, and they **bypass Unity Gateway** (no MCP-level grants, policies, guardrails, or tracing tables).
- **External/custom MCP:** the MCP page describes self-hosted or third-party servers added through an MCPs UI, and custom tools via Databricks Apps templates. The managed-MCP page does not describe UC connections for third-party MCP servers. Treat that as unverified.
- **Limits:** Beta MCPs require account admin to enable the Unity Gateway beta.

### Lakebase

- **Source:** https://docs.databricks.com/aws/en/oltp/ (page fetched, updated Oct 9, 2026)
- **Discovery gives:** a Postgres store for Discovery state: Candidate records, interview notes, approvals, shadow-mode run results.
- **Automation gives:** low-latency state for Automations and agent memory (also an agent memory backend). Synced UC tables can serve apps.
- **Status:** only "Store Postgres changes in the lakehouse" is labeled **Public Preview**. Lakebase as a whole is not labeled on the page. Available on Neon.com as well.
- **Limits:** regions on a separate page not fetched. Key limits (size, connections, throughput) not stated. Restore history window length not stated.

### Genie

- **Source:** https://docs.databricks.com/aws/en/genie/ (page fetched, updated Oct 27, 2026)
- **Discovery gives:** natural-language query over governed data for process owners and the CoE to explore evidence. Genie Agents can be configured with verified answers, metrics, business rules.
- **Automation gives:** a user-facing answer layer. Genie One is a simplified business-user interface; Genie Code is an AI coding assistant. Genie spaces are the managed-MCP endpoint `/api/2.0/mcp/genie/{genie_space_id}` (Public Preview endpoint).
- **Status:** not labeled on the page. Genie One and Genie Agents usage by users is free through **Jan 31, 2027**; service-principal usage is excluded and billed. Genie Code pay-as-you-go billing starts **July 8, 2026**.
- **Limits:** table counts and source limits not stated.

### Jobs and Lakeflow Spark Declarative Pipelines

- **Source:** https://docs.databricks.com/aws/en/ldp/ (page fetched, updated Oct 22, 2026); job system tables (see above).
- **Discovery gives:** scheduled extraction and transformation of evidence (for example, building the event log tables from system tables and business data).
- **Automation gives:** batch Automations (Jobs) and streaming/declarative Automations (pipelines); Jobs supports `WORKFLOW_RUN` and `SUBMIT_RUN` run types.
- **Status:** not stated on the pipelines page. Pipeline event log preview is Beta.
- **Sources supported:** S3, ADLS Gen2, GCS; Kafka, Kinesis, Pub/Sub, Event Hub, Pulsar.
- **Limits:** pipeline limitations page linked, not fetched. Job-table limits: many columns empty for rows before early December 2025; timeline slicing changed Jan 19, 2026; saving a job without changes creates no row.

### Apps

- **Source:** https://docs.databricks.com/aws/en/dev-tools/databricks-apps/ (page fetched)
- **Discovery gives:** process-owner interview forms, evidence review screens, Candidate triage.
- **Automation gives:** the human approval and cutover UI, plus the operator dashboard for shadow-mode results.
- **Runtime:** serverless; Python (Streamlit, Dash, Gradio) and Node.js (React, Angular, Svelte, Express). Auth via OAuth; UC for governance.
- **Status:** not stated. Billing per hour of provisioned compute while running. Per-workspace app count limit is in a separate page not fetched. Authorization (user auth / OBO) is referenced but not described on the fetched page.

### Unity Catalog governance (including ABAC)

- **Source:** https://docs.databricks.com/aws/en/data-governance/unity-catalog/abac/ (page fetched, updated Oct 22, 2026)
- **Governance gives:** UC securables for Candidates and Automations (tables, functions, models, MCP servers). ABAC uses governed tags to drive row filters, column masks, and GRANT/DENY policies.
- **Status:** ABAC overall not labeled. **Beta**: metastore-level policies, policies on views, DENY policies.
- **Limits:** compute requirements and policy quotas exist (specifics not on the page); view support is Beta; GRANT/DENY apply only to supported securable types.

### Standards (not Databricks)

- **OCEL 2.0 (Object-Centric Event Log):** specification dated 16 Oct 2023, arXiv:2403.01975 (submitted Mar 2024). Core data: events, objects, event-to-object and object-to-object relations, qualifiers, dynamic object attributes. Exchange formats in 2.0: JSON, XML, SQLite. CSV/Parquet came in 2.1. The ocel-standard.org `/2.0/` PDF currently serves a document labeled 2.1.0pre5 dated 1 Oct 2026, so cite the archived 2.0 version when relying on 2.0 specifics.
  - Source: https://arxiv.org/abs/2403.01975 ; https://www.ocel-standard.org/2.0/ocel20_specification.pdf (the PDF, not the HTML path, is what resolved)
  - Not verified here, but relevant to ticket #3: PM4Py's OCEL support and PM4Py's licence (AGPL implications) are owned by #3.

## Mapping to the Discovery and Automation split

| Block | Discovery | Automation | Notable limit |
|---|---|---|---|
| System tables | Activity, lineage, cost evidence | Baseline cost, outcome verification | ~1 h lag, 365 d, regional, schema drift |
| Lakeflow Connect | Brings ERP/CRM/ITSM data in | Source for Automations | Per-connector status unverified; no incremental on some tables |
| Lakehouse Federation | Read access to external DBs | Read-side input | Read-only |
| Zerobus | Capture landing (task/desktop events) | Outcome events | Default-storage ingest Public Preview; SDK Beta; quotas not on page |
| Agent Bricks / Agent Runtime | Investigation agents | Agent-type Automations | Status/regions not stated; legacy builders deprecated |
| Model Serving | Text classification endpoints | Endpoints for agents/models | Limits on separate page |
| MLflow | Trace records | Shadow-mode comparison | Status not stated |
| MCP (Unity Gateway) | Access to Genie, SQL, Slack, GitHub etc. | External write path via connectors | Beta MCPs need admin beta; workspace MCP endpoints bypass Unity Gateway |
| Lakebase | Discovery state store | Automation state, agent memory | Only one feature Public Preview; limits not stated |
| Genie | NL exploration for process owners | User-facing answers | Status not labeled |
| Jobs / SDP | Evidence pipelines | Batch and streaming Automations | Pipeline limits not on page |
| Apps | Interview and review UIs | Approval/cutover UI | Status not stated |
| Unity Catalog / ABAC | Governance of Candidates and Automations | Governs Automation access | ABAC Beta in parts |

## Follow-up research (not resolved here)

These are the items to verify before any build decision that depends on them. Each one should become its own research ticket or be checked in the relevant spec ticket.

1. Agent Bricks and Mosaic AI Agent Framework: current status labels, supported agent types, regions, and whether the older Agent Framework API (ResponsesAgent, `agents.deploy`) is still the recommended path.
2. Lakeflow Connect per-connector status and regions, including Salesforce, ServiceNow, and any ITSM connector, and the release-types page (returned 404 in this snapshot).
3. Zerobus quotas and ingestion availability pages (not fetched successfully): rates, payload size, regions.
4. Lakehouse Federation GA/Preview status.
5. MLflow tracing status (GA or preview) and the OTel storage path.
6. Model Serving limits and regions page.
7. Lakebase regions and limits.
8. Apps resource limits and user authorization (OBO) details.
9. Whether an official shadow-mode or traffic-split feature exists on Model Serving or Agent Runtime (none found in the docs read here).
10. External-service MCP status (`system.ai.slack`, `system.ai.github`, and others) and whether third-party MCP servers can be registered through UC connections.

## Sources

- System tables index: https://docs.databricks.com/aws/en/admin/system-tables/
- Audit log system table (Public Preview): https://docs.databricks.com/aws/en/admin/system-tables/audit-logs
- Jobs system tables: https://docs.databricks.com/aws/en/admin/system-tables/jobs
- Query history system table: https://docs.databricks.com/aws/en/admin/system-tables/query-history
- Lineage system tables: https://docs.databricks.com/aws/en/admin/system-tables/lineage
- Zerobus Ingest overview (updated Oct 1, 2026): https://docs.databricks.com/aws/en/ingestion/zerobus-overview
- Zerobus Ingest (updated Oct 20, 2026): https://docs.databricks.com/aws/en/ingestion/zerobus-ingest
- Lakeflow Connect (overview): https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/
- Lakehouse Federation: https://docs.databricks.com/aws/en/query-federation/
- Agent Bricks concepts (updated Oct 13, 2026): https://docs.databricks.com/aws/en/agents/concepts/
- Model Serving (updated Oct 6, 2026): https://docs.databricks.com/aws/en/machine-learning/model-serving/
- MLflow 3 GenAI tracing (updated Oct 8, 2026): https://docs.databricks.com/aws/en/mlflow3/genai/tracing/
- MCP overview (summarizing fetch; list of managed servers differs from managed-mcp page): https://docs.databricks.com/aws/en/generative-ai/mcp/
- Managed MCP and Unity Gateway MCPs: https://docs.databricks.com/aws/en/generative-ai/mcp/managed-mcp
- Lakebase (updated Oct 9, 2026): https://docs.databricks.com/aws/en/oltp/
- Genie (updated Oct 27, 2026): https://docs.databricks.com/aws/en/genie/
- Lakeflow Spark Declarative Pipelines (updated Oct 22, 2026): https://docs.databricks.com/aws/en/ldp/
- Databricks Apps: https://docs.databricks.com/aws/en/dev-tools/databricks-apps/
- Unity Catalog ABAC (updated Oct 22, 2026): https://docs.databricks.com/aws/en/data-governance/unity-catalog/abac/
- OCEL 2.0 paper (arXiv): https://arxiv.org/abs/2403.01975
- OCEL 2.0 specification PDF: https://www.ocel-standard.org/2.0/ocel20_specification.pdf
