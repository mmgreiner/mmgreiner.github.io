---
categories:
- programming
- template
date: "2022-08-29T00:00:00Z"
tags:
- F#
- html
- liquid
- handlebars
- fluid
title: Static templating with handlebars, liquid and others
showToc: true
---

Templating is an proofen method to create static web pages. Typically, you write your html pages and fill in some parameters at page assembly time. Well-known templating languages are [Liquid], which is written in ruby and used by [Jekyll], and [Handlebars], which calls itself *minimal templating on steroids*.

For both there exist dotnet versions: [Handlebars.Net] and [DotLiquid].

This post compares these two. In addition, it also takes a brief look at [Scriban] and [Fluid].

Code samples are available at [Github mmgreiner](https://github.com/mmgreiner/Templating).

## The task
You have several persons which have tasks to perform. We will test how F# types, particularely sequences, records and options, are handled.

~~~ fsharp
type Chore = 
    {
        Name: string
        DueDate: DateTime
        Completed: DateTime option
    }

type Person = 
    {
        Name: string
        Chores: Task seq 
    }

module Data = 
    let persons = seq {
        { 
            Name = "Markus"
            Tasks = seq { 
                { Name = "Clean House"; DueDate = DateTime.Today; Completed = Some DateTime.MaxValue}
                { Name = "Feed the cat Milou"; DueDate = DateTime.Today; Completed = None }
            }
        }
        {
            Name = "Milou"
            Tasks = Seq.empty
        }
}
~~~

## Handlebars

Handlebars use a very simple syntax. Expressions are ``{{`` 
followed by content, followed by ``}}``. 
Blocks are used for iterations ``{{#each}}`` or if statements ``{{#if}}``. A simple [Handlebars] line looks like this:

    <p>Hallo {{Name}}</p>

### Installation

~~~~ csh
$ dotnet add package Handelbars.Net
~~~~

And in FSharp:

~~~~ fsharp
open HandlebarsDotNet
~~~~

### Template

~~~~
{% raw %}
let templateString = """
<ul>
{{#each Persons}}
    <li>
    <p>{{this.Name}} has the following tasks to do:</p>
    <ul>
        {{#each this.Chores}}
        <li>{{this.Name}} by {{date_format this.DueDate}}{{#if this.Completed}}, completed {{date_format this.Completed.Value}} {{/if}}
        </li>
        {{else}}
        <li>Free afternoon</li>
        {{/each}}
    </ul>
{{/each}}
</ul>
"""
{% endraw %}
let template = Handlebars.Compile(templateString)
~~~~

The html template is a simple list of persons with a list for each person of it's tasks.

Some points to note:
- ``Completed``is of type ``Option<DateTime>``. So first we have to test that it is not null 
``{{#if this.Completed}}``, and then access it's Value. 
- if a list is empty, you can add ``{{else}}`` which is processed for the empty list.
- ``date_format``is a helper that is not part of standard [Handlebars].

### Helpers

You can create your own helpers, as documented in [Handlebars]. the ``context`` typically corresponds to the ``this`` of the expression, but you can also use parameters for it. 

~~~~ fsharp
    Handlebars.RegisterHelper ("date_format", fun writer (context: Context) (parameters: Arguments) ->
        let dt =
            match parameters |> Seq.head with
            | :? DateTime as d -> d.ToString("yyyy-MM-dd")
            | _ -> sprintf "Type of %s is %A, not DateTime" (context.Value.ToString()) (context.Value.GetType())
        writer.WriteSafeString(dt)
    )
~~~~

### Template location
tbd

### The program
First, we need to define a anonymous record with the field ``Projects``. Then, we create the template and instantiate it with the data:

~~~~ fsharp
let data = {| Persons = persons |}
let result = template.Invoke(data)
~~~~

This should give us:

~~~~ html
<ul>
    <li>
    <p>Markus has the following tasks to do:</p>
    <ul>
        <li>Clean House by 2022-08-29, completed 9999-12-31 
        </li>
        <li>Feed cat by 2022-08-29
        </li>
    </ul>
    <li>
    <p>Milou has the following tasks to do:</p>
    <ul>
        <li>Free afternoon</li>
    </ul>
</ul>
~~~~

## DotLiquid

[Liquid] was made popular by Ruby based static templating engines like [Jekyll]. [DotLiquid] is a dotnet base implementation.

### Installation

~~~~ csh
> dotnet add package DotLiquid
~~~~

And in FSharp:

~~~~ fsharp
open DotLiquid
open FSharp.Reflection
~~~~

We will use reflection to register types.

### Template

~~~~ fsharp
let templateString = """
{%raw%}<ul>
{% for person in Persons %}
    <li>
    <p>{{person.Name}} has the following tasks to do:</p>
    <ul>
        {% for task in person.Chores %}
            <li>{{ task.Name }} by {{task.DueDate | Date: "%Y-%m-%d"}} {%if task.Completed %}, completed {{task.Completed.Value | date: "%Y-%m-%d"}} {%endif%}
            </li>
        {% else %}Free afternoon
        {% endfor %}
    </ul>
{%endfor%}
</ul>{%endraw%}
"""
~~~~

### Registering types
[Liquid] and [DotLiquid] are very careful to only expose record fields that have explicitly been declared to be visible. This requires us to register all necessary types. A little helper function can support here.

~~~~ fsharp
/// Helper function to register types
let RegisterRecordType (typ: Type) =
    let members (typ: Type) =
        if FSharpType.IsRecord typ then
            FSharpType.GetRecordFields(typ)
            |> Seq.map (fun f -> f.Name)
            |> Seq.toArray
        else
            Array.empty

    Template.RegisterSafeType(typ, members typ)

[ typeof<Person>; typeof<Task>; typeof<PersonList> ] |> List.iter RegisterRecordType

// DateTime option is not a record type, so the above registration does not work
Template.RegisterSafeType(typeof<Option<DateTime>>, [|"Value"|])
~~~~

Note that ``DateTime option`` is not a record and has to be registered separately. I did not succeed in registering a generic type like ``Option<_>``.

Note that we also declared the type ``type PersonList = { Persons: Person seq }`` to have a type for the top level.


Helper functions: tbd 

### Naming Conventions

[DotLiquid] supports Ruby style naming convention (snake case) and CamelCase conventions. By default, it uses Ruby casing conventions, meaning that it turns turns names like ``DueDate`` into ``due_date``.

To prevent this, you can set the nameing convention:

~~~~ fsharp
Template.NamingConvention <- new NamingConventions.CSharpNamingConvention()
~~~~

Unfortunately, this has the side effect that standard [Liquid] functions like ``date`` also now follow the CamelCase naming convention and become ``Date``. I had to look into [StandardFilter.cs](https://github.com/dotliquid/dotliquid/blob/master/src/DotLiquid/StandardFilters.cs) to fully understand this.

We created our own naming convention for this:

~~~~ fsharp
type CamelCaseNamingConvention() =
    let UpperFirstLetter (str: string) = 
        string (Char.ToUpperInvariant(str.[0])) + str.[1..]
    let LowerFirstLetter (str: string) = 
        string (Char.ToLowerInvariant(str.[0])) + str.[1..]

    interface NamingConventions.INamingConvention with
        member val StringComparer: StringComparer = StringComparer.Ordinal
        
        member this.GetMemberName name = name

        member this.OperatorEquals(testedOperator, referenceOperator): bool =
            UpperFirstLetter testedOperator = referenceOperator 
            || LowerFirstLetter testedOperator = referenceOperator
            || testedOperator = LowerFirstLetter referenceOperator

~~~~

In the end, we used the supplied `NamingConventions.CSharpNamingConvention` and wrote `Date` for the pipe function instead of `date`.

### Date formats

See [DotLiquid formatting](http://dotliquidmarkup.org/docs/formatting). By default, DotNet formatting for dates is used. You can change this by:

~~~ fsharp
Liquid.UseRubyDateFormat <- true
~~~

### The program

~~~~ fsharp
let data = { Persons = persons }
template.Render(Hash.FromAnonymousObject(data))
~~~~

## Fluid

[Fluid] is another open-source templating language based on [liquid]. It's benchmark compares it to [DotLiquid] and [Handlebars] and claims it to be faster compared to those (see [fluid-benchmark](https://github.com/sebastienros/fluid#benchmarks)).


## Scriban

>  [Scriban] is a fast, powerful, safe and lightweight scripting language and engine for .NET, which was primarily developed for text templating with a compatibility mode for parsing [Liquid] templates.

Its templating language is less baroque than Liquids, but it can handle liquid input correctly.
According to its [benchmark](https://github.com/scriban/scriban/blob/master/doc/benchmarks.md), it outperforms [DotLiquid], [Handlebars.Net], and [fluid].

You install *Scriban* using `dotnet add package Scriban`. The template would look like this:

~~~ html
{%raw%}<ul>
{{- for person in Persons }}
    <li>
    <p>{{person.Name}} has the following tasks to do:</p>
    <ul>
        {{- if person.Chores | array.size == 0 }}
        Free afternoon
        {{ else }}
        {{- for task in person.Chores }}
            <li>{{ task.Name }} by {{task.DueDate | date.to_string "%Y-%m-%d"}}{{if task.Completed -}}, completed {{task.Completed.Value | date.to_string "%Y-%m-%d"}} {{end}}
            </li>
        {{- end }}
        {{- end }}
    </ul>
{{- end -}}
</ul>{%endraw%}
~~~

The code is simple:

~~~ fsharp
open Scriban

let template = Template.Parse(templateString)
let persons = {| Persons = Chores.Data.persons |}

let result = template.Render(persons, fun m -> m.Name)
printfn "%s" result
~~~

By default, Scriban uses standard ruby naming with underlines. 
To prevent this, we have to use the `MemberRenamer`delegate with `fun m -> m.Name`. See [Member Renamer](https://github.com/scriban/scriban/blob/master/doc/runtime.md#member-renamer) in the Scriban documentation for more information.


## Comparison

Without going into too much details, it seems that [Scriban] is by far the easiest to use, and, according to its own benchmark, the fastest.

## Note on this post
Since this is a [Jekyll] post, it uses [Liquid] iself. This means that all references to {{...}} had to be encapsulated in *&#123;%raw%&#125;* and *&#123;%endraw%&#125;*.


[Liquid]: https://shopify.github.io/liquid/ 
[Jekyll]: https://jekyllrb.com
[Handlebars]: https://handlebarsjs.com
[DotLiquid]: http://dotliquidmarkup.org
[Handlebars.Net]: https://github.com/Handlebars-Net/Handlebars.Net
[Fluid]: https://github.com/sebastienros/fluid
[Scriban]: https://github.com/scriban/scriban/blob/master/doc/language.md
