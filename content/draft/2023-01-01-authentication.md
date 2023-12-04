---
categories:
- programming
date: "2023-01-01T00:00:00Z"
draft: true
tags:
- Microsoft login
- Apple-Id
- Bolero
- F#
title: Authentication
showToc: true
---


Collect information about authentication.

### AspNet.Security.OAuth.Providers

Good samples of aspnet MVC with google, twitter, Apple-ID:

<https://github.com/aspnet-contrib/AspNet.Security.OAuth.Providers>

> **AspNet.Security.OAuth.Providers** is a **collection of security middleware** that you can use in your **ASP.NET Core**application to support social authentication providers like **[GitHub](https://github.com/)**, **[Foursquare](https://foursquare.com/)** or **[Dropbox](https://www.dropbox.com/)**. It is directly inspired by **[Jerrie Pelser](https://github.com/jerriep)**'s initiative, **[Owin.Security.Providers](https://github.com/RockstarLabs/OwinOAuthProviders)**.


### Azure AD with bolero

> This is an example of how to authenticate with an `Azure AD` using `MSAL.js`.

<https://github.com/anderslm/fsharp-bolero-azure-ad>

### Overview ASP.NET Core authentication

<https://learn.microsoft.com/en-us/aspnet/core/security/authentication/?view=aspnetcore-7.0>

**Simple authorization**

<https://learn.microsoft.com/en-us/aspnet/core/security/authorization/simple?source=recommendations&view=aspnetcore-7.0>

### Authentication with net7.0

<https://learn.microsoft.com/en-us/aspnet/core/security/authentication/social/?view=aspnetcore-7.0&tabs=visual-studio-code>

Local code: <file:///Users/mmgreiner/Projects/Scrap/TestAuthorization/TestAuthorization>

### External Authentication Services

<https://learn.microsoft.com/en-us/aspnet/web-api/overview/security/external-authentication-services?source=recommendations>

Covers: 
* [Enabling Facebook Authentication](https://learn.microsoft.com/en-us/aspnet/web-api/overview/security/external-authentication-services?source=recommendations#FACEBOOK)
    
* [Enabling Google Authentication](https://learn.microsoft.com/en-us/aspnet/web-api/overview/security/external-authentication-services?source=recommendations#GOOGLE)
    
* [Enabling Microsoft Authentication](https://learn.microsoft.com/en-us/aspnet/web-api/overview/security/external-authentication-services?source=recommendations#MICROSOFT)
    
* [Enabling Twitter Authentication](https://learn.microsoft.com/en-us/aspnet/web-api/overview/security/external-authentication-services?source=recommendations#TWITTER)

### Bolero Authentication

See <https://fsbolero.io/docs/Remoting#authentication-and-authorization>


### Azure

https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIdConnect-DotNet

> This sample demonstrates a ASP.NET web app application that authenticates users against Azure AD.

locally on <file:///Users/mmgreiner/Projects/Scrap/sign-in/AppModelv2-WebApp-OpenIDConnect-DotNet-master>

### Using Scaffolding

See [Scaffold identity into razor without existing authorization](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/scaffold-identity?view=aspnetcore-7.0&tabs=netcore-cli#scaffold-identity-into-a-razor-project-without-existing-authorization)

Create bolero app:

    dotnet new bolero-app -o BoleroIdentityApp
    cd BoleroIdentityApp/src/*Server

Start installing the tools

    dotnet   add package Microsoft.VisualStudio.Web.CodeGeneration.Design
    
Run the identity scaffolder

    dotnet tool install -g dotnet-aspnet-codegenerator
        
Install the packages:

~~~ 
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Identity.UI
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
~~~

Project file should now look like this:

~~~xml
 <ItemGroup>
    <PackageReference Include="Bolero.Server" Version="0.*" />
    <PackageReference Include="Bolero.HotReload.Server" Version="0.*" />
    <PackageReference Include="Microsoft.AspNetCore.Components.WebAssembly.Server" Version="7.0.*" />
    <PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="7.0.4" />
    <PackageReference Include="Microsoft.AspNetCore.Identity.UI" Version="7.0.4" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="7.0.4">
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
      <PrivateAssets>all</PrivateAssets>
    </PackageReference>
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="7.0.4" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="7.0.4">
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
      <PrivateAssets>all</PrivateAssets>
    </PackageReference>
    <PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="7.0.5" />
  </ItemGroup>  
~~~

    
List identity options:

~~~
dotnet aspnet-codegenerator identity -h
Scaffolding failed.
Could not find project file in /Users/mmgreiner/Projects/Tutorials/BoleroIdentityApp/src/BoleroIdentityApp.Server

Usage: aspnet-codegenerator [arguments] [options]

Arguments:
  generator  Name of the generator. Check available generators below.

Options:
  -p|--project             Path to .csproj file in the project.
  -n|--nuget-package-dir   
  -c|--configuration       Configuration for the project (Possible values: Debug/ Release)
  -tfm|--target-framework  Target Framework to use. (Short folder name of the tfm. eg. net46)
  -b|--build-base-path     
  --no-build               

To see more information, enable tracing by setting environment variable 'codegen_trace' = 1.
RunTime 00:00:00.22
~~~

This happens because the scaffolding tool looks for a C# project with `.csproj` file.
Try the same thing with a cs project.

    dotnet new web -o IdentityApp
    cd IdentityApp
    dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
    dotnet add package Microsoft.EntityFrameworkCore.Design
    dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
    dotnet add package Microsoft.AspNetCore.Identity.UI
    dotnet add package Microsoft.EntityFrameworkCore.SqlServer
    dotnet add package Microsoft.EntityFrameworkCore.Tools
    ls                                                                 
    IdentityApp.csproj		Properties			appsettings.json		obj
    Program.cs			appsettings.Development.json	bin

## Bolero and Authentication

The new net7.0 allows the following creatings:

| Web Type    | Description                         | Create              | Authentication    | Language | Comment          |
|-------------|-------------------------------------|---------------------|-------------------|----------|------------------|
| Razor Pages | Server based dynamic pages          | `dotnet new webapp` | `-au Individual`  | C#       | [Razor Tutorial] |
| MVC         | Model View Control with Controllers | `dotnet new mvc`    | `-au Individual`  | C#       | [MVC Tutorial]   |
| Blazor      | Dynamic SPA pages                   |                     |                   |          |                  |

So the idea is, to create a Bolero App and create a Blazor App, and then copy the Blazor App pages that handle login to the Bolero App.

## Authentication in giraffe

This is a good explanation of how to use EF Core also with [F#]: 
<https://carpenoctem.dev/blog/fsharp-giraffe-auth-identity-ef-core/>


[Razor Tutorial]: https://learn.microsoft.com/en-us/aspnet/core/tutorials/razor-pages/razor-pages-start?view=aspnetcore-7.0&tabs=visual-studio-code
[MVC Tutorial]: https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/start-mvc?view=aspnetcore-7.0&tabs=visual-studio

[F#]: {{ site.fsharp_link }}

[Blazor Tutorial]: https://dotnet.microsoft.com/en-us/learn/aspnet/blazor-tutorial/intro

