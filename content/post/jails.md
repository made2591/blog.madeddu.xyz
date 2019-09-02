---
layout: post
title: "Jails: confining the omnipotent root"
date: 2018-01-08
categories:
- theory
- miscellaneous
tags:
- theory
- jails
- freebsd
- container
---

### Preamble
Recently I became nostalgic and fascinated with stuff from the past, so I decided to create a [Vagrantfile](https://github.com/made2591/vagrant-freebsd) to work with FreeBSD[^fbsd]. Why FreeBSD? Because as a developer, I really like Docker and I started looking in the past to find its _historical_ birth: in fact, as a concept, Docker is no so recent as you think, and I think it exists also because of the works of some other bigs from 80', such as Poul-Henning Kamp[^phk]. Starting from [its work](http://phk.freebsd.dk/pubs/sane2000-jail.pdf) and using a FreeBSD installation I did some experiments with jails, to understand better what they really are, how they works - how can you create what it will look like a _vintage container_ - and why you should use them in a FreeBSD environment - at least, to learn something new.

<div class="img_container"><img src="https://i.imgur.com/BC3tgBK.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Introduction
As P.H. Kamp says in its work, _FreeBSD's __jail__ facility provides the ability to partition the operating system environment, while maintaining the simplicity of the UNIX root model_. I think its work is really clear, and I would like to follow logic steps behind its reasoning, but...wait: let's start from the problem.

#### Traditional UNIX Security
The traditional UNIX access model assigns numeric uids to each user of the system. For the purposes of human convenience, uid 0 is canonically allocated to the root user. The system can discover whether special privileges are accorded to a process simply by looking at the uid of the process: if it is equal to 0, then the process is acting with super-user privileges. It's really simple, but there are a few problems.

Formally, let:
- \\(\{root, u_1, u_2\} \in \cal U\\) be respectively the root (\\(uid \; = \; 0\\)), and two common users \\(\{uids \; = \; 1,2\}\\) from the set of users \\(\cal U\\(;
- \\(\{p_1, p_2, p_3, p_4\} \in \cal P \in \cal O\\) be four different privileged operations from the subset of privileged operations \\(\cal P\\) included in the set of all available operations \\(\cal O\\(;
- \\(G \; = \; \{V, E\}\\) be a graph of _dependencies_ between operations, with \\(V = \cal U\\) and \\(E = \{p_1 \rightarrow p_3, p_2 \rightarrow p_3, ...\}\\(;

It's easy to imagine how this scenario could collapse into a difficult dependencies graph that doesn't let the system to be secure at all. In particular, \\(G\\) itself says that many privileged operations in UNIX - even if _seem_ independent - actually are not: instead, they are closely related and the handing out of one privilege may, in effect, be transitive to the many others. Let assume that users \\(u_1\\) and \\(u_2\\) have to be able to do \\(p_1\\) and \\(p_2\\), but no \\(p_3\\(: then you can permit both to be root enabling them doing whetever they want - of course including \\(p_3\\), but also other \\(p_s\\) you want to disable for both the users. What does it means? It means that, if you have to assign different permits over the same operating system - some securelevel mechanism has to be implemented which allows the administrator to block certain configuration and management functions from being performed by root.

However, a system _root-dependent_ is not a good idea, because if the root user is compromised, you're f\*\*\*d up, and second, having a single administrative account is not a good idea, even in the simplest environment. There are some features in _BSD_ system that provides - let me say - security capabilities: for instance, the single-user mode available to block certain functions until reboot. This is a great functionality but, it's not a solution yet to separate several root abilities. A simplest solution should be introducing new security features to split privileged operations, but this often involves introducing new security management APIs. Further, if we assume to introduce fine-grained capabilities to replace the setuid mechanism in UNIX-like operating systems, applications that previously did a check to see if they were running as root before executing __must now be changed__ to know that they need not run as root. This is not a real solution.

#### chroot
The chroot utility can be used to change the root directory of a set of processes, creating a safe and separate environment from the rest of the system: processes created in the chroot environment can not access files or resources out of this: ok, but I think __jails__ improve the concept of the traditional chroot environment in many ways. In a traditional chroot environment, processes are limited only to the portion of file systems that can be accessed. The rest of system resources (such as the set of system users, running processes, or network sub-system) are shared by chrooted processes and host system processes (those not in a chroot environment). The jails expand this model by _virtualizing_ not only the file system access, but also the set of users, the FreeBSD kernel network sub-system and some other things.

### jail
A jail is characterized by four elements:

- A sub-branch of a directory: the starting point from which you enter the jail. Once inside the jail, a process is not allowed to exit this sub-branch. The traditional security issues affecting the original chroot (2) design do not affect FreeBSD jails.

- A hostname: the hostname that will be used inside the jail. Jails are mainly used to host network services, so having a descriptive host name for each jail can really help the system administrator.

- An IP address: this will be assigned to the jail and can not be changed in any way during the life of the jail. The IP address of a jail is usually an alias address of an existing network interface, although this is not strictly necessary.

- A command: the path to an executable to start inside the jail. This is relative to the root directory of the jail environment, and can vary a lot, depending on the specific type of jail environment.

Jail permits to partition a FreeBSD environment (processes, file system, network resources) into a management environment, and optionally subset Jail environments. The administrator of a FreeBSD machine can partition the machine into separate jail and provide access to the super-user account in each of these without losing control of the over-all environment.

Looking for informations about _how jails really work_, I discovered that jails are foundamentally of two types: the "complete" jails, which resemble a real FreeBSD system, and the "service" ones, dedicated to one application or service, possibly running with privileges. Let's make an example of how you can create a jail.

#### Instructions
First, [Vagrantfile](https://github.com/made2591/vagrant-freebsd) and ```vagrant ssh``` your FreeBSD machine

##### Create the environment
{{< highlight sh >}}
1 setenv DESTINATION /path/to/jail
2 mkdir -p $DESTINATION
3 cd /usr/src
4 make buildworld
5 make installworld DESTDIR=$DESTINATION
6 make distribution DESTDIR=$DESTINATION
7 mount -t devfs devfs $DESTINATION/dev
{{< / highlight >}}

Where:
- ```setenv DESTINATION /path/to/jail``` sets an environment variable to specify jail location - where the jail is physically in the host file system. A good practise is to put your jails under ```/usr```, better if in a ```jails``` dedicated subfolder: for instance, ```/usr/jail/mark``` where ```mark``` is the name of the jail;
- ```mkdir -p $DESTINATION``` creates the directory tree including intermediates one if required;
- ```cd /usr/src``` changes the folder to the place in which the source code is normally installed - which contains the several subdirectories, including the ```sys``` one that contains Kernel source files;
- ```make buildworld && make kernel``` creates the world and compiles the kernel source: this will take a loooooong time[^lt], but, using the -j - only in the first make - you can increase speed by running multiple jobs. How many jobs to use depends on the processor (number of cores plus one is a start). The kernel target builds and installs the new kernel;
- ```make installworld DESTDIR=$DESTINATION``` populates the directory subtree chosen with the necessary binaries, libraries, man pages, etc
- ```make distribution DESTDIR=$DESTINATION``` installs every needed configuration file. In simple words, it installs every _installable_ file of ```/usr/src/etc/``` to the ```/etc```

After you create the environment inside the jail subtree, you can create a jail - in term of configure it.

##### Configure the jail
To configure a jail, you have two options: first, use ```jail.conf``` file, a file structured with the format ```key = "value";```. The short version of the story is that FreeBSD jails used to be defined in ```/etc/rc.conf``` using _rc-style_ syntax. During FreeBSD 9.x, a new convention was introduced, using ```/etc/jail.conf```. FreeBSD 10.x defaults to the new style but still supports the ```rc.conf``` style. FreeBSD 11 dropped support for the old style. Inspired by [this short post](https://therub.org/2014/08/11/convert-freebsd-jails-from-rc.conf-to-jail.conf/), I got an simple but complete example of old style configuration:

{{< highlight sh >}}
jail_enable="YES"
jail_list="api web storage"
jail_mount_enable="YES"
jail_devfs_enable="YES"
jail_devfs_rules="devfsrules_jail"

jail_api_rootdir="/jails/api"
jail_api_hostname="api.mycompany.org"
jail_api_ip="10.0.0.1"

jail_web_rootdir="/jails/web"
jail_web_hostname="web.mycompany.org"
jail_web_ip="10.0.0.2"

jail_storage_rootdir="/jails/storage"
jail_storage_hostname="storage.mycompany.org"
jail_storage_ip="10.0.0.3"
{{< / highlight >}}

If you remove everything from rc.conf except for the jail_enable line and move everything shared to the global scope to clean up the individual definitions, you got a new jail.conf file like the one below:

{{< highlight sh >}}
allow.raw_sockets = 0;
exec.clean;
exec.system_user = "root";
exec.jail_user = "root";
exec.start += "/bin/sh /etc/rc";
exec.stop = "/bin/sh /etc/rc.shutdown";
exec.consolelog = "/var/log/jail_${name}_console.log";
mount.devfs;
mount.fstab = "/etc/fstab.$name";
allow.mount;
allow.set_hostname = 0;
allow.sysvipc = 0;
path = "/jails/${name}";

api {
        host.hostname = "api.mycompany.org";
        ip4.addr = 10.0.0.1;
}

web {
        host.hostname = "web.mycompany.org";
        ip4.addr = 10.0.0.2;
}

storage {
        host.hostname = "storage.mycompany.org";
        ip4.addr = 10.0.0.3;
}
{{< / highlight >}}

More info [here](https://www.freebsd.org/cgi/man.cgi?query=jail.conf&sektion=5&n=1).

The second option is to use directly the ```jail``` command with parameters from command line. As the man page states, jail command recognizes two classes of parameters: four fixed parameters, the _true jail parameters_, that are passed to the kernel when the _jail_ is created (which can be seen with _jls_ and, for backward compatibility, can be passed without names: path, hostname, ip, and command), and the pseudo-parameters that are only used by jail itself.

So a jail may be specified with parameters directly on the command line. In this case,the ```jail.conf``` file will not be used.
(And this is the part where you realize that jail utility is the docker daemon.)

##### Start/stop a jail
Using ```service jail start $name``` you start the jail $name: if you don't specify the name all the jails defined in the ```/etc/jail.conf``` will be started. Using the stop statement you can stop the jail.

##### Listing jails
Using ```jls``` you can list the jails in your host. jexec 3 /etc/rc.shutdown

<span style="color:#A04279; font-size: bold;">__NOTE__</span>: you can also shotdown a jail using the ```jexec``` command: look at the ```JID```'s jail you want to stop using the listing command ```jls```, then run ```jexec 3 /etc/rc.shutdown```

If you to find out more about how the jail mechanism is implemented, you can have a look at this files[^mat] ```/usr/src/usr.sbin/jail/jail.c```, ```/usr/src/sys/kern/kern_jail.``` and ```/usr/include/sys/jail.h``` header file: in the latter is defined the jail structure, there is an entry for each of the arguments passed to the jail command, and indeed, they are set during its execution.

{{< highlight sh >}}
struct jail {
        u_int32_t       version;
        char            *path;
        char            *hostname;
        u_int32_t       ip_number;
};
{{< / highlight >}}

### Conclusion
The jail facility provides FreeBSD with a conceptually simple security partitioning mechanism, allowing the delegation of administrative rights within virtual machine partitions. The implementation relies on restricting access within the jail environment to a well-defined subset of the overall host environment. This includes limiting interaction between processes, and to files, network resources, and privileged operations. Administrative overhead is reduced through avoiding fine-grained access control mechanisms, and maintaining a consistent administrative interface across partitions and the host environment. So...does it seem familiar?

<div class="img_container"><img src="https://i.imgur.com/lPETbDl.png"  style="width: 90%; marker-top: -10px;"/></div>

If something is wrong, please let me know, I'm still waiting for the world to be built -.-

Thank you everybody for reading!

__PS__: this is the part where you realize that a jail is similar to a container, the jail.conf is kind of a Dockerfile but - wait for it - could be so much more, like a docker compose: further, jexec looks like the docker exec command, and jls looks like docker ps -a, and so on, am I wrong?

[^phk]: [Poul-Henning Kamp](https://en.wikipedia.org/wiki/Poul-Henning_Kamp)
[^fbsd]: [https://www.freebsd.org/it/](https://www.freebsd.org/it/)
[^lt]: In [this](https://lists.freebsd.org/pipermail/freebsd-questions/2016-November/274690.html) short thread, you can see some order of magnitude: buildworld takes hours so...be patient.
[^mat]: More at [freebsd.org](https://www.cse.buffalo.edu//~kensmith/FreeBSD/book.html)