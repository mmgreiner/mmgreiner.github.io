---
title:  "Mathe Uffzgi"
categories: math 
tags: homework F# math visualization
toc: true
---


Once in a while, I'm supporting Lea with her math homework (Swiss: *Uffzgi*). Here are some tools that help me (remember, that I prefer [F#]({{site.fsharp_link}}) to other languages. Whenever possible, I use interactive Notebooks and make them available on [github][notebooks].

## Mitternachtsformel

The quadractic formula is called *Mitternachtsformel* in German speaking schools, since the student is supposed to know it by heart even when suddenly woken up after midnight.

It goes like this:

$$ (x_1, x_2) = \frac{ - b \pm \sqrt {b^2 - 4ac} }{2a} $$

when 

$$ ax^2 + bx + c = 0 $$

It can easily be computed in [F#]({{site.fsharp_link}}) using this piece of code:

~~~fsharp
let mitternacht (a: float) b c =
    let cof = sqrt (b * b - 4.0 * a * c) 
    let den = 2.0 * a
    ( (-1.0 * b + cof) / den, (-1.0 * b - cof) / den)
~~~~

Try it:

~~~fsharp
mitternacht 1.0 -4.0 -5.0
|> printfn "%A"
// returns (5.0, -1.0)
~~~~

## Microsoft Math Solver

A simple way is to use the [Microsoft Math Solver][mathsolver]. You can type in the mathematical equation like 

    x^2 - 4x - 5 = 0
    
and it will return the solution and visualizes the graph.

![Math]({{site.url}}/images/mathsolver.png)



[mathsolver]: https://math.microsoft.com/en

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

Formulas can also be turned int LaTeX:

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



[notebooks]: {{site.greiner_link | append: "/Notebook" }}