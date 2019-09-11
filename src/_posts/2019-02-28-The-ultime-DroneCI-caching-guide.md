---
layout: default
tag: drone
title: The ultimate DroneCI caching guide
image: images/1500x500.jpg
link: /the-ultimate-droneci-caching-guide
permalink: /the-ultimate-droneci-caching-guide
excerpt: In this article I demonstrate all practical caching solutions in Drone.io - volume based caches, bucket based caches and using Docker's layer cache as a primary caching mechanism.

---

## The ultimate DroneCI caching guide

{{ page.date | date: "%Y-%m-%d" }}

Sisyphos was the king of Corinth in the Greek mythology. He was punished by the gods by being forced to roll an immense boulder up a hill only for it to roll down when it nears the top. Repeating this action for eternity.

Installing dependencies in every CI run can be equally futile, hence the need to cache dependencies arise quickly once there is a basic pipeline running.

In this article I'm going to demonstrate all practical caching solutions in Drone CI: volume based caches, bucket based caches and using Docker's layer cache as a primary caching mechanism.

Let's jump in with a naive volume based cache.

### How to quickly set up volume based caching

Once a repository is set to "Trusted" in Drone, the bellow pipeline will mount the `/tmp/drone/cache/node_modules` host path to the `node_modules` folder in your Drone step. It will also retain its content between pipeline runs.

```
kind: pipeline
name: default

steps:
- name: build
  image: node
  volumes:
  - name: cache
    path: /drone/src/node_modules
  commands:
  - 'echo ''{"devDependencies": {"@angular/cli": "^7.3.3" }}'' > package.json'
  - npm install

volumes:
- name: cache
  host:
    path: /tmp/drone/cache/node_modules
```

Running this synthethic example, it shows a five seconds speedup:

```
➜ ../drone exec --trusted
[build:0] + echo '{"devDependencies": {"@angular/cli": "^7.3.3" }}' > package.json
[build:1] + npm install
[build:7]
[build:8] added 295 packages from 179 contributors and audited 17414 packages in 9.827s
[build:9] found 0 vulnerabilities
[build:10]

➜ ../drone exec --trusted
[build:0] + echo '{"devDependencies": {"@angular/cli": "^7.3.3" }}' > package.json
[build:1] + npm install
[build:7]
[build:8] audited 17414 packages in 5.002s
[build:9] found 0 vulnerabilities
```

The demonstrated caching approach works, however many Drone user prefer using plugins for specific tasks.

### Using the `drillster/drone-volume-cache` plugin

On the Drone plugin site there is a featured volume based cache, it's the `drillster/drone-volume-cache`.

The plugin is [well documented](http://plugins.drone.io/drillster/drone-volume-cache/){:target="\_blank"} and should you have additional questions, the plugin's source code is [hosted on Github](https://github.com/Drillster/drone-volume-cache/blob/master/cacher.sh){:target="\_blank"}.

There are three steps in the drone file after adjusting the previous example to use this volume cache. I wrapped the build step with restore and rebuild steps which are responsible to handle the cache state.

The mount paths also changed. I pointed the Drone host mounts to the `/cache` in-container-path and the plugin is responsible to copy the cache content to the `node_modules` folder.

```
kind: pipeline
name: default

steps:
- name: restore-cache
  image: drillster/drone-volume-cache
  settings:
    restore: true
    mount:
      - ./node_modules
  volumes:
  - name: cache
    path: /cache

- name: build
  image: node
  commands:
  - 'echo ''{"devDependencies": {"@angular/cli": "^7.3.3" }}'' > package.json'
  - npm install

- name: rebuild-cache
  image: drillster/drone-volume-cache
  settings:
    rebuild: true
    mount:
      - ./node_modules
  volumes:
  - name: cache
    path: /cache

volumes:
- name: cache
  host:
    path: /tmp/drone/cache
```

I ran this pipeline twice and a similar 5 second speedup is realized. First, the cache is empty and by the second run the plugin found the cache content and it restored it to the mount path.

In the example below I cleared `node_modules` in the working directory to make sure that the modules are populated from the cache. If I don't run `rm -rf`, local `drone exec` runs can be confusing as it's difficult to be certain who put the content in the `node_modules` folder.

```
➜ sudo rm -rf node_modules && ../drone exec --trusted
[restore-cache:0] No cache for ./node_modules
[build:0] + echo '{"devDependencies": {"@angular/cli": "^7.3.3" }}' > package.json
[build:1] + npm install
[build:7]
[build:8] added 295 packages from 179 contributors and audited 17414 packages in 12.237s
[build:9] found 0 vulnerabilities
[build:10]
[rebuild-cache:0] Rebuilding cache for folder ./node_modules...
➜ sudo rm -rf node_modules && ../drone exec --trusted
[restore-cache:0] Restoring cache for folder ./node_modules...
[build:0] + echo '{"devDependencies": {"@angular/cli": "^7.3.3" }}' > package.json
[build:1] + npm install
[build:7]
[build:8] audited 17414 packages in 5.143s
[build:9] found 0 vulnerabilities
[build:10]
[rebuild-cache:0] Rebuilding cache for folder ./node_modules...
```

### My experience with the drillster/drone-volume-cache plugin

I used the `drillster/drone-volume-cache` for a couple months, running approximately 50 daily builds with it.

Eventually I forked it and changed it to my own needs. The main driver of that was a weird issue of the cached Ruby gems, sometimes one particular of the restored gems was corrupt.

We suspected in the team that perhaps the concurrent writing of the cache could cause the problem, so I changed the plugin so it first tars up the cache content and then switches the current cache pointer symlink in one atomic step.

I can't say for sure that this solved the problem, or a Ruby version update we had in those weeks, but the issue dissappeared.

All in all the drillster plugin is a good go-to solution if you chose volume based caching. And should you ever need to improve it, the codebase is easy to understand.

### Volume based caching however comes with a limitation however

And that is - what you have guessed already - the cached state is only available on a single build agent.

You may try to synchronize the cache between your agents, or you may accept the inconsistency.

Or you can store the cache in a central location, in a bucket.

### Distribute caching with buckets

There are [two](http://plugins.drone.io/tags/cache/){:target="\_blank"} bucket based cache plugins that cover two major cloud providers: Amazon S3 and Google Cloud. I demonstrate the one for [Amazon S3](http://plugins.drone.io/drone-plugins/drone-s3-cache/){:target="\_blank"} here.

The semantics are similar as before, here is the adjusted pipeline:

```
kind: pipeline
name: default

steps:
- name: restore
  image: plugins/s3-cache
  settings:
    pull: true
    root: cachetest1.laszlo.cloud
    # region: "us-east-1"
    access_key:
      from_secret: aws_access_key_id
    secret_key:
      from_secret: aws_secret_access_key
    restore: true

- name: build
  image: node
  commands:
  - 'echo ''{"devDependencies": {"@angular/cli": "^7.3.3" }}'' > package.json'
  - npm install

- name: rebuild
  image: plugins/s3-cache
  settings:
    pull: true
    root: cachetest1.laszlo.cloud
    # region: "us-east-1"
    access_key:
      from_secret: aws_access_key_id
    secret_key:
      from_secret: aws_secret_access_key
    rebuild: true
    mount:
      - node_modules
    when:
      event: push
```

The plugin configuration is not intuitive. It uses the `root` field for the bucket name and there is no official example of how to use the plugin with Amazon S3. It supports other S3 compatible bucket solutions - like minio - and the plugin documentation only showcases the setup of those.

Nevertheless if the `aws_*` secrets are correct, and the `root` field is set, it will cache everything from the `mount` path and places it under the bucket in the `[root]/<owner>/<repo>/<branch>/` naming scheme. This schema can be redefined with the `path` field.

I don't have experience running this plugin for an extended period, but after the initial hurdles of locating the `root` parameter, it does what it says.

```
➜ sudo rm -rf node_modules && ../drone exec --trusted --secret-file ../dummysecrets
[restore:4] time="2019-02-27T19:29:13Z" level=info msg="Restoring cache at /cachetest1.laszlo.cloud/master/archive.tar"
[restore:7] time="2019-02-27T19:29:35Z" level=info msg="Downloaded 69 MB from server"
[restore:8] time="2019-02-27T19:29:35Z" level=info msg="Cache restored"
[build:0] + echo '{"devDependencies": {"@angular/cli": "^7.3.3" }}' > package.json
[build:1] + npm install
[build:7]
[build:8] audited 17414 packages in 4.606s
[build:9] found 0 vulnerabilities
[build:10]
[rebuild:8] time="2019-02-27T19:29:52Z" level=info msg="Putting file in cachetest1.laszlo.cloud at master/archive.tar"
[rebuild:9] time="2019-02-27T19:31:31Z" level=info msg="Uploaded 69 MB to server"
[rebuild:10] time="2019-02-27T19:31:31Z" level=info msg="Cache rebuilt"
```

### What about the network overhead of downloading cache files from buckets?

If you ran the above snippet you saw that it saved the five seconds that we saw can be saved, but downloading and then uploading 69MBs takes lot longer than that.

While this is an artificial example only, sometimes caching made my build slower, simply because what I gained with not installing packages was lost on network transfer.

Caching is no silver bullet, but it can help in many cases. Just make sure you measure the effect.

### How to use Docker layer caching as the primary caching

For a long time I was building the Docker image as the very last step of my pipeline - just before deploying it.

Besides that I had a volume based cache and it happened that in Ruby projects I ran `bundle install` twice. Once in the CI step and then during `docker build`.


A colleague of mine mocked me just long enough that I took a chance on his idea. What if I build the Docker image first, and then run the subsequent build tests and other steps within that image?

Following this approach the well known example looks like this:

```
kind: pipeline
name: default

steps:
- name: prep
  image: node
  volumes:
  - name: cache
    path: /drone/src/node_modules
  commands:
  - 'echo ''{"devDependencies": {"@angular/cli": "^7.3.3" }}'' > package.json'
  - 'echo ''FROM node\nCOPY package.json .\nRUN npm install'' > Dockerfile

- name: docker-builder
  image: plugins/docker
  settings:
    repo: laszlocloud/image-first-test
    tags: latest
    cache_from: "laszlocloud/image-first-test:latest"
    dry_run: true
```

The key for this strategy to work is to set the `cache_from` parameter and provide a list of Docker images that `docker build` will consider as layer cache sources.

The console output shows that first the plugin pulls the listed image from the registry and then includes the `--cache-from` option in the docker build command. At the end you can see that the `npm install` step is using the cache.

If you would take the above snippets, the build would use the cached layers even though you have not built this image before on your machine.


```
[prep:0] + echo '{"devDependencies": {"@angular/cli": "^7.3.3" }}' > package.json
[prep:1] + echo 'FROM node
[prep:2] COPY package.json .
[prep:3] RUN npm install' > Dockerfile
[docker-builder:70] latest: Pulling from laszlocloud/image-first-test
[docker-builder:115] Status: Downloaded newer image for laszlocloud/image-first-test:latest
[docker-builder:116] + /usr/local/bin/docker build --rm=true -f Dockerfile -t 00000000 . --pull=true --cache-from laszlocloud/image-first-test:latest --label [docker-builder:134] Step 3/7 : RUN npm install
[docker-builder:135]  ---> Using cache
[docker-builder:136]  ---> 331dbe3922cb
```

I wrote about the using the Docker layer cache and `--cache-from` in Drone.io in [this article](https://laszlo.cloud/how-using-cache-from-can-speed-up-your-docker-builds-in-droneci){:target="\_blank"}.

And this concludes the fourth approach to caching in Drone.

### We have seen four approaches to caching

Bellow you can see a comparison table of all the demonstrated strategies.

I started out using the drillster plugin, then went straight to the Docker layer caching approach.

Don't be like Sisyphos, pick a caching strategy!

|                              | Pro                                                                                                                        | Con                                                                                                                                                      |
| ---------------------------- | :------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Naive volume mounts          | easy to get started                                                                                                        | - difficult to maintain as each pipeline will have its own flavor of caching<br/>- inconsistent on multiple agents <br/>- doesn't work on cloud.drone.io |
| drillster/drone-volume-cache | <br/>- clean source code<br/>- easy to extend                                                                              | <br/>- inconsistent on multiple agents <br/>- doesn't work on cloud.drone.io                                                                             |
| S3 buckets                   | - works with multiple agents<br/> - works on cloud.drone.io                                                                | - setup is not intuitive<br/>- network transfer can be long                                                                                              |
| Docker --cache-from          | - if you ship Docker images, you don't have to cache twice<br/>- reproducible builds<br/>- works with multiple agents<br/> - works on cloud.drone.io | - network transfer can be long<br/>- building the image first does not come natural                                                                                                                           |
