---
title:  "Mixed integer linear optimiziation for supply chain and production"
categories: math
tags: math OR Optimiziation
toc: true
---

This blog contains formulas which can be used to optimize the supply chain with multiple suppliers, and a general production planning formula, taken from the book [POCHET, Yves und WOLSEY, Laurence A.: *Production planning by mixed integer programming*][PochetWolsey].

## Supply Chain optimization

To produce a certain demand your product *p* at time period *t*, you need to purchase *x* products from *s* suppliers. 

This is a mixed integer linear optimiziation problem, since we are only dealing with integers and one binary variable.

### The suppliers

For each product froom a supplier, there are:

- initial investment: the tools and the R&D cost to be able to produce the product at all
- maybe dependend investments: Some tools build on other tools which the supplier has already invested in. 
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
- $$j_s^p$$ are the dependent startup costs which needed for investment $$i_s^p$$

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
\forall_{p_s} i_s^p & \le j_s^p & \text{dependent investments} \\ 
\end{aligned}
$$

The *objective* says: look at each time period. Count how many parts we purchase from each supplier, multiply this with the unit cost of this part, and minimize this cost.

Then look at investments: if the investment has been made $$y = 1$$, then take these investment costs and add them also while minimizing.

The first *constraint* says that the number of products supplied from supplier *s* must be no more than what this supplier can deliver in this time period, given that the initial investments have been made.
I took the trick with the binary variable *y* from [Mosek]. A supply is only given when this variable moves to 1.

The second constraint ensures that the products supplied from all combined suppliers at a given time period must at least fulfill the demand for that time period.

The third constraint ensures that if investment $$i_s^p$$ is made, then also investment $$j_s^p$$ has been made.

## Production planning

General production structure capacited multi-level lot-sizing model [PochetWolsey], page 45. All the formulas below taken from [PochetWolsey].

### Constants

- $$1 \le i \le m$$: set of items to be produced
- $$1 \le k \le K$$: set of shared resources with limited capacity
- $$1 \le t \le n$$: time periods until final period *n*
- $$p_t$$: unit production cost
- $$q_t$$: fixed production cost
- $$h_t$$: unit inventory cost
- $$d_t^i$$: demand for item *i* to be satisified at period *t*
- $$M_t$$: large positive number, expressing an upper bound on the maximum lot size in period *t*
- $$L_t^k$$: available capacity of resource *k* during period *t*
- $$\alpha^{i k}, \beta^{i k}$$: amount of capacity of resource *k* consumed  
- $$1 \le j \le m$$: items 
- $$D(i)$$ set of direct successors of part *i* in the BOM, i.e. the items consuming some amount of item *i* when they are produced
- $$r^{i j}$$: the amount of item *i* required to make one unit of item *j*. 
- *r* is the dependent demand, and $$d_t^i$$ is the independent demand
- $$\gamma^i$$: lead-time to produce or deliver an lot of *i*
- $$x_t^i$$: the size of product or purchase order of item *i* launched in period *t*, and delivered in period $$t + \gamma^i$$.

### Decision variables

- $$x_t$$: production lot size in period *t*
- $$y_t$$: binary variable indicating whether there is a positive production in period *t* 
- $$s_t$$: inventory at the end of period *t*

### Objective Function

$$
\begin{aligned}
\text{min} \sum_i \sum_t (p_t^i x_t^i + q_t^i y_t^i + h_t^i s_t^i) \\
\text{subject to} \\
s_{t-1}^i + x_{t-\gamma^i}^i & = [d_t^i + \sum_{j \in D(i)} r^{i j} x_t^j] + s_t^i \\
x_t^i & \le  M_t^i y_t^i \\
\sum_i \alpha^{i k} x_t^i + \sum_t \beta^{i k} y_t^i & \le L_t^k \\
\end{aligned}
$$



## References

[Mosek: Modeling Cookbook][Mosek]

<div class="csl-bib-body" style="line-height: 1.35; ">
  <div class="csl-entry">POCHET, Yves und WOLSEY, Laurence A., 2006. <i>Production planning by mixed integer programming</i>. New York ; Berlin: Springer. Springer series in operations research and financial engineering. ISBN&nbsp;978-0-387-29959-4. <div class="csl-right-inline" style="margin: 0 .4em 0 0em;">MLCM 2006/40914 (T)</div>
  </div>
  <span class="Z3988" title="url_ver=Z39.88-2004&amp;ctx_ver=Z39.88-2004&amp;rfr_id=info%3Asid%2Fzotero.org%3A2&amp;rft_id=urn%3Aisbn%3A978-0-387-29959-4&amp;rft_val_fmt=info%3Aofi%2Ffmt%3Akev%3Amtx%3Abook&amp;rft.genre=book&amp;rft.btitle=Production%20planning%20by%20mixed%20integer%20programming&amp;rft.place=New%20York%20%3B%20Berlin&amp;rft.publisher=Springer&amp;rft.series=Springer%20series%20in%20operations%20research%20and%20financial%20engineering&amp;rft.aufirst=Yves&amp;rft.aulast=Pochet&amp;rft.au=Yves%20Pochet&amp;rft.au=Laurence%20A.%20Wolsey&amp;rft.date=2006&amp;rft.tpages=499&amp;rft.isbn=978-0-387-29959-4"></span>
</div>


[BSMI]: https://www.tuvsud.com/en-us/services/product-certification/bsmi
[SBS]: https://www.sbs-soest.de
[Darfon]: https://www.darfon.com.tw/en
[Tritek]: https://tritekbattery.com
[Excel Example]: https://supplychaindetective.com/supply-chain-modeling-optimization/
[Supply Planning]: https://towardsdatascience.com/supply-planning-using-linear-programming-with-python-bff2401bf270
[PuLP]: https://coin-or.github.io/pulp/
[LP Python]: https://realpython.com/linear-programming-python/#what-is-mixed-integer-linear-programming
[Mosek]: https://docs.mosek.com/modeling-cookbook/index.html
[PochetWolsey]: https://dl.acm.org/doi/book/10.5555/1202598

[fsharp]: {{ site.fsharp_link }}
