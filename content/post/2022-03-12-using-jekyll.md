---
categories:
- publishing
date: "2022-03-12T19:00:00Z"
tags:
- jekyll
- github
- F#
- websharper
title: Using Jekyll
showToc: true
---

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

## Post filenames

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file, for instance `.md`. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{{< highlight ruby >}}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{{< / highlight >}}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/


## Installing

Under OSX, the following line has to be added to the Gemfile:

~~~~ruby
gem "github-pages", "~> 225", group: :jekyll_plugins
gem "webrick"
~~~~

## Deploying

These posts are stored at [GitHub](https://github.com/mmgreiner/mmgreiner.github.io) and published to Github Pages [mmgreiner.github.io](https://mmgreiner.github.io).

##  Compiling

To compile into pages, use: 

````csh
 % bundle exec jekyll serve   
````

## Supported languages

To find out which programming languages are supported, use `rogify`:

~~~csh
% rogify list
== Available Lexers ==
abap: SAP - Advanced Business Application Programming

actionscript: ActionScript [aliases: as,as3]

ada: The Ada 2012 programming language

apache: configuration files for Apache web server

...

yaml: Yaml Ain't Markup Language (yaml.org) [aliases: yml]

yang: Lexer for the YANG 1.1 modeling language (RFC7950)

zig: The Zig programming language (ziglang.org) [aliases: zir]

~~~

[rogify]: https://simpleit.rocks/ruby/jekyll/what-are-the-supported-language-highlighters-in-jekyll/
