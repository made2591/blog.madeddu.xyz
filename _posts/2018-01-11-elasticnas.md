---
title: "Elasticsearch over My home Network Attached Storage"
tags: [coding, elastic, kibana, search, nas]
---

### Introduction
I always owned a lot of hard drives: I don't know why, I always used and still use to look for space to save my data. In the years, I started using disks, then I create a proliant I don't want to go the cloud because I'm stupid, and so I decided to make order in my huge amount of files. The first thing you have to do when you are handling terabytes and terabytes of __both__ well-ordered __and__ _no-ordered-at-all_ of data is literaly pray that someone else, like a magician, or druid come to you with a magic wand and settles the mess for you, in a way you do not know but you'll like it. This article is the right one if you don't want to pray, but you still need to order your stuff. I have done it using elastisearch and kibana!

<p align="center"><img src="http://craigconnects.org/wp-content/uploads/W-S-files.jpg" style="width: 100%; marker-top: -10px;"/></p>

#### What you need before starting
Ok, here the ingredients:
- Elasticsearch: download available [here](http://www.elastic.co/downloads/elasticsearch);
- Kibana: download available [here](https://www.elastic.co/downloads/kibana);
- Python: actually, you don't need this if you are an AWK expert 😎😎😎;

#### Recipe
To order, you first have to make some sort of statistics: elasticsearch + kibana can really help you in doing that. Go to the folder in which you downloaded Elasticsearch binary and:

{% highlight bash %}

cd elasticsearch-<version>
./bin/elasticsearch

{% endhighlight %}

You can add -d if you want to run it in the background as a daemon - but you should prefer a docker container to create something always available in time. If you're running Elasticsearch on Windows, simply run ```bin\elasticsearch.bat``` instead. For the scope of this article, you're are simply done: ok, what?! Yes, your files are ok. Just kidding

####
