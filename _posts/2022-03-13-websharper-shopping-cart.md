---
layout: post
title:  "Websharper Shopping cart"
date:   2022-03-12 10:41:39 +0100
categories: websharper
tags: f# template
---

## Websharper Shopping cart

I found an interesting article from 2020 by Adam Granizc on the implementation of a WebSharper shopping cart in https://github.com/intellifactory/blogs/blob/master/user/granicz/20201231-variations-for-a-websharper-shopping-cart.md

Unfortunately, the code was not published, and as an exercise, I wanted to re-implement it in two ways:

- as shown in the article using the reactive UI
- as an elmish MVU implementation

I found the code later:

https://github.com/websharper-samples/ShoppingCart

### Confustion about HTML Templating

WebSharper allows several ways to instatiate HTML, and It took me some thinking to get these terms right:

- `ws-replace`
- `ws-hole`
- `ws-template`
- `ws-template-children`

For quick reference, I created this table:

#### Replace

```html
<div ws-replace="Body">
</div>
```

```F#
MyTemplate()
	.Body(h1 [] [text "Hallo"])
	.Doc()
```

```html
<h1>Hallo</h1>
```
#### Hole

```html
<div ws-hole="Body">
</div>
```

```F#
MyTemplate().Body(h1 [] [text "Hallo"])
```

```html
<div>
  <h1>Hallo</h1>
</div>
```

#### Template

```html
<div ws-template="MyParagraph">
  <p>
    Hallo ${Name}
  </p>
</div>
```

```F#
MyTemplate.MyParagraph().Name("Markus")
```

```html
<div>
  <p>
    Hallo Markus
  </p>
</div>
```

#### Holes and Templates

Holes and templates are often used to fill lists:

```html
<div>
  <ul>
    <div ws-replace="MyItems"></div>
    <div ws-template="MyItem">
      <li>Item: ${Name}</li>
    </div>
  </ul>
</div>
<div
```

```F#
let viewMyItem item =
	MyTemplate.MyItem()
		.Name(item)
		.Doc()
 
MyTemplate.MyItems(
	[ "Apples"; "Pears"] 
	|> Seq.map viewMyItem
	|> Doc.Concat
)
.Doc()
```

```html
<div>
  <h1>Hallo</h1>
</div>
```


Hole

