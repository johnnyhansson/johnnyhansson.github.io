---
layout: post
title: Running a ASP.NET Core app in a Docker container on Heroku 
categories: .NET Core, Docker, Heroku
---

Today, on the train back from work a question popped up in my head. I wanted to run a ASP.NET Core app and make it available to the rest of the Internet without setting up a server at home, and I wanted the hosting to be free. [Azure](https://azure.microsoft.com) was an alternative, but I wanted to try something different. I then remembered I had tried a service called [Heroku](http://www.heroku.com) a couple of years ago when I played with Ruby. Back then, .NET Core didn't exist and I doesn't seem to be supported out-of-the-box right now either. However, Heroku does support Docker so why not create a Docker images of your app and publish it to Heroku?

To get started you need a couple of things.

1. You need an account on [Heroku](http://www.heroku.com), which you can sign-up to for free.  
2. You need to install [Docker](https://www.docker.com) on your machine.
3. You need to install the [.NET Core SDK](https://www.microsoft.com/net/download/core).  

### Create the app on Heroku

When you have created your Heroku account it's time to create the app in Heroku. This can be made either from the Heroku Dashboard or via the CLI. In this quick tutorial I will show how you will do it from the CLI. Instructions for installing the Heroku CLI can be found [here](https://devcenter.heroku.com/articles/heroku-cli).

```
# Login to your Heroku account
heroku login

# Create a new app named aspnet-core-on-heroku
heroku create aspnet-core-on-heroku

# Open your new app in the browser
heroku open -a aspnet-core-on-heroku
```

When visiting your newly created app it will look like this.

<img src="/images/AspNetCore-On-Heroku/newapp.png" />

### Create the ASP.NET Core app

The ASP.NET Core app we will publish to Heroku will be based on the default MVC template.

```
# Create a new ASP.NET Core app based on the MVC template
dotnet new mvc -n aspnet-core-on-heroku

cd aspnet-core-on-heroku

# Build the app using the Release configuration and publish the artifacts to a folder named publish
dotnet publish -c Release -o publish
```

### Create a Dockerfile and build the Docker image

Create a new file in the project directory and name it Dockerfile. Below you will find the content of the `Dockerfile`. Heroku doesn't allow us to set the publish port for the container. Instead we will use the $PORT which value will be injected by Heroku when starting the container.

```
FROM microsoft/aspnetcore:2.0.0

WORKDIR /app
COPY /publish .

CMD ASPNETCORE_URLS=http://*:$PORT dotnet aspnet-core-on-heroku.dll
```

Also create another file, name it `.dockerignore` and paste in the content below. This will prevent unnecessary files to be transferred to the Docker deamon when building the image.

```
**
!publish/
```

Now, it's time to build the image. Because we will publish it to Heroku's container registry we need to tag the image correctly. The naming convention used by Heroku is `registry.heroku.com/<app-name>/<process-type>`.

```
docker build -t registry.heroku.com/aspnet-core-on-heroku/web .
```

### Publish the image to Heroku

The application is created, the Docker image has been built, so now it's time to publish the image to Heroku.

```
# Before pushing the image we need to login to the container registry
heroku container:login

# Push image to Heroku
docker push registry.heroku.com/aspnet-core-on-heroku/web

# Open the published app in browser
heroku open -a aspnet-core-on-heroku
```

Now your published app will look like this. Enjoy!

<img src="/images/AspNetCore-On-Heroku/newapp.png" />