---
layout: post
title:  "Mathe Uffzgi"
categories: math 
tags: homework fsharp math visualization
---

Once in a while, I'm supporting Lea with her math homework (Swiss: *Uffzgi*). Here are some tools that help me (remember, that I prefer [F#]({{fsharp_link}}) to other languages.

## Symbolic algebra and F#

There is an interesting library out there called [Math.NET Symbolics](https://symbolics.mathdotnet.com). It can be used to do algebraic calculations. 

First, the variables are declared as symbols:

~~~~fsharp
open System.Numerics
open MathNet.Numerics
open MathNet.Symbolics
open Operators

let x = symbol "x"
let y = symbol "y"
let a = symbol "a"
let b = symbol "b"
let c = symbol "c"
let d = symbol "d"

let aa = a + a
let kompliziert = (a/b/(c*a))*(c*d/a)/d 
~~~~

Now it can be used to simplify formulas. They can be printed using `Infix.format`.

~~~fsharp
a + a       // return 2*a
kompliziert |> Infix.format     // returns 1/(a*b)
~~~

Formulars can also be turned int LaTeX:

~~~fsharp
kompliziert |> LaTex.format
// returns \frac{1}{ab} 
~~~

And this LaTeX can then be used in markdown documents. How to do this with [Jekyll]({{site.jekyll_link}}), is shown in the post [Liquid tricks]({% post_url 2022-04-02-liquid-tricks %}).


~~~fsharp
let symbols = Map.ofList [ "a", FloatingPoint.Real 2.0; "b", FloatingPoint.Real 3.0 ]
Evaluate.evaluate symbols (1/(a*b))  // Returns Real 0.1666666667 (as float)
~~~

Unfortunately, this crashed for me. See also as notebook on [github](https://github.com/mmgreiner/Notebook/blob/main/Symbolics.ipynb)

## Mathematical graphs

For visualization, I typically use [Plotly.NET](https://plotly.net).

An example of plotting herbizide influence on plant growth can be found as an interactive F# [notebook]( https://github.com/mmgreiner/Notebook/blob/main/Herbizid.ipynb).



