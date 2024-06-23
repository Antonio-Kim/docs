## Read operation of webapi

There are more intricacy as to reading data from database and simply sending data. For example, say you have 55,000 items in a database and someone requests GetAll. Imagine how much you would have to send over the internet for that specific dataset! This isn't feasible and there are multiple ways to mitigate this issue. Several techniques will be explained here, mainly: **Paging**, **Sorting**, and **Filtering**

### Paging

We don't want our users to send request all the data. We want only portion of the dataset sent to the user instead, and if they would like more, they can request more parts. There are two methods of paging techniques:

1. Client-side paging: client gets all the data from the server and shows only the content that user may be viewing.
2. Server-side paging: server only send specific amount of data and the client has to comply with this

In the ideal world, servers do not care about the paging and have the client deal with this. However, this isn't the case we have to do our due diligent and take performance and security into account. Thus, we'll learn how to implement paging to our read operation.
Say we have the following GET() method:

```
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

```
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

```
public async Task<RestDTO<Apple[]>>(int pageIndex = 0, int pageSize = 0) {
	var apples = await _context.Apples()
		.Skp(pageIndex * pageSize)
		.Take(pageSize);

	return new RestDTO<Apple[]> {
		Data = apples.ToArrayAsync(),
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
