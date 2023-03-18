---
title:  "RESTful API"
categories: programming 
tags: F# web bulma
toc: true
---

This contains some thoughts how to implement RESTful APIs.

## Background

tbd

## Best practices

A search for RESTful API best practices reveals many sites:

- [Microsoft Guidelines] are extremely useful and handle tricky stuff like date formats.
- [REST API Tutorial](https://restfulapi.net)
- [Stackoverflow](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/) is a good overview of best practices.
- [Swagger](https://swagger.io/resources/articles/best-practices-in-api-design/)
- [Hevodata] good overview, but pushing own tool

## Resources

Use plural nouns to describe resources

## Verbs

Taken from [Hevodata] and [Microsoft Guidelines] 

| REST Verb | Action | Is Idempotent |
|-----------|--------|--------------|
| GET | fetch one or more values from the resource | true |
| POST | create a new resource (or set of resources) | false |
| PUT | replace an existing object entirely or created a new one, if ID does not exist | true |
| DELETE | delete an existing object | true | 
|      | these are used less frequently | |
| PATCH |  modify a field of an existing resource | false |
| OPTION | get information about a request | true

The first four verbs (GET, POST, PUT, DELETE) respond to the the [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operation of persistent storage.

Questions:
- PUT - how should it behave if the ID does not exist?
- POST - does it create the ID?

### POST

Return `201 Created` and should return the created location 

### GET

tbd 

### PATCH

tbd

## Minimal API

> [Minimal API]s are a simplified approach for building fast HTTP APIs with ASP.NET Core. You can build fully functioning REST endpoints with minimal code and configuration. Skip traditional scaffolding and avoid unnecessary controllers by fluently declaring API routes and actions. 

For now, we show how to implement a file upload in [F#] (see also [Minimal API F#] for more examples).

To create a Minimal API program, use:

~~~csh
% dotnet new web --lang f# -o TodoApi
~~~

This is the whole program:

~~~fsharp
[<EntryPoint>]
let main args =
    let builder = WebApplication.CreateBuilder(args)
    let app = builder.Build()

    app.MapPost("/upload", Func<IFormFile, Task<IResult>>(fun file -> task {
        let tempFile = IO.Path.Join(IO.Path.GetTempPath(), file.FileName)
        app.Logger.LogInformation("tempFile", tempFile, "filename", file.FileName)
        use stream = IO.File.OpenWrite(tempFile)
        file.CopyTo(stream)
        stream.Close()
        return Results.Ok("uploaded " + file.FileName)
    })) |> ignore

    app.Run()

    0 // Exit code
~~~

It reads the posted file and copies the file to a local temporary file. A little text message is displayed in the browser. In a real example, you would parse the stream and do something useful with it. 

The corresponding HMTL looks like this:

~~~html
<form method="post" action="/upload" enctype="multipart/form-data">
    <label>
        Upload file:
        <input type="file" name="file"/>
    </label>
    <button type="submit">Upload the file</button>
</form>
~~~

The code looks much simpler than the corresponding MVC controller. It can also be tested using CURL:

    % curl --form file='@filename' http://localhost:PORT/upload


### Minimal API and HTML content

[Minimal API] is aimed at RESTful APIs, not necessarily at HTML pages. To return a HTML page, use:

~~~fsharp
app.MapGet("/", Func<Task<IResult>>(fun () -> task {
        let page = IO.File.ReadAllText("index.html")
        return Results.Content(page, contentType = "text/html")
    })) |> ignore
~~~

This example can also be found in my [github](https://github.com/mmgreiner/Tutorials/tree/master/MinimalApiPost).

## Miscellaneous

### Simple bulma index.html

Often I look for the simplest `index.html` possible. I use this one with [bulma] CSS framework:

~~~html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Title</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.9.4/css/bulma.min.css">
  </head>
  <body>
  <section class="section">
    <div class="container">
      <h1>De finibus bonorum et malorum</h1>
      <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. </p>
    </div>
  </section>
  </body>
</html>
~~~


## References 

Fielding, Roy Thomas (2000). ["Chapter 5: Representational State Transfer (REST)"](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm). Architectural Styles and the Design of Network-based Software Architectures (Ph.D.). University of California, Irvine.

[F#]: {{ site.fsharp_link }}

[Learn Micfrosoft]: https://learn.microsoft.com/en-us/aspnet/web-api/overview/older-versions/build-restful-apis-with-aspnet-web-api

[Representational State Transfer]: https://en.wikipedia.org/wiki/Representational_state_transfer#Architectural_constraints

[Hevodata]: https://hevodata.com/learn/rest-api-best-practices/

[Microsoft Guidelines]: https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#31-recommended-reading

[Minimal API F#]: https://github.com/teodoran/fsharp-aspnetcore-minimal-apis/blob/main/TodoApi/Program.fs

[Minimal API]: https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/overview?view=aspnetcore-7.0

[bulma]: https://bulma.io
