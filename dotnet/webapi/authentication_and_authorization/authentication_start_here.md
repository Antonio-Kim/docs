### Authentication Types

Here are some main authentication techniques for web api's:

- Sessions/cookies: relies and key-value storage service to log user's authentication info. This service will generate unique sessionId and sent back to client. Each request sent to server will require this sessionId
- Bearer tokens: uses encryption token which includes authorization info. The token has expiry date
- API keys: ClientId and ClientSecret is used everytime request is sent
- Signature and certificates: uses TLS certification for hashing based on shared secret. This is great for security but hard to implement on both side

### Authentication using JWT

Here are the required packages in order to implement authentication:

```bash
dotnet add package Microsoft.Extensions.Identity.Core
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

The implementation steps are:

1. Create a new entity class for users
2. Create or update existing ApplicationDbContext to handle users
3. Add apply migration and add database for ASP.NET Core Identity
4. Setup and configure services and middleware
5. Implement new endpoint to handle registration and login process

#### Step 1 - Creating new entity class for Users

**_Side note_**: do not use `User` for entity because this might create confusion between built-in properties (such as ControllerBase.User).

Create a new entity class insde /Model directory. We'll name it ApiUser

```csharp
using Microsoft.AspNetCore.Identity;
namespace Fruits.Models;

public class ApiUser : IdentityUser {}
```

#### Step 2 - Create or update existing ApplicationDbContext to handle users

We're assuming you've already have an ApplicationDbContext class, but even if not, the implementation is the same:

```csharp
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;

public class ApplicationDbContext : IdentityDbContext<ApiUser>
```

The main difference is that you'll inherit IdentityDbContext with type of Entity class we've inherit earlier.

#### Step 3 - Add apply migration and add database for ASP.NET Core Identity

Assuming that database has been already created (see database section), we add new migration

```bash
dotnet ef migrations add Identity
dotnet ef database update Identity
```

#### Step 4 - Setup and configure services and middleware

Two services and a middleware is required to be included and configured in our Program.cs:

- Identity service: perform registration and loginc process
- Authorization service: define rules for issuing and reading JWTs
- Authentication middleware: add JWT reading task to HTTP pipeline

##### Adding identity service

Steps to implement:

1. Add ASP.NET Identity service to service container
2. Configure minimum security requirements, such as password length
3. Add authentication middleware

```csharp
using Microsoft.AspNetCore.Identity;
//... some codes to generate and add services
builder.Services.AddIdentity<ApiUser, IdentityRole>(options => {
	options.Password.RequireDigit = true;
	options.Password.RequireLowercase = true;
	options.Password.RequireUppercase = true;
	options.Password.RequireNonAlphanumeric = true;
	options.Password.RequireLength = 12;
}).AddEntityFrameStores<ApplicationDbContext>();
```

##### Adding Authorization Service

Steps to implement:

1. Define JWT as default authentication method
2. Enabling JWT Bearer authentication method
3. Setting up JWT Validation, issuing, and lifetime settings

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;

builder.Services.AddAuthentication(options => {
    options.DefaultAuthenticateScheme =
    options.DefaultChallengeScheme =
    options.DefaultForbidScheme =
    options.DefaultScheme =
    options.DefaultSignInScheme =
    options.DefaultSignOutScheme =
        JwtBearerDefaults.AuthenticationScheme;
}).AddJwtBearer(options => {
    options.TokenValidationParameters = new TokenValidationParameters {
        ValidateIssuer = true,
        ValidIssuer = builder.Configuration["JWT:Issuer"],
        ValidateAudience = true,
        ValidAudience = builder.Configuration["JWT:Audience"],
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(
          System.Text.Encoding.UTF8.GetBytes(
              builder.Configuration["JWT:SigningKey"]) )
    };
});
```

Issuer, Audience, and signing key has to be set up in order for this to work. Under appsettings.json file, add the following:

```json
"JWT": {
	"Issuer": "FruitBasket",
	"Audience": "FruitBasekt",
	"SigningKey": "R@nd0mStr1ng2H3re"
}
```

##### Adding authentication middleware

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

#### Step 5 - Implement new endpoint to handle registration and login process

We'll create an endpoint for client to register, and login. There are also managing services we'll need to include to our controller to handle managing and logging users. As usual, we'll create DTO for Register and DTO for Login.

```csharp
using System.ComponentModel.DataAnnotations;
// Register DTO
public class RegisterDTO {
    [Required]
    public string? UserName { get; set; }
    [Required]
    [EmailAddress]
    public string? Email { get; set; }
    [Required]
    public string? Password { get; set; }
}

// LoginDTO
public class LoginDTO {
    [Required]
    [MaxLength(255)]
    public string? UserName { get; set;}
    [Required]
    public string? Password { get; set; }
}
```

We'll first begin with required dependencies for the setup of the endpoints, and then go into implementation afterwards. We add both UserManager and SignInManager from Microsoft.AspNetCore.Identity library

```csharp
[Route("[controller]/[action]")]
[ApiController]
public class AccountController : ControllerBase {
    private readonly ApplicationDbContext _context;
    private readonly IConfiguration _configuration;
    private readonly UserManager<ApiUser> _userManager;
    private readonly SignInManager<ApiUser> _signInManager;

    public AccountController(
        ApplicationDbContext context,
        IConfiguration configuration,
        UserManager<ApiUser> userManager,
        SignInManager<ApiUser> signInManager
    )
    {
        _context = context;
        _logger = logger;
        _configuration = configuration;
        _userManager = userManager;
    }
}
```

Let's implement the Registration endpoint. This uses "/Account/Register" which mirrors [controller]/[action] or more specifically reflecting the class name (without the "Controller") and the name of the function

```csharp
[HttPost]
[ResponseCache(CacheProfileName = "NoCache")]
public async Task<ActionResult> Register(RegisterDTO input) {
    try {
        if(ModelState.IsValid) {
            var newUser = new ApiUser();
            newUser.UserName = input.UserName;
            newUser.Email = input.Email;
            var result = await _userManager.CreateAsync(newUser, input.Password);
            if (result.Succeeded) {
                return StatusCode(201, $"User '{newUser.UserName}' has been created.");
            } else {
                throw new Exception(string.Format("Error: {0}", string.Join(" ", result.Errors.Select(e => e.Description))));
            }
        } else {
            var details = new ValidationProblemDetails(ModelState);
            details.Type = "https://tools.ietf.org/html/rfc7231#section-6.5.1";
            details.Status = StatusCode.Status400BadRequest;
            return new BadRequestObjectResult(details);
        }
    } catch (Exception e) {
        var exceptionDetails = new ProblemDetails();
        exceptionDetails.Detail = e.Message;
        exceptionDetails.Status = StatusCodes.Status500InternalServerError;
        exceptionDetails.Type = "https://tools.ietf.org/html/rfc7231#section-6.6.1";
        return StatusCode(StatusCode.Status500InternalServerError, exceptionDetails);
    }
}
```

The validation for password is handled by UserManager so we do not have to worry about that. Most of the code above is actually writing error messages and handling both 400 and 500 error code. This should be straightforward. The Login process looks similar to above but requires JWT implementation. Here are basic steps:

1. Check user in the database, and check their password
2. Create signing credential and encrypt it
3. Create Claims list and add usernam
4. Create JwtObject and configure it with:
   - Issuer
   - Audience
   - Claim
   - Expiry
   - Signing Credential
5. Convert Jwt Object into string and send it back

```csharp
[HttpPost]
[ResponseCache(CacheProfileName = "NoCache")]
public async Task<ActionResult> Login(LoginDTO input) {
    try {
        if (ModelState.IsValid) {
            var user = await _userManager.FindByNameAsync(input.UserName);
            if (user == null || !await _userManager.CheckPasswordAsync(user, input.Password)) {
                throw new Exception("Invalid login attempt.");
            } else {
                // Create Signing Credential
                var signingCredentials = new SigningCredentials(
                    new SymmetricSecurityKey(System.Text.Encoding.UTF8.GetBytes(_configuration["JWT:SigningKey"])),
                    SecurityAlgorithms.HmacSha256);
                // Create Claims
                var claims = new List<Claim>();
                claims.Add(Claim(ClaimType.Name, user.UserName));
                // create JWT Object using configuration and identities created above
                var jwtObject = new JwtSecurityToken(
                    issuer: _configuration["JWT:Issuer"],
                    audience: _configuration["JWT:Audience"],
                    claims: claims,
                    expires: DateTime.Now.AddSeconds(300),
                    signingCredentials: signingCredentails
                );
                // Convert to string and return with jwt token
                var jwtString = new JwtSecurityTokenHandler().WriteToken(jwtObject);
                return StatusCode(StatusCode.Status200Ok, jwtString);
            }
        } else {
            var details = new ValidationProblemDetails(ModelState);
            details.Type = https://tools.ietf.org/html/rfc7231#section-6.5.1";
            details.Status = StatusCodes.Status400BadRequest;
            return new BadRequestObjectResult(details);
        }
    } catch (Exception e){
        var exceptionDetails = new ProblemDetails();
        exceptionDetails.Detail = e.Message;
        exceptionDetails.Status = StatusCodes.Status401Unauthorized;
        exceptionDetails.Type = "https://tools.ietf.org/html/rfc7231#section-6.6.1";
        return StatusCode(StatusCodes.Status401Unauthorized, exceptionDetails);
    }
}
```

Most of them are straight forwad to understand. Here we'll explain little bit more about Claims. Claims are list of users whom we're generating JWT token for. When the endpoints are checking whether the request is allowed, it checks the claims first before allowing the endpoint to go through. For now there's only one.
