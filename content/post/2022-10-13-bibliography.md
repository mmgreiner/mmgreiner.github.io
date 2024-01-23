---
categories:
- publishing
- programming
date: "2022-10-13T00:00:00Z"
tags:
- Zotero
- citation
- bibliography
- C#
- json-schema
title: Bibliographies, citations and dotnet programming
showToc: true
---


Helping out a friend to get a Matura thesis completed, I stumbled onto the topics of citations and bibliographies. In the end, I wanted to get a handle on bibliographies using dotnet code.

## Zotero

I installed [Zotero], which was recommended by my daughter who had used it for her master thesis. It basically is a database (located either locally on your computer or on the web, with optional synchronization) of all the books, articles, etc you want in your bibliography.

Note: On OSX, another commonly used tool is [BibDesk]. 

[BibDesk]: https://bibdesk.sourceforge.io

### Better BibTeX
There is an extension to [Zotero]  called [Better-BibTeX](https://retorque.re/zotero-better-bibtex/), which I installed.

> Better BibTeX (BBT) is a plugin for [Zotero](https://www.zotero.org/) and [Juris-M](https://juris-m.github.io/) that makes it easier to manage bibliographic data, especially for people authoring documents using text-based toolchains (e.g. based on [LaTeX](https://www.latex-project.org/) / [Markdown](https://www.markdownguide.org/)).

#### Features

- Facilities for generating citation keys
- Conversion between formats and encodings
- Facilities for exporting data from Zotero

#### Visual Studio Code

Pick citation from Visual Studio Code: <https://marketplace.visualstudio.com/items?itemName=mblode.zotero>.

It pops up a little window which lets you choose the bibliography entry from Zotero and creates a unique identifier:

  @durrenmattDurcheinandertalRoman1989

However, it is unclear how this can be used inside markdown.

#### Further information

I also found this link <https://lucidmanager.org/productivity/hugo-bibliography/> which is worth reading. It defines particals for citations.



## File Formats

The oldest and best known (but cryptic) file format for a bibliography is [BibTeX]. It comes in a `.bib` file. A typical format looks like this:

~~~
@book{durrenmatt_durcheinandertal_1989,
	address = {Zürich},
	title = {Durcheinandertal: {Roman}},
	isbn = {978-3-257-01820-2},
	shorttitle = {Durcheinandertal},
	publisher = {Diogenes},
	author = {Dürrenmatt, Friedrich},
	year = {1989},
}

@article{noauthor_wer_2022,
	title = {Wer entblösst hier eigentlich wen?},
	issn = {2504-4648},
	journal = {Aargauer Zeitung},
	month = aug,
	year = {2022},
	pages = {6--7},
}
~~~

These files are difficult to parse with a software program. So I looked for other possible formats to store bibliographies. [Zotero] lists these export formats:

![Zotero Export](/images/Screenshot%202022-10-13%20at%2009.21.50.png)

These are described in more detail in [Zotero-data-formats](https://www.zotero.org/support/dev/data_formats).
I chose to take a look into the CSL Json format.

### CSL Json

CSL stands for Citation Style Language. I found a good overview and the JSON schema on [Github-CSL]. 

[Github-CSL]: https://github.com/citation-style-language/schema
[CSL-Data-Json-Schema]: https://github.com/citation-style-language/schema/blob/master/schemas/input/csl-data.json
[CSL-schema-raw]: https://github.com/citation-style-language/schema/raw/master/schemas/input/csl-data.json

[citeproj-js]: https://citeproc-js.readthedocs.io/en/latest/csl-json/markup.html

> This is the official repository for schemas describing the Citation Style Language (CSL). Current schemas include:
>
> - CSL schema - describes CSL style and locale XML files
> - CSL-JSON schema - describes a commonly used JSON data model for storing CSL processor input (such as bibliographic metadata).
>
> For more information about CSL, visit [https://citationstyles.org](). For general quesions and discussions have a look at the CSL-forum.

A sample CLS Json file may look like this:

~~~json
[
  {
    "id": "http://zotero.org/users/10298668/items/SH797ZU6",
    "type": "article-newspaper",
    "container-title": "Aargauer Zeitung",
    "ISSN": "2504-4648",
    "page": "6-7",
    "title": "Wer entblösst hier eigentlich wen?",
    "issued": {
      "date-parts": [
        [
          "2022",
          8,
          27
        ]
      ]
    }
  },
  {
    "id": "http://zotero.org/users/10298668/items/TL3Z9724",
    "type": "book",
    "call-number": "MLCS 90/03506 (P)",
    "event-place": "Zürich",
    "ISBN": "978-3-257-01820-2",
    "number-of-pages": "175",
    "publisher": "Diogenes",
    "publisher-place": "Zürich",
    "source": "Library of Congress ISBN",
    "title": "Durcheinandertal: Roman",
    "title-short": "Durcheinandertal",
    "author": [
      {
        "family": "Dürrenmatt",
        "given": "Friedrich"
      }
    ],
    "issued": {
    "date-parts": [
      [
        "1989"
      ]
    ]
    }
  }
]
~~~

Obviously, this is easier to handle than the `.bib` format.

### CSL Json Schema

The CSL Json Schema describes how json files have to look like to confirm to CSL. There are two CSL Json schema definitions:

- [CSL-Data-Json-Schema] for a **bibliographic** entry
- [CSL-Citaton-Json-Schema](https://github.com/citation-style-language/schema/blob/master/schemas/input/csl-citation.json) for a **citation** entry

The [CSL-Data-Json-Schema] looks like this:

~~~json
{
  "description": "JSON schema for CSL input data",
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://resource.citationstyles.org/schema/v1.0/input/json/csl-data.json",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "type": {
        "type": "string",
        "enum": [
          "article",
          "article-journal",
          "article-magazine",
          "article-newspaper",
          "bill",
          "book",
          "broadcast",
          "chapter",
          "classic",
          "collection",
          "dataset",
          "document",
          "entry",
          "entry-dictionary",
          "entry-encyclopedia",
          "event",
          "figure",
          "graphic",
          "hearing",
          "interview",
          "legal_case",
          "legislation",
          "manuscript",
          "map",
          "motion_picture",
          "musical_score",
          "pamphlet",
          "paper-conference",
          "patent",
          "performance",
          "periodical",
          "personal_communication",
          "post",
          "post-weblog",
          "regulation",
          "report",
          "review",
          "review-book",
          "software",
          "song",
          "speech",
          "standard",
          "thesis",
          "treaty",
          "webpage"
        ]
      },
      "id": {
        "type": [ "string", "number" ]
      },
      "citation-key": {
        "type": "string"
      },
~~~

Since in the end I wanted to use software to parse CSL-formatted bibliographies, I needed to turn these JSON files into dotnet classes.

A good another link to look at, particularly concering the handling of author names, is [citeproj-js]. 

## From Json Schema to dotnet classes

I started with the [CSL-Data-Json-Schema]. This now needed to be converted into dotnet classes. For this, I found the tool [NJsonSchema] by [Rico Suter](https://blog.rsuter.com/about/). 

[NJsonSchema]: https://github.com/RicoSuter/NJsonSchema

This promised to generate C# classes from the given schema. I used this code to do so:

~~~fsharp
let raw = "https://github.com/citation-style-language/schema/raw/master/schemas/input/csl-data.json"

let schema = NJsonSchema.JsonSchema.FromUrlAsync(raw).Result

let generator = new NJsonSchema.CodeGeneration.CSharp.CSharpGenerator(schema)
let file = generator.GenerateFile()

let outfn = Path.Combine(__SOURCE_DIRECTORY__, "gen-classes.cs")
printfn "writing to %s" outfn
File.WriteAllText(outfn, file)

~~~

Now the generated C# code looks like this:

~~~csharp
//----------------------
// <auto-generated>
//     Generated using the NJsonSchema v10.8.0.0 (Newtonsoft.Json v9.0.0.0) (http://NJsonSchema.org)
// </auto-generated>
//----------------------
namespace MyNamespace
{
    #pragma warning disable // Disable all warnings

    [System.CodeDom.Compiler.GeneratedCode("NJsonSchema", "10.8.0.0 (Newtonsoft.Json v9.0.0.0)")]
    public partial class Anonymous
    {
        [Newtonsoft.Json.JsonProperty("type", Required = Newtonsoft.Json.Required.Always)]
        [System.ComponentModel.DataAnnotations.Required(AllowEmptyStrings = true)]
        [Newtonsoft.Json.JsonConverter(typeof(Newtonsoft.Json.Converters.StringEnumConverter))]
        public Type Type { get; set; }

        [Newtonsoft.Json.JsonProperty("id", Required = Newtonsoft.Json.Required.Always)]
        [System.ComponentModel.DataAnnotations.Required(AllowEmptyStrings = true)]
        public double Id { get; set; }

        [Newtonsoft.Json.JsonProperty("citation-key", Required = Newtonsoft.Json.Required.DisallowNull, NullValueHandling = Newtonsoft.Json.NullValueHandling.Ignore)]
        public string CitationKey { get; set; }
        // ...
~~~

I changed two things:

- rename namespace and class name to `CSLData``
- change the type of `Id` to string. In the [CSL-schema-raw], the type of Id had two possibilities: `string` or `double`. The converter picked double which caused run time exceptions.

This code was put into a dotnet library `CSLData`.

## Parsing a CSL File

Now to some code to parse the CSL files. I tried the following:

~~~fsharp
// #r "nuget: Newtonsoft.Json"
open System
open System.Text.Json
open CSLData
open Newtonsoft.Json

let fn = "./Exported-Items.json"
let json = IO.File.ReadAllText(fn)

// standard does NOT work, all data is zero
let csl = System.Text.Json.JsonSerializer.Deserialize<CSLData.CSLData list>(json)
printfn "csl = %A %A" csl csl.Head.Id

// Newtonsoft works
let csl' = Newtonsoft.Json.JsonConvert.DeserializeObject<CSLData.CSLData list>(json)
printfn "cls' = %A %A" csl' csl'.Head.Id
~~~

Turns out that System.Text.Json deserializes without exception, but all the values are zero or null. The Newtonsoft deserializer works.

- [ ] One issue still open is to make this more F# friendly with options instead of nulls.


## More on [Zotero]

I installed Zotero. It is programmed in JavaScript and uses a SQLite database as local storage [Zotero-dev].

You need to register as user, and then you can get an API UserId https://www.zotero.org/settings/keys. In my case, that is 12345678.

Using this user id, you can access your bibliography library using:

~~~bash
$ curl https://api.zotero.org/users/12345678/collections
[
  {
    "key": "KEYKEY",
    "version": 3,
    "library": {
    "type": "user",
    "id": 12345678,
    "name": "mmgreiner",
    "links": {
      "alternate": {
        "href": "https://www.zotero.org/mmgreiner",
        "type": "text/html"
      }
    }
  },
  "links": {
    "self": {
      "href": "https://api.zotero.org/users/10298668/collections/KEYKEY",
      "type": "application/json"
    },
    "alternate": {
      "href": "https://www.zotero.org/mmgreiner/collections/KEYKEY",
      "type": "text/html"
    }
  },
  "meta": {
    "numCollections": 0,
    "numItems": 17
  },
  "data": {
    "key": "XXXXX",
    "version": 3,
    "name": "Lea",
    "parentCollection": false,
    "relations": {}
    }
  },
  {
  "key": "XXXXX",
  "version": 3,
  "library": {
  "type": "user",
  "id": 12345678,
  "name": "mmgreiner",
  "links": {
    "alternate": {
      "href": "https://www.zotero.org/mmgreiner",
      "type": "text/html"
      }
    }
  },
  "links": {
  "self": {
  "href": "https://api.zotero.org/users/10298668/collections/KEYKEY",
  "type": "application/json"
  },
  "alternate": {
  "href": "https://www.zotero.org/mmgreiner/collections/KEYKEY",
  "type": "text/html"
  }
  },
  "meta": {
  "numCollections": 0,
  "numItems": 2
  },
  "data": {
  "key": "XXXXX",
  "version": 3,
  "name": "Greiner",
  "parentCollection": false,
  "relations": {}
  }
} 
~~~


## Citations

There are many standards how the citation and the bibliography could look. there is even a language to define how citations could look: [CSL](https://citationstyles.org/):  

For instance:
* [AAA](https://www.scribbr.com/citing-sources/citation-styles/#aaa)
* [APA](https://www.scribbr.com/citing-sources/citation-styles/#apa)
* [APSA](https://www.scribbr.com/citing-sources/citation-styles/#apsa)
* [ASA](https://www.scribbr.com/citing-sources/citation-styles/#asa)
* [Chicago (Turabian) author-date](https://www.scribbr.com/citing-sources/citation-styles/#chicago-author-date)
* [CSE name-year](https://www.scribbr.com/citing-sources/citation-styles/#cse-name-year)
* [Harvard](https://www.scribbr.com/citing-sources/citation-styles/#harvard)
* [MLA](https://www.scribbr.com/citing-sources/citation-styles/#mla)
* [ACS](https://www.scribbr.com/citing-sources/citation-styles/#acs)
* [AMA](https://www.scribbr.com/citing-sources/citation-styles/#ama)
* [CSE citation-name or citation-sequence](https://www.scribbr.com/citing-sources/citation-styles/#cse-citation-name-sequence)
* [IEEE](https://www.scribbr.com/citing-sources/citation-styles/#ieee)
* [NLM](https://www.scribbr.com/citing-sources/citation-styles/#nlm)
* [Vancouver](https://www.scribbr.com/citing-sources/citation-styles/#vancouver)


## Misc.

### ISBN Numbers

Automatically generate citation from ISBN number

### IMDb for films

Get Citations of films: No standard, but can use film database [IMDb](https://www.imdb.com), which has a sequence number, Ex Manhattan has number [Woody Allen: Manhattan](https://www.imdb.com/title/tt0079522/) tt0079522 

Code to generate from PDF: [bibtex-generator]

### Support in Github

See this [stackoverflow](https://stackoverflow.com/questions/26587527/cite-a-paper-using-github-markdown-syntax) article, or directly on [Github](https://github.blog/2021-08-19-enhanced-support-citations-github/).

Further information:
- <https://github.com/citation-file-format/citation-file-format#tools-to-work-with-citationcff-files-wrench>
- <https://github.com/citation-file-format/ruby-cff>

[Zotero]: https://www.zotero.org
[BibTeX]: http://www.bibtex.org/Format/
[Zotero-dev]: https://www.zotero.org/support/dev/start
[bibtex-generator]: 
https://boberle.com/projects/bibtex-entry-generator/online/
