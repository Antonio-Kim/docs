## Delete operation in webapi

Delete operation is mostly similar to the update operation but instead of POST method we use DELETE, and the parameter of the DELETE will be just an integer. See the Update Operation file for more info

```
[HttpDelete( Name = "DeleteApples")]
[ResponseCache( NoStore = true )]
public async Task<RestDTO<Apple?>>(int Id) {
	var apple = await _context.Apple
		.Where(a => a.Id == Id)
		.FirstOrDefaultAsync();

	if (query != null) {
		_context.Apple.Remove(apple);
		await _context.SaveChangesAsync();
	}
	return new RestDTO<Apple?>()
	{
		Data = apple,
		Links = new List<LinkDTO>()
		{
			new LinkDTO(
				Url.Action(null, "Apple", apple, Request.Scheme)!,
				"self",
				"DELETE"
			)
		}
	};
}
```
