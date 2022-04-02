---
layout: post
title:  "WebSharper simple inputs"
categories: websharper
tags: input html5
---

This post shows various ways to handle input using [WebSharper]({{site.websharper_link}}).

## Client side

There are two basic approaches: using HTML templates or using Websharper's HTML.

HTML templates give a clean separation between design and function, and you can change the HTML file without recompiling. But there are some limits to it also. 

### HTML Templating

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


### Handling of *select*

The HTML code is straight forward:

```html
<label>
  Primary Colors:
    <select ws-var="PrimaryColor" ws-hole="PrimaryColorList">
    </select>
</label>
```

However, the following fsharp will fail:

```fsharp
[<JavaScript>]
...
type Color = Red | Yellow | Blue
let primary = Var.Create PrimaryColors.Blue
let primaryColors = 
  FSharp.Reflection.FSharpType.GetUnionCases typeof<PrimaryColors>
  |> Array.map (fun uc -> uc.Name)
  |> Array.map (fun col -> Tags.option [] [text col])

Template()
  .PrimaryColor(primary)
  .PrimaryColorsList(primaryColors)
```

It fails for two reasons:

- HTML templating can only handle variables of numeric or string types
- FSharpType does not exist for JavaScript. So during compilation, it will claim `WebSharper error FS9001: Type not found in JavaScript compilation:  Microsoft.FSharp.Reflection.FSharpType`.


So try this:

```fsharp
let primary = Var.Create (sprintf "%A" PrimaryColors.Blue)

let primaryColors = 
  [ Red; Yellow; Blue]
  |> Seq.map (sprintf "%A")   
  |> Seq.map (fun col -> Tags.option [] [text col])

```

Unfortunately, i did not find an easy way to enumerate over the union, therefore the explicit list `[ Red; Yellow; Blue]`.

## WebSharper HTML

to be done

## File input

To use input type `file` for uploading a file, there are two possibilities: posting a form and handle the input in the server, or handle it on the client.

Here we discuss handling it on a client. Further information can be found in at [Mozilla][mozilla-file] or on [W3Docs](https://www.w3docs.com/learn-javascript/file-and-filereader.html).


In JavaScript, the corresponding code looks as shown below. The actual content of the file is given in the `fileReader.result`.

````html
<body>
    <input onchange="readFile(this)" type="file">
    <script>
      function readFile(input) {
        let file = input.files[0]; 
        let fileReader = new FileReader(); 
        fileReader.readAsText(file); 
        fileReader.onload = function() {
          alert(fileReader.result);
        }; 
        fileReader.onerror = function() {
          alert(fileReader.error);
        }; 
      }
    </script>
  </body>
````

In the WebSharper html template, we use an `ws-oninput` trigger.

```html
<label>
    file Load:
    <input type="file" ws-oninput="LoadFile">
</label>
```

In F#, we the input event contains the fields `Target` and `Event`. We use target and cast it to `HTMLInputElement`. Two event handlers are used: `Onload` and `Onerror`. The handlers are called from the [`ReadAsText`][moz-readAsText] function. In this simple example, we just print the content of the text file to the console.

````fsharp
open WebSharper.UI.Html
open WebSharper.JavaScript

TemplateMain()
  .LoadFile(fun event -> 
      let inputElem = ev.Target :?> HTMLInputElement
      let files = inputElem.Files
      for i in 0 .. files.Length-1 do
          let file = files.[i]
          printfn "file = %s, %i" file.Name file.Size
          let reader = new TextFileReader()
          reader.Onload <- (fun _ -> printfn "reader = %s" reader.Result)
          reader.Onerror <- (fun _ -> sprintf "cannot read %s" file.Name |> JS.Alert)
          reader.ReadAsText(file)
  )
````


[ws-inputs]: https://developers.websharper.com/docs/v4.x/fs/ui
[mozilla-file]: https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications
[moz-readAsText]: https://developer.mozilla.org/en-US/docs/Web/API/FileReader/readAsText
