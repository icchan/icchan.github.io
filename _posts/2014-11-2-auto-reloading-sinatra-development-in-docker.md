---
title: Auto-Reloading Sinatra Development in Docker
layout: post
---
So I've been investigating [Docker](https://www.docker.com/) recently, and its pretty great.

One cool feature which has been particularly useful for me is [mounting a directory on the host machine inside the docker container](https://docs.docker.com/userguide/dockervolumes/#mount-a-host-directory-as-a-data-volume).
It is great for a dev environment to auto-reload new changes without building a new container or restarting a container.
If you are running an app which auto-reloads you can run a file in a mounted folder, and edit the files locally (on the docker host) and see them reflected in the running containter. Very handy for scripting languages (like Ruby or PHP) as well as cross platform languages (like Java). 

I'll go through how I have been doing it for a [Sinatra](http://www.sinatrarb.com/) app running with [Shotgun](https://github.com/rtomayko/shotgun) (auto-reloader for rackup).

First you need an image with both Sinatra and Shotgun. 
I pushed one [here](https://registry.hub.docker.com/u/icchan/shotgun-sinatra/), but if you prefer to make your own it was just this:

```bash
docker run training/sinatra run gem install shotgun 
```

Now to run it so it auto-reloads from your local folder run a command like this:

```bash
docker run -d -p {port-on-host}:9393 -v {absolute-local-path}:/app icchan/shotgun-sinatra shotgun --host 0.0.0.0 /app/{your-app}.rb 
```

So heres a breakdown:

* `-d` means run as daemon
* `-p {host-port-to-expose}:9393` exposes and publishes port 9393 (shotgun default port) to your host 
* `-v {absolute-local-path}:/app` mounts the local folder to /app in the container
* `icchan/shotgun-sinatra` is the name of the shotgun image
* `shotgun --host 0.0.0.0 /app/{your-app}.rb` is the command to run on the image, the 0.0.0.0 is to make accessible outside the container (default is 127.0.0.1)

So if your app is in `/home/ruby/app/hello.rb` on your host and you want to run it on 8080 you would run this command:

```bash
docker run -d -p 8080:9393 -v /home/ruby/app:/app icchan/shotgun-sinatra shotgun --host 0.0.0.0 /app/hello.rb 
```

It will now be accessible at port 8080 on your host. If you are using boot2docker its probably `http://192.168.59.103:8080`.

Now edit the file `/home/ruby/app/hello.rb` on your host machine, the changes will automagically be reflected in the running app :)
