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

**Event**:
Something that happened at a point in time, related to one or more Objects.
_Avoid_: Log line, record (when meaning the occurrence)

**Object**:
A thing Events happen to or involve: a business document, a person, an application, a table.
_Avoid_: Entity, artifact

**Event Log**:
All Events and Objects Discovery knows about for one deployment; a Process is a view over it, not a partition of it.

**Case**:
The Object a Process view is centred on, e.g. an order, an access request, a monthly report.
_Avoid_: Ticket, instance, item

**Recorded / Derived / Inferred**:
How a fact in the Event Log is known: a system logged it; it was computed deterministically from recorded facts; or an agent or model concluded it, with a confidence.

**Claim**:
A statement by a person or document about how work is done. Claims explain and cross-check the Event Log but are never Events.
_Avoid_: Fact, finding

**Task**:
A single step a person performs within a Process.
_Avoid_: Step, action, activity (when meaning human work)

**Interaction**:
One user-interface action within a Task, such as focusing a window or clicking a control. Interactions say what kind of thing was done, never what was typed or shown.
_Avoid_: Click, keystroke, UI event

**Task Execution**:
One person's performance of one Task, made up of their Interactions; the bridge between what a person did on screen and the business Objects it affected.

**Activity**:
The business-meaningful name of what an Event did, drawn from the deployment's Activity Vocabulary, as distinct from the raw name the source system used.

**Activity Vocabulary**:
The set of Activities a deployment recognises, proposed by Discovery and editable by people.

**Manual Process**:
A Process in which at least one Task is done by a human that a machine plausibly could do.

### Automation

**Automation Candidate**:
A described Manual Process together with its evidence and estimated value; the handoff from Discovery to Automation.
_Avoid_: Opportunity, use case

**Automation**:
The Databricks-native thing that replaces some or all of a Manual Process's human Tasks.
_Avoid_: Bot, agent (unless it literally is one)

### Configuration and scenarios

**Enterprise Profile**:
The configured description of the enterprise the asset serves: its name, industry, shape and systems.
_Avoid_: Reference enterprise, tenant, customer config

**Scenario**:
An Enterprise Profile plus a set of Seeded Processes, from which synthetic data is generated.
_Avoid_: Demo world, fixture

**Seeded Process**:
A Manual Process or Decoy deliberately planted in a Scenario's data, with ground truth to score against.

**Decoy**:
A Seeded Process that looks automatable but should not be flagged, or should be flagged to keep human.
