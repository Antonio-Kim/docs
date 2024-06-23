## Seeding database

Usually this is not necessary but when it comes to testing your CRUD operations you would like some sort of data in there to see if your methods are done correctly. There are several ways to do it but this guide, at least for now, will be broken down to two methods, programming and csv files, and testing method which holds data temporarily using in-memory database.

### Seeding Database using coding

This is useful when you want to have small sample of data for initial test. This section requires bit of theoretical understanding of how Inversion of Control works. By default ASP.NET has Dependency Injection containers that manages services for you. There various default services added, but one of the services you do add to database is DbContext in Program.cs. You might be aware of this code:

```csharp
builder.Services.addDbContext<ApplicationDbContext>(options =>
	options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection)));
```

Here you are adding DbContext service to your app. Knowing this fact, you can manually access these services using IServiceProvider. When you have separate class wanting to use one of these services you would have to inject IServiceProvider to your class in order to modify them. See where this is going?

When creating class to seed your data, you'll need to inject IServiceProvider to your seeding class. We can do it by the following:

```csharp
public static class SeedData {
	public static async Task Initialize(IServiceProvider serviceProvider) {
		using var context = new ApplicationContext(serviceProvider.GetRequiredService<DbContextOptions<ApplicationDbContext>>());
		await context.Database.EnsureCreatedAsync();

		// ... various methods are added here to the context using Database methods

		context.SaveChanges();
	}
}
```

Bit of explanation is required here. First, both the class and method are static since we won't need it to have it as an object (they are helper class and function after all). Second, as mentioned, we inject IServiceProvider so we can access services that Dependency Injection container has. You see that on the next line we do bring one of the services in using "GetRequiredService", and here we specifically bring "<DbContextOptions<ApplicationDbContext>>" which looks very familiar to constructor of the ApplicationDbContext

```csharp
public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options) {}
```

Notice that we're importing using instead of declaring at the top. ApplicationContext implements IDisposable and removes the service at the end of the call. If we were declaring at the top of the class the context would continue on until program ends and that would be wasteful of memory resource, so we declare it inside the function. Lastly, after inserting all the sample data, we end the function with "context.SaveChanges()" just like how we would do in a regular services.

### Seeding Database with csv files

In most cases, DBMS managers comes with the ability of importing csv files into the table. But this isn't ideal when dealing with multiple tables with a single file, so we'll implement our own version. Again, if it was one csv file for one table, it's a lot easier to use DBMS to do it.
We'll implement seeds using webapi. We'll need a third-party library to deal with csv files first. Download CsvHelper using NuGet, or through command line:

```bash
dotnet add package CsvHelper
```

This will download the latest version of the library. Now we'll create an entity class based on the columns of the csv file

```csharp
public class Apple {
	[Name("ID")]
	public int? ID { get; set;}
	[Name("Type of Apple")]
	public string? typeOfApple { get; set; }
	[Name("Users rating")]
	public int? usersRating { get; set; }
}
```

Note that this is ficticious data set with three columns in the csv file with names "ID", "Type of Apple", and "Users rating". The Name attributes are exactly the name of the column shown on csv file, whereas the properties in the class are not. This is because of spacing and thus to map properly to our variable, name attribute was used. The Name attribute comes from CsvHelper library, not Data Annotation library from Microsoft.EntityFrameworkCore.
