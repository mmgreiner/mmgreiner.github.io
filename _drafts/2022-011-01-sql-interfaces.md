---
categories:
    - programming
    - database
tags:
    - F#
    - C#
    - SQLProvider
    - Dapper
title: Sql F# interfaces
toc: true
---

This is a collection of SQL interfaces in [F#]. A good overview can be found at the [FSharp DataAccess guides](https://fsharp.org/guides/data-access/). 

Another introduction, albeit from 2014, is the blog [Using F# for database related tasks](https://fsharpforfunandprofit.com/posts/low-risk-ways-to-use-fsharp-at-work-4/) in the [F# for fun and profit](https://fsharpforfunandprofit.com/) series.

Particular for [F#], the following questions have to be looked at:

- support of record types?
- support of `Option` types?
- support of [discriminated union](https://fsharpforfunandprofit.com/posts/discriminated-unions/) types?
- need to write SQL statements?

## Entity Framework Core

TBD

## FSharp.Data.SqlClient

<https://github.com/fsprojects/FSharp.Data.SqlClient>

TBD

## SQLProvider

<http://fsprojects.github.io/SQLProvider/>

> This repository contains the source for the Old F# type providers called SqlDataConnection, SqlEntityConnection, ODataService, WsdlService and EdmxFile.



## SqlDataConnection Type Provider

<https://fsprojects.github.io/FSharp.Data.TypeProviders/sqldata.html>

TBD

## Dapper

[Dapper] is a micro-[ORM](https://en.wikipedia.org/wiki/Object–relational_mapping) "created by the people behind Stackoverflow" (see [Learn Dapper](https://www.learndapper.com)).   

[Dapper.FSharp] is a

> Lightweight F# extension for StackOverflow Dapper with support for MSSQL, MySQL, PostgreSQL and SQLite

Sample:

~~~fsharp
select {
    for p in personTable do
    selectAll
} |> conn.SelectAsync<Person>
~~~

## Comparison

<http://fsprojects.github.io/FSharp.Data.SqlClient/comparison.html>

[F#]: {{ site.fsharp_link }}
[Dapper]: https://github.com/DapperLib/Dapper
[Dapper.FSharp]: https://github.com/Dzoukr/Dapper.FSharp
