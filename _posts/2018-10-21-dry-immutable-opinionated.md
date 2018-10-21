---
title: "DRY, immutable, opinionated, agnostic"
tags: [dry, immutable, opinionated, agnostic, aws, future, iaas, coding, erlangen, bayern, germany]
---

### Prelude
As far as I know there are many ways to *create* today in IT.

What is becoming more difficult is doing it properly and taking the right decisions but (spoiler)... 

> But... I'm starting feeling that my repository is on the right direction to be *self.deployable* and agnostic. Above the infrastructure, which is provisioned by terragrunt and terraform, one or more actor(s) are placed (vedesi jenkins, but whoever it is), than the pipelines can be restored and the actors will be redeployed and start redeploy applications (even pieces of infrastructure with dependencies) on their behalf, to the various parts of the infrastructure (to be continued...)

Since I'm an IT guy, I continuosly reiterate that I do not know a lot about computer science. It's ok, I'm confortable with this and everyday I feel a little bit more heartened, because I'm not alone and in the end if you properly have defined process to deploy stuff, you can accomplish your job quite easily, even without being an hero. Thus, I would like to talk about three concepts appeared somehow behind the scenes (not so hidden), most probably existing since years, everybody of us (I guess) had already deal with :)

For everybody who simply want to jump to the related git repository I used to experiment with these concepts, the github repo is reachable here: [immutable](https://github.com/made2591/immutable).

#### Opinionated
Several universities make the claim that, when deciding where to put sidewalks, they first let students wear paths through the grass. This told them where to pave and ensured the best use of their walkways: like in the picture below.

![opinionated](https://i.imgur.com/euEnC9n.jpg)

Thus you can think of these well worn paths in cloud architecture as a procedure, or design, that gets repeated over and over again, to the point that it should just become a boilerplate. Rather than everyone composing the same 99% of code, we can generate that code, and focus on the 1% that is unique. Opinionated tooling is designed to guide you down a path that is considered a best practice. Additionally, since best practice is the default, the amount of unique code we maintain is dramatically reduced. Opinionated tooling doesn't eliminate options, however, it simply assumes some sensible defaults and relies on the user to understand when it makes sense to deviate from those defaults.

Instead of writing CloudFormation from scratch to build and orchestrate ECS, ECR, EC2, ELB, VPC, and IAM resources ourselves, we can start with a smart set of defaults, and just fill in a few blanks, customizing only the parts that we want changed, and let our opinionated tool generate the boilerplate CloudFormation. Actually, most of the libraries and framework used by software / platform / site engineers around the world are opinionated - and maybe sometimes we aren't even aware about it.

As far as I learned in the last months, the part of a platform should be for real opinionated is the most critical: the infrastructure part (mainly network) but also everything that is around. Let's say you have to create a simple web server, a security group to control the access, two separated stages that differ between each other only on environment settings (dimensions, HA, etc): there are some best practises you should follow to create this scenario. 

Some of them came out of the box from providers - think about access that is by default denied for obvious reasons - some others needs to be implemented: what you can bet on is that every of them should be opinionated. A configurable server provision process with enabled (re)deployment tools should impose all pre-commit automation happening via recursive makefiles, all provisioning done via ssh, and all deployments completed within a git receive-hook (which is setup during provision, along with the git repo). The fact is that many environments like the ones we are building / will build are in place today / were already built in the past.

Even if I didn't found a set of opinionated terraforms[^opin], I will collect them in [immutable](https://github.com/made2591/immutable) and start using default patterns to accomplish an opinionated state. For what concerns example, I think that [Mu](https://getmu.io/) is a good opinionated cloudformation-oriented framework to taste what does it mean *opinionated library*. Also, what I found interesting is [terraformation](https://github.com/dtan4/terraforming) and a few other tools to move from one language to another. Why? Talk about it later :-)

#### Immutable
The immutable concept is something coming from the functional programming paradigm: if you look on Internet about, the best definition you get of an immutable object (*unchangeable* object) is *an object whose state cannot be modified after it is created*. This is in contrast to a mutable object (changeable object), which can be modified after it is created. Why immutable? It's super simple: because everything that is immutable is *still working and will work forever* - (or not, but is irrelevant from a logical point view), by design, cause it was built and released, full stop - in 80s' we all would say compiled in a *binary*. Think about the old 80s' bank applications running around the world: they still work because they are *binary* objects - at least conceptually - running on mainframes and relying on TCP/IP. It's not a coincidence that binaries, mainframes and TCP/IP are still the only things working (I mean, for real) since 80s', the best gifts we received from our ancestors - that often are our olds collegues or parents, in this just born world made of cables :-)

![immutable](https://i.imgur.com/bIPCBmG.jpg)

The point is: the infrastructure should be immutable. As much as possible serverless, by making use of services - the real ones - like queues and identity management. The machines should be avoided in favour of containers, but when they are needed should be packered and traited like real containers.

Many times I heard the sentence "why we should create an ami for this? then we should mantain it" but honestly, what happens in the end is that you still have to maintain some machines, but without using a provisioner you lost the concept behind the immutability. The immutability is something that let you ignore your binary / object / describing state or whatever, since the exact moment you decided to release it until you will move to the next release, because it represents the best you were able to built until a fixed point in time.

This is similar to what I'm building in [immutable](https://github.com/made2591/immutable): the idea is that you define your design once, trying to decoupling as much as possible from your IasS provider - at least, conceptually - and be able to restore everything from scratch with a single pull. What you should be aware, is that to do this you have to not repeat yourself.

#### DRY
In software engineering, don't repeat yourself (DRY) is a principle of software development aimed at reducing repetition of software patterns, replacing it with abstractions or using data normalization to avoid redundancy. In IaaC, this implies defining resources, configuration, environment, provider and states once, separately, and avoid as much as possible writing twice what you can write once. This starting from the common environment variables shared across different setup to the resource definition that should use parameters, built-in functions and the maximum expressive power of the language you are using to avoid repetition. The code is clearer, more robust, more safe, and we all like formalism - hopefully. More specifically, the DRY principle is stated as "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system".

![dry](https://i.imgur.com/RbFzUWe.jpg)

The [infrastructure folder](https://github.com/made2591/immutable/tree/master/infrastructure) contains two folder inside, ```live``` and ```modules```. To build the infrastructure, I mainly made use (will made use of) of ansible, packer, terraform and terragrunt. The latter let my terraform code be as DRY-driven as possible: unfortunately, this is tricky to achieve especially because dialogue between modules is complicated and not always *plannable* without a run. To define the folder structure at infrastructure level, I created two folder (see the [official doc](https://www.gruntwork.io/) for more information): from high level perspective, the live folder contains the configurations patterns and values for two staging enviroment, called dev and prod. The modules folder my *.tf* files to setup the AWS infrastructure. Both the two folder are configured as much as possible to follow the DRY approach, mostly thanks to terragrunt: unfortunately, there are still some repetition I would like to remove as soon as I understand how :) 

But... I'm starting feeling that my repository is on the right direction to be *self.deployable* and agnostic. Above the infrastructure, which is provisioned by terragrunt and terraform, one or more actor(s) is placed (i.e. Jenkins, but whoever it is), the pipelines are re-established and the actors will be redeployed and start redeploy applications (even pieces of infrastructure with dependencies) on their behalf to the various parts of the infrastructure.

#### Agnostic
I started looking at Terraform as more than just a simple provisioner that allows you to safely and predictably create, change, and improve infrastructure.
From a purely practical point of view, Terraform is nothing but an open source tool that codifies APIs into declarative configuration files that can be shared among team members, treated as code, edited, reviewed, and versioned. So why one should decide to use terraform, given a cloud provider that provides its own language? Let's say the latter is AWS, you could directly use CloudFormation (beware! CloudFormation stays here for *any other language provided by your cloudprovider*, whatever the cloudprovider is). What do these two languages ​​share?

![agnostic](https://i.imgur.com/uhpBCNj.jpg)

They are both languages, with an expressive power. Let's say that the language provided by your cloud provider is based on a $$L_p$$ language and has an expressive power $$x$$. Terraform is powerful $$\leq x$$, because it relies on sdk so it could be *equal* or, at least, *as powerful as* $$L_p$$. What happens for real is that everyone sees it as *less powerful* and takes advantage of CloudFormation, because in the end it can not chase the expressive power of the language of the cloudprovider - still to verify how much greater this expressive power drive you to produce spaghetti, but this is another argument of discussion.
But, at least conceptually, if you act to pursue as much as possible decoupling between you and your IaaS, the more you describe your world with an agnostic dialet, the more tomorrow you will be able to easily move or start integrate with other environments, provided by some other providers. That's the reason why - imho - is better being friends of nobody that provider-addicted for laziness.

### Conclusion
I think this post comes first as a consequence of my new role as a Platform Engineer: all of us build, everyday, mainly because of the graffiti on the wall (*calling all creators*). I wanted to share this kind of - honestly, I really don't know how to call it - *vision* about my work I realized should be - hopefully - a good one to follow.

A still *only-partially* related git repository I used to experiment with these concepts is reachable here: [immutable](https://github.com/made2591/immutable).

Thank you everybody for reading!

[^opin]: But there are repository like [this](https://github.com/GovTechSG/terraform-aws-vpc) that provided opionated configurations to start from.
