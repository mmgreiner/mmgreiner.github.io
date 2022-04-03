---
layout: post
title:  "Operation Research"
categories: math
tags: operation research
---

{% include mathscripts.html %}

[Operation research](https://en.wikipedia.org/wiki/Operations_research) is concerned with advanced analytical methods to improve decision-making.

## What is an optimization problem?

The goal of optimization (adopted from [OR-Tools](or-tools)) is to find the best solution given a set of constraints. The problem has the following elements:

- The *objective* - the quantity you want to optimize, either minimize or maximize
- the *constraints* - restrictions on the set of possible solutions. 

### First example

I found a very good introduction in the [lecture notes][stacho] of Juraj Stacho. He introduces the following problem:

>A toy company makes two types of toys: toy soldiers and trains. Each toy is produced in two stages, first it is constructed in a carpentry shop, and then it is sent to a finishing shop, where it is varnished, vaxed, and polished. To make one toy soldier costs $10 for raw materials and $14 for labor; it takes 1 hour in the carpentry shop, and 2 hours for finishing. To make one train costs $9 for raw materials and $10 for labor; it takes 1 hour in the carpentry shop, and 1 hour for finishing.

> There are 80 hours available each week in the carpentry shop, and 100 hours for finishing. Each toy soldier is sold for $27 while each train for $21. Due to decreased demand for toy soldiers, the company plans to make and sell at most 40 toy soldiers; the number of trains is not restriced in any way.

> What is the optimum (best) product mix (i.e., what quantities of which products to make) that maximizes the profit (assuming all toys produced will be sold)?

We have these decision variables:
- $$ x_1 $$ = Number of soldier
- $$ x_2 $$ = Number of trains

The objective is to maximize the profit:

- $27 − $10 − $14 = $3 profit for selling one toy soldier ⇒ 3x1 profit (in $) for selling $$ x_1 $$ toy soldier
- $21 − $9 − $10 = $2 profit for selling one toy train ⇒ 2x2 profit (in $) for selling $$ x_2 $$ toy train

This gives us the objective function $$ z = 3x_1 + 2x_2 $$, which needs to be optimized for the best profit.

What about constraints? These are the available hours in the carpentry shop and the finishing shop.

> - producing x1 toy soldiers and x2 toy trains requires
> 1. 1 $$ x_1 + 1 x_2 $$ hours in the carpentry shop; there are 80 hours available 
> 1. 2 $$ x_1 + 1 x_2 $$ hours in the finishing shop; there are 100 hours available
> - the number $$x_1$$ of toy soldiers produced should be at most 40

So the *linear program* is defined as:

$$ \max{3 x_1 + 2 x_2} $$

Subject to:

$$ x_1 + x_2 \le 80 $$

$$ 2 x_1 + x_2 \le 100 $$

$$ x_2 \le 40 $$

$$ x_1, x_2 \ge 0 $$

How to solve this in F# is shown [below](#simple-example).


### Simple example

~~~fsharp
open Google.OrTools.LinearSolver
open Google.OrTools.FSharp

let solver = Solver.CreateSolver("GLOP")

// Variables
let x1 = solver.MakeNumVar(0.0, 1000.0, "x1")
let x2 = solver.MakeNumVar(0.0, 1000.0, "x2")

// Constraints
solver.Add(LinearExpr.(<=)(x1 + x2, 80.0))
solver.Add(LinearExpr.(<=)(x1 * 2.0 + x2, 100.0))
solver.Add(LinearExpr.(<=)(x1, 40.0))

// Objective function 3x1 + 3x2
let objective = solver.Objective()
objective.SetCoefficient(x1, 3)
objective.SetCoefficient(x2, 2)
objective.SetMaximization()
solver.Solve()

objective.Value() |> printfn "objective = %A"
x1.SolutionValue() |> printfn "x1 = %f"
x2.SolutionValue() |> printfn "x2 = %f"
~~~

The output will be:

    objective = 180.0
    x1 = 20.000000
    x2 = 60.000000

Note that in FSharp there is no implicit type conversion. So the constraints have to use the explicit way of `LinearExpr.(<=)`. This is nicely described in this [StackOverflow](https://stackoverflow.com/questions/65532910/f-or-tools-sat-solver) post.

Note also that this corresponds the optimal solution as indicated in [stacho]. There, the optimal solution is also given as $$ (x_1, x_2) = (20, 60) $$. 

The Jupyter notebook is availe on [github](https://github.com/mmgreiner/Notebook/blob/main/OR-Tools.ipynb)

## Classification of problems

**To Do**: I had this big table, hand written.


## Google OR-Tools

Google has an impressive suite of open source software for operation research, called [OR-Tools][or-tools]. 

> OR-Tools is open source software for combinatorial optimization, which seeks to find the best solution to a problem out of a very large set of possible solutions. Here are some examples of problems that OR-Tools solves:
>
> - Vehicle routing: Find optimal routes for vehicle fleets that pick up and deliver packages given constraints (e.g., "this truck can't hold more than 20,000 pounds" or "all deliveries must be made within a two-hour window").
> - Scheduling: Find the optimal schedule for a complex set of tasks, some of which need to be performed before others, on a fixed set of machines, or other resources.
> - Bin packing: Pack as many objects of various sizes as possible into a fixed number of bins with maximum capacities.
>
> In most cases, problems like these have a vast number of possible solutions—too many for a computer to search them all. To overcome this, OR-Tools uses state-of-the-art algorithms to narrow down the search set, in order to find an optimal (or close to optimal) solution.

### Programming languages

OR-Tools have APIs for the languages C++, Python, Java, and C#. Unfortunately, there is not API for F#.

[ORTools.FSharp](https://github.com/google/or-tools/tree/stable/examples/dotnet#note-on-googleortoolsfsharp)


### Types of solvers

- Constraint Programming: Find solutions to problems that can be expressed as constraints
- Linear and Mixed-Integer Programming: find the optimal value for a linear objective function
- Vehicle routing: best vehicle routes given constraints
- Graphs Algorithms: find shortest path, min cost flows, ...


## Literature

- [[mosek]]: Mosek Modelling Cookbook
- Yves Pochet and Laurence A. Wolsey: Production Planning by Mixed Integer Programming. Springer Series in Operations Research and Financial Engineering, 2006. ISBN-10: 0-387-29959-9
- [[stacho]]: Juraj Stacho: Introduction to Operation Research. Deterministic Models. Columbia University, 2014


[mosek]: https://docs.mosek.com/modeling-cookbook/index.html
[or-tools]: https://developers.google.com/optimization/
[stacho]: https://www.cs.toronto.edu/~stacho/public/IEOR4004-notes1.pdf