# Serilog.AspNetCore.Enrichers.CorrelationId

(fork of Serilog.Enrichers.CorrelationId)

Enriches Serilog events with a correlation ID for tracking requests.

To use the enricher, first install the NuGet package:

```powershell
Install-Package Serilog.AspNetCore.Enrichers.CorrelationId
```

Then, apply the enricher to your `LoggerConfiguration`:

```csharp
Log.Logger = new LoggerConfiguration()
    .Enrich.WithCorrelationId()
    // ...other configuration...
    .CreateLogger();
```

The `WithCorrelationId()` enricher will add a `CorrelationId` property to produced events.

### Included enrichers

The package includes:

 * `WithCorrelationId()` - adds a `CorrelationId` to track logs for the current web request.
 * `WithCorrelationIdHeader(headerKey)` - adds a `CorrelationId` extracted from the current request header (or created if one does not exist).

## Installing into an ASP.NET Core Web Application

This is what your `Startup` class should contain in order for this enricher to work as expected:

```cs
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Serilog;

namespace MyWebApp
{
    public class Startup
    {
        public Startup()
        {
            Log.Logger = new LoggerConfiguration()
                .MinimumLevel.Debug()
                .WriteTo.Console(outputTemplate: "[{Timestamp:HH:mm:ss} {CorrelationId} {Level:u3}] {Message:lj}{NewLine}{Exception}")
                .Enrich.WithCorrelationId()
                .CreateLogger();
        }

        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
        public void ConfigureServices(IServiceCollection services)
        {
            // ...
            services.AddHttpContextAccessor();
            // ...
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
        {
            // ...
            loggerFactory.AddSerilog();
            // ...
        }
    }
}
```

You need to register the `IHttpContextAccessor` singleton so that the enricher has access to the requests `HttpContext` so that it can attach the correlation ID to the request/response.
