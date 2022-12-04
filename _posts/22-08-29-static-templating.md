---
title:  "Static templating with handlebars and liquid"
categories: programming
tags: F# html liquid handlebars fluid
toc: true
---

## Static templating with handlebars and liquid

Templating is an older method to create static web pages. Typically, you write your html pages and fill in some parameters at page assembly time. Well-known templating languages are [Liquid], which is written in ruby and used by [Jekyll], and [Handlebars], which calls itself *minimal templating on steroids".

For both there exist dotnet versions: [Handlebars.Net] and [DotLiquid].

This post compares these two.

## The task
You have several persons which have tasks to perform. We will test how F# types, particularely sequences, records and options, are handled.

~~~ fsharp
type Task = 
    {
        Name: string
        DueDate: DateTime
        Completed: DateTime option
    }

type Person = 
    {
        Name: string
        Tasks: Task seq 
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

Handlebars use a very simple syntax. Expressions are ``{% raw %} {{ {% endraw %}`` 
followed by content, followed by ``{%raw%} }} {%endraw%}``. 
Blocks are used for iterations ``{%raw%}{{#each}}{%endraw%}`` or if statements ``{%raw%}{{#if}}{%endraw%}``. A simple [Handlebars] line looks like this:

    <p>Hallo {%raw%}{{Name}}{%endraw%}</p>

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
        {{#each this.Tasks}}
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
``{%raw%}{{#if this.Completed}}{%endraw%}``, and then access it's Value. 
- if a list is empty, you can add ``{%raw%}{{else}}{%endraw%}`` which is processed for the empty list.
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
to be done.

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
        {% for task in person.Tasks %}
            <li>{{ task.Name }} by {{task.DueDate | Date: "yyyy-MM-dd"}} {%if task.Completed %}, completed {{task.Completed.Value | date: "%Y-%m-%d"}} {%endif%}
            </li>
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

[fluid] is another open-source templating language based on [liquid]. It's benchmark compares it to [DotLiquid] and [Handlebars] and claims it to be faster compared to those (see [fluid-benchmark](https://github.com/sebastienros/fluid#benchmarks)).



## Note on this post
Since this is a [Jekyll] post, it uses [Liquid] iself. This means that all references to {%raw%}{{...}}{%endraw%} had to be encapsulated in *{%raw%}&#123;%raw%&#125;{%endraw%}* and *{%raw%}&#123;%endraw%&#125;{%endraw%}*.


[Liquid]: https://shopify.github.io/liquid/ 
[Jekyll]: https://jekyllrb.com
[Handlebars]: https://handlebarsjs.com
[DotLiquid]: http://dotliquidmarkup.org
[Handlebars.Net]: https://github.com/Handlebars-Net/Handlebars.Net
[fluid]: https://github.com/sebastienros/fluid
