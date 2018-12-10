---
layout: post
title:  "Dockerize Your Rails Application !"
date:   2018-07-15 14:07:23 +0530
categories: docker 
comments: true
---


This is the right time to invest time and resources in containerization technologies to make sure your projects are 
scalable and portable. Docker is the de-facto containerization platform we've at the moment.

## Prerequisites

* You should know the basics of Containerization and Docker, if not start __[here](https://www.docker.com/resources/what-container)__.
* Your application should be designed according to __[The twelve-factor app](https://12factor.net/)__ standards.
You can dockerize your app without this, but sooner you do this lesser evils you'll have to face.

## Writing Dockerfile

I've created a default blog application for demo with `rails new blogger`, get in to `blogger` app directory.
To create docker image for application, we need to write the `Dockerfile`.

Create a new file with name `Dockerfile`, name of Dockerfile is arbitery which means you can name it `staging.Dockerfile`
if you prefer, let's stick to `Dockerfile` for the demo.

Dockerfile
```
FROM ruby:2.3.7

WORKDIR usr/src/blogger

EXPOSE 3000

RUN apt-get update
RUN apt-get install -y nodejs

COPY Gemfile Gemfile.lock ./
RUN bundle install

COPY . .

ENTRYPOINT ["scripts/init.sh"]

CMD ["bundle", "exec", "rails", "s"]
```
scripts/init.sh
```
#! /bin/sh
bundle exec rake db:migrate
```

## Commands Explained

### FROM
```
FROM ruby:2.3.7
```
We are setting ruby base image for our blogger image, you can select a prebuilt linux distribution with ruby installed so
that you don't have to take care of installing ruby and it's neccessary dependencies. You can chose for a variety of Ruby
versions from their official Dockerhub __[profile](https://hub.docker.com/r/library/ruby/tags/)__.
Take away here is chose an official image whenever possible, so that you don't have to write/maintain base software. You
can opt for smaller sized compact images as well, any image that ends with `slim`, `alpine` are stripped off additional
libraries and kept with minimum viable libraries and apps to keep size in check. Opting for smaller base image reduce your
image size considerably which helps when you download/upload your images.

### WORKDIR
```
WORKDIR usr/src/blogger
```
With `WORKDIR` we are setting up our default directory for operations, which means any file that gets copied or any command
that runs without `absolute path` will be in effect from this directory. `usr/src` is arbitory, it's just that I prefer
putting my code there.

### EXPOSE
```
EXPOSE 3000
```
We use `EXPOSE` command to open up container port to outside world, we opened up `3000` port of container in above example
as it was the rails default port.

### RUN
```
RUN apt-get update
RUN apt-get install -y nodejs
```
`RUN` command is used to run shell commands. Most of the time we need libraries and supporting softwares to run
applications, we need `nodejs` to run Rails application, that's why we're installing nodejs in above example. `-y` flag
will prevent any prompts and install without intervention.

### COPY
```
COPY Gemfile Gemfile.lock ./
```
`COPY` command is used to copy files/directory from host to container image. Usage of `COPY` is just similar to `cp` command,
`COPY source_of_file_in_host destination_of_file_in_container`. If you use destination as relative path then path will start
from `WORKDIR` as it's the default directory in container.
To build docker image, run `docker build -t albertpaulp/blogger:v1 .`

To run the image we just builded,  `docker run -p 80:3000 albertpaulp/blogger:v1`

### ENTRYPOINT
```
ENTRYPOINT ["scripts/init.sh"]
```
Applications can have certain chores to do before starting up everytime, for example a typical backend application may need
to run migrations in database everytime before starting up. `ENTRYPOINT` is the place where you put these initialization
scripts, this command accepts a command or a file.

### CMD
```
CMD ["bundle", "exec", "rails", "s"]
```
`CMD` just like `RUN` except it only execute at runtime and doesn't create layer while building, usually the final command
is given to `CMD`, in our scenario it's starting the server.


## Optimization

### Leveraging Docker Cache
If you've noticed the order we arranged Dockerfile, we've copied Gemfile first and built it, then we are adding rest
of the code files. We do this to make use of Docker caching, so that if we keep building without changing Gemfile, docker will
make use of it's caching mechanism to skip installing gems everytime, which saves us a lot of time. Rule of thumb is always
arrange Dockerfile instructions in way that less frequent operations comes first, so that Docker can use cache for those,
Follow __[this](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)__ to learn how Docker caching works. 

### Customizing App Startup with script
You can make use of `ENTRYPOINT` to run specific commands before your application starts, it can be anything from running
migrations, creating databases and checking if database server is up etc.
