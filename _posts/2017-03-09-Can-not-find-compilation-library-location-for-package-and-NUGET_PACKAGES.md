---
layout: post
title: 'Can not find compilation library location for package' and NUGET_PACKAGES
categories: .NET Core
---

I recently started my journey to learn .NET Core and ASP.NET Core and yesterday I hit a roadblock.

In most cases when you create a new .NET Core or ASP.NET Core application and installing NuGet packages,
these will end up in the global NuGet package cache located at `%userprofile%\.nuget\packages`.
However, I wanted that my NuGet packages should end up in another location than the default global package folder.
So, what I did was creating a NuGet.config file in the solution root of my ASP.NET Core project and added the following content.

```xml
<configuration>
    <config>
        <add key="globalPackagesFolder" value=".nuget\packages" />
    </config>
</configuration>
```

This configuration will override the location of the global package folder, so when doing a `dotnet restore` the packages
will end up in the folder `.nuget\packages` in the solution folder. It worked like a charm and the packages ended up in
the location I wanted. The problem appeared when I typed `dotnet run` and then browsed to http://localhost:5000. 
The page didn't show up as expected and when I looked at the console the exception message told me:

`Can not find compilation library location for package dapper`

However, when dropping the dapper folder into the default NuGet package cache folder the exception disappeared.
So, for some reason it didn't look at the package cache folder in my solution.

### NUGET_PACKAGES to the rescue

A while later I stumbled across [this](http://stackoverflow.com/a/40505874) Stackoverflow post that revealed the existence of
the environment variable NUGET_PACKAGES.

What I ended up with in this case was setting the NUGET_PACKAGES variable so it pointed to my local NuGet package folder.
So, in the command prompt I types the command below when my current working directory was the solution folder.

`set NUGET_PACKAGES=%cd%\.nuget\packages`

Next time I started the ASP.NET Core application and navigated to http://localhost:5000 it worked as expected.
