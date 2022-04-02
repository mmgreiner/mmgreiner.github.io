---
layout: post
title:  "WebSharper Remoting"
date:   2022-03-13 09:07:00 +0100
categories: websharper fsharp
tags: f# remoting

---

Note that WebSharper JavaScript does not support ``Async.RunSynchronously`` as stated here [WebSharper Doc]({{site.websharper_link | append: "/remoting"}}).


~~~ fsharp
module Server =

	[<Remote>]
	let getItems () =
		let items = [ "apples"; "pears" ]
		async { return items }

[<JavaScript>]
module Client = 
	let items = 
		async {
		
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

