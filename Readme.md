# Running a dotnet app in Kubernetes

## Run dotnet on your mac

First, you'll need to install dotnet core on your mac. You can easily do this by following the instructions here: https://www.microsoft.com/net/learn/get-started-with-dotnet-tutorial

Next, create a simple new web application. I chose a webapi application, because I wanted to create an example API running in Kubernetes. I passed a flag to not have any auth turned on by default, and no https turned on by defaults: `dotnet new webapi --auth None --no-https`. 

The output of that is something like what's shown below..

```
  ~/dev/src/github.com/karthequian/dotnet-example$ dotnet new webapi --auth None --no-https
The template "ASP.NET Core Web API" was created successfully.

Processing post-creation actions...
Running 'dotnet restore' on /Users/karthik/dev/src/github.com/karthequian/dotnet-example/dotnet-example.csproj...
  Restoring packages for /Users/karthik/dev/src/github.com/karthequian/dotnet-example/dotnet-example.csproj...
  Generating MSBuild file /Users/karthik/dev/src/github.com/karthequian/dotnet-example/obj/dotnet-example.csproj.nuget.g.props.
  Generating MSBuild file /Users/karthik/dev/src/github.com/karthequian/dotnet-example/obj/dotnet-example.csproj.nuget.g.targets.
  Restore completed in 1.42 sec for /Users/karthik/dev/src/github.com/karthequian/dotnet-example/dotnet-example.csproj.

Restore succeeded.
```

This will effectively create the shell for your project (csproj), and everything you'll see in the github project..

## Run your application locally

Type `dotnet run` to run it locally on your mac. The initial build took a little long, but it was up and running pretty quick. The output of the command is shown below:

```
 ~/dev/src/github.com/karthequian/dotnet-example$ dotnet run
Using launch settings from /Users/karthik/dev/src/github.com/karthequian/dotnet-example/Properties/launchSettings.json...
: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[0]
      User profile is available. Using '/Users/karthik/.aspnet/DataProtection-Keys' as key repository; keys will not be encrypted at rest.
Hosting environment: Development
Content root path: /Users/karthik/dev/src/github.com/karthequian/dotnet-example
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
```

To test the application, you can run a simple curl against it, or open up the URL in the website. As there are no default routing paths for /, and there is only a controller defined for `/values`, we'll need to make sure to curl against a `/values` endpoint as shown below, and we should see what we expect.

```
 ~/dev/src/github.com/karthequian/dotnet-example$ curl localhost:5000/api/values/
["value1","value2"]
```

## Dockerize the application

I've included a sample Dockerfile with this, that'll work well for the application. I followed the example to build this from the Docker website: https://docs.docker.com/engine/examples/dotnetcore/#create-a-dockerfile-for-an-aspnet-core-application. I did make a change to the `Entrypoint`, and updated the dll name to match the project name, which, in this case is `dotnet-example.dll`

To build, I ran the following docker build command, and tagged it with my dockerhub name.

```
~/dev/src/github.com/karthequian/dotnet-example$ docker build -t karthequian/dotnetexample .
Sending build context to Docker daemon   1.37MB
Step 1/10 : FROM microsoft/dotnet:sdk AS build-env
 ---> 9e243db15f91
Step 2/10 : WORKDIR /app
 ---> Using cache
 ---> f9654a9137e0
Step 3/10 : COPY *.csproj ./
 ---> Using cache
 ---> 3269eb744d01
Step 4/10 : RUN dotnet restore
 ---> Using cache
 ---> a7f034c5b6bb
Step 5/10 : COPY . ./
 ---> be9452f1d481
Step 6/10 : RUN dotnet publish -c Release -o out
 ---> Running in d3fbd21bf58c
Microsoft (R) Build Engine version 15.7.179.6572 for .NET Core
Copyright (C) Microsoft Corporation. All rights reserved.

  Restoring packages for /app/dotnet-example.csproj...
  Generating MSBuild file /app/obj/dotnet-example.csproj.nuget.g.props.
  Generating MSBuild file /app/obj/dotnet-example.csproj.nuget.g.targets.
  Restore completed in 621.9 ms for /app/dotnet-example.csproj.
  dotnet-example -> /app/bin/Release/netcoreapp2.1/dotnet-example.dll
  dotnet-example -> /app/out/
 ---> f5b878778afd
Removing intermediate container d3fbd21bf58c
Step 7/10 : FROM microsoft/dotnet:aspnetcore-runtime
 ---> fcc3887985bb
Step 8/10 : WORKDIR /app
 ---> Using cache
 ---> 6d69aad275ab
Step 9/10 : COPY --from=build-env /app/out .
 ---> Using cache
 ---> 2788fd5829f9
Step 10/10 : ENTRYPOINT dotnet dotnet-example.dll
 ---> Running in 12bfee8e072d
 ---> 15a64794811d
Removing intermediate container 12bfee8e072d
Successfully built 15a64794811d
Successfully tagged karthequian/dotnetexample:latest
```

Next, to verify this works, I can run the container with the command `docker run -d -p 5001:80 karthequian/dotnetexample`, and port maps port 5000 on my machine to port 80 on the container. And again, I can test this with a curl command as shown below, where localhost:5000 is pointed to my container by Docker.

```
~/dev/src/github.com/karthequian/dotnet-example$ curl localhost:5001/api/values
["value1","value2"]
```

## Run in Kubernetes

I've created a sample Deployment.yaml file with this project that you can use. Once you've got a kubernetes cluster running, such as a cluster in Oracle Kubernetes Engine, you can run the command below to deploy the container to Kubernetes:
```kubectl apply -f https://raw.githubusercontent.com/karthequian/dotnet-example/master/Deployment.yaml``` 

This will make a deployment and a service, and will expose the service as a nodeport on port `32080`.
```
 ~/dev/src/github.com/karthequian/dotnet-example$ kubectl apply -f https://raw.githubusercontent.com/karthequian/dotnet-example/master/Deployment.yaml
deployment "dotnetworld" created
service "dotnetworld" created
 ~/dev/src/github.com/karthequian/dotnet-example$ kubectl get deployments
NAME                                      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
dotnetworld                               1         1         1            1           20s
 ~/dev/src/github.com/karthequian/dotnet-example$ kubectl get services
NAME                                      CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
dotnetworld                               10.96.79.93     <nodes>       80:32080/TCP                 26s
kubernetes                                10.96.0.1       <none>        443/TCP                      30d
```

To test, you can get the IP of any of the nodes on your service, and then run a curl against it like the examples below. 

```
 ~/dev/src/github.com/karthequian/dotnet-example$ kubectl get services
NAME                                      CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
dotnetworld                               10.96.79.93     <nodes>       80:32080/TCP                 26s
kubernetes                                10.96.0.1       <none>        443/TCP                      30d
 ~/dev/src/github.com/karthequian/dotnet-example$ kubectl get nodes
NAME              STATUS    AGE       VERSION
129.146.123.174   Ready     30d       v1.9.7
129.146.133.234   Ready     30d       v1.9.7
129.146.162.102   Ready     30d       v1.9.7
 ~/dev/src/github.com/karthequian/dotnet-example$ curl 129.146.162.102:32080/api/values
["value1","value2"]
```

## Fin!

And there you have it, a dotnet core app running in Kubernetes! If you run into issues, feel free to create a github issue against me, or reach me on twitter via @iteration1
