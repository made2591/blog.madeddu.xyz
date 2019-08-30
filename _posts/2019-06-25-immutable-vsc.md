---
layout: post
title: "I run VSC in the browser and I am just fine - Part I"
categories: [coding, aws, immutable]
tags: [coding, aws, cloudformation, guide, route53, vsc, cfn-init]
---

### Introduction
Serverless and managed *things* are the best choices if you don't want to deal with infrastructure (3 2 1: fight) buuuuut...even immutable things are not so bad for this purpose - at least, if they are immutable for real 🤣 Today I wanna talk about a useful way to run an instance(s) of [VSC](https://github.com/microsoft/vscode) server in AWS and code from everywhere (yes, even your iPad): let's start!

<div class="img_container"><img src="https://i.imgur.com/dgm4ksi.png" style="width: 100%; marker-top: -10px;"/></div>

This time I will go native: so no CDK, I'm sorry, but pure Cloudformation instead. If you are not interested in all the astonishing things I have to say, you can find the template [here](https://raw.githubusercontent.com/made2591/vcs-server/master/template.yml).

### Requirements
Ok, before going ahead there are a few things you should have:

- An [AWS account](http://aws.amazon.com)
- A domain
- A certificate
- Docker
- A keypair

The key pair is important: since everything is wrong, starting me, you could need to access your instance to better understand my mistakes. In other words, don't trust me and, please, prepare a keypair to attach to the instance and let you access it.

For what concerns the domain/certificate part, you can even ignore it if you are not interested in having a custom domain and HTTPS - thus, if you want to just run VSC as it is. Keep in my mind you will have to modify the Cloudformation template accordingly to have it working properly.

### Architecture
The architecture I have in mind for this use case is pretty simple:

<div class="img_container"><img src="https://i.imgur.com/2cVDX3F.png" style="width: 100%; marker-top: -10px;"/></div>

I don't wanna discuss the Slack/Lambda action invoke part: there are plenty of guides to help you realize that part. Instead, I will focus this article and the second part of this series on the central core part: the Cloudformation template to provision the VSC server instance as an immutable resource you can create and destroy by the end of the day. The notification part is included only as a concept idea to keep the stack creation process asynchronous and let you write code without worry about anything in a safe enough environment (at least, I hope 😂).

#### If you own a domain...
You would like to use it to expose your immutable IDE(s) and use HTTPS. You can easily register a domain by following the guidelines here [AWS Route 53](https://aws.amazon.com/getting-started/tutorials/get-a-domain/)[^domain]. After that, you can use the open CA [Let's Encrypt](https://letsencrypt.org/) to get a valid certificate for your domain. The steps to produce the certificates can be summarized in a few steps thanks to Docker!

In the case you use direct credentials to access your AWS account, then you can run the following:

{% highlight sh %}
docker run -it --rm -e AWS_ACCESS_KEY_ID="<YOUR_ACCESS_KEY_ID>" -e AWS_SECRET_ACCESS_KEY="<YOUR_SECRET_ACCESS_KEY>" --entrypoint /bin/sh certbot/dns-route53
{% endhighlight %}

I don't know which permissions you need exactly - most probably route53 *CRUD* is enough - but the container it's safe and listed as the first option in the list of the [ACME v2 Compatible Clients](https://letsencrypt.org/docs/client-options/) in Let's Encrypt website.

If you don't trust containers and Docker, or just in the case you have an *admin* role to assume to operate in your account, then run the following

{% highlight sh %}
docker run -it --rm $(aws --profile YOUR_PROFILE_TO_RUN_STS sts assume-role --role-arn YOUR_ARN_ROLE_TO_ASSUME --role-session-name "certbot" | jq '" -e AWS_ACCESS_KEY_ID=" + .Credentials.AccessKeyId + " -e AWS_SECRET_ACCESS_KEY=" + .Credentials.SecretAccessKey + " -e AWS_SESSION_TOKEN=" + .Credentials.SessionToken' | sed s/\"//g) --entrypoint /bin/sh certbot/dns-route53
{% endhighlight %}

Thanks to `jq`, this will propagate your temporary credentials inside the certbot container as environment variables: in both cases, you will enter the container and you will be ok in running the following command.

{% highlight sh %}
certbot certonly \
  -d YOUR_DOMAIN \
  -d *.YOUR_DOMAIN \ # see later
  --dns-route53 \
  -m YOUR_EMAIL \
  --agree-tos \
  --non-interactive \
  --server https://acme-v02.api.letsencrypt.org/directory
{% endhighlight %}

This will generate the certificates inside the container, under the path `/etc/letsencrypt/archive/`. For your interest, given the domain `mydomain.com`, you can detach how many records you want. I decided to have this entire stack running under the third level domain `code.mydomain.com`, and each of the instance(s) running under `[a-z].code.mydomain.com` (I will talk about this later). This is the reason you need the `*.YOUR_DOMAIN`. In a single instance, the setup could be not required.

You can copy the certificates and bring them outside of the container by running something like this:

{% highlight sh %}
docker cp YOUR_CERTBOT_CONTAINER_ID:/etc/letsencrypt/archive/FOLDER_OF_CERTS ./
{% endhighlight %}

Now, two of these parameters will be required by codeserver to run on HTTPS: they will be named `cert1.pem` and `private1.pem`. Since I will use `cfn-init` (more info [here](https://docs.aws.amazon.com/AWSCloudformation/latest/UserGuide/aws-resource-init.html)) to setup everything, there's a good way to safely store these files and it's, of course, the [AWS Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html). In fact, by using the configuration section `files`, the `cfn-init helper` script processes are able to retrieve values from SSM automatically. Thus, store the content of these two files as two separate parameters in AWS Parameter Store with simple string as value - I wasn't able to retrieve them if stored as a *secure string*, even if it should be available as stated by [this announcement](https://aws.amazon.com/about-aws/whats-new/2018/08/aws-Cloudformation-introduces-dynamic-references-to-support-aws-/). You can easily store the content of the files with the `awscli` by running:

{% highlight sh %}
aws ssm put-parameter \
    --name "cert.code.mydomain.com" \
    --type "String" \
    --value "$(cat /path/to/your/cert1.pem)" \
    --overwrite
{% endhighlight %}

and for the private key respectively

{% highlight sh %}
aws ssm put-parameter \
    --name "key.code.mydomain.com" \
    --type "String" \
    --value "$(cat /path/to/your/private1.pem)" \
    --overwrite
{% endhighlight %}

Thus, you don't need to see the content of these files anymore - until the certificate will expire XD. I'm already considering to address this problem with automation by storing these values inside Secret Store and enable automatic rotation. I didn't focus a lot on this topic, but feel free to try it and let me know!

Let's finally start with an overview of the setup.

### Overview
The idea is pretty simple: the guys from [coder.com](https://coder.com) have done a great job in packaging the Visual Studio Code to run it as a simple old fashion *binary* - but even a modern docker container. As you can imagine, it's pretty simple to configure the application and let it run in the cloud... this time, I discarded the idea of having it running in a container - at least, if you wanna run a single instance. So the idea is pretty simple: you have an EC2 instance, you decide the family type and a few other options. This instance doesn't have anything else, because it's intended to be as much as possible immutable as the VSC server binary: it just starts and exposes the service. When everything it's up, you can run a ```git pull``` and start coding from your browser!

#### Parameters
Let's analyze the parameters of the Cloudformation you can find [here](https://github.com/made2591/vcs-server/blob/master/template.yml):

| Parameter Name | Description |
|----------------|-------------|
| **VpcId** | Requires you to specify a valid VPC identifier. If you don't have one created by you, the default will be fine. Just go in the console and retrieve the ID - or choose it from the menu in the Cloudformation console. |
| **SubnetId** | Requires you to specify a valid Subnet identifier, with the subnet belonging of course to the VPC specified before. If you don't have one created by you, one of the default public subnets will be fine. Just go in the console and retrieve the ID - or, again, choose it from the menu in the Cloudformation console. |
| **ExposedPort** | It's a number defining the port to use to expose the application. The default one is `8443`, I replaced with `443`. |
| **SourceIP** | Lets you specify which is the IP or the IP pool you want to allow in the security group. By default a `"0.0.0.0/0"` will let you reach the instance once running from everywhere: this could be really dangerous thus **it's strongly suggested to provide restriction over this**. |
| **CertificateParameterName** | If you own a domain in AWS and you generated the certificate by following the guideline before, you would like to use the domain to expose your immutable IDE and also use SSL for communication. This Cloudformation parameter contains the name of the AWS parameter that contains the certificate - you have to create the AWS parameter before the creation of the stack (in the example above, this value would be `cert.code.mydomain.com`. |
| **PrivateKeyParameterName** | See *CertificateParameterName*. |
| **InstanceFamily** | Enter the family type of your instance (like t3.small) Consider that too small instances will run in failure more frequently. |
| **RootVolumeDimension** | Enter the dimension of the root volume. In the beginning, I thought to attach a secondary EBS volume as a storage layer, but then I changed my mind because this solution is intended to be as much as possible immutable. So just save your work or push it before deleting the stack, or feel free to add EBS volume handling. |
| **KeyPairName** | Enter the name of the key pair you want attached to the instance: you need to create this key before the creation of the stack, and it's for debugging purpose or if you want to have access to the instance too (it truly depends on your setup). |
| **DomainName** | Enter the domain to expose the service (if you read the certificate part above, the `code.mydomain.com`). |
| **AppPassword** | This is the password will be required to access the instance. |

#### cfn-init
The first thing to notice is: cfn-init doesn't run by itself. It has to be called ([line 120](https://github.com/made2591/vcs-server/blob/0d8bd43c24ea22be12d05466bad8075ef69c134d/template.yml#L120)) and this is done by using the `UserData` section:

{% highlight yml %}
Fn::Base64:
  Fn::Join:
    - ""
    - - "#!/bin/bash -xe\n"
      - "/opt/aws/bin/cfn-init -v "
      - "         --stack "
      - Ref: AWS::StackName
      - "         --resource CodeServerPublicInstance"
      - "         --configsets end-to-end"
      - "         --region "
      - Ref: AWS::Region
      - "\n"
{% endhighlight %}

Then, the metadata will be parsed: the [Metadata](https://github.com/made2591/vcs-server/blob/0d8bd43c24ea22be12d05466bad8075ef69c134d/template.yml#L133) section contains both the `configSets` definition and the steps inside every config step. The `Install` step just uses the `package` definition to install `Docker`, `wget` and `gzip`.

{% highlight yml %}
Metadata:
  AWS::CloudFormation::Init:
    configSets:
      end-to-end:
        - Install
        - Setup
        - Service
    Install:
      packages:
        yum:
          docker: []
          wget: []
          gzip: []
{% endhighlight %}

In the [Setup](https://github.com/made2591/vcs-server/blob/0d8bd43c24ea22be12d05466bad8075ef69c134d/template.yml#L146), the certificate files are retrieved from the parameter store (`files` section) and safely stored inside the machine. A second config section (`commands`) will execute the required command to retrieve the binary of codeserver, unzip it and place it under the `bin` folder: 

{% highlight yml %}
Setup:
  files:
    "/etc/pki/CA/private/cert1.pem":
      content:
        Fn::Join:
          - ""
          - - ""
            - !Ref CertificateParameterName
      mode: "000600"
      owner: root
      group: root
    "/etc/pki/CA/private/privkey1.pem":
      content:
        Fn::Join:
          - ""
          - - ""
            - !Ref PrivateKeyParameterName
      mode: "000600"
      owner: root
      group: root
  commands:
    01-wget:
      command: wget https://github.com/cdr/code-server/releases/download/1.1156-vsc1.33.1/code-server1.1156-vsc1.33.1-linux-x64.tar.gz
      cwd: "~"
    02-tar:
      command: tar -xvzf code-server1.1156-vsc1.33.1-linux-x64.tar.gz
      cwd: "~"
    03-copy:
      command: cp code-server1.1156-vsc1.33.1-linux-x64/code-server /usr/bin/
      cwd: "~"
    04-chmod-bin:
      command: chmod +x /usr/bin/code-server
      cwd: "~"
{% endhighlight %}

Then, the [Service](https://github.com/made2591/vcs-server/blob/0d8bd43c24ea22be12d05466bad8075ef69c134d/template.yml#L179) section will configure systemd (buuuuuuu) to run the process. I almost followed the instruction available [here](https://www.linode.com/docs/quick-answers/linux/start-service-at-boot/) to build this config file: I'm not an expert of this kind of thing, so take it as it is without too many warranties :)

{% highlight yml %}
Service:
  commands:
    05-service-creation:
      command: !Sub |
        cat <<EOF >> /etc/systemd/system/code-server.service
        [Unit]
        Description=Codeserver systemd service

        [Service]
        Type=simple
        Restart=always
        RestartSec=5s
        ExecStart=/usr/bin/code-server --disable-telemetry --password ${AppPassword} --port ${ExposedPort} --cert=/etc/pki/CA/private/cert1.pem --cert-key=/etc/pki/CA/private/privkey1.pem

        [Install]
        WantedBy=multi-user.target
        EOF
      cwd: "~"
    06-chmod-service:
      command: "chmod 644 /etc/systemd/system/code-server.service"
      cwd: "~"
    07-chmod-service:
      command: "systemctl --system daemon-reload"
      cwd: "~"
    08-systemctl-start:
      command: "systemctl enable code-server "
      cwd: "~"
    09-systemctl-enable:
      command: "systemctl start code-server "
      cwd: "~"
  services:
    sysvinit:
      code-server:
        enabled: "true"
        ensureRunning: "true"
{% endhighlight %}

And this is pretty much the whole `cfn-init` setup process.

#### Route53
To let Cloudformation propagates correctly Route53 record set to let you reach your machine, there's an `AWS::Route53::RecordSet` that create the record for you in the form `instance-id.code.mydomain.com`

{% highlight yml %}
CodeServerPublicInstanceRecordSet:
  Type: AWS::Route53::RecordSet
  Properties:
    HostedZoneId:
      Ref: HostedZoneId
    Comment: Codeserver public instance domain name
    Name:
      Fn::Join:
      - ''
      - - Ref: CodeServerPublicInstance
        - "."
        - Ref: DomainName
        - "."
    Type: A
    TTL: '60'
    ResourceRecords:
    - Fn::GetAtt:
      - CodeServerPublicInstance
      - PublicIp
{% endhighlight %}

Finally, if you consider the architecture proposed in the beginning, you can even implement your *notification* part to provide (by email, for example), the temporary Route53 address to reach your machine. And why not, maybe even generate the password to access the application at run time and provide it on a different channel (why not, encrypted with a fixed key you define only once).

### Conclusion
This was the first step to go live with you VSC instance and start coding from everywhere. To read about how to transform this stack into a multi-tenant solution to share the same machine across multiple users, then go ahead with the reading and look the Part II - [My team run VSC in the browser and they are just fine - Part II](https://madeddu.xyz/posts/traefik-single-to-multi-tenant) 😉!

Thank you everybody for reading!

[^domain]: if you have a domain registered somewhere else, I'm sorry I don't cover this scenario here because it's out of the scope of the article but you can google and find how to bring it inside route53.