---
layout: post
title: "How to: create a fleet of container-based-go-lambda with one command"
date: 2020-12-12
polly: https://madeddu.xyz/mp3/aws/container-lambda-fleet.mp3
categories:
- coding
- golang
- aws
tags:
- coding
- aws
- lambda
- golang
- grafana
- sentiment
- analysis
---

### Introduction

As you might know, at the re:Invent AWS recently announced the capabilities of running Lambda by getting the code directly from a docker image you provide, as illustrated [here](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/). Moreover, they also announced the [Amazon ECR Public and Amazon ECR Public Gallery](https://aws.amazon.com/about-aws/whats-new/2020/12/announcing-amazon-ecr-public-and-amazon-ecr-public-gallery/) that you can reach at [https://gallery.ecr.aws/](https://gallery.ecr.aws/).

And this was pretty much my reaction:

<div class="img_container"><img src="https://media3.giphy.com/media/vV2F1yUqw7tzHmbx2K/giphy.gif" style="width: 100%; marker-top: -10px;"/></div>

### What you might need to know before begin

First of all: keep calm, because there's already a [Github Repo](https://github.com/made2591/container-lambda-fleet) to build an entire fleet of microservices - with their respective ECR-based repository - with only one command from the shell. Hopefully, you can get up and running by simply opening a shell, cloning my repo, change the AWS_PROFILE_NAME at the top of the `Makefile` to point to your profile, and run:

{{< highlight yaml >}}
make create-everything
{{< / highlight >}}

And This. Is. It.

<div class="img_container"><img src="https://media1.giphy.com/media/n1FGE3eMGbDqloZQC0/giphy.gif" style="width: 100%; marker-top: -10px;"/></div>

Just kidding

### Just kidding.

Is not that easy: you have to change your `AWS_PROFILE` name inside the `Makefile` but the repository is intended to be a good boilerplate to start with new Lambda in a container without going crazy. If you want to know more, go ahead with my article and read more about what you might need to do before start with container Lambda!

The first thing to do is update your `sam cli` because a new option is required to deploy Lambda based on container: `--image-repository`. Your actual version of `sam` most probably will NOT support this option yet, at least if you haven't updated it recently.

So, update your sam cli -> this strictly depends on the method you used to install it in the beginning in your system.

### A template to rule them all

The first thing I tried to do is to have a single template file to both create an ECR repository and deploy a fleet of Lambda. Excluding some tricks you cannot do locally - like having a machine that builds your image - you cannot do this and the reason is simple.

> The docker-image build step is necessarily separated from the package of the Lambda function since the packaging requires the Lambda code to be ready - in this case, pushed.

Thus:

> The `docker push` to ECR must be completed before the Lambda packaging begins, and this requires an ECR repository already deployed and *ready to serve* an image before the `sam deploy` command.

Actually, even before the `sam packaging`. To operate all these steps together, I create a `Makefile` with instructions that let you implement the steps (and even more) described [here](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/) by AWS. By using the `make` command, you will be able to manage both the registry and the fleet of microservices you want to deploy, create them, update them, delete them and even run separately both the build of the single microservices and the deployment of the entire stacks :) 

### Make a fleet of repositories

My idea was to create a single template file for the registry (`repositories.yaml`) and a single template file for the fleet (`fleet.yaml`): you just need to replicate the folder structure inside the repository to add a new microservice and extend the two template files and the `Makefile` to include the docker commands required to build each microservice. The folder structure is pretty much this:

{{< highlight yaml >}}
./container-lambda-fleet
    # this folder contains the microservice files and Dockerfiles
    fleet/
        a_microservice_sample
        another_microservice
        ...
    # this SAM template deploy the functions inside Lambda
    fleet.yaml
    # this Cloudformation template deploy the function repositories inside ECR
    repositories.yaml
    # this pack build package and deploy command for you
    Makefile
{{< / highlight >}}

The `repositories.yaml` is the template in charge of creating the infrastructure to host the microservices repositories inside the ECR registry, and it's located in the root folder of the Github repository. Inside the `Makefile`, there's a simple rule called `container-lambda-fleet-registry-create` that run the command:

{{< highlight yaml >}}
aws cloudformation create-stack --stack-name $(REGISTRY_STACK_NAME) --template-body file://repositories.yaml --capabilities CAPABILITY_AUTO_EXPAND
{{< / highlight >}}

The template just accepts `AMicroserviceSampleRepositoryName` - that is the name we wanna give to our sample microservice - and many more if you want to add them. In fact, there are already some commented placeholder lines to let you extend your repositories - your fleet, accordingly. I won't comment on the template file because it's really as simple as creating a single `AWS::ECR::Repository` resource, you can have a look inside the repository.

### Make a fleet of microservices

The template `fleet.yaml` create a fleet of microservices (Lambda function) based on the new `ImageUri` parameter: the URI has exported automatically from the stack deployed by `repositories.yaml`, in order, for each microservice, and it's named using the naming convention `<stackname>-<microservicename>`

{{< highlight yaml >}}
  # A sample microservice Lambda container-based function: you should replicate for each microservice part of the fleet
  AMicroserviceSample:
    Type: AWS::Serverless::Function
    Properties:
      PackageType: Image
      ImageUri: 
        Fn::ImportValue:
          !Sub "${ECRStackName}-a-microservice-sample"
    Metadata:
      DockerTag: go1.x-v1
      DockerContext: ./a-microservice-sample
      Dockerfile: Dockerfile
{{< / highlight >}}

In this template there are some commented placeholder lines as well, to let you extend your microservices of your fleet. Every service you add must be adapted using the respective `DockerContext` and the respective `Uri` - the name we wanna give to our sample microservice. This means you must first update your `repositories` stack before going for a new service deployment since you need to first create a repository and push your image. 

Inside the `Makefile`, there's a simple rule called `fleet-template-package` that run the command:

{{< highlight yaml >}}
sam package --template-file fleet.yaml --image-repository $(shell aws cloudformation describe-stacks --stack-name $(REGISTRY_STACK_NAME) | jq '.Stacks[0].Outputs[1].OutputValue') --output-template-file fleet-packaged.yaml
{{< / highlight >}}

But... this command will not work immediately if you run it just after creating the registry because

### Building the docker image

You first need to build and push your docker image. To complete these steps, there are two rules: the first one is called `a-microservice-sample-image-build` and the second one is called `fleet-image-push`. The first one can be adapted to build and tags every microservice you add to your fleet (once again, there are some placeholder lines inside the `Makefile` to - ok, you got it), the second to push your image, after ECR login.

There's also a command to build, push, package, and deploy your fleet from zero to hero and it's called `fleet-template-create`:

{{< highlight yaml >}}
fleet-template-create: a-microservice-sample-image-build fleet-image-push fleet-template-package fleet-template-deploy
{{< / highlight >}}

### Conclusion

There are a lot of commands in the middle to delete or update your stack, so there's only one more thing to say: just clone and start coding lambda-container-based in Golang on AWS. Again, you can find the whole code of my experiments in [this Github Repo](https://github.com/made2591/container-lambda-fleet) :)

If you want to explore the stack and leave a comment, feel free! 

Thank you everybody for reading!
