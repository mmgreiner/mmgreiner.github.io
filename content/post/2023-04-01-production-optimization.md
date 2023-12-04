---
categories:
- programming
- math
date: "2023-04-01T00:00:00Z"
draft: false
tags:
- jekyll
- math
- OR
title: "General production optimization"
showshowToc: true
math: true
mermaid: true
---

## Optimization

General production structure capacited multi-level lot-sizing model [PochetWolsey], page 45

- $1 \le i \le m$: set of items to be produced
- $1 \le k \le K$: set of shared resources with limited capacity
- $1 \le t \le n$: time periods until final period *n*
- $p_t$: unit production cost
- $q_t$: fixed production cost
- $h_t$: unit inventory cost
- $d_t^i$: demand for item *i* to be satisified at period *t*
- $M_t$: large positive number, expressing an upper bound on the maximum lot size iin period *t*
- $L_t^k$: available capacity of resource k during period *t*
- $\alpha^{i k}, \beta^{i k}$: amount of capacity of resource *k* consumed  
- $1 \le j \le m$: items 
- $D(i)$ set of direct successors of part *i* in the BOM, i.e. the items consuming some amount of item *i* when they are produced
- $r^{i j}$: the amount of item *i* required to make one unit of item *j*. 
- *r* is the dependent demand, and $d_t^i$ is the independent demand
- $\gamma^i$: lead-time to produce or deliver an lot of *i*
- $x_t^i$: the size of product or purchase order of item *i* launched in period *t*, and delivered in period $t + \gamma^i$.
- **Decision variables**:
    - $x_t$: production lot size in period *t*
    - $y_t$: binary variable indicating wether there is a positive production in period *t* 
    - $s_t$:inventory at the end of period *t*


### Objective Function

$$
\begin{aligned}
\text{min} \sum_i \sum_t (p_t^i x_t^i + q_t^i y_t^i + h_t^i s_t^i) \cr
\text{subject to} \cr
s_{t-1}^i + x_{t-\gamma^i}^i & = [d_t^i + \sum_{j \in D(i)} r^{i j} x_t^j] + s_t^i \cr
x_t^i & \le  M_t^i y_t^i \cr
\sum_i \alpha^{i k} x_t^i + \sum_t \beta^{i k} y_t^i & \le L_t^k \cr
\end{aligned}
$$


## References

<div class="csl-bib-body" style="line-height: 1.35; ">
  <div class="csl-entry">POCHET, Yves und WOLSEY, Laurence A., 2006. <i>Production planning by mixed integer programming</i>. New York ; Berlin: Springer. Springer series in operations research and financial engineering. ISBN&nbsp;978-0-387-29959-4. <div class="csl-right-inline" style="margin: 0 .4em 0 0em;">MLCM 2006/40914 (T)</div>
  </div>
  <span class="Z3988" title="url_ver=Z39.88-2004&amp;ctx_ver=Z39.88-2004&amp;rfr_id=info%3Asid%2Fzotero.org%3A2&amp;rft_id=urn%3Aisbn%3A978-0-387-29959-4&amp;rft_val_fmt=info%3Aofi%2Ffmt%3Akev%3Amtx%3Abook&amp;rft.genre=book&amp;rft.btitle=Production%20planning%20by%20mixed%20integer%20programming&amp;rft.place=New%20York%20%3B%20Berlin&amp;rft.publisher=Springer&amp;rft.series=Springer%20series%20in%20operations%20research%20and%20financial%20engineering&amp;rft.aufirst=Yves&amp;rft.aulast=Pochet&amp;rft.au=Yves%20Pochet&amp;rft.au=Laurence%20A.%20Wolsey&amp;rft.date=2006&amp;rft.tpages=499&amp;rft.isbn=978-0-387-29959-4"></span>
</div>


[fsharp]: {{< param "fsharp_link" >}}

[PochetWolsey]: https://dl.acm.org/doi/book/10.5555/1202598
[Mosek]: https://docs.mosek.com/modeling-cookbook/index.html

