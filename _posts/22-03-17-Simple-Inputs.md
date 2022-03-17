---
layout: post
title:  "Simple Inputs"
categories: websharper
tags: input html5
---

## WebSharper simple inputs

### Client side

On the client side, you can combine [reactive variables][ws-inputs] with input html elements.

```html
<label>
  Any text:
  <input type="text" ws-var="T1" placeholder="enter any text">
</label>
<label>
  Any number:
  <input type="number" ws-var="N1">
</label>
<label>
  Any date:
  <input type="date" ws-var="D1">
</label>
<label>
  Phone number: 
  <input type="tel" ws-var="Tel" placeholder="Phonenumber" />
</label>
<label>
  Password:
  <input type="password" ws-var="PW">
</label>
<label>
  OnInput:
  <input type="text" ws-oninput="OnInput" />
</label>
<label>
  Checkbox:
  <input type="checkbox" ws-var="Check" />
</label>
<label>
  Selection:
  <select ws-var="Color">
    <option>green</option>
    <option>blue</option>
    <option>yellow</option>
  </select>
</label>
<p>
  Text = ${T1}, Number = ${N1}, Check = ${Check}
</p>
```

And in the code:

```fsharp
[<JavaScript>]
module Client =
    let t1 = Var.Create ""
    let n1 = Var.Create 0
    let d1 = Var.Create ""
    let tel = Var.Create ""
    let pw = Var.Create ""
    let check = Var.Create true
    let color = Var.Create ""
    let n2: Var<Client.CheckedInput<int>> = Var.Create (Client.CheckedInput<int>.Blank "0")
    
    let Main () =
        MainTemplate()
          .T1(t1)
          .N1(n1)
          .D1(d1)
          .Tel(tel)
          .PW(pw)
          .Check(check)
          .Color(color)
          .N2(n2)
          .OnInput(fun ev -> printfn "ev = %A" ev.Target)
          .Doc()
```



[ws-inputs]: https://developers.websharper.com/docs/v4.x/fs/ui

