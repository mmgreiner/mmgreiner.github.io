---
title:  "Json Serialization"
categories: programming
tags: F# Json
toc: true
---

Best help: [FSharp.System.TextJson]

~~~fsharp
#r "nuget: FSharp.SystemTextJson";; 
open System.Text.Json
open System.Text.Json.Serialization

let options = JsonSerializerOptions()
options.Converters.Add(JsonFSharpConverter())

let me = {| MiddleName = Some "Max"; Birtday = System.DateTime(1965, 11, 10) |}
let you = {| MiddleName = Option<string>.None; Birtday = System.DateTime(1980, 1, 31) |}
JsonSerializer.Serialize(me, options)
// val it: string = "{"Birtday":"1965-11-10T00:00:00","MiddleName":"Max"}"
JsonSerializer.Serialize(you, options)
// val it: string = "{"Birtday":"1980-01-31T00:00:00","MiddleName":null}"
~~~

## Json serialization with Bolero

tbd

## Json Serialization with Swagger / OpenAPI

[Web-Api-Swagger](https://learn.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger?view=aspnetcore-6.0)

[FSharp.System.TextJson]: https://github.com/Tarmil/FSharp.SystemTextJson
