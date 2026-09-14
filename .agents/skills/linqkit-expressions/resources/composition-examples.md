# Composition examples

Adapt these examples to the installed LINQKit version and domain model. They illustrate expression shape, not a promise that every database provider translates it. Imports used below are `LinqKit`, `System.Linq`, and `System.Linq.Expressions`.

## Mandatory filter plus an optional OR group

Assume `Product` has `TenantId` (int), `Name` (non-null string), and `Description` (nullable string). `terms` is an already normalized collection of nonempty search strings. The contract is: always restrict tenant; no terms means no additional search restriction; any term may match either text field.

```csharp
Expression<Func<Product, bool>> predicate = p => p.TenantId == tenantId;

if (terms.Count > 0)
{
    var matchesAny = PredicateBuilder.New<Product>(false);
    foreach (var term in terms)
    {
        var searchTerm = term;
        matchesAny.Or(p =>
            p.Name.Contains(searchTerm) ||
            (p.Description != null && p.Description.Contains(searchTerm)));
    }

    // Freeze the starter's expression before further composition.
    Expression<Func<Product, bool>> search = matchesAny;
    predicate = predicate.And(search);
}

var query = products.AsExpandable().Where(predicate);
```

Here `products` is an `IQueryable<Product>`. The expansion boundary is explicit; retain or omit it based on the actual expression and existing query setup. Validate the database's string comparison semantics separately.

Verify that a product from another tenant is excluded even when its text matches, and that an empty term list still enforces the tenant condition. Compare zero, one, and multiple terms.

## Reuse a selector inside a DTO projection

Assume `Product` has decimal `UnitPrice` and `DiscountRate`, plus `Id`; `ProductSummary` exposes compatible `Id` and decimal `NetPrice` properties. The example contract uses fractional discounts and leaves rounding unspecified; establish any real rounding rule before using it.

```csharp
Expression<Func<Product, decimal>> netPrice =
    p => p.UnitPrice * (1m - p.DiscountRate);

Expression<Func<Product, ProductSummary>> summary =
    p => new ProductSummary
    {
        Id = p.Id,
        NetPrice = netPrice.Invoke(p)
    };

var query = products.AsExpandable().Select(summary);
```

An alternative boundary is `products.Select(summary.Expand())` when no other expression use requires query-level expansion. Do not add both mechanically.

For an in-memory semantic check, compile the finished expanded expression: `summary.Expand().Compile()`. Test expected arithmetic independently. For database usage, execute the query and verify decimal behavior and selected columns with the real provider. Do not move `netPrice.Compile()` into a captured delegate and expect LINQKit to recover the lost expression body.
