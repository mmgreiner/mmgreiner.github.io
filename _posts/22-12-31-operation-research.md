---
title:  "Operation research"
categories: programming
tags: OR OR-Tools
toc: true
---

Operation Research is defined in [Wikipedia](https://en.wikipedia.org/wiki/Operations_research) as

> Operations research (British English: operational research) ... , often shortened to the initialism OR, is a discipline that deals with the development and application of analytical methods to improve decision-making. It is considered to be a subfield of mathematical sciences.

Typical problems of OR are:

- production or assembly optimization
- routing optimization
- scheduling
- optimal packaging

I have found two good online lecture notes: [Introduction to Operation Research by Juraj Stacho][Stacho] and [Operation and Operations Resarch by Matthew Roughan][Roughan].

In this post, I want to show some examples of OR problems and how they can be solved using either [OR-Tools] and dotnet or xx and Julia.

## Linear optimization example

All OR problems consist of three components:

- **decision variables**: they define the values that have to be decided upon
- **goal** or **objective**: the quanitity that needs to be minimized or maximized
- **objective function**: expression how to compute the goal
- **constraints**: equalities or inequalities describing restrictions.

Our example is taken from [OR-Tools]:

$$
\begin{align}
obj: \text{maximize } 3x + y \text{ subject to:} \\
c_0: x + y & \le 2 \\
d_0: 0 \le x & \le 1 \\
d_1: 0 \le y & \le 2 \\
\end{align}
$$
<figcaption>Equation 1</figcaption>

In this case, there are two **variables**  *x* and *y*, both with defined lower and upper boundaries given in $$ d_0 $$ and $$ d_1 $$.

There is one contraint $$ c_0 $$, with coefficients *1.0* for *x* and *1.0* for *y*.

The objective function *obj* has two variables *x* and *y*, with coefficients *3.0* and *1.0*.

## Google OR-Tools

[OR-Tools] "is open source software for combinatorial optimization, which seeks to find the best solution to a problem out of a very large set of possible solutions." Installation is very easy using dotnet, see [OR-Tools installation](https://developers.google.com/optimization/install). Using dotnet it is as easy as: ``dotnet add package Google.OrTools``.  

[OR-Tools] can be used for many kinds of OR problems. One category is to find any feasible solution (see example xy below), another category is to find an optimal solution, typically trying to minimize or maximize some cost or values.

*Linear optimization* or *linear programming (LP)* lare supported with two algorithms, one from Google (called **GLOP**) and one for integer only variables and constraints, called **SCIP**. See [here](https://developers.google.com/optimization/lp) for more information.

### OR-Tools and Dotnet

[OR-Tools] have support for C++, Python, Java, and C#. [F#] is not supported out of the box. There used to be a wrapper `Google.OrTools.FSharp` which is still available on [nuget](https://www.nuget.org/packages/Google.OrTools.FSharp/#dependencies-body-tab), but the documentation is no longer available.

Particularly for expressing equations directly in the code, C# is much handier with its implicit type conversion.

Compare these two pieces of code to demonstrate this, taken from [OR-Tools MIP Example](https://developers.google.com/optimization/mip/mip_example):

**Maximize $$ x + 10y $$ subject to:**

$$
\begin{align}
x &+ 7y & \le 17.5 \\
x & & \le 3.5 \\
&x \ge 0 \\
&y \ge 0
\end{align}
$$
<figcaption>Equation 2</figcaption>

#### C#

[OR-Tools] uses C# as the targeted dotnet programming language. C# with its automatic type conversion lends itself nicely to express algebraic equations.

~~~csharp
using Google.OrTools;
var solver = LinearSolver.Solver.CreateSolver("SCIP");
var x = solver.MakeIntVar(0.0, double.PositiveInfinity, "x");
var y = solver.MakeIntVar(0.0, double.PositiveInfinity, "y");
solver.Add(x + 7*y <= 17.5);
solver.Add(x <= 3.5);
solver.Maximize(x + 10 * y);
solver.Solve();
~~~

#### F#

~~~fsharp
#r "nuget: Google.OrTools"
open Google.OrTools
let solver = LinearSolver.Solver.CreateSolver("SCIP")
let x = solver.MakeIntVar(0.0, Double.PositiveInfinity, "x")
let y = solver.MakeIntVar(0.0, Double.PositiveInfinity, "y")
let c1 = solver.MakeConstraint(0.0, 17.5, "c1")
c1.SetCoefficient(x, 1.0)
c1.SetCoefficient(y, 7.0)
let c2 = solver.MakeConstraint(0.0, Double.PositiveInfinity, "c2")
c2.SetCoefficient(x, 1.0)
solver.Maximize(x + 10.0 * y)
solver.Solve()
~~~

There is one way to make [F#] look somewhat nicer, as shown on [StackOverflow](https://stackoverflow.com/questions/65532910/f-or-tools-sat-solver). You have to explicitly declare the operators for the `LinearExpr`.

~~~fsharp
module LinearExprOperators =
    let ( ^<> ) (x: LinearExpr) (y: LinearExpr) = LinearExpr.(<>) (x, y)
    let ( ^= ) (x: LinearExpr) (y: LinearExpr) = LinearExpr.(=) (x, y)
    let (^<=) (x: LinearExpr) (y: LinearExpr) = LinearExpr.(<=) (x, y)
    let (^<==) (x: LinearExpr) (y: float) = LinearExpr.(<=) (x, y)
open LinearExprOperators
olver.Add(x + 7.0 * y ^<== 17.5)
~~~

However, this seems to be very cumbersome. In this case, the functional programming style does not lead itself to be more concise.

### Longer example

Back to the equation 1 of above:

**maximize $$ 3x + y $$ subject to:**

$$
\begin{align}
x + y & \le 2 \\
0 \le x & \le 1 \\
0 \le y & \le 2 \\
\end{align}
$$

In [F#], the variables *x* and *y* are defined with the lower and upper bounds of *1* and *2* respectively. We are looking for for integer solutions, therefore we will be using the `SCIP` linear solver.

The objective function *obj* is defined with the two coefficients for $$ 3x $$ and $$ 1y $$.

~~~fsharp
#r "nuget: Google.OrTools"
open Google.OrTools.LinearSolver
let solver = Solver.CreateSolver("SCIP")
assert not (isNull solver)

// create variables x and y
let x = solver.MakeIntVar(0.0, 1.0, "x")
let y = solver.MakeIntVar(0.0, 2.0, "y")

// create constraint
let c1 = solver.MakeConstraint(0.0, 2.0, "c1")
c1.SetCoefficient(x, 1.0)
c1.SetCoefficient(y, 1.0)

solver.Maximize(3.0 * x + y)
let res = solver.Solve()
printfn "%A" res
// OPTIMAL
printfn "objective value: %f" (solver.Objective().Value()) 
// objective value: 4.000000
printfn "x = %f" (x.SolutionValue())
// x = 1.000000
printfn "y = %f" (y.SolutionValue())
// y = 1.000000
~~~

The objective function could also be implemented using an explicit objective with coefficients:

~~~fsharp
let objective = solver.Objective()
objective.SetCoefficient(x, 3.0)
objective.SetCoefficient(y, 1.0)
objective.SetMaximization()
~~~


## A longer Example

Let's take another example with only integers as variables and using arrays, taken also from [OR-Tools Arrays](https://developers.google.com/optimization/mip/mip_var_array). We will also use [F#] 2-dimensional arrays here. These are described in [F# arrays](https://learn.microsoft.com/en-gb/dotnet/fsharp/language-reference/arrays).

This is the optimization:

$$
\begin{align}
\text{Maximize } 7x_1 + 8x_2 + 2x_3 + 9x_4 + 6x_5 \text{ subject to} \\
5x_1	+	7x_2	+	9x_3	+	2x_4	+	1x_5 & \le	250 \\
18x_1	+	4x_2	-	9x_3	+	10x_4	+	12x_5 & \le 285 \\
4x_1	+	7x_2	+	3x_3	+	8x_4	+	5x_5 & \le 211 \\
5x_1	+	13x_2	+	16x_3	+	3x_4	-	7x_5 & \le 315 \\
x_i \text{ are positive integers}
\end{align}
$$

In the [F#] example, whe will represent the equations as a list of list, such that `fst` represents the coefficients, and `snd` the boundaries. We also define `Inf` as an abbreviations. This most easily resembles the layout of the constraints. The coefficients itself will be stored in a 2-dimensional array.

~~~fsharp
open Microsoft.FSharp.Collections
let Inf = Double.PositiveInfinity

let constraints =
    [
        [5; 7; 9; 2; 1], 250
        [18; 4; 9; 10; 12], 285
        [4; 7; 3; 8; 5], 211
        [5; 13; 16; 3; 7], 315
    ]

let coefficients = 
    constraints
    |> List.map fst
    |> array2D

let bounds = constraints |> List.map snd
~~~

The rest ist straight forward. We declare `solver`, then create as many variables called `x{i+1}`, for $$ x_1 $$ through $$ x_5 $$, as there are coefficients in the first constraint.

Then we create all the constraints, as many as the 2-d array has rows (`coefficients.GetLength(0)`). For each constraint *i*, we create all the coefficients *j*, associated with the variable *j*. The constraint is given at `coefficients.[i,j]`.

Finally, we declare the objective maximization function. To check if everything went fine, we can use the function `solver.ExportModelAsLpFormat(false)`, which gives a nice print-out of the internal data.

~~~fsharp
let solver = LinearSolver.Solver.CreateSolver("SCIP")

// make variables
let vars =
    fst constraints.[0] 
    |> List.mapi (fun i _ -> 
        solver.MakeIntVar(0.0, Inf, $"x{i+1}") 
    )

// make constraints
for i in 0..coefficients.GetLength(0)-1 do
    let con = solver.MakeConstraint(0.0, bounds.[i], $"c_{i}")
    for j in 0..coefficients.GetLength(1)-1 do
        con.SetCoefficient(vars.[j], coefficients.[i, j])

solver.Maximize(7.0 * vars.[0] + 8.0 * vars.[1] + 2.0 * vars.[2] + 9.0 * vars.[3] + 6.0 * vars.[4])

solver.ExportModelAsLpFormat(false) |> printfn "%s"
~~~

This should give a listing as follows:

~~~csh
% dotnet run
\ Generated by MPModelProtoExporter
\   Name             : 
\   Format           : Free
\   Constraints      : 4
\   Variables        : 5
\     Binary         : 0
\     Integer        : 5
\     Continuous     : 0
Maximize
 Obj: +7 x1 +8 x2 +2 x3 +9 x4 +6 x5 
Subject to
 c_0_rhs: +5 x1 +7 x2 +9 x3 +2 x4 +1 x5  <= 250
 c_0_lhs: +5 x1 +7 x2 +9 x3 +2 x4 +1 x5  >= 0
 c_1_rhs: +18 x1 +4 x2 +9 x3 +10 x4 +12 x5  <= 285
 c_1_lhs: +18 x1 +4 x2 +9 x3 +10 x4 +12 x5  >= 0
 c_2_rhs: +4 x1 +7 x2 +3 x3 +8 x4 +5 x5  <= 211
 c_2_lhs: +4 x1 +7 x2 +3 x3 +8 x4 +5 x5  >= 0
 c_3_rhs: +5 x1 +13 x2 +16 x3 +3 x4 +7 x5  <= 315
 c_3_lhs: +5 x1 +13 x2 +16 x3 +3 x4 +7 x5  >= 0
Bounds
 0 <= x1 <= inf
 0 <= x2 <= inf
 0 <= x3 <= inf
 0 <= x4 <= inf
 0 <= x5 <= inf
Generals
 x1
 x2
 x3
 x4
 x5
End
~~~

Now we just need so solve it:

~~~fsharp
match solver.Solve() with
    | LinearSolver.Solver.ResultStatus.OPTIMAL ->
        printfn "solution = %f" (solver.Objective().Value())
        for v in vars do
            printfn "%s = %f" (v.Name()) (v.SolutionValue()) 
    | x -> 
        printfn "no optimal solution, %A" x
~~~

which gives us:

    solution = 259.000000
    x1 = 8.000000
    x2 = 19.000000
    x3 = 0.000000
    x4 = 5.000000
    x5 = 1.000000



## Toy Company 
Let'st start with a simple example out of [Stacho]:

> A toy company makes two types of toys: toy soldiers and trains. Each toy is produced in two stages, first it is constructed in a carpentry shop, and then it is sent to a finishing shop, where it is varnished, vaxed, and polished. To make one toy soldier costs $10 for raw materials and $14 for labor; it takes 1 hour in the carpentry shop, and 2 hours for finishing. To make one train costs $9 for raw materials and $10 for labor; it takes 1 hour in the carpentry shop, and 1 hour for finishing.
>
> There are 80 hours available each week in the carpentry shop, and 100 hours for finishing. Each toy soldier is sold for $27 while each train for $21. Due to decreased demand for toy soldiers, the company plans to make and sell at most 40 toy soldiers; the number of trains is not restriced in any way.
>
> What is the optimum (best) product mix (i.e., what quantities of which products to make) that maximizes the profit (assuming all toys produced will be sold)?

Our variables are `trains` and `soldiers`, both being integers. What is the objective function? We want to maximize the profit, so what are the costs. The manufacturing costs are: 

- **toy soldier**: $27 sales price - $10 for raw material + $14 for labor = $3 profit
- **trains**: $21 sales price - $9 raw material - $10 labor = $2 profit

Therefore, our objective function is: $$ \text{maximize } 3 * soldiers + 2 * trains $$.

What are our constraints?

- **carpentry shop**: $$ soldiers * 1h + trains * 1h $$, with 80 hours available
- **finishing shop**: $$ soldiers * 2h + trains * 1h $$, with 100 hours available

Therefore, the set of equations is:

$$
\begin{align}
\text{max } 3 soldiers + 2 trains \text{ subject to:} \\
soldiers + trains \le 80 \\
2 soldiers + trains \le 100 \\
soldiers \le 40 \\
soldiers, trains \ge 0
\end{align}
$$

In [F#], this could be implemented like this:

~~~fsharp
let solver = Solver.CreateSolver("SCIP")

let constraints = 
    [
        [ 1; 1 ], 80
        [ 2; 1 ], 100
        [ 1; 0 ], 40
    ]
let vars = [ "soldiers"; "trains" ] |> List.map(fun v -> solver.MakeIntVar(0.0, 1000.0, v))
let coefficients = constraints |> List.map fst |> array2D
let bounds = constraints |> List.map snd

// make constraints
for i in 0..coefficients.GetLength(0)-1 do
    let con = solver.MakeConstraint(0.0, bounds.[i])
    for j in 0..coefficients.GetLength(1)-1 do
        con.SetCoefficient(vars.[j], coefficients.[i, j])

solver.Maximize(3.0 * soldiers + 2.0 * trains)
solver.ExportModelAsLpFormat(false) |> printfn "%s"

// solve it
match solver.Solve() with
| OPTIMAL ->
    printfn $"Optimal revenue = ${solver.Objective().Value()}"
    for v in vars do
        printfn $"{v.Name()} = {v.SolutionValue()}"
| x -> 
    printfn "no solution found, %A" x
~~~

This will result in: 

~~~
Optimal revenue = $180
soldiers = 20
trains = 60
~~~

## Map coloring problem

The map coloring problem is an example taken from the [MiniZinc](https://www.minizinc.org/doc-2.5.5/en/modelling.html) tutorial. The states of Australia have to be colored with maximum 3 colors, such that no adjacent states have the same color. 

![Australian States](https://www.minizinc.org/doc-2.5.5/en/images/aust.svg)

Now in this case, we don't have an objective function, we just need any feasible solution. We will therefore use the constrained programming solver, not the linear equation solvers. A variable is created for the color of each state. The constraints are such that neighboring states cannot have the same color. 

An implementation in [F#]:

~~~fsharp
open Google.OrTools.FSharp.Sat

let model = new CpModel()

let nofColors = 3L

let wa = model.NewIntVar(1L, nofColors, "wa")
let nt = model.NewIntVar(1L, nofColors, "nt")
let sa = model.NewIntVar(1L, nofColors, "sa")
let q = model.NewIntVar(1L, nofColors, "q")
let nsw = model.NewIntVar(1L, nofColors, "nsw")
let v = model.NewIntVar(1L, nofColors, "v")
let t = model.NewIntVar(1L, nofColors, "t")

// constraints
model.Add(wa ^<> nt) |> ignore
model.Add(wa ^<> sa) |> ignore
model.Add(nt ^<> sa) |> ignore
model.Add(nt ^<> q) |> ignore
model.Add(sa ^<> q) |> ignore
model.Add(sa ^<> nsw) |> ignore
model.Add(sa ^<> v) |> ignore
model.Add(q ^<> nsw) |> ignore
model.Add(nsw ^<> v) |> ignore

let solver = new CpSolver()
let status = solver.Solve(model)
match status with
| CpSolverStatus.Optimal
| CpSolverStatus.Feasible ->
    let vv x = solver.Value(x) |> int
    printfn "wa = %i, nt = %i, sa = %i, q = %i, nsw = %i, v = %i, t = %i"
        (vv wa) (vv nt) (vv sa) (vv q) (vv nsw) (vv v) (vv t)
| _ ->
    printfn "no solution found"

// wa = 2, nt = 1, sa = 3, q = 2, nsw = 1, v = 2, t = 1
~~~

## Stigler Diet

> The Stigler diet is an optimization problem named for George Stigler, a 1982 Nobel Laureate in economics, who posed the following problem:
>
>> For a moderately active man weighing 154 pounds, how much of each of 77 foods should be eaten on a daily basis so that the man’s intake of nine nutrients will be at least equal to the recommended dietary allowances (RDAs) suggested by the National Research Council in 1943, with the cost of the diet being minimal?

(Taken from [Wikipedia](https://en.wikipedia.org/wiki/Stigler_diet)).

It is also one of the [OR-Tools examples](https://developers.google.com/optimization/lp/stigler_diet) for linear optimization. [OR-Tools] contains a full example in C#, but here the challenge was to turn it into [F#].

### Getting the data

I wanted to take the data directly from the web page to avoid any typing errors when copying the values. There are two tables of interests: one for the nutrients and one for the commodities. The get the data, the package [FSharp.Data]() is used. It generates a dynamic type from the given URL and tables can be parsed row by row 

The nutrient table looks like this:

| Nutrient | Daily Recommended Intake |
| Calories | 3,000 calories |
| Protein | 70 grams |

So we also have to extract the pure number from the second colum with the function ``extractFloat``. In addition, calories and vitamin A have to be divided by 1000, since with commodities they are given as kilos.

The nutrients will give us the boundaries, and the commodities the coefficients. Each commodity corresponds to a variable.

~~~fsharp
open FSharp.Data

[<Literal>]
let StiglerURL = "https://developers.google.com/optimization/lp/stigler_diet"
type StiglerData = HtmlProvider<StiglerURL>

let nutrientsTable = StiglerData.Load(StiglerURL).Tables.``Nutrients list``
let commoditiesTable = StiglerData.Load(StiglerURL).Tables.``Commodities list``

let extractFloat (s: string): float =
    Text.RegularExpressions.Regex.Replace(s, @"[a-zA-Z ',]", "") |> float

let nutrients =
    nutrientsTable.Rows
    |> Array.map (fun row ->
        let name = row.Nutrient
        let intake = row.``Daily Recommended Intake`` |> extractFloat
        {|
            Name = row.Nutrient
            Intake =
                if name.Contains("Calories") || name.Contains("Vitamin A")
                then intake / 1000.0
                else intake
        |}
    )

let commodities =
    commoditiesTable.Rows
    |> Array.map (fun row ->
        {|
            Name = row.Commodity
            Price = row.``1939 price (cents)``
            Unit = row.Unit
            Nutrients = 
                [|
                    float row.``Calories (kcal)``
                    float row.``Protein (g)``
                    float row.``Calcium (g)``
                    float row.``Iron (mg)``
                    float row.``Vitamin A (KIU)``
                    float row.``Thiamine (mg)``
                    float row.``Riboflavin (mg)``
                    float row.``Niacin (mg)``
                    float row.``Ascorbic Acid (mg)``
                |]
        |}
    )
~~~

Now we can convert this to the usual arrays:

~~~fsharp
// create the linear solver
let solver = Solver.CreateSolver("GLOP")
assert not (isNull solver)

// create variables
commodities 
|> Array.iter (fun commodity -> 
    solver.MakeNumVar(0.0, Double.PositiveInfinity, commodity.Name) |> ignore
)

let coefficients = 
    commodities
    |> Array.map (fun com ->
        com.Nutrients
    )
    |> array2D

// constraints
nutrients
|> Array.iteri (fun i nut ->
    let con = solver.MakeConstraint(nut.Intake, Double.PositiveInfinity, nut.Name)
    for j in 0..coefficients.GetLength(1)-1 do
        con.SetCoefficient(solver.variables().[j], coefficients.[i, j])
)
~~~

### Optimization

## Julia

TBD

## References

- [Prof. Juraj Stacho, Columbia University, New York: Introduction to Operations Research][Stacho]
- [Prof. Roughan: Optimisation and Operation Research][Roughan]
- [Google OR-Tools][OR-Tools]
- [Stigler Diet with Gurobi](http://a-d-c.ca/solving-the-stigler-diet-problem-with-gurobi-cplex-and-glop/#page-content)
- [Geek culture: A primer on the OR libraries in python and julia]((https://medium.com/geekculture/a-primer-on-the-operations-research-libraries-in-python-and-julia-d10da69591ff))

## Notes

Still looking for the best way to layout these equations with LaTeX.



[Roughan]: https://roughan.info/notes/oorii/03lecture_notes.html

[Stacho]: https://www.cs.toronto.edu/~stacho/public/IEOR4004-notes1.pdf

[OR-Tools]: https://developers.google.com/optimization

[F#]: {{site.fsharp_link}}

[Julia]: https://julialang.org
