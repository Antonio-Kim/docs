## Implementing Database to your webapi using EF Core

To setup your database, please see the database_setup.md for more info. In this documentation we'll be using Sqlite as the database. **WARNING**: assumption here is that you are familiar with basic Database concepts, mainly relationship between tables like one-to-many and many-to-many.

Create a new folder named "Model" to the project folder. We'll start by working with Entities and work our way up.

### Entity creation

Here we'll assume that you know how to create a simple entity. What's important in this section is the attribute added to each properties. Here are some useful ones:

- [Table("<database name>")]: Without this attribute, EF core will use convention name for the table, which is the class name
- [Key]: setting this filed as primary key
- [Required]: will not accept null value
- [MaxLength(100)]: set maximum length of character
- [Precision(5,2)]: mostly for decimal, the first number indicate total length, and second number indicate decimal places. In this case, three digits before period, and two digits after period.

### Adding DbContext

Add a DBcontext file so that EF can utilize the the operations between ASP.NET and database. This will allow the app to perform basic SQL CRUD operations by using C#.

```csharp
public class ApplicationDbContext : DbContext
{
	public ApplicationDbContext(DbContextOptions options) : base(options) { }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
    }
}
```

The above code is typically how you would start out setting up the DbContext file. You might notice that most of the tutorials online does not include OnModelCreating. This is fine, as there are multiple ways to configure on initial. EF core has mainly three configuration methods:

1. Fluent API: this method takes precedences over other two methods. Highly configurable
2. Data Annotation: attributes added to entity classes. Overrides Convention method but gets overriden by Fluent API
3. Convention: naming conventioned used in EF core.

Majority of the apps use Fluent API and we'll continue using Fluent API instead.

### Many-to-many relationship implementation

Assuming you know how to setup an entity and what many-to-many relationship tables are, we'll start adding code into OnModelCreating as stated on Adding DbContext. Once you created all the Entities, we'll need to implement the junction tables. You'll need to bring in primary keys from the tables you reference into the junction table, but also implement Navigation properties into the class' properties. So say I have following entities:

```csharp
public class Apple {
	[Key]
	[Required]
	public int Id { get; set; }
	[Required]
	public string Description { get; set; } = null!;
}
public class Orange {
	[Key]
	[Required]
	public int Id { get; set; }
	[Required]
	public string Description { get; set; } = null!;
}
```

A silly example, I know, but say you have these two tables. You first need to create a junction table referencing both entities, and within the entity you also have to insert navigation properties that reference those classes.

```csharp
public class Apples_Oranges {
	// Primary keys
	[Key]
	[Required]
	public int AppleId { get; set;}
	[Key]
	[Required]
	public int OrangeId { get; set; }
	// Navigation properties
	public Apple? Apple { get; set; }
	public Orange? Orange { get; set; }
}
```

We then have to go into our DbContext and declare relationship for each entities, including the junction table.

```csharp
public class ApplicationDbContext : DbContext
{
	public ApplicationDbContext(DbContextOptions options) : base(options) { }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
		modelBuilder.Entity<Apples_Oranges>()
			.HasKey(i => new { i.AppleId, i.OrangeId });
    }
}
```

We first setup our junction table entity and assign primary keys. This composition method is only available in Fluent API and is required. We then have to establish one-to-many relationship between junction table and single entities, and have EF core to cascade down to create many-to-many relationship. Be careful of the ordering of words here because it matters: one-to-many, from junction table to entity. This way, it will be easier to remember when implementing in the future. We have already implemented "one" in our junction table, which references both Apple and Orange. Now "many" reference has to be included in the single entity.

```csharp
public class Apple {
	[Key]
	[Required]
	public int Id { get; set; }
	[Required]
	public string Description { get; set; } = null!;
	// Many relationship with junction table
	public ICollection<Apples_Oranges>? Apples_Oranges { get; set; }
}
public class Orange {
	[Key]
	[Required]
	public int Id { get; set; }
	[Required]
	public string Description { get; set; } = null!;
	// Many relationship with junction table
	public ICollection<Apples_Oranges>? Apples_Oranges { get; set; }
}
```

If you do not want your endpoints to have same name as the column variable in the entity, use Newtonsoft.Json package and add the JsonPropertyName attribute:

```bash
dotnet add package Newtonsoft.Json
```

and here's how you would configure your key name in the json output:

```csharp
public class Orange {
	[Key]
	[Required]
	public int Id { get; set; }
	[Required]
	[JsonPropertyName("orange_baskets")]
	public string Description { get; set; } = null!;
	// Many relationship with junction table
	public ICollection<Apples_Oranges>? Apples_Oranges { get; set; }
}
```

Now we'll go back to our DbContext class and implement those relationship. Remember that EF core has no idea how the relationship works unless you set the relationship. You also have to declare in both directions

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
	base.OnModelCreating(modelBuilder);
	modelBuilder.Entity<Apples_Oranges>()
		.HasKey(i => new { i.AppleId, i.OrangeId });
	// From junction table to Apple
	modelBuilder.Entity<Apples_Oranges>()
		.HasOne(a => a.Apple)
		.Withmany(j => j.Apples_Oranges)
		.HasForeignKey(k => k.AppleId)
		.IsRequred()
		.OnDelete(DeleteBehavior.Cascade);
	// From junction table to Orange
	modelBuilder.Entity<Apples_Oranges>()
		.HasOne(j => j.Orange)
		.WithMany(o => o.Apples_Oranges)
		.HasForeignKey(k => k.OrangeId)
		.IsRequired()
		.OnDelete(DeleteBehavior.Cascade);
}
```

We then have to instruct EF to create tables for each entities we've created. This is simply done by using DbSet method, which are added after OnModelCreating method:

```csharp
public DbSet<Apple> Apples => Set<Apple>();
public DbSet<Orange> Oranges => Set<Orange>();
public DbSet<Apples_Oranges>> Apples_Oranges => Set<Apples_Oranges>();
```

Note: you might also see alternative way to do this, and this is with getters and setters. They both do the same thing.

```csharp
public DbSet<Apple> Apples { get; set; }
public DbSet<Orange> Oranges { get; set; }
public DbSet<Apples_Oranges> Apples_Oranges { get; set; }
```

### Configuring DbContext

Now we'll add the DbContext to our web app as a service. Let's first setup our configurations on appsetting.Developer.json first. Add the following line to the json file:

```json
  "ConnectionStrings": {
    "DefaultConnection": "Data source=Fruits.db"
  }
```

The setting above is specifically for Sqlite. Depending on the DBMS, the "DefaultConnection" value will be different. For example, PostgreSQL uses the following values:

```json
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Database=Fruits;Username=<user>;Password=<password>"
  }
```

And Microsoft SQL Server:

```json
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost\\SQLEXPRESS;Database=Fruits;User Id=<user>;Password=<password>;Integrated Security=False;MultipleActiveResultSets=True;TrustServerCertificate=True"
  }
```

Now we add the DbContext to our services. Each DBMS syntax is different, but they're very minor changes. Make sure to download the correct package for the database you're working on.

For Sqlite we add the following service into Program.cs

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
	options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));
```

If it was PostgreSQL, we use "UseNpgsql" instead.

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options => options.UseNpgsql(
        builder.Configuration.GetConnectionString("DefaultConnection")
    ));
```

And for Microsoft SQL Server "UseSqlServer"

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options => options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")
    ));
```

With EF core it is easy to use different databases on the fly. The only difference is, as shown above, is the configuration of the connection string, and alternating the Use method to that specific database.

### EF Migration

Finally, you will need setup Migration for your database to be configured.

```bash
dotnet ef database update InitialSetup
```

If you want to remove the database that was created, you use the drop command

```bash
dotnet ef database drop
```

This will effectly remove the database that was created after update command

#### Terminologies

- Navigation property: directly reference single related entity or collection of entities

```

```
