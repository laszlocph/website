---
layout: default
title: How using cache-from can speed up your Docker builds in DroneCI
image: images/1500x500.jpg
link: /how-using-cache-from-can-speed-up-your-docker-builds-in-droneci
permalink: /how-using-cache-from-can-speed-up-your-docker-builds-in-droneci
excerpt: About layer caching in general and its problems in a distributed setup. And how --cache-from is able to solve it reliably in Drone.io even when using the plugins/docker plugin to build Docker images.

--- 

## How using cache-from can speed up your Docker builds in DroneCI
{{ page.date | date: "%Y-%m-%d" }}

Francis I of France paid 4,000 écus in 1540 for Leonardo da Vinci’s Mona Lisa. On the off chance that a few of you have not kept track of the fluctuations of the écu - 4,000 converted out to about $20,000. 

If Francis had kept his feet on the ground and he (and his trustees) had been able to find a 6% after-tax investment, the estate now would be worth something over $1,000,000,000,000,000.00. That's $1 quadrillion.

The above excerpt is from Warren Buffett's 1964 shareholder letter where he showed masterfully how powerful compounding benefits are.

Speeding up your CI pipeline can have similar exponential effects.

### How does Docker layer caching speed up the CI build?

When building an image, Docker steps through the instructions in your Dockerfile, executing each in the order specified. As each instruction is examined, Docker looks for an existing image in its cache that it can reuse rather than building it. This can save a lot of time if the instruction is a slow one like `npm install` or `bundle install`.

### It's a Docker best practice to leverage the layer cache. 

I build my Docker images as the first step of my Drone pipeline - as opposed to building it as the very last step and treating it only as a build artifact - therefore it's important that the layer cache works reliably.

The good news is that it just works if I have a single Drone agent. However with multiple build agents the layer cache state differs between agents and does not work consistently.

Furthermore if I use the `plugins/docker` plugin to build Docker images, the layer cache does not work at all given its docker-in-docker (dind) architecture.

### Making the layer cache consistent
Distributing caches across agents or mounting the layer cache into a dind setup is a gargantuan task. Instead we can use docker build's `--cache-from` option.

With `--cache-from` you can specify a list of images what `docker build` will consider as a source of cached layers. So instead of relying on an unspecified local state you can rely on tagged images in a registry.

### How to use --cache-from?

Let's take a simple Dockerfile with an apt install step and a `.drone.yml` file that uses the `plugins/docker` plugin to build the image.

```dockerfile
FROM debian:stable-slim

RUN apt-get update && apt-get install -y \
    curl \
 && rm -rf /var/lib/apt/lists/*
```

```yaml
kind: pipeline
name: default

steps:
  - name: docker-builder
    image: plugins/docker
    settings:
      repo: laszlocloud/cache-from-test
      tags: latest
      cache_from: "laszlocloud/cache-from-test:latest"
      dry_run: true
```

The console output shows that first the plugin pulls the listed image from the registry and then includes the `--cache-from` option in the docker build command. At the end you can see that the apt step is using the cache.

If you would take the above snippets, the build would use the cached layers even though you have not built this image before.


```bash
➜ ./drone exec
[docker-builder:69] + /usr/local/bin/docker pull laszlocloud/cache-from-test:latest
[docker-builder:70] latest: Pulling from laszlocloud/cache-from-test
[docker-builder:77] 0e3d8c77ad65: Pull complete
[docker-builder:80] + /usr/local/bin/docker build --rm=true -f Dockerfile -t 00000000 . --pull=true --cache-from laszlocloud/cache-from-test:latest --label org.label-schema.schema-version=1.0 --label org.label-schema.build-date=2019-02-17T14:23:07Z --label org.label-schema.vcs-ref=00000000 --label org.label-schema.vcs-url=
[...]
[docker-builder:88] Step 2/6 : RUN apt-get update && apt-get install -y     curl  && rm -rf /var/lib/apt/lists/*
[docker-builder:89]  ---> Using cache
[docker-builder:90]  ---> 880ea2ef13d2
```

### How to use cache-from with multiple branches?

The previous example used the `latest` image tag which is not a good practice in general and it also makes the caching strategy unpredictable if you use multiple branches.

You can't be sure which branch was used to build the `latest` image. If your branch has a change in your `package.json` or `Gemfile` and the `latest` image is from master, your branch builds will not utilize the cache and do npm or bundle install anyway.

To handle multiple branches we have to adjust `.drone.yml`.
```yaml
kind: pipeline
name: default

steps:
  - name: docker-builder
    image: plugins/docker
    settings:
      repo: laszlocloud/cache-from-test
      tags: 
        - "${DRONE_BRANCH}"
        - "${DRONE_BRANCH}-${DRONE_COMMIT}"
      cache_from:
        - "laszlocloud/cache-from-test:master"
        - "laszlocloud/cache-from-test:${DRONE_BRANCH}"
      dry_run: true
```

The trick to have branch specific cache sources is to tag the branch images explicitly.

Should you build this branch the first time, the `master` image will provide some level of caching. Once you built your branch once, the following builds will have a branch specific cache image.

And yes, you can have any number of images specified in `--cache-from`.

### What about time spent downloading those images?

Since you have to pull the `--cache-from` image first from the registry, it can happen that the time you gain with caching the layers you lose on downloading the images. This depends on a few factors, like the size of the image and the time it takes to build it.

It happened to me some cases that my aggressive caching strategy made the build slower, so it's worth keeping in mind the network cost and measuring the build time.


### How I saved 4 minutes build time of a Ruby app?

I had a large Ruby app with a Dockerfile having the following snippet: 

```
WORKDIR /app
COPY Gemfile Gemfile.lock /app/
RUN bundle install
COPY . ./app/
```

First it copies `Gemfile` then did a `bundle install` and only after copied the rest of the source code. With this trick I made sure only a `Gemfile` change busts the cache.


The only problem was that `bundle install` took four minutes. The layer cache was working reliably when I had a single Drone agent but became an issue once there were more.

Using cache-from saved this 4 minutes consistently and bundle install only happened when the Gemfile really changed. I was happy.

### In a summary

First I wrote about layer caching in general and its problems in a distributed setup.

`--cache-from` is able to solve layer caching reliably in Drone.io even when using the `plugins/docker` plugin to build Docker images.

As a closing thought I include this oldie from xkcd. If you can save five minutes on a build that is running five times a day, you can easily spend a day or two optimizing it with `--cache-from`. So what are you waiting for?

![Effort roi](images/times.jpg)

### So how much Mona Lisa worth today?

I left the story by saying that should Francis I of France invested his money in the market he would have made a gajillion by now. But how much Mona Lisa worth today?

Well, the Mona Lisa was assessed at US$100 million on December 14, 1962. Taking inflation into account, the 1962 value would be around US$830 million in 2019.

And some people say art is a good investment ;)
