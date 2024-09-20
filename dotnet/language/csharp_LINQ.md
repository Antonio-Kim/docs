### LINQ in C#

LINQ has several components, some of them are required, and some of them are optional when using LINQ:

- **Extension Methods (required)**: methods such as Where, OrderBy, and Select.
- **LINQ Providers (required)**: these include **LINQ to Objects** for processing in-memory objects, **LINQ to Entities** for processing data stored in external databases and modeled with EF Core, and \*_LINQ TO XML_ for processing data stored in XML
- **Lambda expression (optional)**: can be used instead of named methods to simplify LINQ queries
- **LINQ query comprehension syntax (optional)**: includes C# keywords like from, in, where, orderby, descending, and select. These are aliases for some of the LINQ extension methods

#### Common LINQ methods and examples

- Where: Filtering entities

```csharp
var query = names.Where(name => name.Length > 4);
```

- OrderBy: Sorting by a single Entity

```csharp
var query = names
	.Where(name => name.Length > 4)
	.OrderBy(name => name.Length);
```

- ThenBy: subsequent sorting after OrderBy

```csharp
var query = names
	.Where(name => name.Length > 4)
	.OrderBy(name => name.Length)
	.ThenBy(name => name);
```
