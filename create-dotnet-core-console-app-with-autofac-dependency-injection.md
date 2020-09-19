---
title: "Create a dotnet core Cosnole app template with autofac dependency injections"
date: 2020-09-11T13:45:06+06:00
image: images/blog/dotnet-core-autofac.png
feature_image: images/blog/dotnet-core-autofac-feature.jpg
author: Kevin Wang
summary: I created this dotnet core console app template with autofac dependency injection configure, this would save your precious time for some repetitive job everytime setting up a new project. Install via nuget, if you like the work please share with your mates.
---

Every time I create a new project, I have to go through exactly same steps to setup some open source libraries that I always use for any projects.

For example I often use autofac as my dependency inject container and register a bunch of services and preconfigure them the same way, such as Serilog, Automapper, etc.

There is nothing difficult to setup all these libraries, it is just time consuming and repetitive job that has to be done every single time.

So here I create this small template project that is pre-installed and pre-configured with all these libraries, to speed up your development process.

- Autofac
- Serilog
- AutoMapper
- Newtonsoft.Json

## Install and use the template

This is the link to the published [nuget]((https://www.nuget.org/packages/Superwalnut.NetCoreConsoleTemplate/)) package as a dotnet template,

[![nuget](https://www.mrkevin.wang/images/blog/nuget-logo.png)](https://www.nuget.org/packages/Superwalnut.NetCoreConsoleTemplate/)

You can install this template via nuget using `dotnet new -i`,

```shell
dotnet new --install Superwalnut.NetCoreConsoleTemplate
```

The template will be installed as `core-console-autofac`.

Then you can create your new project using this template,

```shell
dotnet new core-console-autofac -n MyFirstConsole
```

## What is included in your new project

```c#
    ---- Models
        |---- Foo.cs
    ---- Modules
        |---- ConsoleModules.cs
    ---- AutoMapper
        |---- MyAutoMapperProfile.cs
    ---- appsettings.json
    ---- Program.cs
    ---- Startup.cs
```

### Autofac

I created a Startup.cs file to create autofac `ContainerBuilder` and register all the services using `ConfigureServices()` method. To keep service registrations neat and clean, I created an autofac module, for the services only consumed by the console app. 

<script src="https://gist.github.com/superwalnut/c39f7bc8bf3beb805d0447e383c782d6.js"></script>

```c#
        private ContainerBuilder ConfigureServices(IServiceCollection serviceCollection)
        {
            CreateLogger(Configuration);

            serviceCollection.AddAutofac();
            serviceCollection.AddOptions();

            var builder = new ContainerBuilder();
            builder.Populate(serviceCollection);
            builder.RegisterLogger();

            builder.RegisterModule<ConsoleModule>();
            return builder;
        }
```

### appsettings.json

I need to use `ConfigurationBuilder()`, to read the physical app setting json file from its environment, and then `AddEnvironmentVariables()`, so we can override configurations via environment variables.

```c#
#if DEBUG
            var builder = new ConfigurationBuilder()
                .SetBasePath(AppDomain.CurrentDomain.BaseDirectory)
                .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
                .AddEnvironmentVariables();
#else
            var builder = new ConfigurationBuilder()
                .SetBasePath(AppDomain.CurrentDomain.BaseDirectory)
                .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
                .AddJsonFile($"appsettings.{envName}.json", optional: true)
                .AddEnvironmentVariables();
#endif
```

### Serilog

Registered the logger in `ConfigureService()`, 

```c#
        public static void CreateLogger(IConfigurationRoot configuration)
        {
            Log.Logger = new LoggerConfiguration()
                .MinimumLevel.Debug()
                .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
                .Enrich.FromLogContext()
                .ReadFrom.Configuration(configuration)
                .CreateLogger();
        }
```

and configured with Console output sinks.

```json
  "Serilog": {
    "Using": [ "Serilog.Sinks.Console" ],
    "MinimumLevel": "Information",
    "WriteTo": [
      {
        "Name": "Console"
      }
    ]
  }
```

### AutoMapper

Created a default automapper profile with an example `CreateMap()`.

```c#
    public class MyAutoMapperProfile : Profile
    {
        public MyAutoMapperProfile()
        {
            CreateMap<Foo, FooDto>();
            // Use CreateMap... Etc.. here (Profile methods are the same as configuration methods)
        }
    }
```

## Finally,
Congratulations on setup your .net core console app with this template.
If you find my article is helpful and saving you some time, please click like and share my post.

Here is a link to github to grab the full source code.

---

{{< github "https://github.com/superwalnut/dotnet-console-app-template" >}}
