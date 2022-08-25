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

  Page 134 (166)

  TODO: after chapter 5 play with middlewares: add custom ones after routing to see what it gives
  TODO: after chapter 5 - what else does the endpoint routing have? MapConnections? What's that? MapConnectionHandler?

  TODO: try using razorpages to return JSON.
  TODO: find out difference between websockets and signalr
  TODO: try https://docs.microsoft.com/en-us/aspnet/core/web-api/handle-errors?view=aspnetcore-6.0
