## Read operation of webapi

There are more intricacy as to reading data from database and simply sending data. For example, say you have 55,000 items in a database and someone requests GetAll. Imagine how much you would have to send over the internet for that specific dataset! This isn't feasible and there are multiple ways to mitigate this issue. Several techniques will be explained here, mainly: **Paging**, **Sorting**, and **Filtering**

### Paging

We don't want our users to send request all the data. We want only portion of the dataset sent to the user instead, and if they would like more, they can request more parts. There are two methods of paging techniques:

1. Client-side paging: client gets all the data from the server and shows only the content that user may be viewing.
2. Server-side paging: server only send specific amount of data and the client has to comply with this

In the ideal world, servers do not care about the paging and have the client deal with this. However, this isn't the case we have to do our due diligent and take performance and security into account. Thus, we'll learn how to implement paging to our read operation.
Say we have the following GET() method:

```csharp
public async Task<RestDTO<Apple[]>> Get() {
	var apples = await _context.Apples;

	return RestDTO<Apple[]>()
	{
		Data = await apples.ToArrayAsync(),
		Links = new List<LinkDTO>() {
			new LinkDTO(
				Url.Action(
					null,
					"Apples",
					new { pageIndex, PageSize},
					Request.Scheme)!,
				"self",
				"GET")
		}
	};
}
```

Easy enough. All you have to understand is that the service will return all the Apples in the database, whatever the values might be. The data will return all the Apples in the database. Problem with this is that if there are tons of data inside the database, returning all the apples would take too long to occur. This is where paging comes into play. We set our GET method with parameters, plus default parameters of page size and page index, and then have our DTO return specific number of data to the client. This also mean we'll need to change our EF implementation.

Let's start with RestDTO. Assuming you have similar setup as the DTO in start_here section of this, we'll add PageIndex, PageSize, and RecordCount

```csharp
public class RestDTO<T> {
	public T Data { get; set; } = default!;
	public List<LinkDTO> Links { get; set; } = new List<LinkDTO>();
	// Paging parameters
	public int? PageIndex { get; set; }
	public int? PageSize { get; set; }
	public int? RecordCount { get; set; }
}
```

Now let's modify our GET Method. We'll set default parameter of PageIndex and PageSize, and have EF to skip number of data (size \* index) and take limited amount of data (size).

```csharp
public async Task<RestDTO<Apple[]>>(int pageIndex = 0, int pageSize = 0) {
	var apples = await _context.Apples
		.Skip(pageIndex * pageSize)
		.Take(pageSize);

	return new RestDTO<Apple[]> {
		Data = apples.ToArrayAsync(),
		PageIndex = pageIndex,
		PageSize = pageSize,
		RecordCount = await _context.Apples.CountAsync(),
		Links = new List<LinkDTO>() {
			new LinkDTO(
				Url.Action (
					null,
					"Apples",
					new { pageIndex, pageSize },
					Request.Scheme)!,
				"self",
				"GET")
			}
	};
}
```

### Sorting

There are two ways to implement sort: programmatically and using the DBMS. The best way to do this is through DBMS since DBMS has some sort implementation of this, but also programming requires getting all the dataset which is not ideal.
We can try implementing it ourselves,

```csharp
var query = _context.Apples.OrderBy(a = a.Name).ToArrayAsync();
```

but the main issue is that this only sort by name. If we want to have it sort in different columns, we'll need to create a hugh switch statements to get it queried. This isn't ideal so we'll implement third part library for this.

```bash
dotnet add package System.Linq.Dynamic.Core
```

Once this is done we simply add the sort column into the parameter and have our query sorted.

```csharp
public async Task<RestDTO<Apple[]>>(int pageIndex = 0, int pageSize = 0, string? sortColumn = "Name") {
	var apples = await _context.Apples
		// Here is where sort is added
		.OrderBy(sortColumn)
		.Skip(pageIndex * pageSize)
		.Take(pageSize);

	return new RestDTO<Apple[]> {
		Data = apples.ToArrayAsync(),
		PageIndex = pageIndex,
		PageSize = pageSize,
		RecordCount = await _context.Apples.CountAsync(),
		Links = new List<LinkDTO>() {
			new LinkDTO(
				Url.Action (
					null,
					"Apples",
					new { pageIndex, pageSize },
					Request.Scheme)!,
				"self",
				"GET")
			}
	};
}
```

How about sorting by ascending or descending order? Simply add the parameter with default values, and then modify by OrderBy. The OrderBy LINQ follows `.OrderBy($"{sortColumn} {sortOrder}")` format

```csharp
public async Task<RestDTO<Apple[]>>(int pageIndex = 0, int pageSize = 0, string? sortColumn = "Name", string? sortOrder = "ASC") {
	var apples = await _context.Apples
		// Here is where sort is added
		.OrderBy($"{sortColumn} {sortOrder}")
		.Skip(pageIndex * pageSize)
		.Take(pageSize);

	return new RestDTO<Apple[]> {
		Data = apples.ToArrayAsync(),
		PageIndex = pageIndex,
		PageSize = pageSize,
		RecordCount = await _context.Apples.CountAsync(),
		Links = new List<LinkDTO>() {
			new LinkDTO(
				Url.Action (
					null,
					"Apples",
					new { pageIndex, pageSize },
					Request.Scheme)!,
				"self",
				"GET")
			}
	};
}
```

### Filtering

Adding filtering requires modification to our previous code, but the idea is the same: we'll have EF and DBMS do the filtering since this is common in all our DBMS. The idea is converout our DTO into IQueryable in our Controller and then have EF to only retrieve with that filter. Note that because LINQ uses lazy loading, the whole method does not execute until the Filter is implemented so there's bit of performance there.

```csharp
public async Task<RestDTO<Apple[]>>(
	int pageIndex = 0,
	int pageSize = 0,
	string? sortColumn = "Name",
	string? sortOrder = "ASC",
	string? filterQuery = null) {
	// Filter parameter is added above
	var apples = await _context.Apples.AsQueryable();
	if (!string.IsNullOrEmpty(filterQuery))
		apples = query.Where(a => a.Name.Contains(filterQuery))
	var recordCount = await. apples.CountAsync();

	// after conditions above, query is done here
	apples = apples
		.OrderBy($"{sortColumn} {sortOrder}")
		.Skip(pageIndex * pageSize)
		.Take(pageSize);

	return new RestDTO<Apple[]> {
		Data = apples.ToArrayAsync(),
		PageIndex = pageIndex,
		PageSize = pageSize,
		RecordCount = recordCount,
		Links = new List<LinkDTO>() {
			new LinkDTO(
				Url.Action (
					null,
					"Apples",
					new { pageIndex, pageSize },
					Request.Scheme)!,
				"self",
				"GET")
			}
	};
}
```
