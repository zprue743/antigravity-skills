---
name: vue-vitest-unit-tests
description: Writes and improves Vitest unit and component tests for TypeScript and Vue frontends using independent testing standards. Use for Vue components, composables, utilities, and regression tests; does not copy existing repository test patterns.
---

# Vue and TypeScript unit tests with Vitest

## Approach

Use this skill's standards and official documentation as the testing baseline. Existing tests, snapshots, helper factories, and mocking conventions are not templates or evidence of best practice. Inspect them only when necessary to diagnose failures, identify already-covered behavior, or understand test discovery and setup. Reuse a helper only after independently checking its isolation, types, and fidelity.

Read production code, public interfaces, requirements, and relevant callers to identify intended behavior. Do not turn an apparent implementation bug into the expected result. If a missing requirement changes the expected outcome materially, clarify it; continue with unambiguous cases.

Default to Vue 3 with Vue Test Utils 2 for new setups. Detect installed versions first; adapt to supported APIs without upgrading a frontend merely to write tests. This skill covers unit and focused component tests, not a full end-to-end or visual test suite.

## Establish the execution context

- Inspect package manifests, lockfiles, Vitest/Vite configuration, TypeScript configuration, and required plugins. Preserve working aliases, SFC transforms, test discovery, and package-manager selection. These are integration facts, not test-design standards.
- If setup is missing, add the smallest compatible setup: Vitest, Vue Test Utils, the Vue SFC plugin, and a DOM environment when needed. Check version requirements before selecting dependencies. A separate Vitest configuration must explicitly preserve the needed Vite settings; do not assume both configs are automatically combined.
- Prefer explicit imports from `vitest` and TypeScript `.test.ts` or `.spec.ts` files. Place files where the configured runner discovers them; do not copy assertion style from neighboring tests.
- Use Node for pure utilities and DOM-free composables. Use `jsdom` as the default for mounted Vue components; retain a compatible `happy-dom` setup when it supplies the required APIs. DOM emulators cannot establish layout or real browser fidelity. Identify those cases for browser testing instead of fabricating geometry or browser behavior to make assertions pass.

Sources: [Vitest setup](https://vitest.dev/guide/), [test environments](https://vitest.dev/guide/environment.html).

## Design tests from the contract

Before writing code, identify the relevant inputs/actions, observable outcomes, and meaningful failure or boundary cases. Choose cases by risk, not a fixed test count or coverage percentage.

- For utilities, test representative results and domain boundaries. Include invalid input only where runtime validation or an external-input contract requires it.
- For components, drive props, slots, and user interactions; assert rendered content, semantic attributes, emitted payloads, and externally observable effects. Avoid private refs, `wrapper.vm` internals, direct method calls, or `setData` to force a desired state.
- Prefer `mount` with real children when they participate in the behavior. Use selective stubs for unrelated expensive or platform-specific dependencies, preserving any props, slots, or events needed by the scenario. Do not default to blanket shallow mounting.
- Use meaningful element selectors: accessible roles/names where the chosen query API supports them, labels, or stable test IDs. Vue Test Utils does not provide Testing Library's `getByRole` API. Avoid CSS styling classes and positional selectors as behavioral contracts.
- Keep one coherent behavior per test, with explicit expected values and descriptive names. Several related assertions are appropriate. Avoid existence-only smoke tests and broad snapshots as primary evidence of correctness.

Source: [Vue testing guidance](https://vuejs.org/guide/scaling-up/testing).

## Vue reactivity and asynchronous behavior

- Await `trigger`, `setValue`, and `setProps`. Use `nextTick` for Vue updates after changing reactive inputs directly.
- Use `flushPromises` for promise callbacks outside Vue's update queue. It does not resolve deliberately pending promises or advance timers. For loading states, control a deferred dependency, assert the pending UI, then resolve/reject it and assert the outcome.
- Await promise assertions, including `.rejects`. Avoid sleeps, arbitrary repeated flushing, and retries that conceal races.
- Mount async `setup` components under `Suspense` and await their dependencies before asserting the resolved UI.
- Test independent composables directly through their returned public API. Use a minimal host component for lifecycle hooks or injection, provide required dependencies, and unmount it to exercise cleanup. Use an effect scope when needed to stop reactive effects created outside components.
- Unmount mounted wrappers with `enableAutoUnmount(afterEach)` or explicit teardown. Remove attached DOM containers and Teleport targets created by the test. Use fresh plugin/store/router instances when state is involved; choose real implementations when their behavior is part of the assertion.

Sources: [async behavior](https://test-utils.vuejs.org/guide/advanced/async-suspense.html), [composables](https://test-utils.vuejs.org/guide/advanced/reusability-composition.html), [Vue Test Utils API](https://test-utils.vuejs.org/api/).

## Mock boundaries and isolate state

Keep the subject under test real. Replace uncontrolled network, clock, randomness, or platform dependencies at a narrow boundary. Keep fast deterministic collaborators real. Assert dependency calls only when the interaction itself is part of the contract; also verify the resulting behavior where applicable.

Use `vi.fn` for injected callbacks and `vi.spyOn` to observe or replace object methods. For module mocks, respect `vi.mock` hoisting: create factory-local mocks or use `vi.hoisted` for shared factory state. Do not capture ordinary initialized top-level variables in a hoisted factory. `vi.mocked` adds typing; it does not create a runtime mock.

Create fresh fixtures per test. Clean up what each test changes:

- Clear mock call history when retaining implementations; reset mocks when discarding implementations and queued responses, then configure their defaults explicitly.
- Restore spies to their original methods. `vi.restoreAllMocks` does not undo module mocking or replace cleanup for standalone `vi.fn` mocks.
- Restore stubbed globals/environment variables using the corresponding Vitest cleanup APIs.
- Use fake timers only for time-dependent behavior. Advance the relevant interval, awaiting async timer APIs when callbacks involve promises. Unmount/dispose consumers, handle remaining timers deliberately, and restore real timers.

Do not run cases concurrently when they share module mocks, fake clocks, or global mutable state.

Sources: [Vitest mocking](https://vitest.dev/guide/mocking.html), [Vitest utilities and cleanup APIs](https://vitest.dev/api/vi.html).

## Preserve TypeScript guarantees

Use domain types for fixtures and function signatures for mocks. Prefer typed fixture builders and `satisfies` where supported. Avoid `any`, double casts, and error suppressions used just to force test compilation. If deliberately invalid external input requires a cast, isolate it at that boundary and explain the case.

Vitest runtime success does not prove type correctness. Ensure test files are included in an appropriate TypeScript project and run the project's Vue-aware type check (`vue-tsc` for SFCs) or an appropriate `tsc` check for pure TypeScript. Do not weaken compiler settings or widen production interfaces to expose internals for tests.

Sources: [Vue TypeScript guidance](https://vuejs.org/guide/typescript/overview), [TypeScript satisfies](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html).

## Verify and deliver

1. Run the focused files with the locally installed Vitest in non-watch mode (`vitest run` via the project's package manager). Confirm the intended cases actually ran; zero collected tests is not success.
2. For a regression fix, first demonstrate that the test fails for the reported behavior, not setup errors. Change production code only when fixing it is in scope. Never weaken assertions to accommodate a bug.
3. Run relevant type checks and lint checks, then the affected suite. If shared test configuration changed, run all suites affected by that change. Inspect uncovered branches when coverage is available; add meaningful cases rather than coverage-only assertions.
4. Remove focused tests, unintended skips, debug output, and leaked state. Investigate warnings and unhandled rejections instead of silencing them globally.
5. Report files changed, behaviors covered, commands and results, and any checks that could not run. Distinguish pre-existing failures from new failures. Do not claim passing tests from static review alone.

Consult the linked official docs for version-sensitive APIs. If documentation cannot be reached, use this baseline with locally verified APIs and state any unresolved compatibility limitation.
