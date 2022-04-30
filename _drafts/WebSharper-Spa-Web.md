---
layout: post
title:  "WebSharper Spa vs Web"
date:   2022-04-28
categories: websharper fsharp
tags: f# remoting
---

Compare Single Page application to Web Application

## Startup.fs

Single Page: 

~~~ fsharp
type Startup() =

    member this.ConfigureServices(services: IServiceCollection) =
        ()
    member this.Configure(app: IApplicationBuilder, env: IWebHostEnvironment) =
        if env.IsDevelopment() then app.UseDeveloperExceptionPage() |> ignore
        // ...

module Program =

    [<EntryPoint>]
    let main args =
        WebHost
            .CreateDefaultBuilder(args)
            .UseStartup<Startup>()
            .Build()
            .Run()
        0
~~~

Web:

~~~ fsharp
type Startup() =

    member this.ConfigureServices(services: IServiceCollection) =
        services.AddSitelet(Site.Main)
            .AddAuthentication("WebSharper")
            .AddCookie("WebSharper", fun options -> ())
        |> ignore

    member this.Configure(app: IApplicationBuilder, env: IWebHostEnvironment) =
        if env.IsDevelopment() then app.UseDeveloperExceptionPage() |> ignore

module Program =
    let BuildWebHost args =
        WebHost
            .CreateDefaultBuilder(args)
            .UseStartup<Startup>()
            .Build()

    [<EntryPoint>]
    let main args =
        BuildWebHost(args).Run()
        0

~~~ 

For single page, Sitelet needs not be added as Service.

## Client side

Single Page
~~~ fsharp
[<JavaScript>]
module Client = 
    [<SPAEntryPoint>]
    let Main () =
        // ...
         MainTemplate.Main()
            .Title()
            // ...
            .Doc()
        |> Doc.RunById "main"
~~~

Web:

~~~ fsharp
[<JavaScript>]
module Client =
let Main () =
        let rvReversed = Var.Create ""
        MainTemplate.Main()
            .Title()
            // ...
            .Doc()
~~~
