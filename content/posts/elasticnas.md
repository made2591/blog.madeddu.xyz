---
layout: post
title: "Elasticsearch over My home Network Attached Storage"
date: 2018-01-13
polly: https://madeddu.xyz/mp3/elasticnas.mp3
categories:
- coding
- python
tags:
- coding
- python
- elastic
- kibana
- search
- nas
---

### Introduction
I always owned a lot of hard drives: I don't know why, I always used and still use to look for space to save my data. In the years, I started using disks, then I assembled a HP Proliant to be a Synology Based System - I don't want to go the cloud because I'm stupid - and... in the last week, I decided to make order in a huge amount of files. The first thing you have to do when you are handling terabytes and terabytes of __both__ well-ordered __and__ _no-ordered-at-all_ data is literaly pray that someone else, like a magician, or druid comes to you with a magic wand and fixes all the mess for free, in a way you do not know but you will like.
This article is the right one if you don't want to pray, you really don't believe in miracle but you still need to order your stuff. I have done it using elastisearch and kibana!

<div class="img_container"><img src="http://craigconnects.org/wp-content/uploads/W-S-files.jpg" style="width: 100%; marker-top: -10px;"/></div>

In the next paragraphs, I will talk about how you can setup your envinroment locally, without losing too much time in configurations and start building what you need to explore your file systems.

### What you need before starting
Ok, here the ingredients to prepare the environment:

- Elasticsearch: download available [here](http://www.elastic.co/downloads/elasticsearch);
- Kibana: download available [here](https://www.elastic.co/downloads/kibana);
- Python: actually, you don't need this if you are an AWK expert 😎😎😎;
- Of course, a file system to explore;

### Recipe
To order your files, you first have to make some sort of statistics: Elasticsearch + Kibana can really help you in doing that. Of course, you also have to create data to explore your file systems, but this is really a simple task I will show in the end. First, let's start with the core: Elasticsearch.

#### Step 1: run Elasticsearch
Elasticsearch is a distributed, RESTful search and analytics engine capable of solving different _scenarios_: what do I mean? That it could be use to centrally store your data, let you _discover the expected_ and _uncover the unexpected_ - cit. This is perfect for my original purpose (make order in my file systems)! How can you install Elastisearch in a portable way? It's really simple: using Docker OR... if you don't use Docker - I mean, why!? - open a shell and do the following:

{{< highlight bash >}}
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.1.1.tar.gz
tar -xvf elasticsearch-6.1.1.tar.gz
cd elasticsearch-6.1.1/bin
./elasticsearch
{{< / highlight >}}

If you're running Elasticsearch on Windows, simply run ```elasticsearch.bat``` instead. If you are a mac user, you can also install Elasticsearch using [Homebrew](https://brew.sh/index_it.html) by running ```brew install elasticsearch``` from your shell. In any case... congratulations, you are done[^est]! You can add -d if you want to run it in the background as a daemon - I would prefer a Docker container to create something always available. Let's move one step forward.

#### Step 2: run Kibana
Kibana gives _shape_ to your data and is the extensible user interface for configuring and managing all aspects of the Elastic Stack: it lets you visualize your Elasticsearch data and navigate them. The core ships with the classics histograms, line graphs, pie charts, sunbursts, and more. They leverage the full aggregation capabilities of Elasticsearch: it's a really awesome tool. Again, how to install it?

For mac OS
{{< highlight bash >}}
curl -O https://artifacts.elastic.co/downloads/kibana/kibana-6.1.1-darwin-x86_64.tar.gz
shasum kibana-6.1.1-darwin-x86_64.tar.gz
tar -xzf kibana-6.1.1-darwin-x86_64.tar.gz
cd kibana-6.1.1-darwin-x86_64/
{{< / highlight >}}

For linux 64bit
{{< highlight bash >}}
wget https://artifacts.elastic.co/downloads/kibana/kibana-6.1.1-linux-x86_64.tar.gz
sha1sum kibana-6.1.1-linux-x86_64.tar.gz
tar -xzf kibana-6.1.1-linux-x86_64.tar.gz
cd kibana-6.1.1-linux-x86_64/
{{< / highlight >}}

In any case[^kt], simply run Kibana with ```./bin/kibana```. And... again, congratulations, you are done! If you point your browser to [http://localhost:5601](http://localhost:5601) you should see the Kibana web console. Now that you have a running Elasticsearch core and a Kibana web console to look at your stack - cool, isn't it? - let's move to the most important part: how to generate data starting from the file systems to make some sort of analysis.

#### Step 3: gather filesystem informations
For this step you can use whatever you want. I found inspiration by the output of the ```ll -R``` command, but I'm lazy so I decided to use Python instead. How can you explore the file systems efficiently? Well, it depends on your file systems type: Python offer quite efficient and ready to use library to walk directory recursively but... if you have to explore a file system from a network attached storage, striped on more than one physical devide, you could take advantage of multiprocessing. However, if your file system resides on a single physical device, is entirely possible that parallelizing the work will be slower than just doing the work in a single process. This is because IO on the hard disks - any techs - underlying your shared filesystem may be the bottleneck and attempting many disk reads in parallel could make them all slower, if the disks needs to seek more often rather than reading longer linear sections of data. Even if the IO is a little faster, the overhead of communicating between the processes could eat up all of the gains.

In any case, I created a [repository](https://github.com/made2591/python-multi-walker) to share the few lines of code I used to create my data: using the method ```os.stats``` - more info [here](https://docs.python.org/2/library/stat.html) - of Python standard lib, I created a script that log to a JSON file - named ```files.json``` - all info about each file in my file system. Each file is defined as a dict with this data:

{{< highlight json >}}
{
	"name":  filename,
	"ext":   ext,
	"mode":  stats[0],
	"ino":   stats[1],
	"dev":   stats[2],
	"nlink": stats[3],
	"uid":   stats[4],
	"gid":   stats[5],
	"size":  stats[6],
	"atime": stats[7],
	"mtime": stats[8],
	"ctime": stats[9]
}
{{< / highlight >}}

The stats for each file are extened with, of course, file name and extensions - even if filename (complete path) is sufficient, I wanted also a separated field with ext to create filters more easly later. You can call the explorer with a specific starting directory, with a custom number of processes - if you have a distributed file systems - and saving the output wherever you want.

{{< highlight python >}}
explorer(path = "/", process = 1, output = "files.json")
{{< / highlight >}}

__NOTE__: you can of course play with AWK and print your JSON struct to a file with a single command - but again I'm lazy ok!? And I'm also working on layer 2 for my network saga, it takes - believe me - A LOT of time.

Then, following the instruction of the [getting started guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/_exploring_your_data.html), I load my data in Elasticsearch using the bulk api: it is amazing because it makes it possible to perform many index/delete operations in a single API call - greatly increasing the indexing speed.

{{< highlight bash >}}
curl -H "Content-Type: application/json" -XPOST "localhost:9200/system/files/_bulk?pretty&refresh" --data-binary "@files.json"
{{< / highlight >}}


#### Step 4: insert data, create index, start dashboarding
Ok, until now 1) we created the environment to analyze data 2) we gathered data from our file system 3) we loaded data in our stack: how can we create visualization? Using the last tool we didn't use yet: kibana! First, navigate to [http://localhost:5601](http://localhost:5601), then in sidebar on the left click on ```Management```, than ```Index Patterns```, then select the index pattern - in my case and by following examples above you only have to type ```system*``` in search bar, click ```Next Step``` in the right, then type a pattern ID for your pattern index if you want, then click on ```Create index pattern```. Magic is done!

A table should appear with all informations and inferred types for each field about the data you load previously using the bulk api - more details [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html).

##### Create a Dashboard
If you click to the ```Dashboard``` menu in the left sidebar, you should see an empty page with a button __Create a dashboard__: a Dashboard is nothing more than a set of visualizations. Let's create a new one! If you click on ```Add```, a message appear:

	This dashboard is empty. Let's fill it up!
	Click the Add button in the menu bar above to add a visualization to the dashboard.

##### Create a visualization
Imagine you want to discover how many files you have for each extensions with a simple histogram. Click on ```Add``` and then on ```Add new visualization``` (both blue buttons): then select your index - the one you create with the command line (see below) or via Kibana ```Management``` section - then add a new Bucket on X-Axis. Select as aggregations from the select box, in this case ext.keyword (see documentations about that) and then click play in the upper part to see live your visualization!

<div class="img_container"><img src="https://i.imgur.com/ac4H2FN.png" style="width: 100%; marker-top: -10px;"/></div>

Here's mine: you can create whatever you want and save the visualizations to reused in different dashboard. Then, you can schedule a raspberry to run command over your attached storage in the night, and find a beautifull dashboard in the morning with all the stats you need.

##### Indexing
To make search over text field you may use Mapping APIs - more info [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html). I created my own version using this call:

{{< highlight bash >}}
curl -XPUT 'localhost:9200/system?pretty' -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "files": {
      "properties": {
        "name":    { "type": "text", "fielddata": true },
        "ext":     { "type": "text", "fielddata": true , "fields": { "keyword": { "type": "keyword" } } },
        "size":    { "type": "integer" },
        "ctime":  {
          "type":   "date",
          "format": "strict_date_optional_time||epoch_millis"
        },
        "atime":  {
          "type":   "date",
          "format": "strict_date_optional_time||epoch_millis"
        },
        "mtime":  {
          "type":   "date",
          "format": "strict_date_optional_time||epoch_millis"
        }
      }
    }
  }
}
'
{{< / highlight >}}

If you try - for instance, to retrieve every Python source file in your file systems - you can do a GET request to ```curl -XGET 'localhost:9200/system/files/_search?q=*py&sort=name:asc&pretty'``` and you should get a response with hits!

Thank you everybody for reading!

[^est]: If you have any trouble with Elasticsearch try look at the [official installation guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html)

[^kt]: If you have any trouble with Kibana try look at the [official installation guide](https://www.elastic.co/guide/en/kibana/current/install.html)




