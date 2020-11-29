---
layout: post
title: "Security and Docker: tips and tricks"
date: 2019-02-07
polly: https://madeddu.xyz/mp3/docker-security.mp3
categories:
- theory
- docker
tags:
- container
- docker
- security
- informative
- hacking
---

### Introduction
Everyone use Docker and normally when something is so diffused, there's always someone else that try to figure out how to leverage the diffusion to do bad things (you know what happened in Breaking Bad). Only [a few months ago](https://techcrunch.com/2018/06/15/tainted-crypto-mining-containers-pulled-from-docker-hub/) it happened that someone pushed some malicious software - cryptomining - over lot of images: this happened because, despite the fact that everyone use Docker, not so many people are really aware about security over Docker. Fortunately, during the years Docker put in place several tools and features to avoid (or at least drastically reduce) the damages that *bad attitude* could cause in your containerized systems or applications.

<div class="img_container"><img src="https://i.imgur.com/syc0Wkn.png"  style="width: 40%; marker-top: -10px;"/></div>

If you are interested in discover more about that, I will say something about:
- Isolation: docker network, stop and diff;
- Least Privilege, USER and Linux/Docker Namespaces;
- Delete data, Segregation and Docker privileges;
- Defang Docker images and Docker Capabilities;
- Secure computing mode (seccomp) and limiting of resources;
- Handle secrets, docker-slim and automation;

### Attack happens: what to do
Let's think about a potentially real scenario: you have an application running over one or more Docker images - that could or could not be scaled out to many containers, it doesn't matter for the purpose of the reasoning. Let imagine that this application runs some malicious code and/or code with bugs well known or - even worst - not known yet. This bring your system to be hacked by someone else - i.e. someone else take control of the Docker container running your application by using some exploits and accidentally - even worst - scale privileges to your host system too.

The first question is: what should you do if this happens to your application?

### Incident Response
There's a manual incident response. This are the main things you should keep in mind:

- **Immediately act to isolate compromised container and/or VM/host**. This is the first thing to do to avoid escalation of the issue and, if you are lucky, maybe even investigate more easily the problem(s) by being one step forward your attacker;
- **Do not delete anything**. This is crucial: very often not skilled people scared about what is happening or already happened, decide to delete everything to both be sure it will not happen - and maybe sometimes also to delete traces of some human errors. This is WRONG: human errors will always happen, but **The First Principle of Reliability** teach you that

> You can always avoid what you already now (already happened to you) and you should act to avoid as much as possible what you don't know (never happened to you).

Thus, even if the damage is huge, having traces will prevent you (or your replacement if you caused a mess and you got fired 😅) to repeat the same mistakes already did once;
- **Establish attack extension**. This for sure it's something important to enroll the right amount of time / resources in investigation during (and after) the attack: how urgent is this - which systems are impacted - etc;
- **KEEP CALM**. At least, try to 😂;

<div class="img_container"><img src="https://i.imgur.com/JkqnRZI.png"  style="width: 40%; marker-top: -10px;"/></div>

### Docker tools
As we said, during the years Docker put in place different tools and features to deal with security of your containers: this start from the engineering of the images and include also the run time.

#### Isolation
The Docker commands to work with isolation are at least two:

- `docker network`;
- `docker stop`;
- `docker diff`;

Sorry, three.

##### Docker network
One of the reasons Docker containers and services are so powerful is that you can connect them together, or connect them to non-Docker workloads. Docker's networking subsystem is pluggable, using drivers. Several drivers exist by default, and provide core networking functionality.

The Docker network command, as stated by the documentation, *let you manage networks*. What does it means? It means that if you have to isolate something from something else, the network layer is a good starting point to apply a potentially already sufficient cutoff. Let's see the commands in details

| Command                     | What it does                                          |
| --------------------------- | ----------------------------------------------------- |
| `docker network connect`    | connect a container to a network;                     |
| `docker network create`     | create a network;                                     |
| `docker network disconnect` | disconnect a container from a network;                |
| `docker network inspect`    | display detailed information on one or more networks; |
| `docker network ls`         | list networks;                                        |
| `docker network prune`      | remove all unused networks;                           |
| `docker network rm`         | remove one or more networks;                          |

Of course, the most useful programs if you are still admin of your system - and you should, otherwise just RUN FOREST, RUN!!! - is detach the network interface of the container(s): which one? Well, this is something more related to the way you monitor your application but usually you should be able to identify where is the cause if you're owner (or you know the owner) of the application. What I mean? For instance, if you're homepage get hacked, you should be able to know at least which part of the application serve the homepage and start from there, etc.

##### Docker stop
The well known Docker stop terminate your container: if your container is orchestrated by some other tools (ecs, k8s, etc), it could not be enough and you should most probably act over deployment, services, or any other abstraction layer that *manage* your container for you.

The point is: a stopped container is something you can work on to understand what happened (for instance by looking at the logs with `docker log` command) and in general is a good idea to avoid escalation of the problem.

<div class="img_container"><img src="https://i.imgur.com/wNTuPmp.png" style="width: 100%; marker-top: -10px;"/></div>

##### Docker diff
Despite the Docker diff is a really powerful command, it doesn't seem to be always well known: what it does and why it's important?

To quote [the official doc](https://docs.docker.com/engine/reference/commandline/diff/), the Docker diff inspect changes to files or directories on a container's filesystem.

So this command will show any differences in the filesystem inside the container, compared to the image it was built from. In our top example, we could see if any new files have been created/modified related to our hacked service. If a hacker had installed any tools or scripts, we would see those too.

### Least Privilege
In information security, computer science, and other fields, the principle of least privilege requires that in a particular abstraction layer of a computing environment, every module - such as a process, a user, or a program, depending on the subject - must be able to access only the information and resources that are necessary for its legitimate purpose.[^wiki] In containers world, this translate directly in:

> A (process/container) should only have access to the data and resources essential to perform its function.

Take a moment to reflect over this: this is **crucial** to understand, because the violation of this principle it's most probably the main cause of exploits available in most of the Docker images available online.

Ok, know that we saw how to isolate and/or stop a container, let's see how to mitigate an attack by following some - almost always - simple smart rules.

### Attack mitigation
The main goal of security is to make attacks difficult - ideally, impossible - to be perpetrated by anyone else who don't have privileged access to your system. How can you do this with Docker?

- **Set a USER**: this come from unix operating system so it's really nothing more than the concept of multiuser / least privilege concepts ported to Docker world;
- **Run a read-only filesystem**: this could seem trivial, but we will see how much troubles could cause to the majority of the attackers, i.e. the more inexperienced ones;

#### Why setting a USER
By default, users are not namespaced in Docker. What does it means namespaced and why is this important to know? Well, because namespaces are a fundamental aspect of **containers on Linux** (yes, Docker didn't invent anything, I already wrote [about this](https://madeddu.xyz/posts/jails) once). Thus, let's do a recall of this concept first.

##### Linux namespaces
Namespaces are a feature of the Linux kernel[^namespaces] that partitions kernel resources such that one set of processes sees one set of resources while another set of processes sees a different set of resources. The feature works by having the same namespace for these resources in the various sets of processes, but those names referring to distinct resources. Examples of resource names that can exist in multiple spaces, so that the named resources are partitioned, are process IDs, hostnames, user IDs, file names, and some names associated with network access, and interprocess communication.

Thus, back to USER and Docker, this is what you should now:

| Fact                                                                                                    | Reaction                                        |
| ------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| <span style="color:#A04279; font-size: bold;">UIDs are the same on the host and in the container</span> | BOOOOOOOM;                                      |
| <span style="color:#A04279; font-size: bold;">Root in the container is root on the host</span>          | BOOOOOOOM^2;                                    |
| Consider the possibility of container breakout                                                          | yes, containers are not *jails* (ready: fight); |

From this three fundamental information (practise examples in a while) you should learn that attackers should be constrained with the container, because you wouldn't run apps in VMs as root: and thus, you should not do it in a container even.

#### Docker Namespaces
Since Docker since 1.10, the namespaces where introduced to automatically maps users in container to high-numbered user on host. You can configure the mapping set on the daemon - and not per container - due to some complications with ownership of shared layers. Unfortunately, there are some problems yet with volume permissions and volume plugings, you can't use --pid=host or --net=host (to share the network interface) and some privileged operations are not allowed (like the mknod to create files for peripheral).

Ok so... how to prevent privilege escalation?

#### Prevent privilege escalation
The following statement should be like a mantra:

> The best way to prevent privilege-escalation attacks from within a container is to configure your container's applications to run as unprivileged users.

For containers whose processes must run as the root user within the container, you can re-map this user to a less-privileged user on the Docker host. The mapped user is assigned a range of UIDs which function within the namespace as normal UIDs from 0 to 65536, but have no privileges on the host machine itself. Thus...

##### How to setup USER in a Docker
Creating a USER in a Docker container is as simple as start by creating the user in the Dockerfile:

{{< highlight sh >}}
RUN groupadd -r myuser && useradd -r -g myuser myuser
{{< / highlight >}}

Then, you can simply do anything that needs root privileges like installing packages and so on:

{{< highlight sh >}}
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
{{< / highlight >}}

Then change to the user:

{{< highlight sh >}}
USER myuser
{{< / highlight >}}

It could happen that sometimes you need root privileges in start-up script, for example when you have to change ownership of files. In this case you can't use USER statement, and you should done it in the start-up script.

##### Running a read-only Filesystem
It's really simple to run a read-only filesystem: there's a flag available to enable this feature and it's called `--read-only`. Just try it out by launching the following command:

{{< highlight sh >}}
docker run --read-only debian sh -c 'echo "whatever" > /file'
{{< / highlight >}}

And this can be combined also with the volume directive:

{{< highlight sh >}}
docker run --read-only -v "$PWD":/tmp debian sh -c 'echo "whatever" > /tmp/file'
{{< / highlight >}}

One option I found useful when you have to deal with filesystem issues - but also for security reason could be the use of temporary in memory - not persistent - volume

{{< highlight sh >}}
docker run --read-only --tmpfs /tmp:size=65536k debian sh -c 'echo "whatever" > /tmp/file'
{{< / highlight >}}

that are deleted when the container die. More on them [here](https://docs.docker.com/storage/tmpfs/).

##### Deleting sensible data
Be careful about deleting data in Dockerfiles because remember that every step is a layer - a tarball shared, preserved and most important available for manipulation - does things like the following don't work:

{{< highlight sh >}}
FROM debian
RUN apt-get update && apt-get install -y curl
RUN curl http://myfile.com/secret.tgz -o /secret.tgz
RUN tar xzf /secret.tgz && <do-stuff>
RUN rm /secret.tgz
{{< / highlight >}}

because your secret will be stored until the last layer and thus available. This is something simple to avoid because it only requires you to remember that every layer is defined by a statement. Thus something like this

{{< highlight sh >}}
FROM debian
RUN apt-get update && apt-get install -y curl
RUN curl http://myfile.com/secret.tgz -o /secret.tgz && tar xzf /secret.tgz && <do-stuff> && rm /secret.tgz
{{< / highlight >}}

it's almost ok from - even if maybe not so best-practises compliant (see at the end). Of course, avoid copy and deleting ssh private keys in separated steps. Those kind of things should be - in the worst case - injected as environment variables from outside but even in this case they would be available in the env thus... if you have to use secret, just port them outside of your logic and trust your container somewhere else (more about this later).

##### Segregation
To reduce privilege and containing attacks, you can even segregate containers by *context*. What does this means? Well, containers run on hosts, either virtual on bare-metal machines, and hosts provide an extra layer of isolation.
This can be used to segregate containers by security context, and placing low-risk containers on one set of hosts and high-risk ones on another set, in such a way that - in case of escalation - at list critical part will be *physically* separated.

##### Docker privileges
The point is that unfortunately anyone who can start Docker containers effectively has root access to your system if can mount and read/change any file: this is super easy to verify. Open a shell and try to run the following:

{{< highlight sh >}}
/etc/passwd
{{< / highlight >}}

Now try to run:

{{< highlight sh >}}
docker run -v /etc/passwd:/passes debian cat /passes
{{< / highlight >}}

So how to avoid this? Let's figure out what actually you can do from inside a container.

#### Backdoors
There are two special permissions that can be set on executable files: set User ID (`setuid`) and set Group ID (`sgid`). These permissions allow the file being executed to be executed with the privileges of the owner or the group. For example, if a file was owned by the root user and has the `setuid` bit set, no matter who executed the file it would always run with root user privileges. If you have access to docker you could even start a container, create a suid binary and docker cp (docker copy, see [here](https://docs.docker.com/engine/reference/commandline/cp/)) the binary out of the container. Chances are that your application does not need any elevated privileges, and thus `setuid` or `setgid` binaries.

I found [a nice article](http://redhatgov.io/workshops/security_containers/exercise1.3/) from Redhat about how to *defang your binaries*: I ported the key points in the following two paragraphs.

##### Find a list of binaries
To get a list of binaries with special permissions in a container image, the following command can be used:

{{< highlight sh >}}
sudo docker run debian:jessie find / -perm +6000 -type f -exec ls -ld {} \; 2> /dev/null
{{< / highlight >}}

over your image. At least now you now what could be dangerous. So, how to remove this privileges?

##### Defang your Docker images

You can then defang the binaries with the `chmod a-s` command to remove the suid bit. For example, you can defang your debian image by running:

{{< highlight sh >}}
FROM debian:jessie
RUN find / -xdev -perm +6000 -type f -exec chmod a-s {} \; || true
{{< / highlight >}}

Why `|| true`? The `|| true` allows you to ignore any errors from find. The `setuid` and `setgid` binaries run with the privileges of the owner rather than the user. These are normally used to allow users to temporarily run with escalated privileges required to execute a given task, such as setting a password.

It's more likely that your Dockerfile will rely on a `setuid/setgid` binary than your application. Therefore, you can always perform this step near the end, after any such calls and before changing the user (removing setuid binaries is pointless if the application runs with root privileges).

#### Limit the capabilites
The linux kernel defines various capabilities (CAP_AUDIT_CONTROL CAP_CHOWN CAP_FSETID CAP_LEASE CAP_MKNOD, etc). Directly from the [man](http://man7.org/linux/man-pages/man7/capabilities.7.html), for the purpose of performing permission checks, traditional UNIX implementations distinguish two categories of processes: privileged processes (whose effective user ID is 0, referred to as superuser or root), and unprivileged processes (whose effective UID is nonzero). Privileged processes bypass all kernel permission checks, while unprivileged processes are subject to full permission checking based on the process's credentials (usually: effective UID, effective GID, and supplementary group list).

Starting with kernel 2.2, Linux divides the privileges traditionally associated with superuser into distinct units, known as capabilities, which can be independently enabled and disabled. Capabilities are a per-thread attribute.

Docker containers get a bunch of these capabilities by default, but some of them are not present: try run this command from a shell.

{{< highlight sh >}}
docker run --rm debian date -s "25 SET 1991 10:00:00"
{{< / highlight >}}

The flag `--privileged` let you run with almost all of them. The flag `--cap-add` and `--cap-drop` can respectively add and remove them. Thus if run something like

{{< highlight sh >}}
docker run --rm --cap-drop CHOWN debian chown 777 /tmp
{{< / highlight >}}

<span style="color:#A04279; font-size: bold;">we are preventing from changing ownership using the `chmod` command and we will obtain an error!!</span>

<div class="img_container"><img src="https://i.imgur.com/prlZbUn.jpg" style="width: 100%; marker-top: -10px;"/></div>

#### Secure computing mode
Secure computing mode (`seccomp`) is a [Linux kernel feature](https://en.wikipedia.org/wiki/Seccomp). What it does? Shortly, seccomp allows a process to make a one-way transition into a secure state where it cannot make any system calls except exit(), sigreturn(), read() and write() to already-open file descriptors.

This feature is available only if Docker has been built with `seccomp` and the kernel is configured with CONFIG_SECCOMP enabled. To check if your kernel supports `seccomp`:

{{< highlight sh >}}
grep CONFIG_SECCOMP= /boot/config-$(uname -r)
CONFIG_SECCOMP=y
{{< / highlight >}}

The nice thing is that you can use it to restrict the actions available within the container. The `seccomp()` system call operates on the `seccomp` state of the calling process. You can use this feature to restrict your application's access by running with this `--security-opt seccomp=myprofile.json`:

{{< highlight sh >}}
docker run --rm -it --security-opt seccomp=myapiprofile.json golang
{{< / highlight >}}

There's default seccomp profile (it can be found [here](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)) that provides a sane default for running containers with seccomp and disables around 44 system calls out of 300+ (as stated in the Docker doc).

Now the point is: what do you need inside your Docker? Who knows. Good luck

But hey! There's a tool for everything: [docker-slim](https://github.com/docker-slim/docker-slim) can help you build your secomp profile.

##### Docker-slim
If you only want to auto-generate a Seccomp profile (along with other interesting image metadata) use the profile command.

{{< highlight sh >}}
docker-slim build --http-probe your-name/your-app
docker run --security-opt seccomp:<docker-slim directory>/.images/<YOUR_APP_IMAGE_ID>/artifacts/your-name-your-app-seccomp.json <your other run params> your-name/your-app
{{< / highlight >}}

Feel free to copy the generated profile. You can use the generated Seccomp profile with your original image or with the minified image. But docker-slim do so much more like minifying your images: if you want to auto-generate a Seccomp profile AND minify your image use the build command (more in the official repo)

#### Limiting resources
Another important thing that can be done to avoid attacks (specialy DoS attacks or similar) is limiting resources. This is something that can be done by docker natively - even outside of orchestration environment and can be useful when you have high and low priority containers.

##### Limit CPUs
How can you *keep under control* - and thus act differently if you notice something weird - CPUs across containers? Well, containers get a default relative weighting of 1024. Share this value is as simple as run:

{{< highlight sh >}}
docker run -d -c 512 --name a myimage
{{< / highlight >}}

You can have another image running at default 1024, and this will imply 66% for it and 33% a. With other two flags you can control even how many CPU and time period dimension available for use for each container:

{{< highlight sh >}}
docker run -it --cpu-period=50000 --cpu-quota=25000
{{< / highlight >}}

or even the more recent `--cpus 1.0` to specify the amount of cpus you want to use directly. Nice, but working over the memory could be more useful.

##### Limit memories
By default, a container can use as much memory and swap as it likes. This could potentially let the attacker or some other bugs to cause memory problems and thus stop your application working properly. If you accept to encounter some performance overhead problems, you can control both memory and swap by doing using flags like:

{{< highlight sh >}}
docker run -it -m 256M --memory-swap 512M
{{< / highlight >}}

and avoid bad things like - just to say - [fork bombs attacks](https://en.wikipedia.org/wiki/Fork_bomb).

##### About secrets
There's no a safe place to store secret. You should avoid copy them inside, thus the first things is to load them inside the magical safe place called ENVIRONMENT or ENV - as stated in the [12factor-app](https://12factor.net/config). Everyone did almost ones something like:

{{< highlight sh >}}
docker run -e API_TOKEN=MY_SECRET_KEY myimage
{{< / highlight >}}

At least, I do this every day. It's super simple and easy to use, but if you do:

{{< highlight sh >}}
docker inspect & ps
{{< / highlight >}}

The major problem is that it makes the secret too visible from the linked containers and also as output of `env` command - then of course you cat your credentials files every day thus everything is in there in the history XD.

Now the point is: if not inside the ENV, where the f\*\*k do I have to place my secrets?!?! Maybe mounting a read-only file could seem an option:

{{< highlight sh >}}
docker run -v /nested_dir_in_host/key_file:/key_file:ro myimage
{{< / highlight >}}

but who likes files?! And this nested_dir_in_host/key_file is really secure? The solution is using a Vault (never did it) or even better port your authentication out - and define roles and authorization outside. The process should be something like *I trust my immutable object* and I provide to him *temporary credentials* it can use to do things. This is to decouple your image (your vulnerable part) from your system (locked part).

### If it's not enough...
Unfortunately, even by apply everything you haven't solved the initial problem: because in the end you cannot lock everything. The general guidance is to - first of all - avoid vulnerable software: in fact, if you have a bug in your application the only real solution is to replace the library because running old and known-vulnerable code will get you hacked. Then prefer minimal images (like alpine ones are often the most checked and not vulnerable, but of course it depends on the images you are using). And finally, since it's almost an impossible task keeping the images you build up-to-date with CVEs and actually is even harder to know what libraries are in use, you can leverage an image scanner that automatically scan images for known vulnerabilities.

Thank you everybody for reading!

[^wiki]: Directly from [Wikipedia](https://en.wikipedia.org/wiki/Principle_of_least_privilege).
[^namespaces]: For more information about this topic, see [Linux namespaces](https://en.wikipedia.org/wiki/Linux_namespaces).
