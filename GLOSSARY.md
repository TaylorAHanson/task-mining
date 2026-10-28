# Process Discovery & Automation

Finding manual work wherever an enterprise leaves traces of it, and replacing it with automations native to the Databricks platform.

## Language

### Discovery

**Discovery**:
Finding and describing how work is actually done, from any source available.
_Avoid_: Mining (as the name of the whole effort), process intelligence

**Process Mining**:
A Discovery technique that reconstructs Processes from an event log of case-tagged, timestamped activities.

**Task Mining**:
A Discovery technique that reconstructs how Tasks are performed from user-interaction-level events.

**Process**:
A repeatable, end-to-end flow of work with an identifiable Case.
_Avoid_: Workflow (overloaded with orchestration), procedure

**Case**:
The single thing a Process instance works on, e.g. an order, an access request, a monthly report.
_Avoid_: Ticket, instance, item

**Task**:
A single step a person performs within a Process.
_Avoid_: Step, action, activity (when meaning human work)

**Manual Process**:
A Process in which at least one Task is done by a human that a machine plausibly could do.

### Automation

**Automation Candidate**:
A described Manual Process together with its evidence and estimated value; the handoff from Discovery to Automation.
_Avoid_: Opportunity, use case

**Automation**:
The Databricks-native thing that replaces some or all of a Manual Process's human Tasks.
_Avoid_: Bot, agent (unless it literally is one)
