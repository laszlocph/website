---
layout: default
---

## Mastering test environments with Docker

In the [first part](Simple-Jenkins-and-Docker-workflow) of the series I addressed three out of four requirements

* we build in an independent environment
* we version the build environment together with the source code
* so as we version the build pipeline in the source repository

The last outstanding requirement makes the workflow so exciting to me, **the ability to provision an environment with a handpicked set of components and branches**, 
for local and QA environments, unlocks productivity to a great extent.

## Build every branch
But first we have to make sure that all branches get built automatically. 

Besides the Jenkinsfile, **Jenkins has another new feature in 2.0, it is now aware of branches in the source code repository.** 
This nifty feature spares us a ton of boilerplate work as we don't have to set up various flavors of the build pipeline for every branch ourselves.

By picking *Multibranch Pipeline* on the job creation page we get the same behavior as I showcased in part one, plus Jenkins automatically creates a sub-project for each branch that it finds in a repository with a Jenkinsfile.

![Multi-branch support](multibranch.png)

We also got a new item on the side bar to reindex branches on demand. It is able to detect new branches and delete pipelines that belong to deleted ones.
 
![Multi-branch support](branch-indexing.png)

I made one more modification to the pipeline to make it practically useful. In a new final step I **push the built Docker container image to a [registry](https://hub.docker.com/r/laszlocph/spring-boot-dummy/tags/)**. 
With this step it is now available to other processes for verification and deployment.

## Define stacks
At this point components are built continuously on every branch. As a next step I describe their interrelatedness with Docker Compose and handle them as a logical unit to bring up local and later remote environments.

Docker Compose has a simple yaml syntax, and while it has [powerful options](https://docs.docker.com/compose/compose-file/), it is very easy to define simple stacks.

<pre>
version: '2'

services:
 web-python:
   image: laszlocph/composetest
   ports:
    - "${PORT}:5000"
   depends_on:
    - redis
 boot:
   image: laszlocph/spring-boot-dummy
 redis:
   image: redis

</pre>

Above I defined three services, 
* *web-python* is based on a dummy webapp, the laszlocph/composetest image and depends on a running redis container
* *boot* is the well known Spring Boot application from part one.
* *redis* is just a vanilla Redis container

Running this stack is nothing more than a executing the **docker-compose up** command. We don't have to fiddle with individual *docker run* commands as
it will start all containers in the right order, with the right volumes and exposed ports as they are described in the docker-compose file. 

Furthermore it defines a *Docker network*, so the services are able to communicate with each other in separation to other network traffic. 
They can do that by simply mentioning the other service's name as Compose places an entry to each container's host file with all available services and their IPs.

<pre>
laszlo@~/multi-env: docker-compose up
Creating network "multienv_default" with the default driver
Creating multienv_boot_1
Creating multienv_redis_1
Creating multienv_web-python_1
Attaching to multienv_boot_1, multienv_redis_1, multienv_web-python_1
boot_1        | 
boot_1        |   .   ____          _            __ _ _
boot_1        |  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
boot_1        | ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
boot_1        |  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
boot_1        |   '  |____| .__|_| |_|_| |_\__, | / / / /
boot_1        |  =========|_|==============|___/=/_/_/_/
boot_1        |  :: Spring Boot ::        (v1.4.0.RELEASE)
boot_1        | 
redis_1       | 1:C 12 Oct 09:33:23.093 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis_1       |                 _._                                                  
redis_1       |            _.-``__ ''-._                                             
redis_1       |       _.-``    `.  `_.  ''-._           Redis 3.2.4 (00000000/0) 64 bit
redis_1       |   .-`` .-```.  ```\/    _.,_ ''-._                                   
redis_1       |  (    '      ,       .-`  | `,    )     Running in standalone mode
redis_1       |  |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
redis_1       |  |    `-._   `._    /     _.-'    |     PID: 1
redis_1       |   `-._    `-._  `-./  _.-'    _.-'                                   
redis_1       |  |`-._`-._    `-.__.-'    _.-'_.-'|                                  
redis_1       |  |    `-._`-._        _.-'_.-'    |           http://redis.io        
redis_1       |   `-._    `-._`-.__.-'_.-'    _.-'                                   
redis_1       |  |`-._`-._    `-.__.-'    _.-'_.-'|                                  
redis_1       |  |    `-._`-._        _.-'_.-'    |                                  
redis_1       |   `-._    `-._`-.__.-'_.-'    _.-'                                   
redis_1       |       `-._    `-.__.-'    _.-'                                       
redis_1       |           `-._        _.-'                                           
redis_1       |               `-.__.-'                                               
redis_1       | 
...
web-python_1  |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
web-python_1  |  * Restarting with stat
web-python_1  |  * Debugger is active!
web-python_1  |  * Debugger pin code: 238-470-859

</pre>

Docker Compose seamlessly integrates with other Docker tools. You can see that the containers it started are regular containers themselves, just run *docker ps* to verify.

## Operating Docker Compose


You can get the true power of Docker Compose from its [CLI reference](https://docs.docker.com/compose/reference/), I want to highlight a few of them that can get you very far:

* *docker-compose up -d* to run a stack detached
* *docker-compose logs -f* to tail a stack's output
* *docker-compose ps* to see the state of the stack
* *docker-compose stop* to stop a stack
* *docker-compose rm* to clean up once you are done

## Running a stack remotely

As I mentioned above Docker Compose integrates with other Docker tools: you can use Compose locally, but you can also point it at a remote machine, 
or a cluster of Docker Engines, allowing to start up a QA environment or even a production stack just as easy as running one locally. And that has huge value for me. 

Imagine when a new colleague joins the team and she has a local environment up and running within fifteen minutes. Or when she has her first feature ready to showcase, she can share the QA environment with her team with one command. 
**Now this is what I call proper onboarding.** 





Individual components => stack: Docker-compose
Running local

Running remote is just as easy 
Docker-machine
point at: eval
unset

Pick and choose:
components and branches
small video
conclusion: you can have as many envs exposed to QA, PM, to anyone in great convenience

Scaling / production?
Swarm vs swarm mode
conclusions

next steps













