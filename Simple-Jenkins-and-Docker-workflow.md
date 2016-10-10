---
layout: default
---

## A simple Jenkis 2.0 and Docker workflow

---
**tldr**
In this article I show a Docker based workflow in Jenkins 2.0 with both the CI pipeline and the build environment version 
controlled. The end result is a container with code ready to deploy.

<iframe width="560" height="315" src="https://www.youtube.com/embed/OV05bg30FoQ" frameborder="0" allowfullscreen></iframe>

---


The Docker ecosystem gave me that final push I needed to get started with my consulting career. While I could function as a 
**startup veteran**, **interim CTO** or **engineering management consultant** I realized that there is something very profound in the 
maturity of the Docker world i should double down on. I was amazed by how converged and mature the ecosystem is and adopting Docker
is not something for 2017, it's for the here and now. Since I'm a freelance consultant since this Monday, and it is already 
Wednesday, I should start sharing, shouldn't I?

## The power of releasing often
Last Friday, I left [Falcon.io](https://www.falcon.io/) after five years. That naturally puts me in a very reflective mood, not that I don't reflect much in general anyway... But this article is not one of *those*. What should concern you though is that after making an 
inventory of what worked well in that five years, one thing stood out.

The investment we put early on into our automated **CI and Sandbox environment** payed off big time. It allowed developers to provision 
their own sandbox environment with handpicked set of branches from all components, and to release their own feature in separation to all 
others. And that's clever. Clever, since **there is nothing engineers like more than shipping**, seeing the impact they have, getting feedback on their work is a motivator like nothing else.

Now, I'm excited to have a more modern take on that workflow utilizing Docker and modern CI tools. A lot happened since we launched 
our homegrown VM based tooling. 

## Modern requirements
One requirement stands still: the ability to provision an environment with handpicked components and branches, be that locally, or on a 
QA environment.

My other requirements are:

* ability to have the build pipeline versioned in the source code repository
* ability to build in an independent environment
* and versioning that build environment together with the source code

Why these? Because they give more control to the developer. The environment will be the same locally and on the CI server, 
and factors out the need to talk with an operations engineer. While I have a sweet spot for operations engineers, they really 
shouldn't be bothered with *a certain a Ruby gem that is needed on the CI server*. 

**The separation of roles and responsibilities become more clear between dev and ops** this way, the ops people can move from being 
gatekeepers  (having root access to install *that Ruby gem*), to enablers. They make sure that plenty of computing power is provisioned 
to the CI environment, and.. well, that's all.

## An opinionated take
As I see it, the only blocker to get started with Docker, is the paralyzing amount of choice. There is simply too many good options to 
choose from. If there are 10 components with 3 good choices for each, you do the math why there's a **thousands choices** you have to 
make to reach nirvana.
 
In this article - and the many ones that will follow - I give an opinionated take of a working solution. While my choices are often hard
 to quantify, I will provide some reasoning, and I promise you that I always go towards simplicity. Albeit I pick things simple to me, 
 or to my team. I'm only pragmatic in my own universe.
 
## Jenkins 2.0

This choice may seem a bit old school in the abundance of services like [drone.io](http://drone.io), [circle.ci](http://circle.ci), 
[concourse.ci](http://concourse.ci), [this.ci](nope), [that.ci](neither this one) (you pick which one of these are actually a CI 
solution), you can't ignore the ubiquity of Jenkins. For me, the fact that I know it, and the promise of 2.0 being a drop-in replacement of previous versions makes it a simple choice.

In version 2.0 it introduces the Jenkinsfile: a [Groovy based DSL](https://jenkins.io/doc/pipeline/) to describe the build pipeline, ticking the box on one of my 
requirements. The other two boxes can also be ticked by integrating Docker into the Jenkins workflow, albeit with some plumbing. I'm going to showcase the necessary plumbing in this article.

## Running Jenkins in Docker

First and for most, let's get fancy and run Jenkins itself in Docker. To make it easy for you to 
try, and to gain experience for the times when your ops team is ready hosting it. 

For this purpose I extended the [base Docker image for Jenkins](https://github.com/jenkinsci/docker) with sudo capabilities. Other than this little change it's the vanilla Jenkins experience. 

<pre>
git clone git@github.com:laszlocph/jenkins-with-sudo.git
cd jenkins-with-sudo
docker build -t laszlocph/jenkins-with-sudo:latest .

docker run -it --rm --network=host -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker laszlocph/jenkins-with-sudo:latest
</pre>

Once Jenkins is up, chose the default plugin set, and continue as the *admin* user. You can find the login 
credentials in the startup log.


## Running Docker in Docker?
Let me explain the arguments of the *docker run* command.

The first *-v* option provides a **persistent volume** for Jenkins to store the job definitions and workspaces. You could mount here a 
specific location yourself, but I recommend not to hassle with plethora of file permission problems it brings, but let Docker create you this volume at first run.

The other *-v* options are more interesting. We are mounting the local Docker socket and Docker executable into the container. While this
may seem hackish, it is actually the least intrusive way to allow a Docker container to start new containers. These new containers will 
run on the **host's Docker daemon**, making them **siblings of the Jenkins container**. 
 
For alternatives, reasoning and consequences see [this](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/) and 
[this](http://container-solutions.com/running-docker-in-jenkins-in-docker/) article. Trust me, having read those you will find 
this solution quite nifty.

## Dockerfiles
For testing I prepared a small Spring Boot application (yes, Java) with two Dockerfiles. The *Dockerfile* in the *docker* folder is 
representing the runtime image of the application. 

<pre>
FROM anapsix/alpine-java

WORKDIR spring-boot-dummy
ADD spring-boot-dummy-0.1.0.jar .

CMD java -jar spring-boot-dummy-0.1.0.jar
</pre>


It copies the single fat *.jar* file to the container, then executes it with a simple *java -jar*. You can testdrive it yourself with the 
provided *build-image.sh*, and validate it on [http://localhost:8080](http://localhost:8080)

<pre>
git clone git@github.com:laszlocph/spring-boot-dummy.git
cd spring-boot-dummy
sh build-image.sh
docker run -it --rm --network=host laszlocph/spring-boot-dummy
</pre>

The *Dockerfile* in the root is the build environment. While for Java projects dependencies might not collide, Python and 
other projects benefit greatly from an independent build environments.

<pre>
FROM anapsix/alpine-java:8_jdk

WORKDIR spring-boot-dummy
ADD . .
CMD sleep 1h
</pre>

All needed dependencies for our simple Java app are provided in the SDK or are fetched by Gradle, so the only thing we have to do to 
compile our project is taking a small Java SDK image and adding all our files to the image. Then we let it sleep for an hour. 

Wait, what? 

[Yes](http://unix.stackexchange.com/a/270996), the image will not do anything itself, just provides the necessary dependencies. We will interact with it from the Jenkins workflow.

## Jenkinsfile

[My example pipeline](https://github.com/laszlocph/spring-boot-dummy/blob/master/Jenkinsfile) contains three stages:
* the first one to prepares the Build container image
* the second runs the actual build inside that container
* the last one takes the artifact and build the production container image.

![Pipeline](pipeline.png)

## Interacting with the Build container

The key in this pipeline is the second step where I start up the **Build container** and run the app specific build script with 
the *docker exec* command. 

Remember, all the Build container does is sleeping. It does that to be around even after the build script finished
generating the build artifacts. We can take that artifact further in the pipeline and stop the build container. 

<pre>
...
sh "sudo docker exec ${CONTAINER_ID} ./gradlew build"
sh "sudo docker cp ${CONTAINER_ID}:/spring-boot-dummy/build/libs/spring-boot-dummy-0.1.0.jar docker/spring-boot-dummy-0.1.0.jar"
sh "sudo docker stop ${CONTAINER_ID}"
sh "sudo docker rm ${CONTAINER_ID}"
...
</pre>

## To see it in action

Simply navigate to the [New Item](http://localhost:8080/view/All/newJob) page, provide an 
adequate name to your project and click *Pipeline* from the various presets.

On the configuration page edit the *Pipeline* group by selecting the *Pipeline script from SCM* option and configure the SCM url to git@github.com:laszlocph/spring-boot-dummy.git

Save it and click the *Build now* button. You should see all the steps executing successfully.

## Next steps
What we just did is remarkable from many aspects. 

* We incorporated Docker into our Jenkins workflow as all the builds run now in a designated Docker container. 
* furthermore, both the build and runtime environments are specified in simple text files and they live together with the source code. This enables developers to fully
control the build and runtime experience.

My next step is to continue on this path, and make this workflow aware of multiple system components, allowing developers 
to pick any component with any set of branches to provision a local, test, or even production environment.

Onwards!

---
I might be wrong, you know.. if you feel strongly about one or an other solution in this article, reach out at <a href="mailto:laszlo@laszlo.cloud">laszlo@laszlo.cloud</a> and let's have a chat! 

---

Full disclosure: many of Jenkins's design issues got highlighted in the most recent [Thoughtworks Tech Radar](https://www.thoughtworks
.com/radar/tools#blip-link-976). Those can't be overlooked. There are many challengers in the space, however Jenkins being a true 
workhorse of our industry, it is here to stay for many more years. And with these few steps, we better prepared it for the next era of CI 
automation.

---

<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-84825803-1', 'auto');
  ga('send', 'pageview');

</script>