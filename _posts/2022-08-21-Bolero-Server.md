---
layout: post
title:  "Bolero Post Endpoint"
date:   2022-08-21
categories: bolero fsharp
tags: f# bolero file-upload

---

## Bolero Server endpoint for Posts

I came across this topic trying to implement a file upload in [Bolero]. The It starts with the problem that ``<input type="file" />``is not supported by [Bolero].

Searching for a solution, I found this conversation about [Bolero uploading files](https://github.com/fsbolero/Bolero/issues/66), which made the comment to create a ASP.NET Core endpoint. This post explores how to do this.

### The HTML

We use a standard html form with post to upload the file:

~~~ html
<form action="Uploader/PostFiles" method="post" enctype="multipart/form-data">
    <label>
        Data files
        <input type="file" multiple accept=".csv,.json" id="myfiles" name="myfiles"/>
    </label>
    <button type="submit">Upload files</button>
</form>
~~~

Remember the name ``myfiles``. It will be used later on in the ASP.NET controller.

### The ASP.NET controller

It wasn't easy to find the simplest tutorial on how to write an ASP.NET controller in F#. In the end, [stackoverflow](https://stackoverflow.com/questions/39626220/f-syntax-for-async-controller-methods-in-asp-net-core) was helpful, as was this Microsoft introduction to getting started with [web-app and f#](https://docs.microsoft.com/en-us/visualstudio/get-started/tutorial-fsharp-web-app?view=vs-2022)
and this tutorial [MVC file-upload](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads?view=aspnetcore-6.0)
Here is our simple controller:

~~~ fsharp
namespace HalloController.Server

open System
open Microsoft.AspNetCore.Mvc
open Microsoft.Extensions.Logging

[<ApiController>]
[<Route("[controller]")>]
type UploaderController (logger : ILogger<UploaderController>) =
    inherit ControllerBase()

    [<HttpGet>]         // match GET Uploader/
    member _.Get() =
        logger.LogInformation("Inside UploadController Get")
        "this is a simple GET"

    [<HttpGet("Info")>]     // match GET Uploader/Info
    member _.Info() =
        "this is a GET Info"

    // important: parameter name has to match id and name in Http <input>
    [<HttpPost("PostFiles")>]
    member _.PostFiles(myfiles: System.Collections.Generic.List<Microsoft.AspNetCore.Http.IFormFile>) =
        logger.LogInformation("Inside Uploader PostFiles")
        for f in myfiles do
            printfn "filename = %s" f.FileName
        "success"
~~~

It took me some time to understand how the routes are mapped.
- The class name ``UploaderController`` provides the route part ``Uploader``.
- the member attribute provides the next route part ``Info`` or ``PostFiles``.
- So the complete routes are: http://localhost:12345/Uploader/Info.

It also took some time to discover that the name of the parameter of the post, ``myfiles`` must match exactly the name of the ``<input type="file" name="myfiles" />`` for the parameter to be set.

### The server side

So how do we make the server recognize our controller? This is where the magic of dependency injection kicks in. We just have to add it to the services and endpoints.

For the services, add the line ``services.AddControllers()`` to the ConfigureServices member.

For the endpoints, add the line ``endpoints.MapControllers()`` to the mapped endpoints.

~~~ fsharp
type Startup() =

    // This method gets called by the runtime. Use this method to add services to the container.
    // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
    member this.ConfigureServices(services: IServiceCollection) =
        services.AddControllers() |> ignore     // added
        services.AddMvc() |> ignore
        services.AddServerSideBlazor() |> ignore
        services
            .AddAuthorization()
            .AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
                .AddCookie()
                .Services
            .AddRemoting<BookService>()
            .AddBoleroHost()
#if DEBUG
            .AddHotReload(templateDir = __SOURCE_DIRECTORY__ + "/../HalloController.Client")
#endif
        |> ignore

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    member this.Configure(app: IApplicationBuilder, env: IWebHostEnvironment) =
        app
            .UseAuthentication()
            .UseRemoting()
            .UseStaticFiles()
            .UseRouting()
            .UseBlazorFrameworkFiles()
            .UseEndpoints(fun endpoints ->
#if DEBUG
                endpoints.UseHotReload()
#endif
                endpoints.MapControllers() |> ignore    // added
                endpoints.MapBlazorHub() |> ignore
                endpoints.MapFallbackToBolero(Index.page) |> ignore)
        |> ignore
~~~

That's it! 

Now when you run the server, and post the files, ASP.NET will log the following message:

~~~ bash
info: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[3]
    Route matched with {action = "PostFiles", controller = "Uploader"}. Executing controller action with signature Void PostFiles(System.Collections.Generic.List`1[Microsoft.AspNetCore.Http.IFormFile]) on controller HalloController.Server.UploaderController (HalloController.Server).
~~~


[receive-file]: https://makolyte.com/aspnetcore-receive-a-file-in-the-request/
[Bolero]: https://fsbolero.io/docs/

[exercises]: {{site.exercise_link}}

