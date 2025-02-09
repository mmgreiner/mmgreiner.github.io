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
title: PDF to text conversion
---

There is once in a while the need to extract pure text from PDF files. This blog shows some attempts to do it with dot, with [ruby], and with command line utitilites.

## Dotnet

### SautinSoft

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

### iText7

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

## Command line utilities

[xpdf]: http://foolabs.com/xpdf
[poppler]: https://poppler.freedesktop.org/

I found two command lines utilities, [xpdf] andn [poppler] which does a lot of pdf conversions. 

### xpdf

It can be installed using:

    brew install xpdf
    
Another option according to above is [poppler](http://foolabs.com/xpdf/download.html):

    brew install poppler
    
Poppler is a library, not a command line utility.

I haven't tried either so far.

### Other utilities

[xpdf] installs the following command line tools:

* **xpdf**: PDF viewer ([click for a screenshot](http://www.xpdfreader.com/img/screenshot.png))
* **pdftotext**: converts PDF to text
* **pdftops**: converts PDF to PostScript
* **pdftoppm**: converts PDF pages to netpbm (PPM/PGM/PBM) image files
* **pdftopng**: converts PDF pages to PNG image files
* **pdftohtml**: converts PDF to HTML
* **pdfinfo**: extracts PDF metadata
* **pdfimages**: extracts raw images from PDF files
* **pdffonts**: lists fonts used in PDF files
* **pdfdetach**: extracts attached files from PDF files

~~~
% pdftotext -h
pdftotext version 4.05 [www.xpdfreader.com]
Copyright 1996-2024 Glyph & Cog, LLC
Usage: pdftotext [options] <PDF-file> [<text-file>]
  -f <int>               : first page to convert
  -l <int>               : last page to convert
  -layout                : maintain original physical layout
  -simple                : simple one-column page layout
  -simple2               : simple one-column page layout, version 2
  -table                 : similar to -layout, but optimized for tables
  -lineprinter           : use strict fixed-pitch/height layout
  -raw                   : keep strings in content stream order
  -fixed <number>        : assume fixed-pitch (or tabular) text
  -linespacing <number>  : fixed line spacing for LinePrinter mode
  -clip                  : separate clipped text
  -nodiag                : discard diagonal text
  -enc <string>          : output text encoding name
  -eol <string>          : output end-of-line convention (unix, dos, or mac)
  -nopgbrk               : don't insert a page break at the end of each page
  -bom                   : insert a Unicode BOM at the start of the text file
  -marginl <number>      : left page margin
  -marginr <number>      : right page margin
  -margint <number>      : top page margin
  -marginb <number>      : bottom page margin
  -opw <string>          : owner password (for encrypted files)
  -upw <string>          : user password (for encrypted files)
  -verbose               : print per-page status information
  -q                     : don't print any messages or errors
  -cfg <string>          : configuration file to use in place of .xpdfrc
  -listencodings         : list all available output text encodings
  -v                     : print copyright and version info
  -h                     : print usage information
  -help                  : print usage information
  --help                 : print usage information
  -?                     : print usage information

~~~

