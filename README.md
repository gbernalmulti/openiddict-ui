# OpenIddict UI

A first step to provide some UI features to the [OpenIddict](https://github.com/openiddict/openiddict-core) stack. 

Currently it provides API's for managing `Scopes` and `Applications`.

On top of that it ships API's for the `Usermanagement` when using [ASP.NET Core Identity](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/identity?view=aspnetcore-5.0&tabs=visual-studio).

As a goodie the samples demonstrates this features by an Angular SPA client that uses the [Authorization Code flow](https://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth).

## Running the sample

Assuming you downloaded the sources and opened the directory with [VS Code](https://code.visualstudio.com/) you should be good to go! Ahh and of course you need [.NET Core](https://dotnet.microsoft.com/download) and [node.js](https://nodejs.org/en/) installed on your development environment.

### Running the Server
1. Open the integrated terminal in VS Code and type

```bash
dotnet build
```

That ensures you are able to build the dotnet related stuff!

2. Go to the VS Code Debug tab (Ctrl+Shift+D) and run the Server project.

3. After the Server is running navigate within your favorite command line to the `Client` directory and type:

```bash
npm i
```

This will install all the Client's required dependencies.

```bash
npm run start
```

This will start Angular's development server.

4. Now open your browser of choice and point to the well known Angular dev url.

```bash
http://localhost:4200
```

You should see now the login screen.


## Using it

Follow the original setup of the OpenIddict in the `Startup - ConfigureServices(...)` - method and add the two additional extension hooks `AddUIStore(...)` and `AddUIApis<TIdentityUser>()` and you should be good to go.

```csharp
...
services.AddOpenIddict()
  // Register the OpenIddict core components.
  .AddCore(options =>
  {
    // Configure OpenIddict to use the Entity Framework Core stores and models.
    options.UseEntityFrameworkCore();
    // Enable Quartz.NET integration.
    options.UseQuartz();
  })
  // Register the OpenIddict server components.
  .AddServer(options =>
  {
    options.SetIssuer(new Uri("https://localhost:5000/"));
    // Enable the authorization, device, logout, token, userinfo and verification endpoints.
    options.SetAuthorizationEndpointUris("/connect/authorize")
           .SetDeviceEndpointUris("/connect/device")
           .SetLogoutEndpointUris("/connect/logout")
           .SetTokenEndpointUris("/connect/token")
           .SetUserinfoEndpointUris("/connect/userinfo")
           .SetVerificationEndpointUris("/connect/verify");
    // Enable the desired flows.
    options.AllowAuthorizationCodeFlow()
           .AllowRefreshTokenFlow();
    // Enable the supported scopes.
    options.RegisterScopes(
      Scopes.OpenId,
      Scopes.Email,
      Scopes.Profile,
      Scopes.Roles,
      "demo_api");
    // Register the signing and encryption credentials.
    options.AddDevelopmentEncryptionCertificate()
           .AddDevelopmentSigningCertificate();
    // Force client applications to use Proof Key for Code Exchange (PKCE).
    options.RequireProofKeyForCodeExchange();
    // Register the ASP.NET Core host and configure the ASP.NET Core-specific options.
    options.UseAspNetCore()
           .EnableStatusCodePagesIntegration()
           .EnableAuthorizationEndpointPassthrough()
           .EnableLogoutEndpointPassthrough()
           .EnableTokenEndpointPassthrough()
           .EnableUserinfoEndpointPassthrough()
           .EnableVerificationEndpointPassthrough()
           .DisableTransportSecurityRequirement();
  })
  // Register the OpenIddict validation components.
  .AddValidation(options =>
  {
    // Import the configuration from the local OpenIddict server instance.
    options.UseLocalServer();
    // Register the ASP.NET Core host.
    options.UseAspNetCore();
  })
  // Register the EF based UI Store
  .AddUIStore(options =>
  {
    options.OpenIddictUIContext = builder =>
      builder.UseSqlite(Configuration.GetConnectionString("DefaultConnection"),
        sql => sql.MigrationsAssembly(typeof(Startup)
                  .GetTypeInfo()
                  .Assembly
                  .GetName()
                  .Name));
  })
  // Register the API for the EF and ASP.NET Identity based UI Store
  .AddUIApis<ApplicationUser>(new OpenIddictUIApiOptions
  {
    // Tell the system about the allowed Permissions it is built/configured for.
    Permissions =
    {
      Permissions.Endpoints.Authorization,
      Permissions.Endpoints.Logout,
      Permissions.Endpoints.Token,
      Permissions.GrantTypes.AuthorizationCode,
      Permissions.GrantTypes.Password,
      Permissions.GrantTypes.RefreshToken,
      Permissions.ResponseTypes.Code,
      Permissions.Scopes.Email,
      Permissions.Scopes.Profile,
      Permissions.Scopes.Roles,
      Permissions.Prefixes.Scope + "demo_api"
    }
  });
  
...
```

## Thoughts and ideas

The project is still very young and there are a lot of ideas like:
- Separating the [ASP.NET Core Identity](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/identity?view=aspnetcore-5.0&tabs=visual-studio) API's from the OpenIddict API's.
- Provide API's for the `Authorization` and `Token` entities (if then really required).
- Scope and permission handling within the sample client is still kind of unclear to me. I am basically waiting for some docs that might be written ;-) or some hints within the code base in the [OpenIddict Core repo]((https://github.com/openiddict/openiddict-core)) to look for.
- Provide a [ASP.NET Razor Page](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-5.0&tabs=visual-studio) based UI.
