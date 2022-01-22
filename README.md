# Cleaning Clean Architecture

The purpose of this repository is to investigate "Clean Architecture" and see if it can be improved upon. The goal is to remove complexity without reducing functionality.

## Round 0 - Base State Validation

To being with we need to see if the application can be compiled and run. Likewise, we need to verify that all of the automated tests run.

The first hurdle is Docker. The application won't compile without it. So Docker Desktop needs to be installed, as well as the Linux Subsystem for Windows.

Next is Node. The application doesn't run under the current version of node (17.4.0). But according to the error messages, we can roll back to the 14.x series. At the time of this writing, 14.18.3 is the latest version that works.

The .NET Core template is broken, so you'll need to manually start the Angular frontend using `npm start` from the ClientApp folder.

All tests are passing.



## Round 1 - Removing Docker

Simply building a .NET Core/Angular application shouldn't require a commercial product such as Docker. And since we're looking at the architecture, not the deployment strategy, we can remove it.

To clear a warning message, we'll also update the TypeScript SDK to use Microsoft.TypeScript.MSBuild.

## Round 2 - Authorization Behavior

Looking at the `AuthorizationBehaviour` class, we noticed that it was never being hit. All of the requests that required authorization, for example `TodoListsController.Get`, were being aborted before we got this far.

As it turns out, ASP.NET Core has a built-in system for handling authorization. And this project is using it. So the `AuthorizationBehaviour` class is completely redundant and can be removed.

Reviewing the commands and handlers, we do see one that uses the `Authorization` attribute directly. This is the `PurgeTodoListsCommand`. In theory we would move the attribute to the controller method. But since this command is never used, we can just delete it. Which means we can also delete the matching test cases.

## Round 3 - Unhandled Exception Behavior

To test the `UnhandledExceptionBehaviour`, we start by adding a division by zero to the `GetTodosQueryHandler`. Then we set a break point in both `UnhandledExceptionBehaviour` and `ApiExceptionFilterAttribute` to see which is actually being used. 
 
And the answer is... both of them. Which means we can combine them. When choosing which of the two to keep, there are a couple considerations.

1. Which has more information available? The `ApiExceptionFilterAttribute` has access to the entire `HttpContext`, while `UnhandledExceptionBehaviour` only has the `ExportTodosQuery` object and the exception itself.
2. Which is broader? The `ApiExceptionFilterAttribute` covers all API requests, `UnhandledExceptionBehaviour` only handles API requests that go through MediatR.
3. Which is eariler in the request pipeline? Not only does `UnhandledExceptionBehaviour` only handles requests that go through MediatR, it can't see errors that occur before or after MediatR does its thing. For example, serialization errors won't be caught.

So clearly we should keep `ApiExceptionFilterAttribute` and move the logging `UnhandledExceptionBehaviour` does into it. (Specifically, what to log is left as an exercise for the reader.)




## Round 4 - Performance Logging

Just like exception logging should be handled in the ASP.NET Core pipeline, so should performance logging.

For this we'll use ASP.NET Core middleware. For simple use cases, the `app.Use` method can be employed to create ad-hoc middleware from a function. But it is cleaner to create a separate class.

## Round 5 - Validation

The next behavior we'll be looking at is validation. This is a strange one. In the startup logic for ASP.NET Core, FluentValidation was added and then disabled. Then a MediatR behavior called `ValidationBehaviour` was created to do the same thing.

That's just plain silly. So `ValidationBehaviour` is going to be deleted and FluentValidation is going to be turned back on.

## Round 6 - Logging

Like performance logging, request logging should be pulled up into the ASP.NET Core pipeline so that all requests are logged.

There are many options for this, but to keep it simple we'll just enabled the built-in request logging via `app.UseHttpLogging();`. 

 <img align="left" width="116" height="116" src="https://raw.githubusercontent.com/jasontaylordev/CleanArchitecture/main/.github/icon.png" />
 
 # Clean Architecture Solution Template
![.NET Core](https://github.com/jasontaylordev/CleanArchitecture/workflows/.NET%20Core/badge.svg) 
[![Clean.Architecture.Solution.Template NuGet Package](https://img.shields.io/badge/nuget-6.0.1-blue)](https://www.nuget.org/packages/Clean.Architecture.Solution.Template) 
[![NuGet](https://img.shields.io/nuget/dt/Clean.Architecture.Solution.Template.svg)](https://www.nuget.org/packages/Clean.Architecture.Solution.Template)
[![Discord](https://img.shields.io/discord/893301913662148658?label=Discord&logo=discord&logoColor=white)](https://discord.gg/p9YtBjfgGe)
[![Twitter Follow](https://img.shields.io/twitter/follow/jasontaylordev.svg?style=social&label=Follow)](https://twitter.com/jasontaylordev)


<br/>

This is a solution template for creating a Single Page App (SPA) with Angular and ASP.NET Core following the principles of Clean Architecture. Create a new project based on this template by clicking the above **Use this template** button or by installing and running the associated NuGet package (see Getting Started for full details). 

## Learn about Clean Architecture

[![Clean Architecture with ASP.NET Core 3.0 • Jason Taylor • GOTO 2019](https://img.youtube.com/vi/dK4Yb6-LxAk/0.jpg)](https://www.youtube.com/watch?v=dK4Yb6-LxAk)

## Technologies

* [ASP.NET Core 6](https://docs.microsoft.com/en-us/aspnet/core/introduction-to-aspnet-core?view=aspnetcore-6.0)
* [Entity Framework Core 6](https://docs.microsoft.com/en-us/ef/core/)
* [Angular 12](https://angular.io/)
* [MediatR](https://github.com/jbogard/MediatR)
* [AutoMapper](https://automapper.org/)
* [FluentValidation](https://fluentvalidation.net/)
* [NUnit](https://nunit.org/), [FluentAssertions](https://fluentassertions.com/), [Moq](https://github.com/moq) & [Respawn](https://github.com/jbogard/Respawn)
* [Docker](https://www.docker.com/)

## Getting Started

The easiest way to get started is to install the [NuGet package](https://www.nuget.org/packages/Clean.Architecture.Solution.Template) and run `dotnet new ca-sln`:

1. Install the latest [.NET 6 SDK](https://dotnet.microsoft.com/download/dotnet/6.0)
2. Install the latest [Node.js LTS](https://nodejs.org/en/)
3. Run `dotnet new --install Clean.Architecture.Solution.Template` to install the project template
4. Create a folder for your solution and cd into it (the template will use it as project name)
5. Run `dotnet new ca-sln` to create a new project
6. Navigate to `src/WebUI/ClientApp` and run `npm install`
7. Navigate to `src/WebUI/ClientApp` and run `npm start` to launch the front end (Angular)
8. Navigate to `src/WebUI` and run `dotnet run` to launch the back end (ASP.NET Core Web API)

Check out my [blog post](https://jasontaylor.dev/clean-architecture-getting-started/) for more information.

### Docker Configuration

In order to get Docker working, you will need to add a temporary SSL cert and mount a volume to hold that cert.
You can find [Microsoft Docs](https://docs.microsoft.com/en-us/aspnet/core/security/docker-https?view=aspnetcore-6.0) that describe the steps required for Windows, macOS, and Linux.

For Windows:
The following will need to be executed from your terminal to create a cert
`dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p Your_password123`
`dotnet dev-certs https --trust`

NOTE: When using PowerShell, replace %USERPROFILE% with $env:USERPROFILE.

FOR macOS:
`dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p Your_password123`
`dotnet dev-certs https --trust`

FOR Linux:
`dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p Your_password123`

In order to build and run the docker containers, execute `docker-compose -f 'docker-compose.yml' up --build` from the root of the solution where you find the docker-compose.yml file.  You can also use "Docker Compose" from Visual Studio for Debugging purposes.
Then open http://localhost:5000 on your browser.

To disable Docker in Visual Studio, right-click on the **docker-compose** file in the **Solution Explorer** and select **Unload Project**.

### Database Configuration

The template is configured to use an in-memory database by default. This ensures that all users will be able to run the solution without needing to set up additional infrastructure (e.g. SQL Server).

If you would like to use SQL Server, you will need to update **WebUI/appsettings.json** as follows:

```json
  "UseInMemoryDatabase": false,
```

Verify that the **DefaultConnection** connection string within **appsettings.json** points to a valid SQL Server instance. 

When you run the application the database will be automatically created (if necessary) and the latest migrations will be applied.

### Database Migrations

To use `dotnet-ef` for your migrations please add the following flags to your command (values assume you are executing from repository root)

* `--project src/Infrastructure` (optional if in this folder)
* `--startup-project src/WebUI`
* `--output-dir Persistence/Migrations`

For example, to add a new migration from the root folder:

 `dotnet ef migrations add "SampleMigration" --project src\Infrastructure --startup-project src\WebUI --output-dir Persistence\Migrations`

## Overview

### Domain

This will contain all entities, enums, exceptions, interfaces, types and logic specific to the domain layer.

### Application

This layer contains all application logic. It is dependent on the domain layer, but has no dependencies on any other layer or project. This layer defines interfaces that are implemented by outside layers. For example, if the application need to access a notification service, a new interface would be added to application and an implementation would be created within infrastructure.

### Infrastructure

This layer contains classes for accessing external resources such as file systems, web services, smtp, and so on. These classes should be based on interfaces defined within the application layer.

### WebUI

This layer is a single page application based on Angular 10 and ASP.NET Core 5. This layer depends on both the Application and Infrastructure layers, however, the dependency on Infrastructure is only to support dependency injection. Therefore only *Startup.cs* should reference Infrastructure.

## Support

If you are having problems, please let us know by [raising a new issue](https://github.com/jasontaylordev/CleanArchitecture/issues/new/choose).

## License

This project is licensed with the [MIT license](LICENSE).
