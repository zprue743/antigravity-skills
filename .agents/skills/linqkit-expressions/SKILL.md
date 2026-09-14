---
name: linqkit-expressions
description: Builds, composes, refactors, and debugs C# expression predicates, projections, and selectors with LINQKit. Discovers existing composition APIs before custom visitors and verifies expression semantics separately from EF/query-provider translation. Use for LINQKit expression reuse, dynamic filtering, and related query failures.
---

# LINQKit expressions

## Establish the execution boundary

Identify the intended result, expression inputs/outputs, and where execution occurs: compiled delegates over objects, a remote `IQueryable` provider, or an intentional combination. Read the relevant callers as well as expression definitions.

Inspect the installed LINQKit package/version, .NET target, EF/EF Core version if present, database provider, and query configuration. Choose documentation/source matching that version; current source is a reference, not proof of older behavior. Do not add an EF-specific LINQKit package to an in-memory-only library or upgrade dependencies merely to compose expressions.

Use the user's contract and official guidance as the quality standard. Existing repository helpers may reveal integration constraints but are not automatically correct patterns. Inspect whether they compile, expand, enumerate, or change providers before reusing them.

Before writing a custom visitor, manual parameter replacement, or duplicate selector, check LINQKit's existing composition APIs. Explain the specific unsupported transformation before introducing a custom implementation. Do not force LINQKit into a simple query that ordinary LINQ already expresses clearly.

Source: [LINQKit documentation and package compatibility](https://github.com/scottksmith95/LINQKit).

## Preserve expressions until execution

Use `Expression<Func<T, bool>>` for reusable predicates and `Expression<Func<T, TResult>>` for projections/selectors intended for provider translation. A `Func` is executable code, not an expression tree the provider can inspect.

Use `Invoke` to compose compatible expressions. `Expand()` operates on an expression; `AsExpandable()` wraps an `IQueryable` to expand supported expression uses before translation. Identify any expansion already configured by the application and choose a deliberate boundary instead of scattering expansion calls everywhere.

Do not use `Compile()`, `AsEnumerable()`, or early materialization to conceal a translation failure. LINQKit can strip specific `Compile()` patterns inside an expandable expression, as its documentation demonstrates; that does not make an independently compiled delegate translatable. Use such patterns only when required and verified for the installed stack. Compiling a finished expression for an in-memory semantic test is appropriate.

Prevent self-reference when composing captured expression variables. Keep the previous expression in a distinct variable or use supported combinators; do not assign a lambda that invokes the same variable being reassigned. Recursive expansion is not supported.

Source: [LINQKit composition and expansion](https://github.com/scottksmith95/LINQKit#combining-expressions).

## Build predicates with explicit semantics

- Define what zero filters means before choosing a starting value: match all, match none, or reject the request. Test zero, one, and multiple conditions. An empty `ExpressionStarter` uses its fallback expression; its first `And`/`Or` starts the predicate rather than retaining that fallback as a permanent clause.
- Create a fresh starter per query construction. Do not share a mutable builder across requests or threads.
- Distinguish starter mutation from plain expression composition. For a plain expression, keep the returned result, for example `predicate = predicate.And(condition)`; calling the extension and discarding its result does not change the original tree.
- Group mixed logic deliberately. Build an OR group separately before AND-ing it with mandatory constraints. Never allow optional search branches to bypass required tenant, authorization, or visibility filters.
- Capture stable per-query values. Avoid capturing changing loop variables, mutable request objects, services, or a context in expressions reused beyond their lifetime.
- Verify the installed implementation rather than assuming every `And`/`Or` leaves invocation nodes. Modern implementations may rebind parameters directly; determine expansion needs from the actual composed expression.

Read [composition examples](resources/composition-examples.md) when implementing grouped optional filters or reusable projections.

Sources: [ExpressionStarter implementation](https://github.com/scottksmith95/LINQKit/blob/master/src/LinqKit.Core/ExpressionStarter.cs), [PredicateBuilder implementation](https://github.com/scottksmith95/LINQKit/blob/master/src/LinqKit.Core/PredicateBuilder.cs).

## Compose projections and selectors

Keep reusable mappings as typed expressions. Invoke them within an outer projection or pass a compatible selector directly to `Select`. Check input/output types at every composition boundary; do not replace typed selectors with `dynamic`, `object`, or broad casts merely to join them.

Define optional-navigation behavior explicitly: null DTO, null field, fallback value, or omitted child. Guard the appropriate member access. For nested collections, verify ordering, filtering, empty results, and materialization against the provider. Do not assume `Include` is necessary for every DTO projection, or that expanding a nested selector proves its query shape is supported.

Distinguish composing a selector with a predicate from merging two DTO initializers. LINQKit is not a general mapping engine that automatically reconciles conflicting member assignments. Prefer a clear outer projection or an existing supported mapping path; add a custom merger only for a demonstrated requirement.

Keep expression factories deterministic and without query execution or side effects. Expansion can evaluate expression-valued members while resolving the tree. Do not hide database calls or per-row service calls in selector factories.

Source: [LINQKit expression expansion implementation](https://github.com/scottksmith95/LINQKit/blob/master/src/LinqKit.Core/ExpressionExpander.cs).

## Diagnose provider translation separately

Expansion substitutes expressions; the resulting operations still need provider support. Reduce failures to the smallest query retaining the failing composition. Inspect the expanded tree, then identify unsupported methods, overloads, conversions, grouping, collection shapes, or navigation operations.

Keep the query provider intact through filtering, ordering, projection, and pagination. If client execution is intentional, explain where it starts and bound the data transferred. EF Core can evaluate some top-level projection work on the client while rejecting unsupported expressions elsewhere; a successful query is not proof that every computation ran in SQL.

Check null semantics, collation/case sensitivity, date functions, and numeric behavior against the target database. A CLR null guard or string comparison may not behave identically in SQL. Do not change global query semantics to make one test pass without considering the application contract.

Sources: [EF Core client/server evaluation](https://learn.microsoft.com/en-us/ef/core/querying/client-eval), [SQL null semantics](https://learn.microsoft.com/en-us/ef/core/querying/null-comparisons).

## Check performance without changing meaning

Inspect representative generated SQL, executed command counts, and query plans where needed. Look for repeated correlated subqueries, duplicated expensive selector bodies, over-fetching, and premature materialization. Reusing an expression in C# does not guarantee a database computes it only once.

Keep variable values parameterizable when constructing dynamic trees. Avoid embedding changing values as fresh constant nodes without checking query-cache behavior. Do not claim expansion optimizes the SQL automatically or introduce EF compiled queries without checking their restrictions for the dynamic query shape.

Source: [EF Core dynamic-query performance](https://learn.microsoft.com/en-us/ef/core/performance/advanced-performance-topics).

## Verify and deliver

1. Build/type-check with the installed stack. Verify the intended `Queryable` versus `Enumerable` overloads and expansion boundary.
2. Test expression semantics using representative objects and independent expected results. Cover required constraints, empty filters, mixed AND/OR logic, and relevant null/collection boundaries. Avoid relying on exact `Expression.ToString()` formatting.
3. For remote queries, execute representative cases against the actual provider in a test environment. Inspect SQL where useful; query-string generation alone is not an execution test. In-memory queryables and EF InMemory do not prove relational translation.
4. For a regression, establish a failure for the reported behavior before fixing it. Preserve filter scope and results while simplifying composition; remove only helpers made unnecessary by the change.
5. Report files, installed versions/provider, behavior covered, commands/results, and unverified translation/performance claims. If database access is unavailable, complete semantic checks and clearly leave provider verification outstanding.

Source: [EF Core testing strategy](https://learn.microsoft.com/en-us/ef/core/testing/choosing-a-testing-strategy).
