---
layout: post
title: SaltStack + Docker
tags: [saltstack, docker]
---


Recently, we were continuously running into issue because we could not easily determine the versions of the applications that we had deployed to our non-production environments. The problem that we were facing was that we had many different servers, each running a few different php microservices (most running in Docker containers), and we had no way of knowing which version of each application was in each environment. This made it difficult to test and troubleshoot issues because project A v1.5.1 depended on project B v1.0.1 and project B depended on project C v2.2.1, and so on. Multiple that by 20 projects across 3 environments, and you quickly run into a management nightmare. Now, I know that right now many of you are already rattling off a thousand different management tools that can be used to deploy applications, keep track of versions, poop unicorns, monitor CPU usage, etc, but my company doesn't use that tool, they don't want to pay for it, and they don't want to invest anyone's time into figuring out how it works.

This is where I come in. Given one to two business days with constant interruptions, I wanted to build a web app that would display all of the versions, of all of the applications, across all environments.


### SaltStack + Docker inspect
At work, we use [SaltStack](https://saltstack.com/) to perform small tasks across multiple servers at once. It allows us to do things like check the rpm versions of packages across all servers, copy files, run scripts, etc. Currently, our applications are running in two different ways, some are served by an Apache server running on the host, and the rest of the applications are run in Docker containers. Using Salt, I was able to come up with two separate commands that captured all of the application version info and dumped it into a single file. For the Docker containers, this was accomplished by using [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/) to capture the fields I wanted and then format them in such a way that was easy to parse later.

Docker inspect supports formatting its output using [Golang templates](https://golang.org/pkg/text/template/). It is a little tricky to work with if you have never used it, but there is plenty of documentation on it and it is pretty powerful. After a little struggling, some tears, then golang hatred, then back to golang love, I finally ended up with this:



>docker inspect --format '{\{\.Name\}\},{\{\.Created}\},{\{\.Config.Image}\},{\{if and (\.Config) (\.Config.Env) (gt (len \.Config.Env) 2)}\}{\{index \.Config.Env 2}\}{\{end}\}' \$(docker ps -q)


Here is a snippet of what the output of that command is:

```
HOSTNAME42:
    /app1_apache_1,2017-07-05T20:56:01.923093619Z,registry.local/positions:v0.0.19-rc,GIT_COMMIT=3cfcaf88803a6ed455a69aee63c3a0bc59ff906b
    /app2_apache_1,2017-07-05T18:12:52.758540195Z,registry.local/sessions:v0.0.6-rc,GIT_COMMIT=08728dc27a1ca1a7be1777811fd118bbd3e17411
    /app2_redis_1,2017-07-05T18:12:49.308514157Z,registry.local/redis:3.2-alpine,REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-3.2.4.tar.gz
    /app3_apache_1,2017-06-30T16:19:59.417421069Z,registry.local/accounts:v0.0.15-rc,GIT_COMMIT=83f53df7348a41f28d142750bf817850ef475974
    /proxy,2017-06-19T20:56:44.896440122Z,registry.local/apache:0.0.22-php7-local,
    /consul_agent,2017-03-13T23:19:06.908757417Z,registry.local/consul:0.7.5,DOCKER_BASE_VERSION=0.0.4
HOSTNAME51:
    /proxy,2017-06-30T15:44:42.588490143Z,registry.local/apache:0.0.22-php7-local,
    /consul_agent,2017-03-13T23:19:06.908757417Z,registry.local/consul:0.7.5,DOCKER_BASE_VERSION=0.0.4
HOSTNAME45:
    /app4_apache_1,2017-07-05T17:19:37.923555027Z,registry.local/configurations:v0.0.15-rc,GIT_COMMIT=1a50020a133acce981cc0361375f57cb47f016fc
    /app4_redis_1,2017-07-05T17:19:34.68029824Z,registry.local/redis:3.2-alpine,REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-3.2.4.tar.gz
    /app1_apache_1,2017-06-30T22:22:58.181982066Z,registry.local/messages:v0.0.5-rc,GIT_COMMIT=289c14f1fc84f4ece3e008225ce56ed03357c4a5
    /proxy,2017-06-19T20:57:08.976574148Z,registry.local/apache:0.0.22-php7-local,
    /consul_agent,2017-03-13T23:19:06.908757417Z,registry.local/consul:0.7.5,DOCKER_BASE_VERSION=0.0.4
```

A few different things happened in order for me to get this output. First of all, Docker inspect formatted the fields I asked for, for every container running on the server. I structured it in such a way ( {{.Name}},{{.Created}},... ) so that the fields are comma delimited and then I used this nasty piece of templating magic: {{if and (.Config) (.Config.Env) (gt (len .Config.Env) 2)}}{{index .Config.Env 2}}{{end}} to get the git commit. I baked the git commit into the container at build time by setting it to an environment variable in the dockerfile using ARGS (maybe a future post will discuss how that works). The problem that I faced was that there is not an easy way to get specific environment variables and their values using Docker inspect. So that command is iterating over all of the containers environment variables and giving me the second one if it exists. (NOTE: you need to check if it exists because if you don't, your docker inspect command will die a horrible death and you won't get your precious version information at all, for any container.) This does pose a problem for containers with more than two environment variables, or with containers where the git commit hash is not second. I still need to come up with a better solution to this problem.

Next you will notice that the application versions are grouped by hostname. This is done by Salt. Although helpful, later on it will make the parsing of this file just a little bit harder.

Ok, now we have our version information in a single file, what are we going to do with it? In the next post, I will talk about how I created a nodejs application that accepts the file, saves it to the database, and servers it via RESTful endpoints.
