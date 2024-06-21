## Starting webapi

Best method is create a folder, add solution file to the folder, and then create an webapi there. Reason for
this is so that later on if you want to add a library (or start with a library for that matter), you can add
the libraries to the project on top of the webapi. So here's how you would start, after creating the folder.

```bash
dotnet new sln
dotnet new webapi -n test -controllers
dotnet sln add test
```

this will create a solution file same name as the directory, and then create a webapi named test using
controllers instead of minimal api. Once the webapi folder is created, it then gets added to the the sln
file. If you want to create a library and then have webapi reference the library, you would do it this way:

```bash
dotnet new classlib -n external
dotnet sln add external
dotnet add test reference external
```

This is also useful because at some point you would (and should) want to add testing libraries like xUnit.
Another useful command is to add known packages to the project. For example, let say I want to add
Sqlite database to the webapi as mentioned above.

```bash
dotnet add test package Microsoft.EntityFrameworkCore.Sqlite
```

Which one should I use? Controller or minimal api? The most practical answer is both. You should use minimal
api if you need quick implementation of a web api. But if you need more configuration you should use controllers.

## Setting up environment

- launchSettings.json
  - Creating a profile for specific uses: Under Properties->launchSettings.json->profiles, you can add a new profile with customization to that specific uses. Useful keys are: - applicationUrl: set http and https url and port numbers here - launchBrower: set it to true or false to have brower open up the app.
  - launchUrl: default route when the app is launched - environmentVariables: here you can set which environment that profile is used. Usually are saved with a subset of key-value pair where key starts with "ASPNETCORE_ENVIRONMENT" and value of either "Development", "Staging", or "Production"
  - Setting up ports: under Properties->launchSettings.json, you have two profiles, named "http" and "https". Set "applicationUrl" to specific ports you'd like on http and https urls.
- appsettings.json
  - you can customize each environments' setting by naming a new file. The default version for all the
    environment is appsetting.json, but you can customize to others by appending the name after appsetting. For example, you would name appsetting.Development.json for development environment, and appsetting.Staging.json for staging environment.
  - Useful key-value pairs:
  - UseDeveloperExceptionPage: true/false
  - UseSwagge: true/false

## Useful Middleware

- UseDeveloperExceptionPage(): should be used in developer mode, like this:

```csharp
if (app.Enviroment.IsDevelopment)
{
	app.UseSwagger();
	app.UseSwaggerUI();
	app.UseDeveloperExceptionPage();
}
```

However the best practice is to set on appSetting.Development.json file and retrieve the configuration there instead. Add
""UseDeveloperExceptionPage": false" inside the appSetting.Development.json and then modify the middleware like this:

```csharp
if (app.Configuration.GetValue<bool>("UseDeveloperExceptionPage"))
	app.UseDeveloperExceptionPage();
else
	app.UseExceptionHandler("/error");
```

Make sure to implement the /error endpoint. I suggest using minimal api to implement, like this:

```csharp
app.MapGet("/error/test", () => { throw new Exception("test"); });
```

## Implementing CORS

There are two steps to implement CORS to your app: setting up policies and Applying them. Let's start with setting
up policies first.

```csharp
builder.Services.AddCors(options => {
	options.AddDefaultPolicy(cfg => {
		cfg.WithOrigins(builder.Configuration["AllowedOrigins"]);
		cfg.AllowAnyHeader();
		cfg.AllowAnyMethod();
	});
	options.AddPolicy(name: "AnyOrigin",
		cfg => {
			cfg.AllowAnyOrigin();
			cfg.AllowAnyHeader();
			cfg.AllowAnyMethod();
		});
})
```

This will set CORS policy by default for any endpoints, plus adding policy named "AnyOrigin" where policy could be set.
Now there are three ways to apply CORS policy, which are: CORS middleware, Endpoint routing, and [EnableCors] attribute.
The best way is the attribute way and it will be only shown here. To apply the attribute, go to the routing endpoint in
the controller and type in the following:

```csharp
[EnableCors("AnyOrigin")]
```

## Implementing Caching

There are three types of caching techniques:

1. Server-side caching (application caching): saves key-value pairs using built-in service or third party such as Memcached, Redis, Couchbase, or managed abstract layers like Amazon Elasticache, or even DBMS
2. Client-side caching: used on browsers but server-side will set policies on the cache based on the header (this is probably be the most important one)
3. Intermediate caching (proxy caching, reverse-proxy caching, or content-delivery network (CDN) caching):
   stores cached data by using proxies or third party services optimized by having contents stored near requester's location.

## Implementing HATEOAS

In most cases people would see only the fetching of the data. However if you want your API to be HATEOAS compliant, you want to add a link to your response so that consumer of API can figure out how to go about
using the API without document. Thus, in order for response to be HATEOAS compliant you need:

- Data
- Link
  This can be implemented by having Link DTO added to RestDTO along with your data set. Let's use an example:

```csharp
public class LinkDTO
{
	public string Href { get; private set; }
	public string Rel { get; private set; }
	public string Type { get; private set; }

	public LinkDTO(string href, string, rel, string type)
	{
		Href = href;
		Rel = rel;
		Type = type;
	}
}
public class RestDTO<T>
{
	public List<LinkDTO> Links { get; set; } = new List<LinkDTO>();
	public T Data { get; set; } = default!;
}
```

### Useful files

- launchSetting.json: settings where launching project in development environment
  - $schema: pointing to a URL taht describes the schema used by the file
  - iisSettings: Containing some base configuration for IIS express server
  - profiles: configuration for different web servers. Typically provided by Kestrel. This is the default option
    when `dotnet run` is used on development.
- appSetting.json: configuration for all enviroment. There's another file called appSetting.Development.json
  which is, as name states, on development environment.
  - whichever is the narrowest in terms of enviroment will override the general appSetting. Thus, any configuration
    done in appSetting.Development.json will override the appSetting.json if they are the same key-value pairs.
- Program.cs: main location of where app starts. Usually includes Services and Middleware

### Terminologies

- Middleware: useful tools to configure your webapi with addtional features on the HTTP level. This specifically
  deals with any requests on HTTP level and are divided into two categories: terminal, where the request can
  end, and its opposite non-terminal middlewares. Examples of terminal include redirectional to different servers.
