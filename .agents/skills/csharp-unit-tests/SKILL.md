---
name: csharp-unit-tests
description: Writes and improves C#/.NET unit tests for domain logic, services, utilities, and regressions using xUnit, NUnit, or MSTest. Applies independent testing standards rather than copying repository patterns; identifies behavior that requires integration testing.
---

# C# unit tests

## Establish scope and compatibility

Read the requested behavior, production interfaces, relevant callers, and implementation. Derive expected outcomes from the contract; do not preserve a likely bug just because the code currently behaves that way. Clarify materially ambiguous requirements while continuing with clear cases.

Use this skill and authoritative documentation as the quality baseline. Existing tests may reveal discovery, setup, coverage, or failures, but are not templates for assertions, mocking, or fixture design. Evaluate any reused helper independently.

Inspect solution/project files, `global.json`, target frameworks, nullable settings, package references, central package management, and test configuration. Honor an explicitly requested framework. Otherwise retain the installed framework; for a new setup, use xUnit as this skill's default, subject to SDK/target compatibility. Do not migrate frameworks or upgrade production dependencies just to add tests.

Create a separate test project only when needed, reference the production project, and use the runner packages appropriate to the framework and installed SDK. Distinguish VSTest from Microsoft.Testing.Platform (MTP); their options and adapters differ. Read [framework guidance](resources/frameworks.md) for the selected framework before writing its tests.

Source: [dotnet test and runner selection](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-test).

## Design meaningful cases

- Identify the behavior, triggering inputs, expected result, and consequential boundary or failure cases before writing tests. Prefer fast, isolated, repeatable tests.
- Use descriptive names such as `Calculate_EmptyCart_ReturnsZero` and clear Arrange, Act, Assert sections. Keep one coherent behavior per test; related assertions may belong together.
- Assert public results, state transitions, or contractual effects. Do not use reflection to test private methods, expose internals solely for assertions, or mock the subject itself.
- Use minimal, explicit fixtures and independently determined expected values. Avoid duplicating the production algorithm in the expected result.
- Parameterize variations of the same rule. Split scenarios when setup or outcomes differ substantially; avoid conditional assertions and unrelated loops.
- Cover meaningful failures as well as success. Avoid trivial getter tests, broad snapshots, arbitrary case counts, and coverage-only assertions. Coverage locates gaps; it does not prove correctness.

Source: [Microsoft's unit-testing guidance](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices).

## Apply C# contracts precisely

Keep fixtures strongly typed and preserve compiler/analyzer checks. Test null, empty, whitespace, overflow, or invalid enums when the public contract makes them relevant. Nullable annotations do not enforce runtime validation; use `null!` only deliberately to exercise a null guard, with a clear explanation.

Choose equality assertions deliberately: reference identity, record/value equality, collection contents, ordering, and numeric tolerance express different contracts. Do not assert collection order unless promised. Use exact decimal expectations for contractual decimal arithmetic; choose justified tolerances for floating-point results.

Assert the intended exception type and meaningful properties such as `ParamName`. Avoid full framework-generated or localized message matching. Verify whether the selected assertion accepts derived exceptions. Cancellation may surface as an `OperationCanceledException` subtype, so match the cancellation contract rather than accidentally requiring an exact base type.

For deferred `IEnumerable` or `IAsyncEnumerable` behavior, enumerate within the assertion/action. Merely creating a sequence does not execute its deferred work. Avoid testing compiler-generated record behavior or framework internals unless application logic changes that behavior.

## Control async work, time, and cleanup

Write asynchronous tests as `async Task`; never `async void`, `.Wait()`, or `.Result`. Await production work and task-returning assertions. Follow the selected framework's actual exception API; an API named `ThrowsAsync` is not necessarily awaitable.

Test cancellation by explicitly controlling a token and the dependency's progress. For in-flight cancellation, signal that work has started before cancelling. Use a controlled task or `TaskCompletionSource<T>` with `RunContinuationsAsynchronously` when appropriate; avoid sleeps, polling races, and timing-based guesses. A bounded timeout can prevent a hang but is not the behavioral assertion. Cancel and observe background work during cleanup, including when assertions fail.

For clock-dependent behavior, prefer an injected `TimeProvider` and compatible `FakeTimeProvider`, or an existing clock abstraction. Advance fake time across the relevant boundary and assert the outcome. Fake time controls only code using that provider; it does not replace all system clocks or guarantee asynchronous continuations have finished.

Use fresh subjects, fixtures, and test doubles per case. Dispose owned resources with `using`, `await using`, or framework teardown. Restore any changed global state. Do not disable parallelism across the suite to hide shared state; isolate unavoidable shared resources at the narrowest supported scope.

Sources: [TimeProvider](https://learn.microsoft.com/en-us/dotnet/standard/datetime/timeprovider-overview), [framework guidance](resources/frameworks.md).

## Substitute external dependencies deliberately

Keep fast deterministic collaborators real. Use a small fake, stub, or compatible mocking library at an existing boundary for external I/O or nondeterminism. Avoid adding interfaces to every class or introducing a mocking library for a simple callback. If testability requires a production change, explain the smallest useful seam and make it only within the requested scope.

Configure dependency responses explicitly, including async faults. Verify calls only when the interaction itself is required, such as sending one message or forwarding a cancellation token. Avoid blanket verification of every call, irrelevant logging, or incidental call order.

For an HTTP client, substitute its handler/transport or an existing application abstraction. Do not issue live requests from unit tests. Assert request details and response/error handling that the application owns.

For EF Core, mock an existing application-level data boundary when testing business logic. Do not claim SQL translation, constraints, transactions, or production-provider semantics are verified by mocked `DbSet`, LINQ-to-Objects, or EF's InMemory provider. SQLite also differs from other providers. Flag those behaviors for integration tests against the relevant provider. Do not add a repository layer solely to avoid acknowledging that boundary.

A controller method tested directly does not exercise routing, middleware, filters, or model binding. Identify host-based checks as integration tests rather than inflating unit-test claims.

Source: [EF Core testing strategy](https://learn.microsoft.com/en-us/ef/core/testing/choosing-a-testing-strategy).

## Verify and report

1. Run the focused tests with the installed SDK and configured runner. Inspect local help before selecting filtering or coverage flags; VSTest and MTP syntax is not interchangeable. Confirm the intended tests were discovered and executed; zero tests is not success.
2. For a regression fix, demonstrate a failure caused by the reported behavior before changing production code. Do not weaken an expectation to make a bug pass. If implementation changes are outside scope, report the failing regression.
3. Build and run the affected test project(s), including relevant target frameworks. Investigate new compiler/analyzer warnings and failures. Expand to other affected projects when shared code or test infrastructure changes justify it.
4. Remove accidental skips, focus filters, debugging output, and leaked state. Report environmental or pre-existing failures separately. Do not use retries or broad exception handling to conceal them.
5. Summarize changed files, covered behaviors, exact commands/results, and any unexecuted checks or remaining integration-test needs. Do not claim execution based only on code review.

Consult official documentation for version-sensitive APIs. If it is unavailable, use locally verified APIs and state unresolved compatibility limits.
