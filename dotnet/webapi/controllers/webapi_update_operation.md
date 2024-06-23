## Update operation in webapi

### POST or PUT?

This design consideration really depends on whether you would want your response to be idempotent. In other words, would you like your response to return the same thing every time when the request is sent? One of the case would be last modified date. If response comes with last modified date field, it would return different result every update operation because, well, update is done every time the request is send.

### Steps to create Update

1. Create our model DTO if it doesn't exist
2. Have our method accept specific format of our model in JSON format
3. Look for our model in our database
4. Check if it is. If it does, update our database
5. Return relevant information

#### Creating our model DTO

We can take our ApplicationDbContext do this but we want to separate our responsibilities to have just single responsibility. Thus, if the model of our database does not have its DTO, we'll create our own. Let's continue on using our Apple Analogy. Add the following to your DTO folder

```csharp
public class AppleDTO {
	[Required]
	public int Id { get; set; }
	public string? Name { get; set; }
	public string? Description { get; set;}
}
```

Now we'll implement our controller which completes step 2, 3, 4, and 5 from above.

```csharp
[HttpPost(Name = "UpdateApple")]
[ResponseCache(NoStore = true)]
public async Task<RestDTO<Apple?>> Post(AppleDTO model) {
	// Step 3
	var apple = await _context.Apple
		.Where(a => a.Id == model.Id)
		.FirstOrDefaultAsync();
	if (apple != null) {
		if (!string.IsNullOrEmpty(model.Name))
			apple.Name = model.Name;
		if (!string.isNullOrEmpty(model.Description))
			apple.Description = model.Description;
		apple.LastModifiedDate = DateTime.Now;
		_context.Apple.Update(apple);
		// Step 4
		await _context.SaveChangesAsync();
	}
	// Step 5
	return RestDTO<Apple?>() {
		Data = apple,
		Link = List<LinkDTO>()
		{
			new LinkDTO(
				Url.Action(null, "Apple", apple, Request.Scheme)!,
				"self",
				"POST"
			)
		}
	}
}
```
