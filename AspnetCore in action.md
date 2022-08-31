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

- (page 223)

TODO: find out difference between websockets and signalr
TODO: try https://docs.microsoft.com/en-us/aspnet/core/web-api/handle-errors?view=aspnetcore-6.0
