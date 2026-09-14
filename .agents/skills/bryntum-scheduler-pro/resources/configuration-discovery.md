# Configuration discovery and interaction checks

Use before adding behavior, changing configuration, or diagnosing an option that appears ineffective. The purpose is to discover existing capabilities and detect interacting settings before writing workarounds. This file is a workflow, not a complete or version-independent option catalog.

## 1. Search the behavior, not just a guessed option name

Translate the request into observable acceptance criteria. Examples: keep the visible time span fitted to the container; prevent event resizing; exclude non-working hours; customize one editor field. Include relevant conditions such as zooming, read-only records, multiple assignments, or remounting.

Search the exact installed Scheduler Pro version using:

1. An available Bryntum documentation MCP connection, filtered by product and version where supported. Check the tool's schema for accepted product identifiers and available versions.
2. The official API browser, matching guides, and version-matched examples. Search synonyms and the names of the owning class/feature; follow related-member links and inherited members.
3. Installed TypeScript declarations and shipped documentation/examples. Use `rg` to find candidate names and follow config interfaces/base classes. Declarations establish types, but may omit defaults and interactions; supplement them with explanatory docs.

If one source is inaccessible or only returns the documentation landing page, use another. Do not infer API support from that landing page. State a version mismatch or unresolved documentation gap rather than treating current docs as exact evidence for an older installation.

Bryntum's MCP supports version-specific documentation searches, including configuration and examples. It is useful but optional; this skill must still work without it. Do not install or configure a server merely to perform a lookup.

Source: [Bryntum documentation MCP](https://bryntum.com/products/grid/docs-llm/guide/Grid/ai-features/mcp-server.md).

## 2. Inspect the correct configuration owner

Use the documented inheritance/mixin tree rather than assuming every option lives on SchedulerPro itself.

| Requested behavior | Places to inspect |
| --- | --- |
| Timeline sizing, zoom, visible dates, ticks | SchedulerPro's inherited Scheduler/Timeline APIs, view presets, time-axis settings |
| Rows, columns, sizing, scrolling, selection | Inherited Grid/Widget APIs and the relevant column type |
| Drag/drop, resize, editing, menus, tooltips | The corresponding feature class and its nested configuration |
| Computed dates, effort, availability, dependencies | ProjectModel, event/resource/assignment models, calendars, scheduling mode |
| Loading, saving, custom fields | Project CRUD configuration, stores, model fields, backend protocol |
| A setting lost during framework updates | Wrapper props/config types, native instance APIs, state ownership |

For each viable option, inspect the whole member description: default, allowed values, inherited overrides, prerequisites, warnings, exclusions, deprecation/version notes, and whether it is initialization-only or mutable at runtime. Read referenced options as well. Verify the native path and wrapper spelling instead of assuming they are identical.

Rank solutions: supported config/feature, then a documented extension point for any remaining gap, then custom implementation. A built-in feature that violates another acceptance criterion is not a complete solution. Explain the tradeoff instead of silently dropping that criterion.

## 3. Audit effective configuration

Trace only settings relevant to the requested behavior through application defaults, shared factories, object spreads, subclasses, feature defaults, wrapper bindings, responsive/state restoration, and runtime setters/listeners. Read resolved public properties after initialization when practical; the source config is not necessarily the active value.

Record a compact evidence table in the task response or an existing design note:

| Requirement | Option and owner | Effective value and origin | Interaction/prerequisite | Evidence and decision |
| --- | --- | --- | --- | --- |

Distinguish these failure modes:

- **Unsupported combination:** documentation explicitly rules out using two features together.
- **Precedence or replacement:** a later setting, restored state, or merge replaces an earlier value.
- **Wrong layer:** a display setting is being used to request scheduling behavior.
- **Unmet prerequisite:** a feature is disabled or the required model/data is absent.
- **Wrong lifecycle/API:** an initialization config is changed as if it were a runtime property.
- **Competing ownership:** a listener or framework watcher undoes the native behavior.

Do not guess nested merge semantics, disable unrelated settings, or introduce a second listener to fight the first. Report unknowns as unknowns and test the smallest suspected interaction.

## 4. Check documented examples of interactions

These are examples to recheck against the installed version, not an exhaustive conflict registry:

- `suppressFit: true` disables `forceFit`; `forceFit` also restricts zooming. Inspect both before implementing custom fit/zoom behavior. See [TimelineBase](https://bryntum.com/products/scheduler/docs-llm/api/Scheduler/view/TimelineBase.md), an inherited API to verify in Scheduler Pro.
- `workingTime` filters time-axis ticks and has documented zoom restrictions. It does not replace engine calendars. See [workingTime](https://bryntum.com/products/schedulerpro/docs/api/Scheduler/view/SchedulerBase#config-workingTime).
- `autoHeight: true` renders all rows instead of using normal row virtualization, with consequences for larger datasets. It is a performance tradeoff, not a universal error. See [inherited Scheduler/Grid configuration](https://bryntum.com/products/schedulerpro/docs/api/Scheduler/view/Scheduler#config-autoHeight).

## 5. Prove the choice

Try the smallest configuration change using representative data. Verify the requested behavior and the important adjacent behavior, such as fit plus zoom or editing plus read-only state. When resolving an interaction, compare each option individually and together if that isolates the cause.

Test initial mount and relevant runtime transitions: prop changes, preset changes, state restoration, data reload, or remount. Use a browser for layout and interaction; use engine tests for calculated scheduling. Passing TypeScript checks alone cannot establish behavioral compatibility.

Before custom code, summarize the native candidates, why they do not fully meet the requirement, the evidence checked, and the smallest remaining extension. If evidence is incomplete, label the workaround provisional and describe the unverified point. Never claim all configs were checked or no conflicts remain based solely on keyword search.
