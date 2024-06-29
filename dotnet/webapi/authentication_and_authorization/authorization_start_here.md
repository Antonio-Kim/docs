### Simple Implementation of Authorization

Say we just want our clients to access our endpoints only if they have registered and signed in, which means they should have a JWT token after the sign in process. All you need to do in this case is add [Authorize] attribute on the method and that method will be restricted. By default, every endpoints are allowed for all users except [Authorize]. To reverse this, we can add [Authorize] to the controller and then add [AllowAnonymous] to have all users access the endpoint without JWT token.

### Role-Based Access Control (RBAC)

For this example, we'll create three roles: **Basic Users**, which allows read-only endpoints and nothing else, **Moderators**, with read-only and update endpoints, but no delete, and **administrators**, which allows every endpoints.
Here are the general steps to implement RBAC Strategy for authentication and authorization:

1. Create roles: add two roles, Moderator and Administrator
2. Add Users, if there are already in the database, to the roles
3. Add Role-base claims to JWT: add the users to Claims we defined in authentication
4. Set up role-based authorization rules

#### Creating Roles

We'll add constants in a new class identifying each roles. Create a directory called /Constants and then add the following class named "RoleNames" with fields

```
public static class RoleNames {
	public const string Moderator = "Moderator";
	public const string Administrator = "Administrator";
}
```

Inside the controller, ensure to inject both UserManager and RoleManager service to the controller. For reminder, inside authentication we used UserManager and SignInManager. Here we are using UserManager but RoleManager instead.
We'll add two new users named "TestModerator" and "TestAdministrator" using the Account/Register endpoint.

```
private readonly RoleManager<IdentityRole> _roleManager;
private readonly UserManager<ApiUser> _userManager;

public AppleController(
	ApplicationDbContext context,
	IWebHostEnvironment env,
	ILogger<AppleController> logger,
	RoleManager<IdentityRole> roleManager,
	UserManager<ApiUser> userManager
)
{
	_context = context;
	_env = env;
	_logger = logger;
	_roleManager = roleManager;
	_userManager = userManager;
}
```

We'll now add an endpoint which will create the roles we set on RoleName.

```
[HttpPost]
[ResponseCache(NoStore = true)]
public async Task<IActionResult> AuthData() {
	int rolesCreated = 0;
	int usersAddedToRoles = 0;

	// Add roles to RoleManager
	if (!await _roleManager.RoleExistsAsync(RoleName.Moderator)) {
		await _roleManager.CreateAsync(new IdentityRole(RoleNames.Moderator));
		rolesCreated++;
	}
	if (!await _roleManager.RoleExistsAsync(RoleName.Administrator)) {
		await _roleManager.CreateAsync(new IdentityRole(RoleNames.Administrator));
		rolesCreated++;
	}

	// Add the newly created users, TestModerator and TestAdministrator, to each roles, respectively
	var testModerator = await _userManager.FindByNameAsync("TestModerator");
	if (testModerator != null && !await _userManager.IsInRoleAsync(testModerator, RoleNames.Moderator)) {
		await _userManager.AddToRoleAsync(testModerator, RoleNames.Moderator);
		userAddedToRoles++;
	}
	var testAdministrator = await _userManager.FindByNameAsync("TestAdministrator");
	if (testAdministrator != null && !await _userManager.IsInRoleAsync(testAdministrator, RoleNames.Administrator)) {
		await _userManager.AddToRoleAsync(testAdministrator, RoleNames.Administrator);
		userAddedToRoles++;
	}

	return new JsonResult(
		new {
			RolesCreated = rolesCreated,
			UsersAddedToRoles = usersAddedToRoles
		}
	);

}
```

#### Add Role-base claims to JWT

When user logs into the api, we'll need to add the user to the claims based on the role they're initially assigned to. Because user can be assigned to different roles, we'll need to add various role that the user has into the claim:

```
var claims = new List<Claim>();
claims.Add(new Claim(ClaimTypes.Name, user.UserName));
claims.AddRange(await _userManager.GetRoelsAsync(user))
	.Select(r => new Claim(Claimtypes.Role, r));
```

#### Set up role-based authorization rules

The last part is quite simple to implement. Wherever we have [Authorize] attribute attached to an endpoint, we add Role parameter with assigned Role. For example, if we want to ad PUT endpoint to be assigned to moderators, we add [Authorize(Roles = RoleNames.Moderator)]. If we want to assign DELETE to Administrators, we add [Authorize(Roles = RoleNames.Administrator)]

### Optional: Setting up Swagger for JWT Token

```
builder.Services.AddSwaggerGen(options => {
	options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme {
		In = ParameterLocation.Header,
		Description = "Please enter token",
		Name = "Authorization",
		Type = SecuritySchemeType.Http,
		BearerFormat = "JWT",
		Scheme = "bearer"
	});
	options.AddSecurityRequirement(new OpenApiSecurityRequirement {
		{
			new OpenApiSecurityScheme {
				Reference = new OpenApiReference {}
			},
			Array.Empty<string>()
		}
	});
})
```
