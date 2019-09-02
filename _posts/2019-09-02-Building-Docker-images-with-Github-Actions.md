---
layout: drone
title: Builing Docker images with Github Actions
image: images/1500x500.jpg
link: /builing-docker-images-with-github-actions
permalink: /builing-docker-images-with-github-actions
excerpt: Github Actions defaults are not ideal for Golang projects. This article shows how I configured GOPATH in Actions.


---

## Builing Docker images with Github Actions

{{ page.date | date: "%Y-%m-%d" }}

Building Docker images in a CI pipeline is a common task. And Actions makes it convenient to do.

In this article I show a way to build Docker images on Github Actions.

### Using an action

First I tried finding an action that does what I planned to do.

There is a Github maintained action under [`actions/docker`](https://github.com/actions/docker){:target="\_blank"}, but it looks to be using the old HCL syntax, not the new yaml one.

If there is no suitable action to find, defaulting to a run step always does the job.

### Defaulting to a run step

A run step can run all the commands that you run on your laptop normally. If the necessary tools are installed of course. 

Luckilly Actions [have it installed](https://help.github.com/en/articles/software-in-virtual-environments-for-github-actions){:target="\_blank"} by default.

See it by running:

```yaml
name: Docker build
on: [push]

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:

    - name: Debug
      run: |
        docker version
```

### Building and pushing images to Docker Hub
```yaml
name: Docker build
on: [push]

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:

    - name: Checkout
      uses: actions/checkout@v1
      with:
        fetch-depth: 1

    - name: Docker build
      run: |
        docker login -u "$DOCKER_USERNAME" -p "$DOCKER_PASSWORD"
        docker build -t laszlocloud/actions-test .
        docker push laszlocloud/actions-test
      env:
        DOCKER_USERNAME: {% raw %}${{ secrets.DOCKER_USERNAME }}
        DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}{% endraw %}
```

The `docker` commands don't need much explanation, the only trick is in using the secrets.

Secrets needs to be first created in the repository settings, then included in a workflow step as an environment variable. See the documentation [here](https://help.github.com/en/articles/virtual-environments-for-github-actions#creating-and-using-secrets-encrypted-variables){:target="\_blank"}


Onwards!
