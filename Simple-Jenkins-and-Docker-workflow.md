---
layout: default
---

## A simple Jenkis 2.0 and Docker workflow

---
**tldr**
In this article I show a Docker based workflow in Jenkins 2.0 with both the CI pipeline and the build environment version 
controlled. The end result is a container with code ready to deploy.

---


When i was contemplating to start a consulting carrier the Docker ecosystem gave me that final push i needed. While I could function as a 
**startup veteran**, **interim CTO** or **engineering management consultant** i realized that there is something very profound in the 
maturity of the Docker world i should double down on. I was amazed by how converged and mature the ecosystem is and the adoption of 
Docker is not something for 2017, it's really for the here and now. Since i'm a freelance consultant since this Monday, and it is already 
Wednesday, i should start sharing, shouldn't i?

## The power of releasing often
Last Friday i left [Falcon.io](https://www.falcon.io/) after five years and that puts me in a very reflective mood. Not that i'm not reflecting way 
too much in general anyway... But this article is not one of *those*. What should concern you though is that after making an 
inventory of what worked well in that five years, one thing shined like thousand stars.

The investment we put early on into our automated **CI and Sandbox environment** payed off brilliantly. It allowed developers to provision 
their own sandbox environment with handpicked set of branches from all components and to release their own feature in separation to all 
others. And that's clever. Clever since **there is nothing that engineers like more than shipping**. Seeing the impact they make, getting
 true feedback of their work is a motivator like nothing else.

Now, I'm excited to have a more modern take on that workflow utilizing Docker and modern CI tools, because a lot happened since we launched 
our homegrown VM based tooling.

## Modern requirements
One requirement stands still: the ability to provision an environment with handpicked components and branches, be that locally, or on a 
QA environment.

The other requirements are:

* the ability to have the build pipeline versioned in the source code repository
* the ability to build in an independent environment
* and to version that build environment in the source code repository

Why these? Because these give even more control to the developer. The build environment will be the same locally and on the CI server, 
and factors out completely the need to talk to an operations engineer. While i have a sweet spot for operations engineers, they really 
shouldn't be bothered with *a certain a Ruby gem that is needed on the CI server*. 

Since **the separartion of roles and responsibilities become more clear between dev and ops**, the ops people should move from being 
gatekeepers  (having root access to install *that Ruby gem*), to enablers. They make sure that plenty of computing power is provisioned 
to the CI environment, and.. well, that's all.

## An opinionated take
As i see it, the only blocker to get started with Docker, is the paralysing effect of choice. There is simply too many good options to 
choose from, and if there are 10 components, with 3 good choices for each. You do the math why it's *thousands choices* you have to 
make to reach nirvana.
 
In this article - and the many ones that will follow - i give an opinionated take of a working solution. While my choices are often hard
 to quantify, i will provide some reasoning and i promise you that i always go towards simplicity. Albeit I pick things simple to me, 
 or to my team. I'm only pragmatic in my own universe.
 
## Jenkins 2.0

While this choice may seem a bit old school in the abundance of services like [drone.io](http://drone.io), [circle.ci](http://circle.ci), 
[concourse.ci](http://concourse.ci), [this.ci](nope), [that.ci](neither this one) (you pick which one of these are actually a CI 
solution), you can't go around the ubiquity of Jenkins, the fact that i know it, and the promise of 2.0 being a drop-in replacement of previous versions.

In version 2.0 it introduces the Jenkinsfile: a Groovy based DSL to describe the build pipeline, ticking the box on one of my 
requirements. The other two boxes can also be ticked by integrating Docker into the Jenkins albeit with some plumbing. Plumbing, what i'm going to 
show you here.

## Running Jenkins in Docker

First and for most let's get fancy, and run Jenkins in Docker to make it easy for you to 
try it yourself, and to gain experience for the times when your ops team is ready hosting it. 

I extended the [base Docker image for Jenkins](https://github.com/jenkinsci/docker) with sudo capabilities, other than that it's the 
vanilla Jenkins experience. Once Jenkins is up, chose the default plugin set, and continue as the *admin* user. You can find the login 
credentials in the startup log.

<pre>
git clone git@github.com:laszlocph/jenkins-with-sudo.git
cd jenkins-with-sudo
docker build -t laszlocph/jenkins-with-sudo:latest .

docker run -it --rm --network=host -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker laszlocph/jenkins-with-sudo:latest
</pre>

## Running Docker in Docker?
Let me explain the arguments of the *docker run* command.

The first *-v* option provides a **persistent volume** for Jenkins to store the job definitions and workspaces. You could mount here a 
specific location yoursef, but i recommend not to hassle with plathora of file permission problems it brings, but let Docker create you this volume at first run.

The other *-v* options are more interesting. We are mounting the local Docker socket and Docker executable into the container. While this
may seem hackish, it is actually the least intrusive way to allow a Docker container to start new containers. These new containers will 
run on the **host's Docker daemon**, making them effectively **siblings of the Jenkins container**. 
 
For alternatives, reasoning and consequences see [this](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/) and 
[this](http://container-solutions.com/running-docker-in-jenkins-in-docker/) article. Trust me, having read those you will find 
this solution quite nifty.

## Dockerfiles
For testing i prepared a small Spring Boot application (yes, Java) with two Dockerfiles. The *Dockerfile* in the *docker* folder is 
representing the runtime image of the application. 

<pre>
FROM anapsix/alpine-java

WORKDIR spring-boot-dummy
ADD spring-boot-dummy-0.1.0.jar .

CMD java -jar spring-boot-dummy-0.1.0.jar
</pre>


It copies the single fat *.jar* file to the container, then executes it with a simple *java -jar*. You can testdrive it yourself with the 
provided *build-image.sh*. The application can be validated on [http://localhost:8080](http://localhost:8080)

<pre>
git clone git@github.com:laszlocph/spring-boot-dummy.git
cd spring-boot-dummy
sh build-image.sh
docker run -it --rm --network=host laszlocph/spring-boot-dummy
</pre>

The *Dockerfile* in the root is the build environment. While for Java projects dependencies might not collide, Python and 
other projects would benefit greatly from an independent build environment.

<pre>
FROM anapsix/alpine-java:8_jdk

WORKDIR spring-boot-dummy
ADD . .
CMD sleep 1h
</pre>

All the dependencies needed for our simple Java app is provided in the SDK, or fetched by Gradle, so we don't have to do any more to 
compile our project. We take a small Java SDK image, we add all our files to the image, then we let it sleep for an hour. Wait, what? 
[Yes](http://unix.stackexchange.com/a/270996), the image will not do anything itself, just provide the necessary dependencies, and we 
interact with it from the Jenkins workflow.

## Jenkinsfile

Jenkins 2.0 provides a new way to define pipelines. All you have to do is to create a simple text file in your source code repository 
called Jenkinsfile, and define the pipeline in a [Groovy DSL format](https://jenkins.io/doc/pipeline/).

[My example pipeline](https://github.com/laszlocph/spring-boot-dummy/blob/master/Jenkinsfile) contains three stages:
* the first one to prepare the Build container image
* the second to run the actual build inside that container
* the last one to take the artifact and build the production container image.

![Pipeline](pipeline.png)

## Creating the Pipeline in jenkins

Once your Jenkins container is running, navigate to the [New Item](http://localhost:8080/view/All/newJob) page, where after providing an 
adequate name to your project click *Pipeline* from the various presets.

On the configuration page select edit the *Pipeline* group by selecting the *Pipeline script from SCM* option and configure the SCM url to git@github.com:laszlocph/spring-boot-dummy.git

Save it, then click the *Build now* button. You should see in the logs all the steps executing successfully.

## Interacting with the Build container

The key in this pipeline is the second step where we start up the previously built container to run the app specific build script with 
the *docker exec* command. Remember, all the build container does is sleeping. It does that to be around even after the build script 
generated the build artifacts. We can take that artifact further in the pipeline and stop the build container. 

<pre>
...
sh "sudo docker exec ${CONTAINER_ID} ./gradlew build"
sh "sudo docker cp ${CONTAINER_ID}:/spring-boot-dummy/build/libs/spring-boot-dummy-0.1.0.jar docker/spring-boot-dummy-0.1.0.jar"
sh "sudo docker stop ${CONTAINER_ID}"
sh "sudo docker rm ${CONTAINER_ID}"
...
</pre>

## Next steps
What we just did is remarkable from many aspects.

We incorporated Docker into our Jenkins workflow as all the builds run now in a designated Docker container. Furthermore, both the build
 and runtime environments are specified in simple text files and they live together with the source code. This enables developers to 
control the whole build and runtime experience.

Many of Jenkins's design issues got highlighted in the most recent [Thoughtworks Tech Radar](https://www.thoughtworks
.com/radar/tools#blip-link-976) what can't be overlooked. There are many challengers in the space, however Jenkins being a true 
workhorse of our industry, it is here to stay for many more years. And with these few steps, we better prepared it for the next era of CI 
automation.

As for me, the next steps are to continue on this path, and make this workflow aware of multiple system components, allowing developers 
to pick any component with any set of branches to provision a local, test, or even production environment.

Onwards!

<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-84825803-1', 'auto');
  ga('send', 'pageview');

</script>