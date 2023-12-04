---
categories:
- programming
date: "2022-08-01T00:00:00Z"
draft: true
tags:
- jekyll
- fsharp
title: Using CSV Helper to write CSV files
showToc: true
---

## Usage of CsvHelper

[CsvHelper] is a dotnet C# library for reading and wrinting CSV files.

It is defined for C#, so there are some things you have to think about for [F#].

There is another library better suited for [F#], the [FSharp.Data] library. However, I found it difficult to use it to write CSV files and I found no documentation, only this entry on [stackoverflow](https://stackoverflow.com/questions/33075932/how-to-create-a-csv-file-and-write-data-into-in-f).



### Streaming stuff

1. reading from a stream
1. reading from a string
1. reading from a file

### Option types

[F#] option types are not natively handled by [CsvHelper]. This [stackoverflow entry](https://stackoverflow.com/questions/66401283/how-do-you-use-csvhelper-csvwriter-with-f-option-types) shows how it can be done:

~~~fsharp
open CsvHelper

type OptionConverter<'T>() =
    inherit CsvHelper.TypeConversion.DefaultTypeConverter()
    override __.ConvertToString(value, row, memberMapData) =
        match value :?> Option<'T> with
            | None -> ""
            | Some x -> base.ConvertToString(x, row, memberMapData)

type Record = { X : string; Y : float option }

let writeString x =
    use writer = new StringWriter()
    use csv = new CsvWriter(writer, CultureInfo.InvariantCulture)
    csv.Context.TypeConverterCache.AddConverter<Option<float>>(OptionConverter<float>())
    csv.WriteRecords(x)
    writer.ToString()

[{X = "a"; Y = None}; {X = "Pi"; Y = Some 3.14159}
~~~

Should give you:

    X,Y
    a,
    Pi,3.14159
    


### Mutable records

If you are reading a CSV file with with [CsvHelper] or [FSharp.Data], you need to attribute the record as mutable otherwise the compiler will not produce a class constructor.

~~~fsharp
[<CLIMutable>]
type Person = { Name: string; Birthday: DateTime }
~~~~


### Aside: names of F# record fields.



[F#]: {{< param "fsharp_link" >}}
[CsvHelper]: https://joshclose.github.io/CsvHelper/
[FSharp.Data]: https://fsprojects.github.io/FSharp.Data/library/CsvProvider.html
