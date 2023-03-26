---
title:  "Mixed integer linear optimiziation of supply chain problem"
categories: math
tags: math OR Optimiziation
toc: true
---

## The question

To produce a certain demand your product *p* at time period *t*, you need to purchase *x* products from *s* suppliers. 

This is a mixed integer linear optimiziation problem, since we are only dealing with integers and one binary variable.

### The suppliers

For each product froom a supplier, there are:

- initial investment: the tools and the R&D cost to be able to produce the product at all
- a unit cost for the product
- a potential ramp-up, since only after the initial investment is done can the supplier slowly start to produce.

### The demand

For each product, there is a demand you have to fulfill in order to be able to manufacture.

## The goal

The goal is to find the best combination of products and suppliers, which fullfills your demands at the minimum possible cost.

## Mathematical formulation

The field of mathematics handling these kind of questions is called [Operation Research](https://en.wikipedia.org/wiki/Operations_research). There is some more information also on the [related post]({% post_url 22-12-31-operation-research %}).

First define the **constants**:

- *t* is the time period, often given in months or quarters
- $$d_t^p$$ is the demand for product *p* at time *t*
- $$q_t^s$$ is what supplier *s* can supply at time *t*, given that the initial investments have been made
- $$c_s^p$$ is the unit cost of product *p* from supplier *s*
- $$i_s^p$$ is the initial, startup cost for product *p* from supplier *s*

The **variables** are:

- $$x_{t,s}^p$$ is the variable indicating how many units of product *p* to purchase from supplier *s* at time *t*
- $$y_s^p$$ is the decision to make the initial investment. $$y \in \{0, 1\}$$.

*x* is an integer variable, and *y* in reality is a binary variable which we simulate using an integer variable that can only be 0 or 1.

This gives us the following optimization function and constraints:

$$
\begin{aligned}
\text{Objective:} \\
cost := & \text{Min} \sum_{t}^{\text{periods}} \sum_{p}^{\text{products}} \sum_{s}^{\text{suppliers}} x_{t,s}^p c_s^p 
 + \sum_{p} \sum_{s} y_s^p i_s^p \\
\\
\text{Constraints:} \\
x_{t,s}^p & \le q_t^s y_s^p  & \text{supply}\\
\sum_{p_s} x_{t,s}^p & \ge d_t^p  & \text{demand}\\
\end{aligned}
$$

The *objective* says: look at each time period. Count how many parts we purchase from each supplier, multiply this with the unit cost of this part, and minimize this cost.

Then look at investments: if the investment has been made $$y = 1$$, then take these investment costs and add them also while minimizing.

The first *constraint* says that the number of products supplied from supplier *s* must be no more than what this supplier can deliver in this time period, given that the initial investments have been made.
I took the trick with the binary variable *y* from [Mosek]. A supply is only given when this variable moves to 1.

The second constraints ensures that the products supplied from all combined suppliers at a given time period must at least fulfill the demand for that time period.

## References

- [Mosek: Modeling Cookbook][Mosek]


[BSMI]: https://www.tuvsud.com/en-us/services/product-certification/bsmi
[SBS]: https://www.sbs-soest.de
[Darfon]: https://www.darfon.com.tw/en
[Tritek]: https://tritekbattery.com
[Excel Example]: https://supplychaindetective.com/supply-chain-modeling-optimization/
[Supply Planning]: https://towardsdatascience.com/supply-planning-using-linear-programming-with-python-bff2401bf270
[PuLP]: https://coin-or.github.io/pulp/
[LP Python]: https://realpython.com/linear-programming-python/#what-is-mixed-integer-linear-programming
[Mosek]: https://docs.mosek.com/modeling-cookbook/index.html



[fsharp]: {{ site.fsharp_link }}