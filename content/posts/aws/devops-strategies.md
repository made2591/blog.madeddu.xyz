---
layout: post
title: "Devops strategies"
date: 2019-12-08
categories:
- devops
- life
- miscellaneous
tags:
- devops
- life
- miscellaneous
---

### Intro

As you might know, I use to work with AWS doing and destroying stuff like many other devopssss out there: this post is all about some challenges and problems I had to deal *recently* - in the last months - and involves some notes about how to deal with multistage environments, decoupling and many other practices you should all know about.

The first thing to emphasize is that each of these best practices is *fairly clear* and *well-defined*, for sure online (perhaps discussed by bloggers more experienced than me XD) and deeply described in many books. The problem - at least, in my experience - is not about understanding these best practices, sharing them within a team and starting to use them as a group. The huge problem is actually the one of verifying and *scaffold a behavioral pattern* to put them in place, by *definition of standards within a context that did not use them previously*.

Any devops, at some point, is forced to work following this direction, which means the handling of duplicated resources and gradually adopted standards. Moreover, all of this is all about consider the impacts the new operating model can have on the development cycle and the evolution of the infrastructure he/she's responsible for.

<div class="img_container"><img src="https://i.imgur.com/4Chy2mo.png"/></div>

### The problem

I want to first highlight the problem we wanna address as devops:

> Verify and **scaffold behavioral patterns** to put best practices in place, using **opinionated standards within a context that did not use them previously**, with the final goal of creating **immutable**, **DRY-driven**, rock-solid **environments** for both development and production purposes, with **decoupled components that acts independently**.

This sentence contains many concepts not necessarily - my bad, not strictly, at least - related to devops world. Before going ahead with some of the technical advice I collected over the past months, let's first focus on some of the concepts mentioned in the sentence above.

### Glossary

The keywords and sentences are partially highlighted already in the problem definition. I wanna focus on some of them:

    - Verify;
    - Scaffold behavioral patterns;
    - Opinionated standards;
    - Immutable environments;
    - DRY-driven approach;
    - Decoupled components;

I will speak about each of these point in the next paragraphs.

#### Verify

What I learned from past working experience can be summarized in a single statement:

> Nobody is right and everyone is wrong, but everyone is right and nobody is wrong.

Meaning,

> You cannot trust **words**: best practices evolve during time, so verify what you are putting in place with other people.

That is more a suggestion to say that you should validate everything you think, even the more used pattern because applying a pattern before having it validated by many parts tends to be *risky*, more than you expected. Of course, no one want you losing time in doing things many times but first think, then act. That is a good mantra, but I would like to highlight the pretty important step in the middle everyone uses to forget: share.

> First think, then share, finally act.

<div class="img_container"><img src="https://i.imgur.com/r8xoJsS.jpg"/></div>

The step of sharing, in my opinion, consists of write clear procedures and documents to better schedule changes impacting critical operations. This step requires a lot of time and can only be achieved through serious brainstorming, so the more people you get involved in difficult decisions, the more you augment your chances to get valuable feedback about what you're trying to achieve - meaning: make sense? - and how you're trying to achieve it.

#### Scaffold behavioral patterns

A good devops *tries* a procedure, a pipeline, a process, a pattern. Like any other software engineering task, in the very first place - just after you defined what you wanna achieve - you really don't care about the details. You just want to complete all the spikes you need to *link the points* and say "Ehy dude! This shit can work!". This step **must keep evidence of behavioral patterns** to be adopted by third parties (our best friends, the developers). In fact, it doesn't make any sense to put in place a *pull request strategy*, if no ones pro-actively try and accept only evaluated changes. Maybe you just wanna first introduce a concept of code immutability, moving forward with a solid branching strategy, and then build on top of it.

As [Morcheeba](https://www.youtube.com/watch?v=FLGJXbl6g8o) used to say in the far 2000 (the hell how old I am 😭😭😭):

> Don't you know that Rome Wasn't Built in a Day.

Meaning,

> Don't try to make it **fast**: make it **easy**, then iterate over it and move on with some new behavioral pattern.

<div class="img_container"><img src="https://i.imgur.com/jip9dHH.jpg"/></div>

You will have more followers then any other influencer out there 😜!

Back to the branching strategy, I experienced a good branching strategy by detaching feature branch only and always from a stable point: that means, detach from master branch, and then merge to dev, iterate, merge on test, and finally bring it back to master, separately. This is quite easy to put in place, and simple to adopt across many developers. Furthermore, everyone becomes more responsible of developed code, and not constrained by other development - **assuming is not working with dependencies part of another feature-branch development**. So, another suggestion is:

> Keep features development as much as possible independent.

If you can't develop a feature together with another feature:

> **Don't hesitate serialize things: well done and late is better than quick and wrong.**

Everything you actively made wrong, has to been review in the future. No exceptions thus... make your choices :-)

#### Opinionated

Several universities make the claim that, when deciding where to put sidewalks, they first let students wear paths through the grass. This told them where to pave and ensured the best use of their walkways: like in the picture below.

<div class="img_container"><img src="https://i.imgur.com/2gd1GU3.jpg"/></div>

Thus you can think of these well worn paths in cloud architecture as a procedure, or design, that gets repeated over and over again, to the point that it should just become a boilerplate. Rather than everyone composing the same 99% of code, we can generate that code, and focus on the 1% that is unique. Opinionated tooling is designed to guide you down a path that is considered a best practice. Additionally, since best practice is the default, the amount of unique code we maintain is dramatically reduced. Opinionated tooling doesn't eliminate options, however, it simply assumes some sensible defaults and relies on the user to understand when it makes sense to deviate from those defaults.

For instance, in the world of AWS, the one I work more with, instead of writing CloudFormation from scratch to build and orchestrate ECS, ECR, EC2, ELB, VPC, and IAM resources ourselves, we can start with a smart set of defaults, and just fill in a few blanks, customizing only the parts that we want changed, and let our opinionated tool generate the boilerplate CloudFormation. Actually, most of the libraries and framework used by software / platform / site engineers around the world are opinionated - and maybe sometimes we aren't even aware about it.

A good put to build immutable thing is use tool like [AWS CDK](https://github.com/aws/aws-cdk) by AWS.

{{< highlight js >}}
export class MyEcsConstructStack extends core.Stack {
  constructor(scope: core.App, id: string, props?: core.StackProps) {
    super(scope, id, props);

    const vpc = new ec2.Vpc(this, "MyVpc", {
      maxAzs: 3 // Default is all AZs in region
    });

    const cluster = new ecs.Cluster(this, "MyCluster", {
      vpc: vpc
    });

    // Create a load-balanced Fargate service and make it public
    new ecs_patterns.ApplicationLoadBalancedFargateService(this, "MyFargateService", {
      cluster: cluster, // Required
      cpu: 512, // Default is 256
      desiredCount: 6, // Default is 1
      taskImageOptions: { image: ecs.ContainerImage.fromRegistry("amazon/amazon-ecs-sample") },
      memoryLimitMiB: 2048, // Default is 512
      publicLoadBalancer: true // Default is false
    });
  }
}
{{< / highlight >}}

This class produces an AWS CloudFormation template of more than 500 lines, and the deploy of it produces more than 50 resources. This template can scaffold an opinionated VPC without have to handle difficult Task Definition, LoadBalancer, TargetGroup, etc.
Other advantages of using opinionated library is that you can easily abstract to another language layer, moving from declarative to imperative approach. This includes the change to use logic (if statements, for-loops, etc) when defining your infrastructure, use object-oriented techniques to create a model of your system, define high level abstractions, share them, and publish them to your team, company, or community, and last but not least share and reuse your infrastructure as a library

#### Immutable environments

The immutable concept is something coming from the functional programming paradigm: if you look on Internet about, the best definition you get of an immutable object (*unchangeable* object) is the following:

> An object whose state cannot be modified after it is created.

This is in contrast to a mutable object (changeable object), which can be modified after it is created. Why immutable? Because everything that is immutable is *still working and will work forever* - (or not, but is irrelevant from a logical point view), by design, cause it was built and released, full stop - in 80s' we all would say compiled in a *binary*.

Think about the old 80s' bank applications running around the world: they still work because they are *binary* objects - at least conceptually - running on mainframes and maybe relying on the only TCP/IP. It's not a coincidence that binaries, mainframes and TCP/IP are still the only things working (I mean, for real) since 80s', the best gifts we received from our ancestors.

<div class="img_container"><img src="https://i.imgur.com/j3wTGqn.jpg"/></div>

The infrastructure should be immutable, but honestly... everything you ship related to your application should be *immutable*. The reason is simple:

> If a piece of your application is completly immutable, this means you own that piece of your application.

Thus, you can rebuild it, because you know the deploy process, and you can automate it - why not, across multiple environments. And go DRY is kind of related to this topic.

#### DRY

In software engineering, Don't Repeat Yourself (DRY) is a principle of software development aimed at reducing repetition of software patterns, replacing it with abstractions or using data normalization to avoid redundancy[^wiki]. In IaaC, this implies defining resources, configuration, environment, provider and states once, separately, and avoid as much as possible writing twice what you can write once. This starting from the common environment variables shared across different setup to the resource definition that should use parameters, built-in functions and the maximum expressive power of the language you are using to avoid repetition. The code is clearer, more robust, more safe, and we all like formalism - hopefully.

More specifically, the DRY principle is stated in one sentence, that is:

> Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.

<div class="img_container"><img src="https://i.imgur.com/WPObHIq.jpg"/></div>

The more you apply the DRY principle, the more you spend time in understanding how to propagate and abstract your application from the environment used: configurations, parameters, defaults. Everything that can be customized, including names of resources if you don't have physically separated context: for instance, inside Cloudformation, this can be done easily by using `Parameters` and `Mapping` accordingly.

{{< highlight yml >}}

Parameters
  BranchName:
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - test
      - master
    Description: Branch name

Mappings:
  BranchToStage:
    dev:
      Name: Development
    test:
      Name: Testing
    master:
      Name: Production

!FindInMap: [ BranchToStage, !Ref BranchName, Name ]

{{< / highlight >}}

The `BranchName` can be easily injected from a CI/CD solution using direct branch name from the build template definition. This is just an example of how to process and remove repetition.

What about decoupling? Let's start with some examples to introduce the problem: unfortunately, the example is strictly related to AWS world, but the same concepts are valid for other scenarios I guess.

### Decouple components

If you run one or more serverless stack in AWS, one of the most common scenario you can deal with is a simple microservice running inside a Lambda function, exposed through API Gateway. This scenario just works and it provides a really simple way to scale up to millions of requests without changing pretty much anything inside your code. This is the biggest pro (or cons) of managed services: you just delegate the ownership and responsibility of infrastructure behind your app to someone else, and party rock. Easy peasy.

However, after a while, this microservice starts becoming too big - I'm thinking about execution time to complete a single request - to handle the whole operations it has to complete. This could happen for many reasons, the first one is the **wrong and bad software engineering decisions taken behind the development of the microservice itself**.

<div class="img_container"><img src="https://i.imgur.com/342Md7E.png"/></div>

Even if out of topic, I would like to notice that **going REST doesn't mean ONLY going resources-oriented**, and once you go REST, you should follow some basic idea behind the network communication that support your service. I would like to highlight some crucial points:

- Paginate or, at least, put limit behind your operation;
- Delegate processing time to clients, because the attitude of running out of time it's really annoying for end-users;
- If you find yourself stuck around the scenario of handling difficult and expensive operations, at least go async;

Back to the devops room, we can only focus on the last of the points, because the first two points involve *a bigger part of development*. So, a good iteration to solve the problem is to delegate the execution to another Lambda function. This is pretty easy to implement, until you don't have to understand some logic you actually don't want to understand. I recently found a good pattern, especially if you already adopt *subscription pattern somewhere in your application* to handle real-time communication. To avoid *reverse engineering* in the backend, you can follow the pattern in the picture, described just after.

<div class="img_container"><img src="https://i.imgur.com/Lh8hDZ5.png"/></div>

In order:

- Create a clone of the Lambda that handles your microservice - let call the cloned Lambda `B`, and the original one `A`;
- Create an S3 bucket to store artifact;
- Create an SQS queue;
- Every time a request arrives to `A`, just dump a serialized version of the event of `S3`: if your API Gateway is attached to a Custom Domain Name, you can use as a key to store your event-object a unique identifier of the request simply looking inside the request Headers. After that, publish a message in the SQS Queue with the key used to store the event, and provide an answer to the user;
- The user now subscribe to a predefined place to look at the status of the job: this place is updated as well by the Lambda `B` just before finishing the job;

### Conclusion

All of us build, everyday. I wanted to share this kind of - honestly, I really don't know how to call it - *vision* about my work. I realized should be - hopefully - a good one to follow.

[^opin]: But there are repository like [this](https://github.com/GovTechSG/terraform-aws-vpc) that provided opionated configurations to start from.
[^wiki]: [Wikipedia](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)