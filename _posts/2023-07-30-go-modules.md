---
categories:
    - programming
tags:
    - go
title: How to work with modules and packages in Go
toc: true
---

I had to practice a long time to understand the [go] concepts of modules, packages, and how to publish them to [github]. Here are my learnings.

## Modules and packages and files

Modules contain one or several packages. A module is basically folder which contains a `go.mod` file.

~~~~
mkdir mymodule
cd mymodule
go mod init github.com/mmgreiner/mymodule
~~~~

This initializes a module which is called `github.com/mmgreiner/mymodule`. Note the prefix `github.com/mmgreiner`: this is later useful when deploying the module to github. 

`go mod init` creates the file `go.mod`, which has the same package name:

~~~~go
module github.com/mmgreiner/mymodule

go 1.19

require (
    ...
)
~~~~

### Executable modules

If this module is an executable, this folder must contain a file, typically called `main.go`, and the following content:

~~~~ go
package main

import (
    "fmt"
    "log"

func DoWhatever() {
    log.Println("hallo")
    fmt.Println("hallo")
}

func main() {
    println("hallo world")
}
~~~~

The package name must be `main` for an executable module.

### Packages
All go files within the same folder must have the same package name. You can create subpackages by creating sub-folders.

### Go files
All go files within the same (sub-) folder must have the same package name. 

### Library modules
Library modules do not have a `main` package, and there may be no go files in the top folder of the module.

The relationship between modules, packages and files is well explained in this [stack overflow article](https://stackoverflow.com/questions/61940117/go-modules-vs-package) 

## Using github

Just after you create the top folder, it is useful to initialize the local github repository with:

    git init .

If you did forget this step right in the beginning, you can always still perform:

    git init g
    git add .

After you are completed programming and testing, you commit it locally:

    git commit -m "initial commit"

You would also tag it with a version number. To understand version numbering in [go], see [module version numbers](https://go.dev/doc/modules/version-numbers#in-development). Initally, a good number would be `v0.0.1` for a development phase, or `v1.0.1-alpha` for a pre-release version.

### Create remote repository
I usually do this using Github Desktop or creating on [github] web page. To connect the local repository with the remote repository, use:

    git remote add origin "https://github.com/mmgreiner/<repository-name>.git"


### Tag with version number
You have to tag it locally, and then push it to the remote repository on `github.com`:

    git tag v0.0.1
    git push origin v0.0.1

*Note*: sometimes I messed up the tags and then had to delete them again, since [go] could not find the module. I followed this instruction [*How to delete git tags*](https://devconnected.com/how-to-delete-local-and-remote-tags-on-git/) to do this.

### Publishing
Very important: don't forget to list it in the go directory

~~~
$ GOPROXY=proxy.golang.org go list -m github/mmgreiner/mymodule@v0.0.1
~~~



### Publishing with homebrew

TBD

See <https://docs.brew.sh/Formula-Cookbook>.

See <https://docs.brew.sh/Adding-Software-to-Homebrew#formulae>

To create the ruby script for executable apps, use:

    brew create --go --set-version 0.0.1 https://github.com/mmgreiner/mymodule 



### Open questions

- How does it work with private git repositories? See [here](https://dev.to/gopher/how-to-use-go-modules-with-private-git-repository-53b4).



### References
This whole process is well explained in this go [article](https://go.dev/doc/modules/publishing).

## Local development
If you prefer not to publish your modules to [github], but keep them locally, proceed as follows.

TO-BE-DONE



[fsharp]: {{ site.fsharp_link }}
[go]: {{ site.go_link }}
[github]: https://github.com
