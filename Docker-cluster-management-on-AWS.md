---
layout: default
---

## Docker cluster management on AWS

---
**tldr**
In this episode I reuse the well known Docker Compose files from part 2 and run the stack on AWS. I also explore production usage on AWS ECS.

---

By the end of [part 2](Mastering-test-environments-with-Docker) of the series I was able to run any set of services with any feature branch on a remote machine. The setup was quite pleasing to me since the local and test envirnments were identical and I could showcase the state of development easily for Product Management or QA.

I could reach that point relatively easily since I used only Docker's built in tooling. They were perfectly integrated and very powerful, thanks to Docker's *batteries included* approach.  


![Tailored environment](env.png)

I was also moaning a bit that I couldn't point my Docker Compose script to a cluster of Docker Engines in swarm mode, but I generally wasn't sure about the next steps.

* I could translate my stack described in compose files to the new *service* concept - introduced in Docker 1.12 - and explore [swarm mode](https://docs.docker.com/engine/swarm/)
* and later manage that cluster through [Rancher](http://rancher.com/).
* I could also try Amazon AWS's managed cluster solution: [ECS](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
* or dive into competing stacks like [Kubernetes](http://kubernetes.io), and later on experience with [Google Cloud's](https://cloud.google.com/container-engine/) managed options.

## ECS: Elastic Cointainer Service
Eventually I opted for ECS since there are already plenty of new concepts to juggle with. Managing my own cluster - while it is intellectually pleasing - seemed an overkill.

Especially as [AWS provides tooling to reuse my Docker Compose files](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html), and it took only a few changes to fit my work to ECS.

You can check those changes [here](https://github.com/laszlocph/multi-env/commit/2f2cd6b2069bbc355f63b5945cb979344e338315). Basically I had to drop the *depends_on* definition in favor of *links*, and extended the compose file with CPU and memory restrictions.

While running compose on ECS for QA environments was a breeze, I had to create AWS flavored service definitions anyway for my production stack. But more on that later.

## Launching a cluster

ECS has a very straightforward [documentation](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/create_cluster.html), and it's easy to create a cluster with the AWS dashboard based on that. 

I generally favor scripts and configuration management tools simply because it's easier to recreate and automate the work later on. 
For the sake of fewer moving parts I created my first cluster through the [AWS CLI](http://docs.aws.amazon.com/cli/latest/userguide/tutorial-ec2-ubuntu.html) and didn't use tools like Ansible, Cloudformation or Terraform.

<pre>
ecs-cli configure --region eu-west-1 --cluster cluster01   
ecs-cli up --keypair ecs --capability-iam --size 2 --instance-type t2.micro

</pre>

The above two commands create a complete ECS cluster with all the network infrastructure it requires. It uses a CloudFormation template in the background, where you can validate the actual complexity of the stack. 

![Network infrastructure](cloudformation.png)

## ECS services and tasks

ECS has two primitives to work with: *services* and *tasks*.

*Tasks* basically mirror the service definition section of the Docker Compose file. You define the used Docker image here, exposed ports, volumes and a [bunch of other things](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html).

*Services* represent a higher level of abstraction. You define what task to run, how many instances, what load balancer to use, and you set the auto scaling parameters here too. The ECS *service* makes sure that the preferred amount of containers run, it starts new containers if one dies, and you also perform rolling updates through the *service* definition by changing the referenced task. ECS takes care of the redeployment.

## Compose up

Once the cluster is running, I start up the environment defined in the docker-compose.yml file with the 

<pre>
ecs-cli compose service up
</pre>

command. 

It creates the required *task* definitions from the compose file and also creates a *service*. 

Similarly to Docker Compose, I can check the status, or stop the environment with the *ecs-cli compose service ps* and *ecs-cli compose service stop* commands. It also shows where can I access the exposed port.

<pre>
.../web-python  RUNNING  52.210.151.19:5000->5000/tcp  ...-multi-env:2
.../boot        RUNNING                                ...-multi-env:2
.../redis       RUNNING                                ...-multi-env:2
</pre>

At this point the service is accessible on the displayed IP, I only have to do a small modification in the AWS Security Group because I use a non standard port. And while I'm there, I make sure that the service is only accessible to me. That you can do by limiting the incoming traffic to your office IP, but I rather suggest to set up VPN  access for higher security.

Since the flow is identical to the flow in part 2, I simply modified the [start-env](https://github.com/laszlocph/multi-env/blob/ecs/start-env.sh) script to operate with the *ecs-cli* commands, and **I achieved the same flexibility using an ECS cluster**. 

Each time I run the script I get a new environment exposed on a random port, on one of the cluster node's IP.

Full disclosure: this is not full cluster transparency, but it is sufficient for QA environments. I will address this by Load Balancers in my production setup.

## Production setup

There are a few [notable limitations](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/application_architecture.html) which prevent me from running my production stack directly from compose files:

* *ecs-cli compose service up* puts all my components into one *task* definition, and all components in a single task are deployed together on the same cluster node.
* Putting all components in the same *task* means that I can only scale them together with the means of *services*. This is way too rigid for production deployment.
* A *task* can only take up to 10 components
* Referencing components by their name through *links* only work within one *task* definition. This I have to address with an extra layer of abstraction.

So essentially I'm facing the same problem as with using compose with swarm mode. **I have to architect my production layout differently than my local and QA environments.**

This may feel like giving up Docker's main promise, but I argue that the value is in the flexibility I achieved with the local/QA environments by handling the whole stack together. 

It's also time to mention that in those environments I generously overlooked the complex network layout what production setups have. 
Furthermore some elements of my stack may never be Dockerized for production use. Large scale SQL databases, or managed database services - like RDS or ElastiCache - may not benefit of the agility of containers. 

So the task ahead is to create the fine grained version of the *service* definitions, and solve the problem of *Service Discovery*.

## Service Discovery light

Service Discovery is a problem that became prominent recently in the highly volatile environment of immutable infrastructures. Since services come and go, scale up and down, one can never be sure on what IP address a service is accessible. Putting IP addresses into application configuration files is too rigid to handle the fast changes. The need arrised to turn things up side down. 

Service discovery tools like [Netflix Eureka](https://github.com/Netflix/eureka), [Consul](https://www.consul.io/) become very popular. Essentially every service registers its access information in these components, and the service clients obtain the connection information from these service registries.

In my example though I solve the problem by simply using AWS's Elastic Load Balancer (ELB) service. Especially as it is [now aware of ECS services](https://aws.amazon.com/about-aws/whats-new/2016/08/amazon-ec2-container-service-now-integrated-with-application-load-balancer-to-support-dynamic-ports-and-path-based-routing/). 
**Services automatically get registered and deregistered in ELB**, so I can be sure that by accessing the service on a given DNS name, my request is routed to one of the containers in an ECS *service*. 

This combined with the new **path based routing** in ELB, and the ability to run a load balancer on internal networks allows me to not introduce any complex Service Discovery component.

## Launching the production service

I had to provide a few parameters in order to create the load balancer. The subnets are the ones where the ECS cluster nodes are created, and I created a brand new security group for the load balancer. Later on the cluster nodes will only allow connections from the load balancer, while the load balancer takes the ingress traffic. 

<pre>
aws elbv2 create-load-balancer --name elb01 --subnets subnet-ae0f49ca subnet-573e4221 --security-groups sg-08e22e6e

aws elbv2 create-target-group --name target01 --protocol HTTP --port 80 --vpc-id vpc-3803615c

aws elbv2 create-listener --load-balancer-arn arn:aws:elasticloadbalancing:eu-west-1:782027979363:loadbalancer/app/elb01/355e0f7219b527e9 --protocol HTTP --port 80 --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:eu-west-1:782027979363:targetgroup/target01/ff172e9976a871c4
</pre>

You can find out more about the  *target group* and *listener* components in [this article](https://aws.amazon.com/blogs/compute/microservice-delivery-with-amazon-ecs-and-application-load-balancers/).

The last thing I had to prepare is the *task* and *service* definition files.
 
<pre> 
{
    "containerDefinitions": [
        {
            "cpu": 300,
            "essential": true,
            "image": "laszlocph/spring-boot-dummy:exposed",
            "memory": 300,
            "name": "boot",
            "portMappings": [
                {
                    "containerPort": 8080,
                    "hostPort": 8080
                }
            ]
        }
    ],
    "family": "boot",
    "volumes": []
} 
</pre>
 
<pre>
{
  "cluster": "cluster01",
  "serviceName": "boot",
  "taskDefinition": "boot:1",
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:eu-west-1:782027979363:targetgroup/target01/03f4ca3155c8db4a",
      "containerName": "boot",
      "containerPort": 8080
    }
  ],
  "desiredCount": 2,
  "clientToken": "11yzFkCnk6FD8QBtBIYN8GBSqpOYieIk",
  "role": "ecsServiceRole",
  "deploymentConfiguration": {
    "maximumPercent": 200,
    "minimumHealthyPercent": 50
  }
}  
</pre>
 
Both were based on a blank definition generated by 

<pre>
aws ecs register-task-definition --generate-cli-skeleton
aws ecs create-service --generate-cli-skeleton
</pre>

commands respectively, but I also had success using [this container transform utility](https://github.com/micahhausler/container-transform) that transforms Docker Compose files to ECS task definitions, or Kubernetes Pods, if that's what you need.

<pre>
aws ecs register-task-definition --cli-input-json file://boot-task.json
aws ecs create-service --cli-input-json file://boot-service.json
</pre>

After registering the task and service in ECS, the container instances get registered in the ELB Target Group and the service is available on load balancer's hostname.

![Healthy nodes in the cluster](nodes.png)


## Rolling out updates

Deploying a new version of the service requires to update the service definition, what only takes one command. Then the changes are shown in the ECS logs.

<pre>
aws ecs update-service --cluster cluster01 --service arn:aws:ecs:eu-west-1:782027979363:service/boot --task-definition boot:2
</pre>

![Event log](events.png)

Full disclosure: the update took many minutes, but most of the time was spent in draining connections from the ELB. The default five minutes can be changed to lower values to speed up deployment.

## More services
Since a single instance of the load balancer can handle many services - each mapped into different subpath or port - the only requirement to create a new service is to create the task and service definitions, and to create a new *target group* for each service that you reference in the service definition. Containers will be automatically registered to the load balancer.

## Next steps
It was a good exercise to create the first service with AWS CLI. Mapping all services by hand though call for a configuration management tool, where the version state of the service is stored.  As a next step I'm going to look into this problem, either using Ansible, Terraform or CloudFormation.  

Onwards!

---
I might be wrong, you know.. if you feel strongly about one or an other solution in this article, reach out at <a href="mailto:laszlo@laszlo.cloud">laszlo@laszlo.cloud</a> and let's have a chat! 

<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-84825803-1', 'auto');
  ga('send', 'pageview');

</script>
