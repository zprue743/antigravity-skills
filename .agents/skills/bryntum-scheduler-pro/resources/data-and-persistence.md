# Data and persistence

Use for loading, missing events, store synchronization, and backend round trips.

## Validate the linked dataset

Check events, resources, assignments, dependencies, and calendars together. IDs must be stable and references must resolve, including consistent numeric/string ID handling. Prefer explicit assignment records for multi-resource events; do not assume an event's single `resourceId` represents every assignment.

For a missing bar, inspect the event record, assignment link, resource visibility, filters, date range, and calculation state in that order. Distinguish an unassigned event from a deleted event. An assignment removal and an event deletion have different effects.

Choose one loading strategy. Coordinate linked stores through the project instead of racing independent loaders. Do not replace whole datasets on every reactive update and erase pending edits or selection. Verify custom field mapping, date conversion, and persistence flags.

Source: [displaying Scheduler Pro data](https://bryntum.com/products/schedulerpro/docs/guide/SchedulerPro/data/displayingdata).

## Choose one persistence owner

The project's built-in CRUD facilities can coordinate load/sync across stores. If the application already uses a custom API adapter, preserve that ownership and define the mapping explicitly. Avoid overlapping project auto-sync, per-store endpoints, and application watchers that submit the same edit twice.

For the CRUD protocol, verify the installed version's request/response contract: store sections, added/updated/removed records, success/error handling, and server IDs. Newly added records require phantom-to-server ID reconciliation, including dependent assignment links. Do not acknowledge persistence locally before the server accepts the edit.

Handle failed saves and edits made during an in-flight request deliberately. Serialize or reconcile competing changes; prevent old responses from overwriting newer state. A successful engine commit and a successful HTTP response are separate checks.

Test a create with assignments, update, unassign/delete, failure, and reload as applicable. Verify that IDs, links, and calculated dates survive the round trip.

Sources: [CRUD protocol](https://bryntum.com/products/schedulerpro/docs/guide/Scheduler/data/crud_manager_in_depth), [CRUD manager](https://www.bryntum.com/products/schedulerpro/docs/api/Scheduler/data/CrudManager). Use the Scheduler Pro project as the owner rather than instantiating an unrelated Scheduler CRUD manager from an example.
