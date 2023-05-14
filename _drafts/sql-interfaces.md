---
title:  "Sql F# interfaces"
categories: programming 
tags: F# C#    
toc: true
---

This is a collection of SQL interfaces in [F#].

Good overview: <https://fsharp.org/guides/data-access/>


## FSharp.Data.SQLProvider

<http://fsprojects.github.io/SQLProvider/>




## Dapper

<https://github.com/Dzoukr/Dapper.FSharp> 

> Lightweight F# extension for StackOverflow Dapper with support for MSSQL, MySQL, PostgreSQL and SQLite

Sample:

~~~fsharp
select {
    for p in personTable do
    selectAll
} |> conn.SelectAsync<Person>
~~~

[F#]: {{ site.fsharp_link }}