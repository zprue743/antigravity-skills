---
name: bryntum-scheduler-pro
description: Implements, configures, debugs, and tests Bryntum Scheduler Pro integrations. Discovers built-in options before custom code and audits conflicting settings across inherited APIs, features, project data, and wrappers. Use for Scheduler Pro, including Vue/TypeScript; not generic scheduling algorithms or ordinary Scheduler tasks.
---

# Bryntum Scheduler Pro

## Establish the product and version

Read the requested outcome, package manifest/lockfile, imports, wrapper, project configuration, and relevant application code. Record the exact Scheduler Pro version and framework. Keep the current architecture unless the task requires a change.

Scheduler Pro combines a scheduler UI with a scheduling engine. Its assignments use an AssignmentStore, and dependencies can change calculated dates. Do not transfer assumptions from ordinary Scheduler's visual dependencies or from Gantt-only features without verifying support.

Use the installed package's types, matching official documentation, and version-matched examples to confirm API names, callback signatures, and feature support. Inspect inherited public APIs in the Scheduler Pro docs when needed. Treat `-next` documentation, old forum answers, and examples from other products as leads requiring verification. A docs page that only renders the overview does not establish an API signature.

Keep core, framework wrapper, themes, and any thin-package dependencies compatible. Do not mix full bundles from different Bryntum products or independently upgrade wrappers. Check the documented multi-product packaging strategy when applicable. Preserve the user's licensed/trial package choice and configured registry; do not copy credentials into code or add licensed distribution files to the repository.

Source: [Scheduler Pro documentation](https://bryntum.com/products/schedulerpro/docs/).

## Discover configuration before custom code

Before implementing or changing Scheduler Pro behavior, follow [configuration discovery and interaction checks](resources/configuration-discovery.md). Search by the desired outcome, including inherited configs, feature options, project/model settings, and wrapper mappings. Read defaults, prerequisites, exclusions, and runtime restrictions for each serious candidate.

Prefer an existing supported configuration or feature when it satisfies the requirement. Before introducing a custom listener, renderer, subclass, DOM workaround, or scheduling calculation, explain the native options considered and the specific remaining gap. An unsuccessful keyword search alone is not evidence that no option exists.

For configuration changes, audit the effective values and their origins, including defaults, config spreads, wrapper props, saved state, and runtime assignments. Check related options together and verify behavior after initialization and subsequent updates. Record documented conflicts separately from suspected application interactions; do not claim the audit covers every possible option combination.

## Choose the relevant workflow

- For dates, dependencies, calendars, or unexpected rescheduling, read [scheduling diagnostics](resources/scheduling.md).
- For imports, missing events, backend sync, or duplicate records, read [data and persistence](resources/data-and-persistence.md).
- For Vue/TypeScript mounting, reactivity, or lifecycle issues, read [Vue integration](resources/vue-integration.md).
- For other frameworks, consult the matching Scheduler Pro integration guide before using wrapper-specific APIs.

Identify the layer that owns the behavior: application logic, project data, scheduling engine, widget feature, framework wrapper, or backend. Reproduce the issue with the smallest representative dataset before changing multiple layers.

## Preserve project ownership

Use one authoritative ProjectModel for a related scheduling dataset. Its stores coordinate events, resources, assignments, dependencies, and calendars. Multiple views may deliberately share that project; do not create disconnected copies of its stores.

Apply edits through documented model/store APIs. Treat model instances as live objects, not plain DTOs: do not mutate internal data bags, spread/JSON-clone them into substitutes, or overwrite calculated fields to force a visual result. Define custom persisted fields with the supported model field system and register the model class with the correct store/project configuration.

Await engine completion before reading calculated results or serializing a completed schedule. `project.commitAsync()` completes engine work; it is not a server-save acknowledgement or a guarantee that browser layout is finished. Check supported conflict results/events as well as thrown errors.

Sources: [ProjectModel](https://bryntum.com/products/schedulerpro/docs/api/SchedulerPro/model/ProjectModel), [displaying data](https://bryntum.com/products/schedulerpro/docs/guide/SchedulerPro/data/displayingdata).

## Customize through supported UI APIs

Use documented columns, renderers, feature configuration, editor item overrides, and event listeners. Verify whether a setting is an initialization config, a writable runtime property, or a method; changing the original config object may not reconfigure a running instance.

Scheduler Pro uses the `taskEdit` feature for its task editor. Verify its editor item names and event hooks for the installed version instead of copying ordinary Scheduler editor configuration. Check the documented cancellation/finalization contract for asynchronous drag, resize, or edit validation; do not invent event names or assume all listeners accept promises.

Keep rendering callbacks synchronous, cheap, and free of store mutations or network calls. Use safe text output, supported DOM configuration, or appropriate HTML encoding for dynamic text. Avoid direct mutation of virtualized event/row DOM, cached element references, and brittle selectors into private widget markup.

For a blank or clipped widget, inspect theme loading, host dimensions, flex/grid sizing, visibility, and mount timing before changing scheduling data. Investigate performance separately in engine calculations, data loading, renderer cost, and layout; avoid rebuilding the entire project on routine UI updates.

Source: [Scheduler Pro customization example](https://bryntum.com/blog/how-to-integrate-a-react-ag-grid-with-a-react-bryntum-scheduler-pro/).

## Verify the application behavior

- Unit-test application mappers, validation, and decision logic independently. Do not mock the scheduling engine and then claim to have verified its calculations.
- Use real project/model instances for scheduling checks when the installed distribution supports that test environment. Fix dates, calendar rules, and time-zone assumptions; await calculations and assert outcomes.
- Use a real browser for rendering, drag/drop, resize, task-editor workflows, keyboard behavior, and virtualized scrolling. A DOM emulator cannot establish geometry or full widget behavior.
- Test persistence separately: create/update/delete, assignment links, save failures, and reload. UI success alone does not establish backend durability.
- Dispose fixtures and owned widgets/projects, detach listeners, and settle pending work. Test remounting or route changes when lifecycle behavior changed.

Run relevant build/type checks and focused tests. Report the version, files changed, observed behavior, commands/results, and remaining limitations. If packages, credentials, or a browser are unavailable, finish independent work and clearly identify unexecuted checks. Do not replace missing Bryntum APIs with invented implementations or claim runtime verification from static review.
