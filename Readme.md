Running a dotnet app in Kubernetes

h1. Running dotnet on your mac
First, you'll need to install dotnet core on your mac. You can easily do this by following the instructions here: https://www.microsoft.com/net/learn/get-started-with-dotnet-tutorial

Next, create a simple new web application. I chose a react application because react is the future (or at least that's what Jeff says)! `dotnet new react`

```
 ~/src/github.com/karthequian/dot-net-example$ dotnet new react
The template "ASP.NET Core with React.js" was created successfully.

Processing post-creation actions...
Running 'dotnet restore' on /Users/karthik/dev/src/github.com/karthequian/dot-net-example/dot-net-example.csproj...
  Restoring packages for /Users/karthik/dev/src/github.com/karthequian/dot-net-example/dot-net-example.csproj...
  Generating MSBuild file /Users/karthik/dev/src/github.com/karthequian/dot-net-example/obj/dot-net-example.csproj.nuget.g.props.
  Generating MSBuild file /Users/karthik/dev/src/github.com/karthequian/dot-net-example/obj/dot-net-example.csproj.nuget.g.targets.
  Restore completed in 12.9 sec for /Users/karthik/dev/src/github.com/karthequian/dot-net-example/dot-net-example.csproj.

Restore succeeded.
```

h1. Run your application

Type `dotnet run` to run it locally on your mac.
