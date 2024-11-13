---
categories:
- programming
date: "2022-12-01T00:00:00Z"
draft: false
tags:
- pdf
- dotnet
- F#
- ruby
title: PDF converter in dotnet and ruby
---

There is once in a while the need to extract pure text from PDF files. This blog shows some attempts to do it with dot, and also with [ruby].

## SautinSoft

See [SautinSoft](https://sautinsoft.com/products/pdf-focus/examples/convert-pdf-file-to-text-file-csharp-vb-net.php).

~~~fsharp
#r "nuget: sautinsoft.pdffocus"

let fn = "filename"
let focus = new SautinSoft.PdfFocus();
focus.OpenPdf(fn)
printfn "page count = %i" focus.PageCount
let result = focus.ToText()
printfn "result: %s" result
~~~

Unfortunately, the result was always empty.

## iText7

See [itext7-dotnet](https://github.com/itext/itext7-dotnet).

See [dotnet-queries](http://dotnetqueries.com/Article/239/Extracting-text-from-pdf-using-iText7-c).

~~~fsharp
#r "nuget: itext7"

open iText.Kernel
open iText.Kernel.Pdf

let reader = new PdfReader(fn)
let doc = new PdfDocument(reader)
let pages = [ for pn in 1 .. doc.GetNumberOfPages() do yield doc.GetPage(pn) ]

pages 
|> Seq.map (fun page -> Canvas.Parser.PdfTextExtractor.GetTextFromPage(page))
|> String.concat Environment.NewLine

~~~

This works nicely and extracts the text from the PDF file. Note that this was not tried on PDF files which were scanned.

## Ruby

With [ruby], I found some quite useful libraries ([ruby] calls them *gems*) to do this.

[pdf-reader] will read a pdf file (not a scanned one) and transfer it per page into text. To use it, you first have to install the gem:

[pdf-reader]: https://github.com/yob/pdf-reader

~~~sh
$ gem install pdf-reader
~~~

Then use it in your ruby code:

~~~ruby
require 'pdf-reader'

reader = PDF::Reader.new("myfile.pdf")
text = reader.pages.map do |page|
    page.text
end
~~~

The docmentation of [pdf-reader] is quite good, it worked successfully for me to analyse several thousand pdf files.

[ruby]: https://www.ruby-lang.org/en/