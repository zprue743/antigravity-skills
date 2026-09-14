# Scheduling diagnostics

Use for unexpected dates, dependencies, duration/effort changes, or calendar behavior.

## Trace the calculation inputs

Capture the event, its assignments/resources, dependencies, applicable calendars, project settings, and the edit that triggers the issue. Reduce to a few linked events and resources while preserving the failing rule. Compare input values with results after engine completion.

Check automatic/manual scheduling, constraint type/date, dependency direction/type/lag, duration/unit, and any supported effort-related mode. Verify the installed Scheduler Pro mode set; do not assume every Gantt scheduling mode is available. Determine which values the user controls and which the engine derives before proposing a change.

Prefer documented operations such as `setStartDate`, `setEndDate`, or `setDuration` when they express the edit. Verify their parameter semantics, including whether duration is preserved. Await supported async setters or explicitly await project calculation after store changes. Do not use sleeps or repeated forced refreshes as scheduling synchronization.

Source: [Scheduler Pro EventModel](https://bryntum.com/products/schedulerpro/docs/api/SchedulerPro/model/EventModel).

## Distinguish calendars from display settings

Project calendars supply default availability; events and resources may override them. Inspect the applicable working intervals, inheritance, exceptions, and how assigned resources' calendars participate. Do not assume the default calendar is a weekday business calendar.

Working duration is not necessarily elapsed end-minus-start time. Project conversion settings such as `hoursPerDay` convert duration units; they do not create working shifts. Confirm availability independently, and test overnight shifts, holidays, and daylight-saving boundaries when relevant.

Time-axis filtering such as `workingTime`, non-working-time shading, and the visible date range are presentation settings. Hiding weekend ticks does not by itself make weekends unavailable to the scheduling engine.

Use explicit date serialization and time-zone assumptions. Avoid locale-dependent parsing and ad hoc UTC/local offsets that compensate for an unexplained discrepancy.

Sources: [calendar rules](https://bryntum.com/products/gantt/docs/guide/SchedulerPro/basics/calendars) (shared SchedulerPro guide), [time-axis workingTime](https://bryntum.com/products/schedulerpro/docs/api/Scheduler/view/SchedulerBase#config-workingTime).

## Diagnose before overriding

Check unresolved links, circular dependencies, incompatible constraints, and calendars with insufficient working time. Inspect supported scheduling-conflict results/events. Do not silently suppress conflicts or repeatedly overwrite dates after each calculation.

Clarify whether the requirement is availability, overlap validation, utilization display, or automatic resource leveling. Verify the exact capability and version before promising automatic conflict-free resource allocation.

Validate the intended rule with a small before/after case, including a relevant boundary. Keep the expectation independent of the engine output under test.
