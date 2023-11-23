---
categories:
    - programming
tags:
    - F#
title: Command line parsing
toc: true
---

## CommandLineParser

<https://github.com/commandlineparser/commandline>

<https://www.nuget.org/packages/CommandLineParser.FSharp/>

<https://github.com/commandlineparser/commandline/wiki>

~~~~fsharp
type options = {
  [<Option('r', "read", Required = true, HelpText = "Input files.")>] files : seq<string>;
  [<Option(HelpText = "Prints all messages to standard output.")>] verbose : bool;
  [<Option(Default = "русский", HelpText = "Content language.")>] language : string;
  [<Value(0, MetaName="offset", HelpText = "File offset.")>] offset : int64 option;
}

let main argv =
  let result = CommandLine.Parser.Default.ParseArguments<options>(argv)
  match result with
  | :? Parsed<options> as parsed -> run parsed.Value
  | :? NotParsed<options> as notParsed -> fail notParsed.Errors
~~~~


## Argu

<https://fsprojects.github.io/Argu/>

<https://fsprojects.github.io/Argu/tutorial.html>

~~~fsharp
type Arguments =
    | Working_Directory of path:string
    | Listener of host:string * port:int
    | Log_Level of level:int
    | Detach

~~~


## GoFlag

<https://github.com/mikeruhl/Goflag>

