---
layout: drone
title: Drone Environment Variables - Three tips
image: images/1500x500.jpg
link: /drone-environment-variables-three-tips
permalink: /drone-environment-variables-three-tips
excerpt: These are tricks I gathered over the years using Drone.io environment variables. All testable with `drone exec`.


---

## Drone Environment Variables - Three tips

{{ page.date | date: "%Y-%m-%d" }}

These are tricks I gathered over the years using Drone.io environment variables.

 - generating a practical Docker image tag
 - fixing an annoying issue with env vars
 - the sweetness of yaml anchors

All testable with `drone exec`.

### Using built-in vars to generate a practical Docker image tag

Drone has a set of built in variables that you can use to synamizally name your artifacts. You can find most Git and Drone related metadata on the supported list of vars [here](https://docs.drone.io/reference/environ/){:target="\_blank"}.

Drone also provides Bash parameter substitution like features to further manipulate the environment variables.

Hence my favorite Docker image tag is generated like this:

```yaml
${DRONE_BRANCH//\//-}-${DRONE_COMMIT_SHA:0:8}
```

First I replace forward slashes with a dash and only take the first 8 characters of the commit hash. You can use this `.drone.yml` and run it with `drone exec`.

```yaml
kind: pipeline
name: default

steps:
  - name: docker-tag-demo
    image: debian:stable-slim
    commands:
      - echo "The current branch is ${DRONE_BRANCH}"
      - echo "The current commit hash is ${DRONE_COMMIT_SHA}"
      - echo "The image tag is ${DRONE_BRANCH//\//-}-${DRONE_COMMIT_SHA:0:8}"
```

```
DRONE_COMMIT_SHA=3c1c9db713758d57a254eed96b0457cc9af3ae6b \
./drone exec \
  --branch feature/new-feature
[docker-tag-demo:1] The current branch is feature/new-feature
[docker-tag-demo:3] The current commit hash is 3c1c9db713758d57a254eed96b0457cc9af3ae6b
[docker-tag-demo:5] The image tag is feature-new-feature-3c1c9db7
```

### When variables resolve to empty string

There is a gotcha with environment variables that is best presented with the following pipeline.


```yaml
kind: pipeline
name: default

steps:
  - name: docker-tag-demo
    image: debian:stable-slim
    commands:
      - echo "The value is ${MY_VAR}"
      - echo "It was empty, but not this one $MY_VAR
      - echo "Nor this one $${MY_VAR}"
```

```bash
echo "MY_VAR=dummyvalue" > varfile
drone exec --env-file=varfile
[docker-tag-demo:1] The value is 
[docker-tag-demo:3] It was empty, but not this one dummyvalue
[docker-tag-demo:5] Nor this one dummyvalue
```

This is an annoying one to fix. Due to the demonstrated substitution functions, the `${variable}` expressions are subject to pre-processing. If you do not want the pre-processor to evaluate your expression, it must be escaped with an extra dollar sign: `$${variable}`.

### Reusing common variable groups with Yaml anchors

It happens that I needed to set the same environment variables in several of my Drone pipeline steps.

This quickly grows the yaml file so that it difficult to work with it. But then I figured out the syntax of yaml anchors.

```yaml
global-variables:
  debian_image: &debian_image debian:7.11-slim
  environment: &default_environment
    HOST: postgres
    USER: postgres

kind: pipeline
name: default

steps:
  - name: anchor test
    image: *debian_image
    environment:
      <<: *default_environment
    commands:
      - echo "The host is $${HOST}"
      - echo "The user is $${USER}"
```

### Thanks for reading

Onwards!
