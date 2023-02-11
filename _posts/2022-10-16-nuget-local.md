---
title:  "Using local NuGet source / repository"
date: 2022-10-16
categories: programming
tags: nuget source
toc: true
---

Had some problems creating a local nuget repository. This is how I succeeded:

## Create a local nuget repository / source

    $ cli nuget add source ~/.mynuget

This leads to these changes in the configuration file:

    % cat ~/.nuget/NuGet/NuGet.Config
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
      <packageSources>
        <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
        <add key="Package source 1" value="/Users/mmgreiner/.mynuget" />
      </packageSources>
    </configuration>


## Prepare the project file

To the project config file (`*.fsproj` or `*.csproj`), add:

    <PackageId>CSLData</PackageId>
    <Version>1.0.0</Version>
    <Authors>Markus Greiner</Authors>
    <Company>mmgreiner</Company>

**Note** that the PackageID should not contain the version name, so don't: `CSLData.1.0.0.Test`. This will not work with publishing the nuget package.

## Pack and publish

    % dotnet pack
    MSBuild version 17.3.0+92e077650 for .NET
      Determining projects to restore...
      All projects are up-to-date for restore.
      CSLData -> /Users/mmgreiner/Projects/bibTeX/CSL/CSLData/bin/Debug/net6.0/CSLData.dll
      Successfully created package '/Users/mmgreiner/Projects/bibTeX/CSL/CSLData/bin/Debug/CSLData.1.0.0.nupkg'.

Using nuget directly, you can add it locally.

    % nuget add /Users/mmgreiner/Projects/bibTeX/CSL/CSLData/bin/Debug/CSLData.1.0.0.nupkg -source ~/.mynuget
 
Using the CLI it works like this: 

    % dotnet nuget push bin/Debug/CSLData.1.0.0.nupkg -s ~/.mynuget

**Note**: in the dotnet CLI, the command is called `push` which corresponds to `nuget add`. The CLI command `dotnet nuget add` adds a new source, ie. repository, not a package.

This generates the following file structure:

    /Users/mmgreiner/.mynuget
    ├── csldata
    │   └── 1.0.0
    │       ├── csldata.1.0.0.nupkg
    │       ├── csldata.1.0.0.nupkg.sha512
    │       └── csldata.nuspec

## Use it
Now you can use it in another project:

    $ dotnet add package CSLData

The source does not have to be given.

