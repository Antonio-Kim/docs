## GraphQL for ASP.NET Core

GraphQL is used on cases where you need to combine and expose your data as a service. It's protocol agnostic, and usually have only one endpoint for accessing and modifying data, which is `/graphql`.

There are various free, open-source libraries available since ASP.NET Core does not ship with native libraries for GraphQL. Two major libraries are:

- GraphQL.NET
- Hot Chocolate from ChiliCream

This guide will work with Hot Chocolate only. Install the following packages

```bash
dotnet add package HotChocolate.AspNetCore
dotnet add package HotChocolate.AspnetCore.Authorization
dotnet add package HotChoclate.Data.EntityFramework
```

There may be a chance that Data Attribute `[DefaultValue]` may cause some error when building project. Replace `using System.ComponentModel;` with `using DefaultValueAttribute = System.ComponentModel.DefaultValueAttribute;` instead.

### Main Concepts

GraphQL Schema defined how we want to expose data to clients that are requesting it, and how CRUD operations allowed. Schema compose one or more root operation types that allow clients to perform read, write, and subscription tasks:

- **Query**: Expose all possible data-retrieval queries that we want to make available to our client. Serves as centralized access layer of our data model, with one or more methods corresponding to the various ways to retrieve them
- **Mutation**: Allows client to perform insert, update, and/or dlete oeprations on our data model
- **Subscription**: Enables real-time messaging mechanism that clients can use to subscribe to various events and be notified of their occurrences

Out of all the root operations, Query must be present, whereas other operations are optional.

### Query Schema

Query Schema contains entry points that maps directly onto EF Core DBSets inside ApplicationDbContext. For example, one of the entry point would look like this:

```csharp
[Serial]
[UsePaging]
[UseProjection]
[UseFiltering]
[UseSorting]
public IQueryable<Apples> GetApples([Service] ApplicationDbContext context)
	=> context.Apples;
```

- Serial: Configures GraphQL in runtime to execute certain tasks in serial instead of parallel mode
- UsePaging: Add Paging middleware to GraphQL runtime
- UseProjection: Add dedicated middleware that project the incoming GraphQl queries to database queries through IQueryable objects
- UseFilter: Add filtering middleware that translate into LINQ queries and then to DBMS queries by GraphQL runtime
- UseSorting: Add sorting middleware
