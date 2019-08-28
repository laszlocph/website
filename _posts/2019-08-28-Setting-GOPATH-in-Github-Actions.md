---
layout: drone
title: Setting GOPATH in Github Actions
image: images/1500x500.jpg
link: /setting-gopath-in-github-actions
permalink: /setting-gopath-in-github-actions
excerpt: Github Actions defaults are not ideal for Golang projects. This article shows how I configures GOPATH in Actions.


---

## Setting GOPATH in Github Actions

{{ page.date | date: "%Y-%m-%d" }}

If you are yet to adopt Go modules, you ought to have GOPATH set in Github Actions to have your builds succeed.

This article shows you a way to set $GOPATH.

### First you need Go installed

It's early days with Github Actions to say for sure, but seemingly the canonical way to install tools on the runner environment is to use the `actions/setup-go@v1` Github Action.

It gets the job done, but the folder structure and `go env` is not set properly for builds that rely on $GOPATH

```
name: Go
on: [push]

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:

    - name: Set up Go 1.12
      uses: actions/setup-go@v1
      with:
        go-version: 1.12.4
    
    - name: Debug
      run: |
        pwd
        echo ${GOPATH}
        echo ${GOROOT}
```

$GOROOT is set but $GOPATH is not:

```
/home/runner/work/woodpecker/woodpecker

/opt/hostedtoolcache/go/1.12.4/x64
```

### Controlling the checkout path

Go requires a specific folder structure for the checked out code to be in. Unfortunately Github Actions's checkout path is not matching this layout. 

By default the checked out code is placed under `/home/runner/work/REPOSITORY/REPOSITORY`. This slightly odd path can be changed by parameterizing the checkout action:

```
    - name: Check out code to a GOPATH compatible directory
      uses: actions/checkout@v1
      with:
        fetch-depth: 1
        path: go/src/github.com/laszlocph/woodpecker
```

Running the debug commands again, the current folder will be `/home/runner/work/woodpecker/go/src/github.com/laszlocph/woodpecker` which resembles a valid $GOPATH.

### Setting $GOPATH

Now that the code is checked out to a folder that matches the layout of a typical GOPATH, we can set the GOPATH variable to the right path.

```
    - name: Test
      run: |
        go test -cover $(go list ./... | grep -v /vendor/)
      env:
        GOPATH: /home/runner/work/woodpecker/go
```

After setting this environment variable `go test` and other `go` commands start working. Unfortunately this variable needs to be set in every step where the Go tooling is used.

This shows that Github Actions is little rough around the edges and not optimized for Go workflows. I'm certain this is going to be improved, like in [this](https://github.com/actions/setup-go/issues/12){:target="\_blank"} issue where the setup-go action is getting better support for GOPATH.

Onwards!

And for reference, here is my complete pipeline:

```
name: Go
on: [push]

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:

    - name: Set up Go 1.12
      uses: actions/setup-go@v1
      with:
        go-version: 1.12.4

    - name: Check out code into the Go module directory
      uses: actions/checkout@v1
      with:
        fetch-depth: 1
        path: go/src/github.com/laszlocph/woodpecker

    - name: Debug
      run: |
        pwd
        echo ${HOME}
        echo ${GITHUB_WORKSPACE}
        echo ${GOPATH}
        echo ${GOROOT}
      env:
        GOPATH: /home/runner/work/woodpecker/go

    - name: Test
      run: |
        go get -u golang.org/x/tools/cmd/cover
        go get -u golang.org/x/net/context
        go get -u golang.org/x/net/context/ctxhttp
        go get -u github.com/golang/protobuf/proto
        go get -u github.com/golang/protobuf/protoc-gen-go
        go test -cover $(go list ./... | grep -v /vendor/)
      env:
        GOPATH: /home/runner/work/woodpecker/go

    - name: Build
      run: ./.drone.sh
      env:
        GOPATH: /home/runner/work/woodpecker/go

```
