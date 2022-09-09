# 1

- Reverse proxy: normal proxy proxies clients, reverse proxy proxies services

## Aspnet Core

- There is Kestrel that handles requests as a web server
- There is a set of handlers to handle specific requests sent to app by Kestrel Web server

- HttpContext is just a bag of the request-associated data

- Program and Startup have different responsibilities:
  - Program configures server, logging, IIS integration. _Usually changes rarely_
  - Startup configures middleware pipeline, DI, endpoint configuration. _Changes more often_.

# 2

- There are PageModels, that are something between Model and Controller.
- They bypass Controller call, so rendering is coming straight to Pagemodel

There is also an article about the WebHostBuilder and HostBuilder. Turned out after .net core 3 they merged into one IHostBuilder.

# 3

- Anything configured in ConfigureServices is available in Configure in Startup.cs
- There are PageModels that you can set inside cshtml file, where you add some logic. Basically view logic from controller moves there.
- BindProperty in PageModel allows to bind the input values to properties inside PageModel.
- _ExceptionHandlerMiddleware_ is for prod, _DeveloperExceptionPageMiddleware_ is for dev. _StatusCodePagesMiddleware_ is to show friendly status codes
- HttpContext is what is passed through middleware
- EndpointMiddleware is in the end

- AspnetCore 6 adds DeveloperExceptionPage, and only then everything else
- **Use(Middleware) sets the order of middleware created** So ErrorHandlingMiddleware should go first
- Wow, Configure method in Startup defines the middleware pipeline. Why not call it ConfigureMiddleware?
- In the end there is always a "dummy" middleware that returns 404
- ExceptionHandlerMiddleware reacts only to response from deeper middlewares So ErrorHandling should be FIRST

StatusCodesPageMiddleware is used to show errors containing status codes
UseStatusCodePagesWithReExecute vs UseStatusCodePagesMiddleware?

- ViewModel is used specially for Razor pages. Otherwise it's Model or Contract or whatever

# 4

- Razor pages are grouped by features. No need to look at specific view and specific model etc.

- Mapping query parameters, body, headers and stuff into .net classes is called _model binding_
- MVC is part of EndpointMiddleware pipeline

# 5

- Routing consists of 2 pieces:

  - _EndpointMiddleware_ is used to register all the endpoints (_app.UseEndpoints_)
  - _EndpointRoutingMiddleware_ chooses which of endpoints to executes now (it is also called RoutingMiddleware) (_app.UseRouting_)
    So the RoutingMiddleware is getting the specific endpoint from the dictionary set in EndpointMiddleware, and EndpointMiddleware executes the endpoints

- You can add default value to the route parameter
- You can add optional parameter (with '?')
- Obviously you can specify constraints (type, length and others) for a route parameter
- You can add params-like stuff with {\*\*others} in route. It's called _catch-all parameters_ Example: "products/{\*\*others} will match products/shoes/hats. _Note: catch-all parameters smell badly_
- In catch-all parameters you can use one asterisk or two. One asterisk encodes forward slashes, and two-asterisk don't.
- Route overlapping is pain in the butt; you should avoid it by
- You can customize pascalcase vs dashes in Configure<RouteOptions>.

[v] TODO: after chapter 5 play with middlewares: add custom ones after routing to see what it gives (routing parameters set in RouteValues)

# From https://docs.microsoft.com/en-us/aspnet/core/fundamentals/routing?view=aspnetcore-6.0#route-constraint-reference

- If you don't write UseRouting, the routing middleware will still make it work, but all the custom middlewares will go after _useRouting_
- There is such a thing as terminal middleware. Terminal middleware terminates all the middleware things, returning the response created.

# 6 Model binding

- Algorithm is:
  - Model binding
  - Model validation
  - Page handler
- Binding models can be useful for DRY and some other stuff.
- If you want to bind property for GET requests, don't forget 'SupportsGet=true'

That's all for #6 as the author mostly talks about razorpages there, really hard to distinguish

# 7 Rendering HTML with Razor view

- You can use ViewImports.cshtml to include common stuff like "using" in every view
- ViewStart.cshtml is called before execution of each Razor Page

That's all for Razor views - hopefully I'm not going to write a lot of code using it

# 8 Building forms with tag helpers

Not really relevant - skipped

# 9 Creating a Web Api for mobile/client applications using MVC

- Just like in usual MVC, use AddControllers, UseEndpoints(+MapControllers)
- _Controller_ base class should be used only for server-side MVC: the helpers defined there are useless in ApiController. ApiController should use _ControllerBase_ as parent
- ApiController attribute:

  - wraps not-ok-results into normal JSON class (standard ProblemDetails format; not working for NotFound, for example)
  - Returns 400 automatically if model is invalid
  - Automatically treats input model as [FromBody]
  - You can change the default behaviour for ApiController in ConfigureApiBehaviorOptions

- Token replacement is /api/[controller]

- Determining format of returned value is called _content negotiation_
- Interesting: in Accept headers you can set weights for different content-types
- Whe you return null, the WebApi by default returns NoContent (204)
- If request comes from a browser, typically the response will use just default content-type, ignoring Accept header (but you can configure otherwise in program.cs:
  `services.AddControllers(options => { options.RespectBrowserAcceptHeader = true; });)`

# 10 Service configuration Dependency injection

- Interesting thing, not widely used though: you can put an IEnumerable<T> in a constructor, register several T implementations, and have all of them during runtime:

```
public void ConfigureServices(IServiceCollection services)
{
services.AddControllers();
services.AddScoped<IMessageSender, EmailSender>();
services.AddScoped<IMessageSender, SmsSender>();
services.AddScoped<IMessageSender, FacebookSender>();
}

...
public class UserController:ControllerBase
{
  private readonly IEnumerable<IMessageSender> _messageSenders;
public UserController(
IEnumerable<IMessageSender> messageSenders)
{
_messageSenders = messageSenders;
}
}
```

(Here and all the way down I try to add my feelings and expressions on things I read. That should help me understand the book better)

- The DI just uses last implementation registered for the interface. That's exactly what puzzled me in one of our services, where come cached repo implementation was registered right after the normal one. In that case they had IService, and two implementations ServiceA and ServiceB. But worse: ServiceB was registered the last, _AND_ had IService in its constructor. I personally feel that's a smell.

- Lifetime of a service:
  - Transient
  - Scoped
  - Singleton
- If you inject a scoped dependency into a singleton, the dependency in this scenario will also be a singleton, as it will be created only once. It's called _captured dependency_. So main rule is: inject the more singleton-like services into the transient ones, and not the other way around.
- There is an automatic check for this, but it's slow, so use it only on dev stand (it also has flaws):

```
.UseDefaultServiceProvider(options =>
{
options.ValidateScopes = true;
options.ValidateOnBuild = true;
});
```

# 11 Configuration

- UseDefaultWebHostBuilder uses standard setings for loading configuration.

So, configuration.
You set up the configuration providers in ConfigureAppConfiguration. You set the IConfigurationProviders in the builder, then write _Build_.
There are plenty of configuration providers (XML, Json, CommandLine, Environment etc), but you also can write your own.
CreateDefaultBuilder sets up:

- JSON file provider (appsettings.ENV.json)
- User secrets
- Env variables
- Command line arguments
  If you decide to set all the providers customly in ConfigureAppConfiguration, don't forget to clear() them first

You can address the configuration parameter like
`Configuration["MapSettings:DefaultLocation:Latitude"]`
instead of
`Configuration.GetSection(...)`

- There is such a thing as "User secrets manager", which manages user secrets for the local/development machine
- You can add "ReloadOnChange" flag to the appsettings, to reload automatically, without redeploy, for example
- In order to use settings class and refresh it every time the settings file is changed, use IOptionsSnapshot

The order you add configuration determines the order it is called and overridden

# 12 Saving data with EF Core

- Skipped

# 13 MVC and Razor Pages filter pipeline

- There are filters in aspnetcore.
  There are five types of filters:
- authorization filters (filter out unauthorized requests)
- resource filters (run before model binding and in the end of the request). Use case: add metrics, check content type etc.
- action filters (run after model binding, so you can change the model that comes to controller)
- exception filters (run in case of exception) (_can and mostly should be replaced with exception handling middleware_)
- result filters (if the action method returns result, will be run before and after executing of IActionResult)

Why do we need them?
They are like middleware, but somehow different:

- instead of middleware, you can customize which actions/controllers are gone through the filters. Middleware is applied to _everything_
- filters contain such information as ModelState and IActionResult. Middleware generally can't get this data

So basically, filters are middleware that can be applied to specific actions/controllers. This is the main rule: consider using only if customization needed for different requests.

- Filters are executed in order from most global to most local. You can change this behaviour if your filter implements _IOrderedFilter_
- Filters are basically attributes
- When implementing filter, you can use `context.HttpContext.RequestServices` to get the service from DI. _BUT this is anti-pattern!_
- If you want to catch exceptions in ApiController and return a valid ProblemResult, you should write an ExceptionFilter - especially when you're mixing webapi with razor pages
- Each filter has async brother - better to implement async one, of course
- If you want to run the filter even if some filters 'short circuited', use IAlwaysRunResultFilter
- Short circuiting filters in aspnet core is really messy. If you get there, just open the docs, as it is absolutely _not_ transparent
- DI inside a filter: Use `TypeFilterAttribute` or `ServiceFilterAttribute` for that.

# 14 Authentication

- There is HttpContext that's associated with every request. It has HttpContext.User in it. It is Principal. It is also called ClaimsPrincipal
- ClaimsPrincipal has different Claims in it. Claim is actually any property of a user: email, phone number, and others.
- On authorization we get the principal, encrypt it and send as a cookie
- On subsequent requests browser sends the principal to the server, server decrypts it with AuthenticationMiddleware and checks the permissions. _Important: before the authentication middleware all the middleware see the request as unauthenticated_

AspnetCore Identity is a usual solution when you need an Identity for your services. Almost always it's a good idea to use Identity Server.

Well, there is a lot of info about the Identity Server, how to set it up and all. Probably will forget all of it to the time it's needed.

# 15 Authorization

- Apply [AllowAnonymous] to open publicly endpoints when you need to add [Authorize] attribute globally for the whole service

## 15.2 Handling unauthorized requests

If you're unauthorized, the AuthorizationMiddleware generates one of two different responses:

- Challenge - means that you're not logged in. Response in webapi: 401
- Forbid - means that you don't have enough rights. Response in webapi: 403

When you want to make requests to specific action authorized, you put [Authorize] attribute (of course!)
If you want to be specific (what should the user have), you put the requirement in the brackets inside the attribute. This "requirement" is called "policy"

- You set up which policy requires which claims in `services.AddAuthorization` (requireClaim).
- You also can require role, and not claim - (requireRole) BUT it's legacy and it's advised not to use that stuff.
- You can implement IAuthorizationRequirement to add some logic to the requirement (for example, age of a user)
- If you want to combine different policies, you can
  - use policyBuilder when adding a policy, and
  - write a handler that handles the logic, as IAuthorizationRequirement is purely data
- Important thing that you can write several handlers for one requirement, and if even one of handlers succeed, the whole requirement is met

(stopped here)

TODO: find out difference between websockets and signalr
TODO: try https://docs.microsoft.com/en-us/aspnet/core/web-api/handle-errors?view=aspnetcore-6.0
TODO: bit.ly/designBooksReview
TODO: bit.ly/sysDesignPrimer
TODO: bit.ly/EssArchIntro
TODO: https://github.com/donnemartin/system-design-primer

```

```
