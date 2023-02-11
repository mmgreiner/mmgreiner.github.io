---
title:  "Input file element with Bolero"
categories: programming
tags: input html5 bolero F#
toc: true
---

> [Bolero] is a set of free and open-source libraries and tools to write web applications in [F#]. With it, you can write fully dynamic client-side web applications from the comfort of a strongly-typed functional language, all with great performance.

You develop [F#] code for the client side (inside the browser), which is compiled into [WebAssembly]. The views are either F# codes or Html templates.

The client contains predefined `<input>` types for text, numbers and booleans, but not for file uploads. How can this be done?

## Client and Server

[Bolero] uses the [Elmish] Model-View-Update architecture. 

You develop [F#] code for the **client** side (inside the browser), which is compiled into [WebAssembly]. The views are either F# codes or Html templates.

The client communicates with the server using [SignalR](https://dotnet.microsoft.com/apps/aspnet/signalr) connections. The server uses standard ASPNET. 

There are two potential approaches:

1. Upload a file to the client and use the file's data on the client.

2. Upload a file to the client and pass it on to the server for more complex file data procedures.

## Simple example

It contains predefined `<input>` types for text, numbers and booleans, but not for file uploads. How can this be done?

### On client

~~~fsharp
type Model = { Name: string }
let init = { Name = "" }

type Message =
    | SetName of string

let update message model =
    match message with
    | SetName n -> { model with Name = n }

let viewInput modelData setValue =
    input {
        attr.value modelData
        on.change (fun e -> setValue (unbox e.Value))
    }

let view model dispatch =
    div {
        input {
            attr.value = model.Name
            on.change (fun e -> dispatch (SetName (unbox e.Value)))
        }
        $"Hallo, {model.Name}!"
    }
~~~

Now the problem is, that there exist views for `<input>` types string, numbers and booleans, but not for file input.

## Approach 1: Using JavaScript

TBD

## Approach 2: Using Blazor component

TBD

## Approach 3: Using Server posts

An example for this can be found on my [Github/BoleroApp]({{site.greiner_link}}/BoleroApp) repository. It implements a simple AspNet MVC controller for file uploads.

See also this [post]({% post_url 23-02-10-github-fsharp %}).

## Further information

See <https://procodeguide.com/programming/file-upload-in-aspnet-core/> for an example how to implement file upload in Asp.Net Core 6.0.


[Bolero]: https://fsbolero.io
[F#]: https://fsharp.org/
[Elmish]: https://elmish.github.io/elmish/
[WebAssembly]: https://webassembly.org/
