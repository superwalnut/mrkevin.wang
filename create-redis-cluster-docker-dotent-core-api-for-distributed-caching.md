---
title: "Create Redis cluster + Docker + .Net Core 3.1 API for distributed caching"
date: 2020-09-10T13:45:06+06:00
image: images/blog/redis-docker.png
feature_image: images/blog/redis-docker-feature.png
author: Kevin Wang
summary: I am creating a template project to help you speed up development with redis distributed caching. This post will explain this in detail how you started with a 1-master + 3-slaves cluster and consume it using ServiceStack.Redis client in a .net core 3.1 web API.
---

redis cluster .net core with docker project template

I am creating a template project to help you speed up development with redis distributed caching. This post will explain this in detail how you started with a 1-master + 3-slaves cluster and consume it using ServiceStack.Redis client in a .net core 3.1 web API.

## Part 1. Setup redis in docker

I am using [Bitnami](https://github.com/bitnami/bitnami-docker-redis)’s docker image, it has a lot of cool features that really help us speed up things, such as setting up the configurations through those environment variables.

First, create a docker-compose.yml file, I am exposing the port to 6379, so that I can access it from localhost.

Also I set to REDIS_REPLICATION_MODE to master, and ALLOW_EMPTY_PASSWORD to allow no password, if you would like to be password protected, do set REDIS_PASSWORD.

<iframe src="https://medium.com/media/9c0d924a84c65f0dcc27caca165ec088" frameborder=0></iframe>

If you may notice, I have attached the data/ folder for the redis data persistence, and setup a network appnet that I am going to put the whole cluster into the same network.

Add redis slaves, and expose ports 6380–6382, because we are setting deploy to 3, when we run this in docker swarm mode, it will need 3 ports to satisfy that. I did the same thing for ALLOW_EMPTY_PASSWORD, also set its master host address, so it will find its way to the cluster.

<iframe src="https://medium.com/media/8bc0f308f1205a1a82284f3c242209fa" frameborder=0></iframe>

Note that redis-replica container is dependent on its master container, so only the redis-master is built and running successfully, it will process replicas.

## Part 2. Create .net core app & configure it in docker

Create a .net core 3.1 api app, install ServiceStack.Redis,

    Install-Package ServiceStack.Redis -Version 5.9.2

I am adding an entry in the appsettings.json that we will need to set redis connections,

    "Redis": [ "localhost:6379", "localhost:6380", "localhost:6381", "localhost:6382" ],

Let’s register the ServiceStack.Redis client manager in our startup.cs, and use the Redis config as string array to pass a list of redis nodes. And let the RedisManagerPool to manage which node the request it hitting.

<iframe src="https://medium.com/media/5e92dce3e16c2c055fa81338a5c61960" frameborder=0></iframe>

I created a ICacheItem interface, you can use that govern all the objects you want to work with cache, make it must implement the interface properties or methods that is required for your caching. I only set an Id for demonstration purpose.

<iframe src="https://medium.com/media/eaea43b375c82da2fe88b23d4f66e1d3" frameborder=0></iframe>

That I am going to need this to create my CacheClient implementation and its interface,

<iframe src="https://medium.com/media/513f963a2c653fbe828f4badf998722c" frameborder=0></iframe>

With the simplest methods that everyone is going to need: set, get and delete,

<iframe src="https://medium.com/media/03b63fe0c51635d3bf7ad0b85fc9363d" frameborder=0></iframe>

Finally, create a RedisController to work with your redis cache cluster,

<iframe src="https://medium.com/media/b6f53008ac03ef3a508764e712137d06" frameborder=0></iframe>

Now I have everything I needed for this api project, let’s create a DockerFile, so that we can build our api into container and runs in docker.

<iframe src="https://medium.com/media/a354154d2b152129904f16bc00311b0a" frameborder=0></iframe>

Add that to our docker-compose, you need to specify the DockerFile path, so the docker compose will find it in the right place.

<iframe src="https://medium.com/media/1c54a0c55fbce81245c05491bd0d2176" frameborder=0></iframe>

We want to override the environment variables from appsettings.json for those locahost:xxxx, because inside docker’s containers, they won’t be able to access localhost, we need to access by their container name, which docker has preconfigured as their dns names.

In my example, I am setting up 1 master + 3 slaves, so set them to Redis__0, Redis__1, Redis__2, Redis__3 to override the string array for Redis section.

When I am running this, I had a road block which is because .net core app force https connections, that my container doesn’t have any proper ssl to be successfully served via secured channel. I need to create a dev-cert to my system and let kestrel to use for https connections.

Run this command to generate a dev-certs, I am using Mac, for windows or other OS, please take a look [here](https://docs.microsoft.com/en-us/aspnet/core/security/docker-https?view=aspnetcore-3.1).

    dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p password

If you already had one and don’t remember the password and you don’t bother to find out the password, this is how you clean it out and do it over again.

    dotnet dev-certs https --clean

Set these env variables that your web server Kestrel will know what to do,

<iframe src="https://medium.com/media/1ce2511e323b97b5788730dd553c3564" frameborder=0></iframe>

## Part 3. Run

Once you have done above steps, happy days, let’s run your app.

    docker-compose --compatibility up --build

with compatibility mode, it will apply preset `deploy` to run docker swarm mode that will spin up 1 master + 3 slaves + 1 api.

Try access your api at localhost:8089 and test it using postman, I have shared my postman collection with all the source code in github, click the link at the bottom to grab all.

## Part 4. Project template

Also I have built a nuget package for a project template, you can use dotnet new command to install the template from nuget.org.
[**Superwalnut.RedisClusterTemplate 1.0.8**
*This is a project template for Redis Cluster + Docker + .Net Core 3.1 API, using `dotnet new -i…*www.nuget.org](https://www.nuget.org/packages/Superwalnut.RedisClusterTemplate/)

    dotnet new -i Superwalnut.RedisClusterTemplate

After you installed it, run dotnet new -l to browse all your templates, you should find ‘**Redis .Net Core Template**’ with short name ‘**redis-dotnet-core**’,

![dotnet new -l templates](https://cdn-images-1.medium.com/max/2148/1*va7t8PciU-JrCeF7N3u_TQ.png)*dotnet new -l templates*

Run this command to create a new application that contains redis+api+docker and instantly ready to run.

    dotnet new redis-dotnet-core -n <project-name> -o <project-folder>

This is an example what is included in the generated project from the template.

![](https://cdn-images-1.medium.com/max/2000/1*g4joSBRnqnEteWrXo-UkiA.png)

## Finally,

Congratulations on setup your Redis & .net core app for distributed caching locally and enjoy developing.

If you find my article is helpful, please click **Applaud** to support me.

![](https://cdn-images-1.medium.com/max/2000/0*QfqLWr4dcaupb0mK.png)

Here is the link to GitHub to grab the complete source code for the app.
[**superwalnut/redis-cluster-dotnet-core-template**
*This is a project template that saves your time create redis cluster in docker with a dotnet core app to consume from…*github.com](https://github.com/superwalnut/redis-cluster-dotnet-core-template)