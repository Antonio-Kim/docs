Logging refers to tracking activities of app. This is saved in either structured, semistructured, or unstructured format, and then outputting into some sort of display. The storage method is called secondary, because the primary is usually through user's UI like the command line. In order to implement loggin, Ilogger interface has to be injected to our ApiController

```csharp
private readonly ILogger<ApplesController> _logger;
public ApplesController (
	ApplicationDbContext context,
	Ilogger<ApplesController> logger
) {
	_context = context;
	_logger = logger;
}
```

### Quick logging test

Once ILogger is injected, we can simply do the following to log some sort of message

```
_logger.LogInformation("Hello, world!");
```

This will be displayed or saved at whichever interface is displayed/stored. LogInformation isn't the only method available. Common used methods are:

- LogTrace (0) - for low-level debugging or system admin tasks. Never used due to sensitivity info
- LogDebug (1) - dev information. Useful for analysis and debugging
- LogInformation (2) - usually disabled for production due to verbose information
- LogWarning (3) - abnormal or unexpected beavior
- LogError (4) - noncrictical events or activities that would likely halt the operation.
- LogCritical (5) - critical error that prevented the application starting

All the methods above are assigned to severity level of the error.

### Logging provider

Logging provider either displays or stores logging informations, and uses the ILogger to do so. There several providers that can be used to Log information and errors.

#### Built-in providers

Microsoft.Extensions.Logging comes with several providers, namely:

- Console: outputs to console
- Debug: Log message to output window as well as log files or register
- EventSource: outputs the log as runtime event trace
- EventLog: outputs the log in windows event log (only for windows OS)

By default it comes with all the providers above. To keep only the console and debug, we do the following in the Program.cs:

```
builder.Logging
	.ClearProviders()
	.AddSimpleConsole
	.AddDebug();
```

You can also configure each providers but they are different from each other in terms of parameters and thus you'll need to read through the documentation to do so. Those providers will not be covered in this section

### Types of Logging

- Unstructured: no order or format is declared when logging is done. Usually using plain text record
- Semi-structured: comes with some sort of organization that makes it easier to analyze or parse, such as csv
- Structured: logging is done in a repository such as DBMS.

Each types have its benefits and costs. Unstructured is easy to implement but it gets difficult to analyze as the code gets bigger and bigger. Structured is the best format for handling logging. There are several providers for this, including Microsoft Azure, but the most popular third-party is Serilog.

#### Brief introduction on Serilog

(Will need to modify this for PostgreSQL later on. For now, Microsoft SQL server is used)
Import the following packages for Serilog

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.MSSqlServer
```

and add the following to the Program.cs

```csharp
builder.Host.UseSerilog((ctx, lc) => {
	lc.ReadFrom.Configuration(ctx.Configuration);
	lc.WriteTo.MSSqlServer(
		connectionString: ctx.Configuration.GetConnectionString("DefaultConnection"),
		sinkOptions: new MSSqlServerSinkOptions
		{
			TableName = "LogEvents",
			AutoCreateSqlTable = true
		});
	},
	writeToProvider: true);
```
