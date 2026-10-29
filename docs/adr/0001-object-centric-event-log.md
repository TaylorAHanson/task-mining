# Object-centric Event Log, not a case-centric one

Every Discovery source normalises into one object-centric Event Log per deployment, shaped to map losslessly onto OCEL 2.0 (Events, Objects, event-to-object and object-to-object relations, object attribute history), rather than the case-centric, one-Case-per-Event log most process-mining tools ingest. Lineage, job runs, Task Executions and cross-system business documents are many-to-many, and picking a single Case per Event would throw away the relations Discovery most needs; a Case is therefore a view chosen at analysis time, and XES is produced only as a derived, flattened export.

## Consequences

- Algorithms that only understand flat logs get a flattened projection per chosen Case Object type, so results can differ by projection.
- OCEL 2.1 (CSV/Parquet exchange) was pre-release when this was decided; the schema targets 2.0 and adds 2.1 export once it is final.
