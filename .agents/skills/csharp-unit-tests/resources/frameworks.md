# Framework-specific guidance

Read only the section for the framework in the target project. Use its installed version's APIs and assertion semantics; do not mix attributes or assertions from multiple frameworks in one test project.

## xUnit

Use `[Fact]` for an individual scenario and `[Theory]` with appropriate data attributes for variations. Await task-returning async assertions. Choose exact-type versus assignable exception assertions intentionally.

A test-class instance is created per test, but class/collection fixtures share their instances. Keep mutable per-test state out of shared fixtures. Use constructor setup for synchronous work and supported lifecycle interfaces for asynchronous setup/cleanup. xUnit v2 and v3 have different asynchronous lifecycle signatures; do not copy one version's implementation into the other.

Sources: [xUnit getting started](https://xunit.net/docs/getting-started/v3/getting-started), [shared context](https://xunit.net/docs/shared-context).

## NUnit

Use `[Test]` or `[TestCase]`/`[TestCaseSource]` and constraint assertions such as `Assert.That`. Prefer `async Task` methods for asynchronous scenarios.

NUnit's `Assert.ThrowsAsync<T>` accepts an async delegate but returns the exception synchronously; do not blindly await its return value. Check the installed version before adopting alternative async assertion APIs.

Fixtures normally share one instance. Recreate mutable state in `[SetUp]`, or use supported `FixtureLifeCycle(LifeCycle.InstancePerTestCase)` when suitable. With that lifecycle, one-time setup/teardown must be static. Per-case instances do not isolate static fields or external resources.

Sources: [fixture lifecycle](https://docs.nunit.org/articles/nunit/writing-tests/attributes/fixturelifecycle.html), [async exception assertions](https://docs.nunit.org/articles/nunit/writing-tests/assertions/classic-assertions/Assert.ThrowsAsync.html).

## MSTest

Use `[TestClass]` and `[TestMethod]`; add `[DataRow]` or `[DynamicData]` for parameterized scenarios. Use per-test initialization/cleanup for mutable state and the installed version's supported async lifecycle methods.

Prefer public test methods and `async Task` for broad compatibility. Await task-returning async assertions. Exception assertion names and exact/derived-type behavior vary across versions; verify the chosen API rather than translating names from xUnit or NUnit. Prefer assertions scoped to the operation expected to throw instead of method-wide exception attributes.

Source: [MSTest authoring guidance](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-mstest-writing-tests).
