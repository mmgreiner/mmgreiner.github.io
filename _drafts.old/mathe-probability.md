---
title: "Math question on probability"
categories: math 
tags: probability homework school
---

{% include mathscripts.html %}

## First question

Question: What is the probability that with a dice you first get a 3 and then an even number? Why do I have to multiply?

~~~mermaid
stateDiagram-v2
    [*] --> 1
    [*] --> 2
    [*] --> 3
    [*] --> 4
    [*] --> 5
    [*] --> 6
    1 --> 1'1
    1 --> 1'...
    1 --> 1'6
    2 --> 2'...
    3 --> 1'3
    3 --> 2'3
    3 --> 3'3
    3 --> 4'3
    3 --> 5'3
    3 --> 6'3
    4 --> 4'...
    5 --> 5'...
    6 --> 6'...
    2'3 --> [*]
    4'3 --> [*]
    6'3 --> [*]
~~~

Visualization:

[![](https://mermaid.ink/img/pako:eNpdkTEPgjAQhf8KucXEAAkUOnRwcnVytA4NrUq0YEoxMYT_bqUFPTr1e-_1Ls0boGqlAgadFVbta3E1QievnDeRO6ftOUqSXZRhXLkEY4GxxEg9Zn7uJsOYpikWQjz3e38-CT75xxwjwVhgLDHSGYsQXlaVIb4INDxYBLd3ktwfw4y1QNfCfCAGrYwWtXQdDF-Pg70prTgwd5XC3DnwZnQ50dv2-G4qYNb0Kob-KX-VAbuIR-dUJWvbmoMvdep2_AA1ZHFf?type=png)](https://mermaid.live/edit#pako:eNpdkTEPgjAQhf8KucXEAAkUOnRwcnVytA4NrUq0YEoxMYT_bqUFPTr1e-_1Ls0boGqlAgadFVbta3E1QievnDeRO6ftOUqSXZRhXLkEY4GxxEg9Zn7uJsOYpikWQjz3e38-CT75xxwjwVhgLDHSGYsQXlaVIb4INDxYBLd3ktwfw4y1QNfCfCAGrYwWtXQdDF-Pg70prTgwd5XC3DnwZnQ50dv2-G4qYNb0Kob-KX-VAbuIR-dUJWvbmoMvdep2_AA1ZHFf)

## Second question

Aus einer Urne mit 7 roten und 3 schwarzen Kugeln wird dreimal nacheinander eine Kugel gezogen und jeweils nach der Ziehung wieder in die Urne zurückgelegt.

Wie gross ist die Wahrscheinlichkeit

1.  jedesmal eine rote Kugel zu ziehen?
2. genau zweimal eine schwarze Kugel zu ziehen?
3. mindestens eine schwarze Kugel zu ziehen?

### Jedesmal eine rote:

$$ P(3 rote |10 total) = 3/10, drei mal: 3/10 * 3/10 * 3/10 = 9 / 1000 $$

### genau zwei mal eine schwarze Kugel
## References

https://www.math.net/probability-and-statistics

[mermaid-state-diagram]: https://mermaid-js.github.io/mermaid/#/stateDiagram