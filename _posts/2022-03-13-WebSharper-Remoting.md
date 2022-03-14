---
layout: post
title:  "WebSharper Remoting"
date:   2022-03-13 09:07:00 +0100
categories: websharper removing

---

## WebSharper Remoting

WebSharper JavaScript does not support ``Async.RunSynchronously`` as stated here:

https://developers.websharper.com/docs/v4.x/fs/remoting

```F#
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
```



https://github.com/AlexPeret/websharper-cookbook-tutorial/blob/master/articles/cookbook-chapter-03.md

