---
layout: post
title: "?(DRY(KIS(afe)S)) => CF(ALB+TLS+SM);"
date: 2020-02-01
polly: https://madeddu.xyz/mp3/aws/alb-tls-termination.mp3
categories:
- coding
- aws
tags:
- coding
- aws
- guide
- alb
- ecs
- fargate
---

### Intro

If you work with AWS, you might be involved in building infrastructure to enable some of your customers (both internal and external) to use a particular service, or just to try one of the hundreds open-source application available on Github. Furthermore, most of the ML/AI tools are shipped in docker containers and the philosophy -> if it runs on docker, it runs everywhere has been spread up to the highest level of management (nice, but... sometimes dangerous 😅 ed. ) And it's pretty much true, buuuuut...

<div class="img_container"><img src="https://i.imgur.com/zhZXCcD.jpg" style="width: 100%; marker-top: -10px;"/></div>

### The Problem

I recently found myself in the following situation: I had to deploy and expose the umpteenth docker-pulled-app running as a service on AWS, for a PoC. The main problem I had to address was to enable HTTPS connection for this service, to keep the communication safe as much as possible. I also wanted to preserve the basic authentication exposed by the service. Furthermore, since it was a PoC, I didn't want to put in place a huge piece of infrastructure, neither maintaining it over time: at the same time, my personal goal is always to build-to-recycle, as much as possible, all of my YAMLs for (possible) future releases of a product. In a sense... I just tried to do my best as a DevOps.

#### The Enemy

The involved application to run is called `plot.ly`[^plot.ly]. With a simple docker container, we are able to collect everything we need to run our application. Let's write the required `Dockerfile`:

{{< highlight docker >}}
FROM python:3
WORKDIR /usr/src/app
COPY requirements.txt requirements.txt
RUN pip3 install --no-cache-dir -r requirements.txt
COPY app.py app.py 
CMD [ "python3", "./app.py" ]
{{< / highlight >}}

I tend to produce my own version of the `requirements.txt` file using a virtual environment (more [here](https://virtualenv.pypa.io/en/latest/)). To act like this, open a shell and simply run:

{{< highlight bash >}}
cd project-folder
virtualenv -p python3 .venv
source .venv/bin/activate
pip3 install plotly boto3 dash-auth
pip3 freeze
{{< / highlight >}}

Done! You have an updated version of `requirements.txt` to be used to build your container. Now, the tricky parts.

#### Its Weapon

If you have a quick look at the [official documentation](https://dash.plot.ly/authentication), you can see that authentication for dash apps is provided through a separate `dash-auth` package. `dash-auth` provides two methods of authentication: HTTP Basic Auth and Plotly OAuth. 

HTTP Basic Auth is one of the simplest forms of authentication on the web - OF COURSE, WE DON'T WANT. Also because as a dash developer, you hardcode a set of usernames and passwords in your code and send those usernames and passwords to your viewers. There are a few limitations to HTTP Basic Auth:

- Users can not log out of applications
- You are responsible for sending the usernames and passwords to your viewers over a secure channel
- Your viewers can not create their own account and cannot change their password
- You are responsible for safely storing the username and password pairs in your code.

The second option is passing through Plotly OAuth, which provides authentication through your online Plotly account or through your company's Plotly Enterprise server. As a Dash developer, this requires a paid Plotly subscription (€€€), and so on...

<div class="img_container"><img src="https://i.imgur.com/EoB6TLs.png" style="width: 100%; marker-top: -10px;"/></div>

### The Strategy

Ok at this point you should start seeing the good thing of this story: at least, we already have a working HTTP authentication. And of course, we don't wanna put in place nothing more than a working and safe form of auth. Then... how can we proceed?

#### The Formula

With a magic trick, we can solve our problem: have you noticed the title of the post? It's pretty weird, but the logic behind it is actually all contained in it. Let's take the weird formula:

`?(DRY(KIS(afe)S)) => CF(ALB+TLS+SM);`

- ?: How to 
- DRY: Don't Repeat Yourself
- KIS(afe)S: Keep It Simple/Safe, Stupid 
- =>: Answer
- CF: Write infrastructure code (`C` loud `F` ormation)
- ALB+TLS+SM: leverage Application Load Balancer, TLS termination, and Systems Manager parameter store.

Even if I'm not a security expert, I always try to respect some fundamentals. First of all, I wanted to marry the 2 (actually, 3) principles that *imho* should drive you in the path of building really immutable infrastructures: Don't Repeat Yourself[^dry], Keep It Simple, Stupid![^kiss] - in this scenario, I wanna put the focus of this `s` on Safeness - and of course, write reusable code. I spoke about some of this principle in the past[^myoldpost]: let's define the counterattack to implement a valid fast solution.

#### The Counterattack

The idea is pretty simple: use an `ALB` and leverage `TLS` termination to safely provide `SSL` connection from the user to `ALB`. Then, enable `HTTP` basic auth to provide a username and password authentication between ALB and the application. What I achieved is actually pretty simple and pretty much similar to this:

<div class="img_container"><img src="https://raw.githubusercontent.com/nathanpeck/aws-cloudformation-fargate/master/images/public-task-public-loadbalancer.png" style="width: 100%; marker-top: -10px;"/></div>

Let's comment the YAML code to reach this result: some of the things you can see here are actually not deployed by the template (I assumed you have VPC and Subnet already defined, and a bit more things). 

#### Minimum Requirements to run the game

To start working with the CF, you must first:

- Have a registered domain name, and a certificate inside AWS Certificate Manager;
- Have a VPC and a couple of Public Subnet defined inside it;
- Patience, as usual, if something goes wrong;

#### ALB + TLS + SM

You just define your parameters:

{{< highlight yml >}}
StackName:
  Type: String
  Description: A valid vpc identifier
VpcId:
  Type: AWS::EC2::VPC::Id
  Description: A valid vpc identifier
PublicSubnetOne:
  Type: AWS::EC2::Subnet::Id
  Description: Enter a valid (public) subnet identifier
PublicSubnetTwo:
  Type: AWS::EC2::Subnet::Id
  Description: Enter a valid (public) subnet identifier
ServiceName:
  Type: String
  Description: A name for the service running with ECS Fargate
DesiredCount:
  Type: Number
  Description: How many copies of the service task to run in parallel (augment to scale the number of request)
ContainerPort:
  Type: Number
  Description: What port number the application inside the docker container is binding to
ContainerCpu:
  Type: Number
  Default: 256
  Description: How much CPU to give the container. 1024 is 1 CPU
ContainerMemory:
  Type: Number
  Default: 512
  Description: How much memory in megabytes to give the container
CertificateArn:
  Type: String
  Description: Optional ARN of the certificate to associate with the load balancer to implement TLS Termination
Path:
  Type: String
  Description: A path on the public load balancer that this service should be connected to.
  Default: "*"
Priority:
  Type: Number
  Description: The priority for the routing rule added to the load balancer.
{{< / highlight >}}

These parameters will let us parametrize some of the values. Amazon ECS allows you to run and maintain a specified number of instances of a task definition simultaneously in an Amazon ECS cluster: this is called a `service` and will be the first resource we are gonna create. If any of your tasks should fail or stop for any reason, the Amazon ECS service scheduler launches another instance of your task definition to replace it and maintain the desired count of tasks in the service depending on the scheduling strategy used.

{{< highlight yml >}}
Service:
  Type: AWS::ECS::Service
  DependsOn: LoadBalancerRule
  Properties:
    ServiceName: !Ref 'ServiceName'
    Cluster: !Ref ECSCluster
    LaunchType: FARGATE
    DeploymentConfiguration:
      MaximumPercent: 200
      MinimumHealthyPercent: 75
    DesiredCount: !Ref 'DesiredCount'
    NetworkConfiguration:
      AwsvpcConfiguration:
        AssignPublicIp: ENABLED
        SecurityGroups:
          - !Ref 'FargateContainerSecurityGroup'
        Subnets:
          - !Ref 'PublicSubnetOne'
          - !Ref 'PublicSubnetTwo'
    TaskDefinition: !Ref 'TaskDefinition'
    LoadBalancers:
      - ContainerName: !Ref 'ServiceName'
        ContainerPort: !Ref 'ContainerPort'
        TargetGroupArn: !Ref 'TargetGroup'
{{< / highlight >}}

The `LoadBalancers` let define how this service is attached to the ALB: every load balancer needs a target group. This is used for keeping track of all the tasks, and what IP addresses/port numbers they have. It's just connected to the application load balancer (see later), or a network load balancer, so it can automatically distribute traffic across all the targets.

{{< highlight yml >}}
TargetGroup:
  Type: AWS::ElasticLoadBalancingV2::TargetGroup
  Properties:
    HealthCheckIntervalSeconds: 6
    HealthCheckPath: /health
    HealthCheckProtocol: HTTP
    HealthCheckTimeoutSeconds: 5
    HealthyThresholdCount: 2
    TargetType: ip
    Name: !Ref 'ServiceName'
    Port: !Ref 'ContainerPort'
    Protocol: HTTP
    UnhealthyThresholdCount: 2
    VpcId: !Ref 'VpcId'
{{< / highlight >}}

You should pay attention to point to a valid `/health` to let the ALB validate the sanity of your targets: this is something you can easily define in your `app.py` if you are running plot.ly with a specific route:

{{< highlight python >}}
@app.server.route("/health")
def ping():
    return "ok"
{{< / highlight >}}

After that, we can create a rule on the load balancer for routing traffic to the target group.

{{< highlight yml >}}
LoadBalancerRule:
  Type: AWS::ElasticLoadBalancingV2::ListenerRule
  Properties:
    Actions:
      - TargetGroupArn: !Ref 'TargetGroup'
        Type: 'forward'
    Conditions:
      - Field: path-pattern
        Values: [!Ref 'Path']
    ListenerArn: !Ref 'PublicLoadBalancerListener'
    Priority: !Ref 'Priority'
{{< / highlight >}}

To prepare your application to run on Amazon ECS, you create a task definition. The task definition is a text file, in JSON format, that describes one or more containers, up to a maximum of ten, that form your application. Task definitions specify various parameters for your application. Examples of task definition parameters are which containers to use, which launch type to use, which ports should be opened for your application, and what data volumes should be used with the containers in the task.

{{< highlight yml >}}
TaskDefinition:
  Type: AWS::ECS::TaskDefinition
  Properties:
    Family: !Ref 'ServiceName'
    Cpu: !Ref 'ContainerCpu'
    Memory: !Ref 'ContainerMemory'
    NetworkMode: awsvpc
    RequiresCompatibilities:
      - FARGATE
    ExecutionRoleArn: !Ref 'Role'
    TaskRoleArn: !Ref 'Role'
    ContainerDefinitions:
      - Name: !Ref 'ServiceName'
        Cpu: !Ref 'ContainerCpu'
        Memory: !Ref 'ContainerMemory'
        LogConfiguration:
          LogDriver: "awslogs"
          Options: {
            "awslogs-group": "/ecs/plotly",
            "awslogs-region": !Sub '${AWS::Region}',
            "awslogs-stream-prefix": "ecs"
          }
        Secrets:
          - 
            Name: "PLOTLY_USER"
            ValueFrom: <ARN_OF_YOUR_USERNAME>
          - 
            Name: "PLOTLY_PASS"
            ValueFrom: <ARN_OF_YOUR_PASSWORD>
        Environment:
          - 
            Name: "AWS_REGION"
            Value: "eu-west-1"
        Image:
          Fn::Join:
            - ""
            - - !Ref AWS::AccountId
              - ".dkr.ecr."
              - !Ref AWS::Region
              - ".amazonaws.com/"
              - !Ref ECRRepository
              - ":latest"
        PortMappings:
          - ContainerPort: !Ref 'ContainerPort'
{{< / highlight >}}

Following the [guideline](https://dash.plot.ly/authentication), it's sufficient adding something like this:

{{< highlight python >}}
...
PLOTLY_USER = os.getenv('PLOTLY_USER', None)
PLOTLY_PASS = os.getenv('PLOTLY_PASS', None)

auth = dash_auth.BasicAuth(
    app,
    { PLOTLY_USER: PLOTLY_PASS }
)
...
{{< / highlight >}}

to your `app.py` to enable dash-authentication. To avoid storing this value in the code, we can some environment variables that define the credentials needed to connecto to internal databases. This value are all passed as environment variables to the Docker container. Since some of these values are considered *secrets*, they are stored as SM Parameters on AWS. ECS can inject these secret automatically when the Task starts, as defined in the section `Secrets` in the `TaskDefinition` YAML.

The ECR repository to store the Docker container:

{{< highlight yml >}}
ECRRepository:
  Type: "AWS::ECR::Repository"
  Properties:
    RepositoryName: <DOCKER_COMPLIANT_URL_REGISTRY>
{{< / highlight >}}

The Role to be used as Execution Role and Task Role:

{{< highlight yml >}}
Role:
  Type: "AWS::IAM::Role"
  Properties:
    AssumeRolePolicyDocument:
      Version: "2012-10-17"
      Statement:
        - 
          Effect: "Allow"
          Principal: 
            Service: 
              - "ecs-tasks.amazonaws.com"
          Action: 
            - "sts:AssumeRole"
    Description: Let ECS Cluster to run Plotly Server task definition
    ManagedPolicyArns:
      # Refine this policy to be more restricted, unless you are working in a development environment :-)
      - arn:aws:iam::aws:policy/CloudWatchLogsFullAccess
      - arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
      - arn:aws:iam::aws:policy/AmazonSMReadOnlyAccess
{{< / highlight >}}

The ECS Cluster doesn't need any particular configuration.

{{< highlight yml >}}
ECSCluster:
  Type: AWS::ECS::Cluster
{{< / highlight >}}

A security group for the containers that will run thanks to Fargate.

{{< highlight yml >}}
FargateContainerSecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: Access to the Fargate containers
    VpcId: !Ref 'VpcId'
{{< / highlight >}}

The security group rule to allow network traffic from ALB to containers:

{{< highlight yml >}}
EcsSecurityGroupIngressFromPublicALB:
  Type: AWS::EC2::SecurityGroupIngress
  Properties:
    Description: Ingress from the public ALB
    GroupId: !Ref 'FargateContainerSecurityGroup'
    IpProtocol: -1
    SourceSecurityGroupId: !Ref 'PublicLoadBalancerSG'
{{< / highlight >}}

The security group rule to allow network traffic from other containers with the same security group: it shouldn't be required, but can be useful.

{{< highlight yml >}}
EcsSecurityGroupIngressFromSelf:
  Type: AWS::EC2::SecurityGroupIngress
  Properties:
    Description: Ingress from other containers in the same security group
    GroupId: !Ref 'FargateContainerSecurityGroup'
    IpProtocol: -1
    SourceSecurityGroupId: !Ref 'FargateContainerSecurityGroup'
{{< / highlight >}}

The security group rule for the application load balancer:

{{< highlight yml >}}
PublicLoadBalancerSG:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: Access to the public facing load balancer
    VpcId: !Ref 'VpcId'
    SecurityGroupIngress:
        # Allow access to ALB from anywhere on the internet
        - CidrIp: 0.0.0.0/0
          IpProtocol: -1
{{< / highlight >}}

A public-facing application load balancer, this is used for accepting traffic from the public internet and directing it to public-facing microservices:

{{< highlight yml >}}
PublicLoadBalancer:
  Type: AWS::ElasticLoadBalancingV2::LoadBalancer
  Properties:
    Scheme: internet-facing
    LoadBalancerAttributes:
    - Key: idle_timeout.timeout_seconds
      Value: '30'
    Subnets:
      # The load balancer is placed into the public subnets, so that traffic from the internet can reach the load balancer directly via the internet gateway
      - !Ref PublicSubnetOne
      - !Ref PublicSubnetTwo
    SecurityGroups: [!Ref 'PublicLoadBalancerSG']
{{< / highlight >}}

A dummy target group is used to setup the ALB to just drop traffic initially before any real service target groups have been added:

{{< highlight yml >}}
DummyTargetGroupPublic:
  Type: AWS::ElasticLoadBalancingV2::TargetGroup
  Properties:
    HealthCheckIntervalSeconds: 6
    HealthCheckPath: /
    HealthCheckProtocol: HTTP
    HealthCheckTimeoutSeconds: 5
    HealthyThresholdCount: 2
    Port: 80
    Protocol: HTTP
    UnhealthyThresholdCount: 2
    VpcId: !Ref 'VpcId'
{{< / highlight >}}

and finally.... the Application load balancer listener rule to enable HTTPS connection and TLS termination

{{< highlight yml >}}
PublicLoadBalancerListener:
  Type: AWS::ElasticLoadBalancingV2::Listener
  DependsOn:
    - PublicLoadBalancer
  Properties:
    DefaultActions:
      - TargetGroupArn: !Ref 'DummyTargetGroupPublic'
        Type: 'forward'
    LoadBalancerArn: !Ref 'PublicLoadBalancer'
    Port: 443
    Protocol: HTTPS
    Certificates:
      - CertificateArn: !Ref CertificateArn
{{< / highlight >}}

that, together with the `LoadBalancerRule` (of type `AWS::ElasticLoadBalancingV2::ListenerRule`), let us expose throught 443 in HTTPS and forward request to our service.

### Conclusion

This solution is far from being considered perfect, for many reasons: first of all, there are no restricted policy boundaries, neither it uses a more safe schema like private subnets to hide application even more (instead of leverage the only security groups ingress rule). Also, from a syntax point of view, the Subnet could be passed as a list of Subnets, and there are no autoscaling policies defined for the service. Furthermore, the basic authentication doesn't provide a good path to follow, since there are many others like Cognito integration in front of the ALB to do it properly. 

The solution is also far from being considered production-ready, but... at least you put your PoC in place by building the minimum required things you need. You can recycle this idea to actually safely PoC every other container ready application that already offers a basic authentication system!

Thank you for reading! If you like this post, please upvote it on HackerNews [here](https://news.ycombinator.com/submitted?id=made2591)!

[^myoldpost]: Stay in my blog and read more about [immutable things](https://madeddu.xyz/posts/dry-immutable-opinionated/)
[^dry]: Read more about [Don't Repeat Yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) Principle
[^kiss]: Read more about [Keep It Simple Stupid](https://en.wikipedia.org/wiki/KISS_principle) Principle
[^plot.ly]: More in the [official website](https://plot.ly/)