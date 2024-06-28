### Adding custom caching middleware

This will set caching policy to all our endpoints if [ResponseCache] policy isn't set. Add this to Program.cs' Middleware

```csharp
App.UseAuthorization();

app.Use((context, next) => {
	context.Response.GetTypedHeaders().CacheControl = new Microsoft.Net.Http.Headers.CacheControlHeaderValue()
	{
		NoCache = true,
		NoStore = true
	};
	return next.Invoke();
});
```

### Adding custom cache profile

There may be multiple times where you implement the same cache profile. Instead of doing this, we can create cache profile in the services part of Program.cs and then use it inside our controller:

```csharp
builder.Services.AddControllers(options => {
	options.CacheProfiles.Add("NoCache", new CacheProfile() { NoStore = true });
	options.CacheProfiles.Add("Any-60", new CacheProfile() {
		Location = ResponseCacheLocation.Any,
		Duration = 60
	});
})
```

And then inside our controller we add the following attribute:

```csharp
[HttpGet(Name = "GetApples")]
[ResponseCache(CacheProfileName = "Any-60")]
public async Task<RestDTO<Apples[]>> Get()
```

### Server-side response caching

There are different ways to cache responses, but here we'll stick with Middleware. The next section will cover the external proxy server caching. Add the Caching service to our webapp:

```csharp
builder.Services.AddResponseCaching();
// ...
app.UseCors();
app.UseResponseCaching();
```

Because caching middleware has fixed size, we can also configure it to have a larger memory. This is done in the service section when we add the response caching

```csharp
builder.Services.AddResponseCaching(options => {
	options.MaximumBodySize = 32 * 1024 * 1024;
	option.SizeLimit = 50 * 1024 * 1024;
}) ;
```

Both option uses bytes, so to configure them larger, we'll multiply (1024 \* 1024) for MB.

### In-memory caching

The step after server caching but before external server/app for caching is the in-memory cache. The problem with server-side caching is that it does not enforce the caching policy of the server; it honors the client side instead. Many problems can occur by honoring client-side caching, for example Distributed Denial of service attacks.

#### Setting up In-memory caching

```
builder.Services.AddMemoryCache();
```

There are options we can add, such as:

- ExpirationScanFrequency
- SizeLimit
- CompactionPercentage

We'll use default parameters from now on.

#### Implementing In-memory caching

Here's the general step for implementing in-memory caching inside our controller:

1. Create a cacheKey string variable based on the GET parameters.
2. Check out the \_memoryCache instance to see whether a cache entry for that key is present.
3. If a cache entry is present, we use it instead of querying the database; otherwise, retrieve the data by using EF Core, and set the cache entry with an absolute expiration of 30 seconds.

We'll inject memory cache to our controlelr

```csharp
private readonly IMemoryCache _memoryCache;
public AppleController(
	ApplicationDbContext context,
	ILogger<ApplesController> logger,
	IMemoryCache memoryCache
)
{
	_context = context;
	_logger = logger;
	_memoryCache = memoryCache;
}
```

We then implement the caching inside our controller

```csharp
[HttpGet(Name = "GetApples")]
public async Task<RestDTO<Apples[]>> Get ([FromQuery] RequestDTO<AppleDTO> input) {
	var query = _context.Apples.AsQuerable();
	if (!string.IsNullOrEmpty(input.FilterQuery))
		query = query.Where(a => a.Name.Contains(input.FilterQuery));
	var recordCount = await query.CountAsync();

	Apple[]? result = null;
	// Implementation of caching begins here
	var cachekey = $"{input.GetType()}--{JsonSerializer.Serialize(input)}";
	if (!_memoryCache.TryGetValue<Apples[]>(cacheKey, out result))
	{
		query = query
			.OrderBy($"{input.SortColumn} {input.SortOrder}")
			.Skip(input.PageIndex * input.PageSize)
			.Take(input.PageSize);
		result = await query.ToArrayAsync();
		_memoryCache.Set(cacheKey, result, new TimeSpan(0, 0, 30));
	}
}
```

### Distributed Caching

In-memory caching is better than HTTP caching, but problem occurs when there's multiple servers running one service. This becomes bloted but also hard to maintain the caching technique because it will have to be applied to all the servers. Luckily there's another method of doing this, and this is through having dedicated server for caching. Even if it is only one server running, it will give both performance benefit and independent lifetime unlike what we implemented above.

There are two techniques to implement caching: DBMS and in-memory caching like redis. This will not be discussed in this section.
