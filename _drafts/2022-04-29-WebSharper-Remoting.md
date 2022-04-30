---
layout: post
title:  "WebSharper Remoting - Under construction"
date:   2022-03-13 09:07:00 +0100
categories: websharper fsharp
tags: f# remoting

---

**Still working on this page**

See samples on [exercises][exercises], WSList and SpaList

Note that WebSharper JavaScript does not support ``Async.RunSynchronously`` as stated here [WebSharper Doc]({{site.websharper_link | append: "/remoting"}}).


~~~ fsharp
[<JavaScript>]
type Person = 
    {
        Name: string
        Birthday: DateTime
    }
    static member init name year month day =
        {
            Name = name
            Birthday = DateTime(year, month, day)
        }

module Server =

    let persons = 
        [
            Person.init "Donald Knuth" 1938 1 10
            Person.init "Barbara Liskov" 1039 11 7
            Person.init "Tony Hoare" 1934 1 11
            Person.init "Niklaus Wirth" 1934 2 15
            Person.init "Susan L. Graham" 1942 9 16 
        ]

    [<Remote>]
    let GetPersons () =
        async {
			// just testing with some delay
            let! sl = Async.Sleep 2000 
            return persons
        }
~~~



Topics:

- `list` cannot be converted to JSON internally, use array or ??
- or do I have to add FSharp.Types?
- Same problem for `Option`
- On client side, you cannot use Async.RunSynchronously. Need a reactive variable and load at the start.
- 


### Authentication

See [WebSharper Cookbook][cookbook]


[cookbook]: https://github.com/AlexPeret/websharper-cookbook-tutorial/blob/master/articles/cookbook-chapter-03.md

[exercises]: {{site.exercise_link}}

