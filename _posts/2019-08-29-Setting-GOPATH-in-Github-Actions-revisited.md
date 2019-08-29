---
layout: drone
title: Setting GOPATH in Github Actions - revisited
image: images/1500x500.jpg
link: /setting-gopath-in-github-actions-revisited
permalink: /setting-gopath-in-github-actions-revisited
excerpt: Github Actions defaults are not ideal for Golang projects. This article shows how I configured GOPATH in Actions.


---

## Setting GOPATH in Github Actions - revisited

{{ page.date | date: "%Y-%m-%d" }}

If you are yet to adopt Go modules, you ought to have GOPATH set in Github Actions to have your builds succeed.

I [already found](https://laszlo.cloud/setting-gopath-in-github-actions){:target="\_blank"} a way how to set $GOPATH and in this article I demonstrate a slightly different approach.

But first a detour to understand the environment.

### Actions runner environment

With the `runs-on: ubuntu-latest` field you can set the runtime environment for the job. This will define the host VM the job will run in.

Github Actions provide a few ways to override the execution environment - or technically to start Docker containers inside this VM and perform steps in these Docker containers.

#### The run step

The most simple way to perform custom steps in Actions is to use the `run` field. This will run shell commands in the host VM's shell.

So this is not really an override of the execution environment, but more like the simplest way to use the default.

```yaml
name: Go
on: [push]

jobs:
  my-first-job:
    name: My first job
    runs-on: ubuntu-latest
    steps:
    - name: Hello
      run: |
        echo "Hello world"
        echo "Yaml allows me to have multiline commands"
```

See the [reference](https://help.github.com/en/articles/workflow-syntax-for-github-actions#jobsjob_idstepsrun){:target="\_blank"}.

#### Using a custom action

Github Actions - the product - is about the reusable building blocks, called *actions*.

Actions are Docker containers made to perform one specific task. They accept incoming parameters in the `with` field, but don't allow customized behavior, eg a `run` field can't be used to override behavior.

```yaml
    - name: Check out code
      uses: actions/checkout@v1
      with:
        fetch-depth: 1
```

Under the hood the runner pulls the actions Docker image and runs it on the host VM.

This is a way to run Docker containers, but with predefined behavior. You can write your own actions too, [check out how](https://help.github.com/en/articles/building-actions){:target="\_blank"}.

#### The docker runner environment

There is a third way to define the exection environment. By using the `container` field, you can run custom commands within a specific Docker container. 

```yaml
name: Go
on: [push]

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    container:
      image: golang:1.12.4
    steps:
    - name: Debug
      run: |
        go version
```

The above example sets the execution environment to be the `golang:1.12.4` container, and all steps will be performed in that image, thus the Go tooling will be available throughout the job.

The `runs-on` field is still mandatory as there is always a host machine, but with this approach I can prebake images that have the needed tools installed. Like `kubectl` or `go`.

This also differs from the canonical examples on the Github Actions page where the `actions/setup-go@v1` action is used to install Go - the approach I also used in the first part of this article.

### Setting GOPATH using the docker execution environment

Putting the above in practice, and using the checkout path trick from the [first part](https://laszlo.cloud/setting-gopath-in-github-actions){:target="\_blank"} of this article, here is how I compiled go with Github Actions.

```yaml
name: Go
on: [push]

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    container:
      image: golang:1.12.4
      env:
        GOPATH: /__w/woodpecker/go/
    steps:

    - name: Check out code into the Go module directory
      uses: actions/checkout@v1
      with:
        fetch-depth: 1
        path: go/src/github.com/laszlocph/woodpecker

    - name: Test
      run: |
        go get -u golang.org/x/tools/cmd/cover
        go get -u golang.org/x/net/context
        go get -u golang.org/x/net/context/ctxhttp
        go get -u github.com/golang/protobuf/proto
        go get -u github.com/golang/protobuf/protoc-gen-go
        go test -cover $(go list ./... | grep -v /vendor/)
```

### I was keen on using the containerized environment for couple of reasons

First, `actions/setup-go@v1` is not ready yet, and setting $GOPATH manually is cumbersome.

Second, I'm a big supporter of the Drone.io CI system which to this day has revolutionary architecture. In Drone each step of the pipeline is performed in a specific Docker image. Replicating that behavior with Actions was my desire and I could achieve some of it.

The `jobs.container` field does an okay job setting runtime environment, but it also differs from Drone as there is only one image to be used in the whole job.

I'm not sure what to think of the Github Actions containerized environment and  Github doesn't seem to be sure either. There are multiple ways to control the runtime as I listed in this article, and the Github tutorials seem to favor the host machine examples, not the container ones.

One thing is clear though, Actions is flexible and the best practices will solidify over time.

Oh and one more thing, did you know that you can run `docker-compose up` in Actions? [Proof](https://github.com/peter-evans/docker-compose-actions-workflow){:target="\_blank"}.

Onwards!
