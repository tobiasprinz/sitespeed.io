---
layout: default
title: Use Docker to run sitespeed.io.
description: With Docker you get a prebuilt container with sitespeed.io, Firefox, Chrome and XVFB. It's super easy to record a video and get visual metrics like Speed Index and First Visual Change.
keywords: docker, configuration, setup, documentation, web performance, sitespeed.io
nav: documentation
category: sitespeed.io
image: https://www.sitespeed.io/img/sitespeed-2.0-twitter.png
twitterdescription: Use Docker to run sitespeed.io.
---

[Documentation]({{site.baseurl}}/documentation/sitespeed.io/) / Docker

# Docker
{:.no_toc}

* Lets place the TOC here
{:toc}

## Containers

Docker is the preferred installation method because every dependency is handled for you for all the features in sitespeed.io.

We have a ready made container with [Chrome, Firefox & Xvfb](https://hub.docker.com/r/sitespeedio/sitespeed.io/). It also contains FFMpeg and Imagemagick, so we can record a video and get metrics like SpeedIndex using [VisualMetrics](https://github.com/WPO-Foundation/visualmetrics).

### Structure

The Docker structure looks like this:

[NodeJS with Ubuntu 18](https://github.com/sitespeedio/docker-node) -> [VisualMetrics dependencies](https://github.com/sitespeedio/docker-visualmetrics-deps) ->
[Firefox/Chrome/xvfb](https://github.com/sitespeedio/docker-browsers) -> [sitespeed.io](https://github.com/sitespeedio/sitespeed.io/blob/master/Dockerfile)

The first container installs NodeJS (latest LTS) on Ubuntu 18. The next one adds the dependencies (FFMpeg, ImageMagick and some Python libraries) needed to run [VisualMetrics](https://github.com/WPO-Foundation/visualmetrics). We then install specific version of Firefox, Chrome and lastly xvfb. Then in last step, we add sitespeed.io and tag it to the sitespeed.io version number.

We lock down the browsers to specific versions for maximum compatibility and stability with sitespeed.io's current feature set; upgrading once we verify browser compatibility.
{: .note .note-info}

## Running in Docker

The simplest way to run using Chrome:

```bash
docker run --rm -v "$(pwd)":/sitespeed.io sitespeedio/sitespeed.io:{% include version/sitespeed.io.txt %} -b chrome https://www.sitespeed.io/
```

In the real world you should always specify the exact version (tag) of the Docker container to make sure you use the same version for every run. If you use the latest tag you will download newer version of sitespeed.io as they become available, meaning you can have major changes between test runs (version upgrades, dependencies updates, browser versions, etc). So you should always specify a tag after the container name(X.Y.Z). Know that the tag/version number will be the same number as the sitespeed.io release:

```bash
docker run --rm -v "$(pwd)":/sitespeed.io sitespeedio/sitespeed.io:{% include version/sitespeed.io.txt %} -b chrome https://www.sitespeed.io/
```

If you want to use Firefox (make sure you make the shared memory larger using --shm-size):

```bash
docker run --shm-size 2g --rm -v "$(pwd)":/sitespeed.io sitespeedio/sitespeed.io:{% include version/sitespeed.io.txt %} -b firefox https://www.sitespeed.io/
```

Using `-v "$(pwd)":/sitespeed.io` will map the current directory inside Docker and output the result directory there.
{: .note .note-info}

## More about volumes

If you want to feed sitespeed.io with a file with URLs or if you want to store the HTML result, you should setup a volume. Sitespeed.io will do all the work inside the container in a directory located at _/sitespeed.io_. To setup your current working directory add the _-v "$(pwd)":/sitespeed.io_ to your parameter list. Using "$(pwd)" will default to the current directory. In order to specify a static location, simply define an absolute path: _-v /Users/sitespeedio/html:/sitespeed.io_

If you run on Windows, it could be that you need to map a absolute path. If you have problems on Windows please check [https://docs.docker.com/docker-for-windows/](https://docs.docker.com/docker-for-windows/).

## Update (download a newer sitespeed.io)

When using Docker upgrading to a newer version is super easy, change X.Y.Z to the version you want to use:

```bash
docker pull sitespeedio/sitespeed.io:X.Y.Z
```

Then change your start script (or where you start your container) to use the new version number.

## Tags and version

In the real world you should always specify the exact version (tag) of the Docker container to make sure you use the same version for every run. If you use the latest tag you will download newer version of the container as they become available, meaning you can have major changes between test runs (version upgrades, dependencies updates, browser versions, etc). So you should always specify a tag after the container name(X.Y.Z). This is important for sitespeed.io/browsertime/Graphite/Grafana containers. It's important for all containers you use. Never use the *latest* tag!

## Synchronise docker machines time with host

If you want to make sure your containers have the same time as the host, you can do that by adding <code>-v /etc/localtime:/etc/localtime:ro</code> (Note: This is specific to Linux).

Full example:

```bash
docker run --shm-size 2g --rm -v "$(pwd)":/sitespeed.io -v /etc/localtime:/etc/localtime:ro sitespeedio/sitespeed.io:{% include version/sitespeed.io.txt %} -b firefox https://www.sitespeed.io/
```

## Setting time zone

If you want your container to run in a specific time zone you can do that with TZ.

```bash
docker run -e TZ=America/New_York --rm -v "$(pwd)":/sitespeed.io sitespeedio/sitespeed.io:{% include version/sitespeed.io.txt %} -n 1 https://www.sitespeed.io
```

## Change connectivity

To change connectivity you should use Docker networks, read all about it [here]({{site.baseurl}}/documentation/sitespeed.io/browsers/#change-connectivity).

## Access localhost

If you run a server local on your machine and want to access it with sitespeed.io you can do that on Mac and Windows super easy if you are using Docker 18.03 or later by using _host.docker.internal_.

```bash
docker run --shm-size 2g --rm -v "$(pwd)":/sitespeed.io sitespeedio/sitespeed.io:{% include version/sitespeed.io.txt %} -b firefox http://host.docker.internal:4000/
```

If you are using Linux you should use `--network=host` to make sure localhost is your host machine.

```bash
docker run --shm-size 2g --rm -v "$(pwd)":/sitespeed.io --network=host sitespeedio/sitespeed.io:{% include version/sitespeed.io.txt %} -b firefox http://localhost:4000/
```

## Access host in your local network
Sometimes the server you wanna test is in your local network at work and Docker cannot reach it (but you can from your physical machine). Usually you can fix that by making sure Docker uses the same network as your machine. Add `--network=host` and it should work.


## Extra start script

You can run your extra start script in the Docker container:

```bash
docker run -e EXTRA_START_SCRIPT=/sitespeed.io/test.sh --rm -v "$(pwd)":/sitespeed.io ...`.
```

## Troubleshooting

If something doesn't work, it's hard to guess what't wrong. Then hook up x11vnc with xvfb so that you can see what happens on your screen.

### Inspect the container

We autostart sitespeed.io when you run the container. If you want to check what's in the container, you can do that by changing the entry point.

```bash
docker run -it --entrypoint bash sitespeedio/sitespeed.io:{% include version/sitespeed.io.txt %}
```

### Visualise your test in XVFB

The docker containers have `x11vnc` installed which enables visualisation of the test running inside `Xvfb`. To view the tests, follow these steps:

- You will need to run the sitespeed.io image by exposing a PORT for vnc server. By default this port is 5900. If you plan to change your port for VNC server, then you need to expose that port.

```bash
docker run --rm -v "$(pwd)":/sitespeed.io -p 5900:5900 sitespeedio/sitespeed.io:{% include version/sitespeed.io.txt %} https://www.sitespeed.io/ -b chrome
```

- Find the container id of the docker container for sitespeed by running:

```bash
docker ps
```

- Now enter into your running docker container for sitespeed.io by executing:

```bash
docker exec -it <container-id> bash
```

- Find the `Xvfb` process using `ps -ef`. It should be using `DISPLAY=:99`.
- Run

```bash
x11vnc -display :99 -storepasswd
```

Enter any password. This will start your VNC server which you can use by any VNC client to view:

- Download VNC client like RealVNC
- Enter VNC server : `0.0.0.0:5900`
- When prompted for password, enter the password you entered while creating the vnc server.
- You should be able to view the contents of `Xvfb`.
