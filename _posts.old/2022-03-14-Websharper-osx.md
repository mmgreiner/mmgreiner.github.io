---
title:  "Websharper OSX"
categories: programming 
tags: osx websharper dotnet
toc: true
---

## Installing WebSharper

I'm developing web applications using [WebSharper] on macOS Monterey. 

To install the the templates, use:

~~~~sh
% dotnet new -i WebSharper.Templates
~~~~

The following templates are installed:

```sh
% dotnet new --list websh
These templates matched your input: 'websh'

Template Name                           Short Name       Language  Tags          
--------------------------------------  ---------------  --------  --------------
WebSharper 5 Client-Server Application  websharper-web   [C#],F#   WebSharper/Web
WebSharper 5 Extension                  websharper-ext   F#        WebSharper    
WebSharper 5 Html Site                  websharper-html  [C#],F#   WebSharper/Web
WebSharper 5 Library                    websharper-lib   [C#],F#   WebSharper    
WebSharper 5 Minimal Application        websharper-min   [C#],F#   WebSharper/Web
WebSharper 5 Proxy                      websharper-prx   [C#],F#   WebSharper    
WebSharper 5 Single Page Application    websharper-spa   [C#],F#   WebSharper/Web
```

To create a project, use `dotnet new websharper-web -lang F# -o outputFolder`.

After the project has been created, the project file looks as follows:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net5.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <Compile Include="Client.fs" />
    <Compile Include="Startup.fs" />
    <None Include="wsconfig.json" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="WebSharper" Version="5.0.0.120" />    
    <PackageReference Include="WebSharper.FSharp" Version="5.0.0.120" />
    <PackageReference Include="WebSharper.UI" Version="5.0.0.120" />
    <PackageReference Include="WebSharper.AspNetCore" Version="5.0.0.120" />
  </ItemGroup>

</Project>
```

These `5.0...` versions of WebSharper do not compile on OSX:

```sh
dotnet build
Microsoft (R) Build Engine version 17.0.0+c9eb9dd64 for .NET
Copyright (C) Microsoft Corporation. All rights reserved.

  Determining projects to restore...
  All projects are up-to-date for restore.
  chmod: /Users/XXX/.nuget/packages/websharper.fsharp/5.0.0.120/build//../tools/net5.0/osx-x64//wsfsc.sh: No such file or directory
/Users/XXX/.nuget/packages/websharper.fsharp/5.0.0.120/build/WebSharper.FSharp.targets(61,5): error MSB3073: The command "chmod u+x '/Users/XXX/.nuget/packages/websharper.fsharp/5.0.0.120/build//../tools/net5.0/osx-x64//wsfsc.sh'" exited with code 1. [/Users/XXX/Projects/mywebsharper/Exercise1/Exercise1.fsproj]

Build FAILED.
```

Problem seems to be that for version 5 of [WebSharper][websharper-home], the targets for `osx-x64` are missing:

```sh
% ls /Users/XXX/.nuget/packages/websharper.fsharp/5.0.0.120/build//../tools/net5.0
linux-musl-x64	linux-x64	win-x64
```

### Remidy

Change all the references to `5.0...` in the project file to `4.*`:

```xml
<ItemGroup>
  <PackageReference Include="WebSharper" Version="4.*" />    
  <PackageReference Include="WebSharper.FSharp" Version="4.*" />
  <PackageReference Include="WebSharper.UI" Version="4.*" />
  <PackageReference Include="WebSharper.AspNetCore" Version="4.*" />
</ItemGroup>
```

Another way is to set a variable for the version:

```xml
<PropertyGroup>
  <TargetFramework>net5.0</TargetFramework>
  <WSVersion>4.*</WSVersion>
</PropertyGroup>
...
  <PackageReference Include="WebSharper" Version="$(WSVersion)" />  
```



[WebSharper]:  {{site.websharper_link}}
