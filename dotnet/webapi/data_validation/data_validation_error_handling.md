## Error Handling

ASP.NET handles errors very well out of the box. If you send wrong type of data for the parameter required, it will send 400 error code with explanation as to why the error occur. This error handling is done at [ApiController] level. Error handling by [ApiController] occurs at two stages of incoming data: at model binding and model state. The system handles two important stages that can redirect the http request from 200 to 400 error codes:

1. During Model binding: converts the input into model parameter's type. If the types do not match, it sends HTTP error 400.
2. During Model validation: this could be out of the box attributes, or custom validation with IValidateableObject

### Custom error messages

As mentioned above, there are two paths that error can occur when handling web requests. We'll look into handling them on those stages. For model binding errors, the custom message can be added on the Program.cs where AddController is being inserted:

```csharp
builder.Services.AddControllers(options => {
    options.ModelBindingMessageProvider.SetValueIsInvalidAccessor(
        (x) => $"The value '{x}' is invalid.");
    options.ModelBindingMessageProvider.SetValueMustBeANumberAccessor(
        (x) => $"The field {x} must be a number.");
    options.ModelBindingMessageProvider.SetAttemptedValueIsInvalidAccessor(
        (x, y) => $"The value '{x}' is not valid for {y}.");
    options.ModelBindingMessageProvider.SetMissingKeyOrValueAccessor(
        () => $"A value is required.");
});
```

For model validation, we use attribute with "ErrorMessage= <Message>" parameter:

```
[Required(ErrorMessage="This value is required")]
[Range(1,100,ErrorMessage="The value must be between 1 and 100")]
```

### Manual model validation

Notice that handling error is done by ApiController and only replies back with 400 error code. To modify its behaviour, we'll need to customize its redirection. First, we configure our ApiController behaviour in the main program like this, adding the code above app is declared:

```csharp
builder.Services.Configure<ApiBehaviorOptions>(options =>
    options.SuppressModelStateInvalidFilter = true));

var app = builder.Build();
```

Let's start with an example. Instead of returning Error 400, let's implement Error 501 and state that it has not been implemented just yet. Here are the steps to implement the handling inside our controller:

1. Ensure that the function method returns Task<ActionResult<RestDTO<Apple[]>>>
2. Check if the validation error occurs, if so, create ValidationProblemDetails object
3. Find which error has occured. If it is something that you are expected, set the ValidationProblemDetails' status code to 501, and return ObjectResult of ValidationProblemDetail with StatusCode set to 501.
4. If it is not something not expected, set the ValidationProblemDetails status code 400 and return BadRequestObjectResult of ValidationProblemDetail object.

Here's the actual implementation

```csharp
if (!ModelState.IsValid) {
	var details = new ValidationProbelmDetails(ModelState);
	details.Extensions["traceId"] = System.Diagonstics.Activity.Current?.Id ?? HttpContext.TraceIdentifier;
	if (ModelState.keys.Any(k => k == "PageSize")) {
		details.Type = "https://tools.ietf.org/html/rfc7231#section-6.6.2";
		details.Status = StatusCodes.Status501NotImplemented;
		return new ObjectResult(details) {
			StatusCode = StatusCodes.Status501NotImplemented
		};
	} else {
		details.Type = "https://tools.ietf.org/html/rfc7231#section-6.5.1";
		details.Status = StatusCodes.Status400BadRequest;
		return new BadRequestObjectResult(details);
	}
}
```
