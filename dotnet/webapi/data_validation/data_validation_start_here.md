Data binding is a technique that ASP.NET uses to convert request's body, header, and any information from the client and convert into .NET's primitive and/or reference type. This allows conversion from raw string data into readable and modifiable data that our web app can use.

In most cases ASP.NET does this for us automatically. However, it does not validation whether the raw string data are correct, nor it has a way to gracefully handle the error other than 400 Error code (this is debatable since ASP.NET displaying 400 error is very helpful). In an ideal world we assume that the information sent by client is correct and fits into our model. In reality this is rarely the case. We'll implement some techniques to validate these data.

### Data validation attribute

These attributes are added in the parameter section of our request. Here are some of the useful attributes:

- [CreditCard]
- [EmailAddress]
- [MaxLength(n)]
- [MinLength(n)]
- [Range(n,m)]
- [RegularExpression(regex)]
- [StringLength]
- [Url]
  These validations are considered trivial (regex may not be trivial however. It could get pretty difficult!). There are cases where custom validation is required, and we usually create our own attribute by inheriting ValidationAttribute. For example, consider whether our sortOrder parameter is either ASC or DESC. We can simply create our own logic by if (sortOrder == "ASC" || sortOrder == "DESC"), use Regular Expression attribute [RegularExpression("ASC|"DESC")], which are all valid. However, we'll use this as an example, and create our own validation with custom output as an example of how to implement custom validation.

```csharp
public class SortOrderValidatorAttribute : ValidationAttribute {
	public string[] AllowedValues { get; set; } = new[] { "ASC", "DESC" };
	public SortOrderValidatorAttribute() : base("Value must be one of the following: {0}") { }

	protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
	{
		var strValue = value as string;
		if (!string.IsNullOrEmpty(strValue) && AllowedValues.Contains(strValue))
			return ValidationResult.Success;

		return new ValidationResult(
			FormatErrorMessage(string.Join(",", AllowedValues)));
	}
}
```

We inherit ValidationAttribute class and override one of the method, IsValid(). IsValid takes in two parameters, but we primary deal with value, which is the value assigned to parameter of our webapi method. Take GET method for example:

```csharp
public async Task<RestDTO<Apple>> Get(
	int pageIndex = 0,
	[SortOrderValidator] string? sortOrder = "ASC"
)
// ...
// Method is implemented below
```

Assuming our webapi takes a request in without any value, ASP.NET will take ASC as the default value for sortOrder. This in turn, is the "value" inside the IsValid method on our custom validation attribute. We assign our value into strValue after casting into a string, and then checks whether the strValue is empty and in our array of AllowedValue. If it is - and in our case it will - it will return Success and our validation is complete. Otherwise, it returns with error message and the data is invalidated.

This is nice and easy to implement. And you may argue this isn't necessary since there's only two AllowedValues, namely, ASC and DESC. But what happens when there's dynamically growing list of values you need to maintain? For example, there might be cases where our Model may have additional types added on, which increases database's column. This might be okay to hard code if the size of the columns are small, but cumbersome when the size gets bigger.

```csharp
public class SortColumnValidatorAttribute : ValidationAttribute
{
	public Type EntityType { get; set; }
	public SortColumnValidatorAttribute(Type entityType) : base("Value must match an existing column")
	{
		EntityType = entityType;
	}
	protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
	{
		if (EntityType != null)
		{
			var strValue = value as string;
			if (!string.IsNullOrEmpty(strValue) &&
				EntityType.GetProperties().Any(p => p.Name == strValue))
				return ValidationResult.Success;
		}
		return new ValidationResult(ErrorMessage);
	}
}
```

This requires some explanation. The Type is a reference type that will be checked during runtime. Notice that constructor here assigns entityType to a variable named EntityType, which is passed inside the validation attribute. The method GetProperties() will check whether the class that is passed onto EntityType has fields that's same as the value that's being passed in the parameter. For example, going back to the previous GET method we used

```csharp
public async Task<RestDTO<Apple>> Get(
	int pageIndex = 0,
	[SortOrderValidator] string? sortOrder = "ASC",
	[SortColumnValidator(typeof(AppleDTO))] string? sortColumn = "Name"
)
```

Again, assuming the request does not indicate what sortColumn is, it will assign that parameter into sortColumn. It will then check AppleDTO and see if it has a field called "Name".

#### Adding Custom Validation attribute info to Swagger

Swagger is often used as client app for testing webapi in ASP.NET. Swagger can understand built-in validation attributes provided by ASP.NET, but cannot understand custom validations we've described above. To do so, we need to follow these two steps:

1. Create a new filter class that implements IParameterFilter for each of the custom validator we created.
2. Implements the method that IParameterFilter requests, which is Apply() method

We'll go through both of the custom validator we created above and add to swagger. Create a folder name Swagger and add the following Filters

**NB**: Having bit difficulty understanding how this is working, so this will be revisited later on

```csharp
public class SortOrderFilter : IParameterFilter
{
	public void Apply(OpenApiParameter parameter, ParameterFilterContext context)
	{
		var attributes = context.ParameterInfo?
			.GetCustomAttributes(true)
			.OfType<SortOrderValidatorAttribute>();

		if (attributes != null)
		{
			foreach (var attribute in attributes)
			{
				parameter.Schema.Extensions.Add(
					"pattern",
					new OpenApiString(string.Join("|", attribute.AllowedValues.Select(v => $"^{v}$")))
				);
			}
		}
	}
}
```

```csharp
public class SortColumnFilter : IParameterFilter
{
	public void Apply(OpenApiParameter parameter, ParameterFilterContext context)
	{
		var attributes = context.ParameterInfo?
			.GetCustomAttributes(true)
			.OfType<SortColumnValidatorAttribute>();

		if (attributes != null)
		{
			foreach (var attribute in attributes)
			{
				var pattern = attribute.EntityType
					.GetProperties()
					.Select(p => p.Name);
				parameter.Schema.Extensions.Add(
					"pattern",
					new OpenApiString(string.Join("|", pattern.Select(v => $"^{v}$")))
				);
			}
		}
	}
}
```

Then go to main Program.cs and under AddSwaggerGen() service, add the custom validations:

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.ParameterFilter<SortColumnFilter>();
    options.ParameterFilter<SortOrderFilter>();
});
```
