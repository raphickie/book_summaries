# 1

- Reverse proxy: normal proxy proxies clients, reverse proxy proxies services

## Aspnet Core

- There is Kestrel that handles requests as a web server
- There is a set of handlers to handle specific requests sent to app by Kestrel Web server

- HttpContext is just a bag of the request-associated data

- Program and Startup have different responsibilities:
  - Program configures server, logging, IIS integration. _Usually changes rarely_
  - Startup configures middleware pipeline, DI, endpoint configuration. _Changes more often_.

There is also an article about the WebHostBuilder and HostBuilder. Turned out after .net core 3 they merged into one IHostBuilder.

- Anything configured in ConfigureServices is available in Configure in Startup.cs
- There are PageModels that you can set inside cshtml file, where you add some logic. Basically view logic from controller moves there.
- BindProperty in PageModel allows to bind the input values to properties inside PageModel.
- _ExceptionHandlerMiddleware_ is for prod, _DeveloperExceptionPageMiddleware_ is for dev. _StatusCodePagesMiddleware_ is to show friendly status codes
- HttpContext is what is passed through middleware
- EndpointMiddleware is in the end

- AspnetCore 6 adds DeveloperExceptionPage, and only then everything else
- **Use(Middleware) sets the order of middleware created** So ErrorHandlingMiddleware should go first

# 2

There are PageModels, that are something between Model and Controller.
They bypass Controller call, so rendering is coming straight to Pagemodel

# 3

Page 65
