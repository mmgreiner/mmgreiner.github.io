---
title:  "Bolero hosting Razor page"
categories: programming
tags: F# C# Bolero Bulma
toc: true
---

In this post we investigate how to use Razor pages on the host and [Bolero] WebAssembly on the client side. The final code can be found on [My github - Tutorials - BoleroAppRazor](https://github.com/mmgreiner/Tutorials).


## Create Bolero Razor hosting

This is described in more detail in the [Bolero hosting documentation](https://fsbolero.io/docs/Hosting#razor). To create such a project, use:

    dotnet new bolero-app --hostpage=razor -o BoleroAppRazor

This creates the following files:

~~~~
.
├── BoleroAppRazor.sln
└── src
    ├── BoleroAppRazor.Client
    │   ├── BoleroAppRazor.Client.fsproj
    │   ├── Main.fs
    │   ├── Startup.fs
    │   ├── bin
    │   └── wwwroot
    │       ├── css
    │       │   └── index.css
    │       ├── favicon.ico
    │       └── main.html
    └── BoleroAppRazor.Server
        ├── BoleroAppRazor.Server.fsproj
        ├── BookService.fs
        ├── Pages
        │   └── _Host.cshtml
        ├── Startup.fs
        ├── bin
        ├── data
        │   └── books.json
~~~~

The initally served file is ``_Host``.

Unfortunately, when you try to run this program, it throws the following error:

~~~~
dotnet run --project src/*Server
info: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[62]
      User profile is available. Using '/Users/mmgreiner/.aspnet/DataProtection-Keys' as key repository; keys will not be encrypted at rest.
Hosting environment: Production
Content root path: .../src/BoleroAppRazor.Server
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/1.1 GET http://localhost:5000/ - -
fail: Microsoft.AspNetCore.Server.Kestrel[13]
      Connection id "0HMPQBS3HOHR9", Request id "0HMPQBS3HOHR9:00000001": An unhandled exception was thrown by the application.
      Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation.CompilationFailedException: One or more compilation failures occurred:
      /Users/mmgreiner/Projects/Scrap/BoleroAppRazor/src/BoleroAppRazor.Server/Pages/_Host.cshtml(3,21): error CS0234: The type or namespace name 'RazorHost' does not exist in the namespace 'Bolero.Server' (are you missing an assembly reference?)
~~~~

You have to change ``@using Bolero.Server.RazorHost`` to ``Bolero.Server``.

~~~html
@page "/"
@namespace BoleroAppRazor.Server
@using Bolero.Server.RazorHost
@inject IBoleroHostConfig BoleroHostConfig
<!DOCTYPE html>
<html>
...
~~~

to

~~~html
@page "/"
@namespace BoleroAppRazor.Server
@using Bolero.Server
@inject IBoleroHostConfig BoleroHostConfig
<!DOCTYPE html>
<html>
...
~~~

Now you will get the error:

~~~~
Hosting environment: Production
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/1.1 GET http://localhost:5000/ - -
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint '/_Host'
info: Microsoft.AspNetCore.Mvc.RazorPages.Infrastructure.PageActionInvoker[103]
      Route matched with {page = "/_Host"}. Executing page /_Host
info: Microsoft.AspNetCore.Mvc.RazorPages.Infrastructure.PageActionInvoker[107]
      Executing an implicit handler method - ModelState is Valid
info: Microsoft.AspNetCore.Mvc.RazorPages.Infrastructure.PageActionInvoker[109]
      Executed an implicit handler method, returned result Microsoft.AspNetCore.Mvc.RazorPages.PageResult.
info: Microsoft.AspNetCore.Mvc.RazorPages.Infrastructure.PageActionInvoker[104]
      Executed page /_Host in 235.5552ms
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint '/_Host'
fail: Microsoft.AspNetCore.Server.Kestrel[13]
      Connection id "0HMPQBUHEQ0DL", Request id "0HMPQBUHEQ0DL:00000001": An unhandled exception was thrown by the application.
      System.MissingMethodException: Method not found: 'Elmish.Program`4<!!3,!!4,!!5,!!7> Elmish.ProgramModule.map .....
      
~~~~

The simplest way to correct is is to change ``BoleroAppRazor.Client/Main.fs``.

~~~fsharp
type MyApp() =
    inherit ProgramComponent<Model, Message>()

    override this.Program =
        let bookService = this.Remote<BookService>()
        let update = update bookService
        Program.mkProgram (fun _ -> initModel, Cmd.ofMsg GetSignedInAs) update view
        |> Program.withRouter router
#if DEBUG
        |> Program.withHotReload
#endif

~~~

Remove the ``withHotReload``.

Now everything works fine. The server side looks like this:

~~~~fsharp
type Startup() =

    // This method gets called by the runtime. Use this method to add services to the container.
    member this.ConfigureServices(services: IServiceCollection) =
        services.AddMvc().AddRazorRuntimeCompilation() |> ignore
        services.AddServerSideBlazor() |> ignore
        services
            .AddAuthorization()
            .AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
                .AddCookie()
                .Services
            .AddBoleroRemoting<BookService>()
            .AddBoleroHost()
#if DEBUG
            .AddHotReload(templateDir = __SOURCE_DIRECTORY__ + "/../BoleroAppRazor2.Client")
#endif
        |> ignore

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    member this.Configure(app: IApplicationBuilder, env: IWebHostEnvironment) =
        if env.IsDevelopment() then
            app.UseWebAssemblyDebugging()

        app
            .UseAuthentication()
            .UseStaticFiles()
            .UseRouting()
            .UseAuthorization()
            .UseBlazorFrameworkFiles()
            .UseEndpoints(fun endpoints ->
#if DEBUG
                endpoints.UseHotReload()
#endif
                endpoints.MapBoleroRemoting() |> ignore
                endpoints.MapBlazorHub() |> ignore
                endpoints.MapFallbackToPage("/_Host") |> ignore)
        |> ignore

module Program =

    [<EntryPoint>]
    let main args =
        WebHost
            .CreateDefaultBuilder(args)
            .UseStaticWebAssets()
            .UseStartup<Startup>()
            .Build()
            .Run()
        0
~~~~

## Change pages

The pages typically consist of a `.cshtml` file and the corresponding C# file `.cshtml.cs`. For our purpose, we have to change all these to [F#][fsharp]. For instance:

~~~csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace BoleroAppRazor.Server.Pages;

public class IndexModel : PageModel
{
    private readonly ILogger<IndexModel> _logger;

    public IndexModel(ILogger<IndexModel> logger)
    {
        _logger = logger;
    }

    public void OnGet()
    {
    }
}
~~~
is changed to:

~~~fsharp
namespace BoleroAppRazor.Server.Pages

open Microsoft.AspNetCore.Mvc
open Microsoft.AspNetCore.Mvc.RazorPages

open Microsoft.Extensions.DependencyInjection
open Microsoft.Extensions.Logging

type IndexModel(_logger: ILogger<IndexModel>) =
    inherit PageModel()

    member this.OnGet() = ()

~~~

Don't forget to add them to the project file `.fsproj`:

~~~xml
 <ItemGroup>
    <Compile Include="Pages/Index.cshtml.fs" />
    <Compile Include="Pages/Error.cshtml.fs" />
    <Compile Include="Pages/Privacy.cshtml.fs"/>
    <Compile Include="BookService.fs" />
    <Compile Include="Startup.fs" />
  </ItemGroup>
~~~

## Change layout

Change the starting page:

~~~fsharp
            .UseEndpoints(fun endpoints ->
#if DEBUG
                endpoints.UseHotReload()
#endif
                endpoints.MapBoleroRemoting() |> ignore
                endpoints.MapBlazorHub() |> ignore
                //endpoints.MapFallbackToPage("/_Host") |> ignore
                endpoints.MapRazorPages() |> ignore // added
            )
~~~

The new `_layout.cshtml` will only call `@RenderBody()`.  The index page `Index.cshtml` will be calling the [Bolero] Client.

~~~fsharp
@page
@model IndexModel
@namespace BoleroAppRazor.Server.Pages
@using Bolero.Server
@inject IBoleroHostConfig BoleroHostConfig
@{
    ViewData["Title"] = "Home page";
}

<div class="content">
    <div id="main1">
        @(await Html.RenderComponentAsync<BoleroAppRazor.Client.Main.MyApp>(BoleroHostConfig))
    </div>

</div>

@Html.RenderBoleroScript(BoleroHostConfig)
~~~

## Aside: Bulma and navigation

On server, `wwwroot/js/nav_bulma.js`. Taken from [Bulma Navbar](https://bulma.io/documentation/components/navbar/). 


~~~javascript
document.addEventListener('DOMContentLoaded', () => {

    // Get all "navbar-burger" elements
    const $navbarBurgers = Array.prototype.slice.call(document.querySelectorAll('.navbar-burger'), 0);

    // Add a click event on each of them
    $navbarBurgers.forEach(el => {
        el.addEventListener('click', () => {

            // Get the target from the "data-target" attribute
            const target = el.dataset.target;
            const $target = document.getElementById(target);

            // Toggle the "is-active" class on both the "navbar-burger" and the "navbar-menu"
            el.classList.toggle('is-active');
            $target.classList.toggle('is-active');

        });
    });
});
~~~

[fsharp]: {{ site.fsharp_link }}
[Bolero]: {{site.bolero_link}}
[Bulma]: https://bulma.io/
