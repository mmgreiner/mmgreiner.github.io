---
title:  "PDF converter in dotnet"
categories: programming
tags: pdf dotnet F# 
---

There is once in a while the need to extract pure text from PDF files within dotnet. This blog shows some attempts.

## SautinSoft

see [SautinSoft](https://sautinsoft.com/products/pdf-focus/examples/convert-pdf-file-to-text-file-csharp-vb-net.php).

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
