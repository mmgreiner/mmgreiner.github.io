---
title:  "WebSharper templates"
date:   2022-03-13 13:00:00 +0100
categories: programming
tags: F# template websharper
---


WebSharper allows several ways to instatiate HTML, and It took me some thinking to get these terms right:

- `ws-replace`
- `ws-hole`
- `ws-template`
- `ws-template-children`

## Replace

Replace the given node with the provided string or list of docs.

```html
<div ws-replace="Body">
</div>
```

```fsharp
MyTemplate()
	.Body(h1 [] [text "Hallo"])
	.Doc()
```

```html
<h1>Hallo</h1>
```
## Hole

Fills the child nodes with the given string or sequence of `Doc`s.

```html
<div ws-hole="Body">
</div>
```

```fsharp
MyTemplate().Body(h1 [] [text "Hallo"])
```

```html
<div>
  <h1>Hallo</h1>
</div>
```

## Template

```html
<div ws-template="MyParagraph">
  <p>
    Hallo ${Name}
  </p>
</div>
```

```fsharp
MyTemplate.MyParagraph().Name("Markus")
```

```html
<div>
  <p>
    Hallo Markus
  </p>
</div>
```

## Holes and Templates

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

```fsharp
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


