---
layout: default
title: CI is a timesink, and it is our fault
image: images/1500x500.jpg
link: /ci-is-a-timesink-and-it-is-our-fault
permalink: /ci-is-a-timesink-and-it-is-our-fault
excerpt: We made CI this generic task engine that it is today. This article highlights why CI is often tedious to maintain and looks for inspiration to solve the problem.

---

## CI is a timesink, and it is our fault

{{ page.date | date: "%Y-%m-%d" }}

We think of GPUs as powerful beasts that are 10 or 100 times faster than CPUs. We see graphics and AI benchmarks proving this every single time.

I remember the day when I installed my first graphics card, the 3dfx Voodoo Banshee in my PC and instantly Quake 2 ran not just faster but way smoother as well. It felt like magic. CPUs looked slow and tedious in comparison.

Later I learned that GPUs are rather primitive, doing only a fraction of what CPUs can do, but they are really good at their job.

### What is CI's job anyway?

CI or build servers came to existence in the early 2000s when extreme programming techniques and agile was introduced.

Progressive developers wanted to integrate their code daily thus they needed to automate the build and run it on a server. Also, they wanted to make their build self testing, so they can make sure that the automated build is functionally correct. The build server was born.

It had two responsibilities: 
 - to build the project and generate an artifact in an automated fashion
 - and to verify this build

This clean cut responsibility of the build server only got more fuzzy over time as we added more and more functionality to our build pipelines. Making CI a timesink.

### But why does this added functionality make CI a timesink?

Because it requires CI to be a general purpose task engine integrated to every other system we have. We made CI the glue that it is today.

We codified our processes and culture in CI, and as every team has different processes, our CI jobs are different from team to team, from project to project. 

CI servers are powerful and flexible, but we also made them difficult to maintain as we overloaded them with various responsibilities.

### How are we overloading our CI pipelines exactly?

As a consultant I like to advocate for drone.io as the CI engine of choice - or more recently my Drone fork, called Woodpecker. And when I do that I'm often faced with the question if it has a plugin for X.

My answer is along the lines of "everything that can be scripted can be run on CI", plus Drone or Woodpecker has the most simple plugin architecture where you can literally pack your shell script in a Docker image and use it as a plugin with all benefits of declarativness in the pipeline.

And because it's so easy, we often extend CI with various functions.

### Examples of how we extend CI with various functions

One example for extending CI's responsibilities is deployment.

We have different deployment targets, we have different strategies for deployment and rollback, and we also have different environments. This often results in custom deploy and cleanup steps scattered into many steps in the pipeline.


Another example for latching on CI is notifications.

We typically notify Slack in various build events. Sometimes we notify task managers as well or external jobs that need to run once the CI job finished.

I have also seen secret management being stuffed in CI.

CI's have an okay process for managing secrets and we use this to manage secrets that are used in the pipeline. Sometimes we also use this facility to manage application secrets from the CI pipeline. If you think about it, CI has nothing to do with those secrets. But since CI has the facilities, and it  is often the only tool in our disposal where we can properly manage secrets, we manage application secrets from the CI pipeline.

These are usually not big issues, but sometimes it becomes a real problem.

### When is it becoming a problem?

It can become a problem when you move between projects. You can never be sure how the pipeline works exactly as even pipelines of the same team vary slightly between projects. Even if you built the pipeline yourself, after some time you forget the nuances you had to implement to make the build work.

Then when you face an issue, you have no other way to debug than to scan every line of the pipeline or print debug statements. Finding a misconfigured environment variable or a rouge `export MY_VAR=xxx` is often not trivial.

Pipelines with many responsibilities also becoming a problem when you try to extend one of the aspects of the build.

If you want a more advanced deployment logic, review app cleanup, rollbacks or blue-green deployments, the single step deploy that was glued at the end of the pipeline easily becomes five steps or more.

Or should you need advanced secret rotation logic like some of the secret vaults have today? I don't even know how to achieve that in a CI job.

These are growing pains in CI job maintenance.

### Where can we look for a solution?

We can look for a solution - or at least inspiration - in other projects or systems that have dealt with overlapping responsibilities before.

### Java build systems went through a similar situation in the mid 00s
Ant was the build system of the time. A procedural, flexible tool where you can iteratively enhance your script as your project grew.

At the time even the folder structure of Java projects was different in every project and a great deal of time was spent in moving files, setting buildpaths and so forth. And once you hit a build error, all you had was a line number where the Ant script failed, so you had to dig up the Ant script's source code and start debugging line by line. Sounds familiar, doesn't it?

Then came Maven with a lot of opinions, conventions and declarativeness. Its primary goal was to *"allow a developer to comprehend the complete state of a development effort in the shortest period of time"*. 

Maven had conventions and supported only a single folder structure and build steps. It argued that all Java projects have to build, test and install their artifacts. No variations, but conventions everywhere. It quickly took the Java space and it still has a large fraction of the market today.

### Netlify has to be a source of inspiration as well
Netlify is a CI system in essence for static or single page web applications. It's a Saas solution and it integrates vertically all things we need to do if we want to deploy our app: TLS, hosting, builds, everything. It's popularity inevitably comes from the fact that it does everything for you in its selected niche, but in essence it's a CI engine with the very same technical foundations as our general purpose CI engines.

What Netlify did well was to bring a bunch of opinions and good defaults so Javascript, Python and other projects build by default, no pipeline yaml was required.

In both examples, adding opinions and enforcing conventions simplified the configuration of the projects, and lessened the maintenance need over time.

### But hey! I like that CI is so simple and flexible.

I hear you, and this is why I also like CI. I like the iterative approach of how we build pipelines. It's dead simple to do and the feedback loop is short. We can have a good feeling about the steady improvements we make.

My point is that it comes with a cost: the things that we also loath in CI. It's often tedious, integrations break and debugging them is not a time well spent.

### We made CI this generic task engine that CI is today

It's important for me to repeat that *we* made CI this generic task engine that CI is today.

I admit that we don't have many alternatives today and CI poses itself a good enough place to put these functionalities. But the cost it comes with is worth talking about. We are giving up efficiency and innovation if we stick to the general purpose CI engines as we know them today.

Just as in the GPU vs CPU analogy I think we can achieve a more efficient state and division of work, if we find a proper place for the functionality that today we just latch on CI.

Onwards!

In a follow up article I'm going to write on how we can break up CI's role. In the meantime, you can follow me on Twitter and get notified when that article is out. I'm [@laszlocph](https://twitter.com/laszlocph){:target="\_blank"}.
