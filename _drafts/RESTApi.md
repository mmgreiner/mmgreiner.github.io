---
title:  "RESTful API"
categories: programming 
tags: F# web 
toc: true
---

This contains some thoughts how to implement RESTful APIs.

## Background


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

### PATCH

## Overview


## References 

Fielding, Roy Thomas (2000). ["Chapter 5: Representational State Transfer (REST)"](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm). Architectural Styles and the Design of Network-based Software Architectures (Ph.D.). University of California, Irvine.

[fsharp]: {{ site.fsharp_link }}

[Learn Micfrosoft]: https://learn.microsoft.com/en-us/aspnet/web-api/overview/older-versions/build-restful-apis-with-aspnet-web-api

[Representational State Transfer]: https://en.wikipedia.org/wiki/Representational_state_transfer#Architectural_constraints

[Hevodata]: https://hevodata.com/learn/rest-api-best-practices/

[Microsoft Guidelines]: https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#31-recommended-reading

[Minimal API F#]: https://github.com/teodoran/fsharp-aspnetcore-minimal-apis/blob/main/TodoApi/Program.fs
