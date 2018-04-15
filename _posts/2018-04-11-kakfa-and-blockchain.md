---
title: "[Not completed yet] Implementing a Blockchain using Kafka"
tags: [coding, kafka, blockchain, experiment]
---

### Introduction
I recently started interesting about the concept of _streaming_ - not videos, of course - mainly because in the last months I collected several sources to be used as starting point: so, waiting for my Google Home Mini to be shipped, I decided to mix togheter two words I often see surfing the Web. The first is Kafka - that is a distributed streaming platform (ok but what exactly does that mean - _for real_? Be patience, I will try to introduce the tool in this article) and the second is Blockchain (I don't like this world, believe me or not I think it's like Big Data, in the sense that _everybody knows the principles but nobody wants to deal with the maths_): this two concepts have kind of _similarity_, I guess 🧐
I have a problem with memory, so I first talk about how Kafka works just to remind me the key concepts for the next month, because I use to forget everything I learn - d\*\*\*q.

<p align="center"><img src="https://s3-eu-west-1.amazonaws.com/spiked-online.com/kafka_review.jpg" style="width: 100%; marker-top: -10px;"/></p>

### Ingriedents
- Kafka;
- Blockchain (?) -> you don't need maths;

#### Kafka: reasons
A streaming platform has three key capabilities:
- Publish and subscribe to streams of records, similar to a message queue or enterprise messaging system: the publish–subscribe is a messaging pattern where senders of messages, called publishers, do not program the messages to be sent directly to specific receivers, called subscribers, but instead categorize published messages into classes without knowledge of which subscribers, if any, there may be. Similarly, subscribers express interest in one or more classes and only receive messages that are of interest, without knowledge of which publishers, if any, there are. We'll see that this are key features of the way Kafka actually works out of the box;
- Store streams of records in a fault-tolerant durable way: this is main due to the _distributed charateristics_ of the Kafka platform - more on this later;
- Process streams of records as they occur: simple? Not at all, but Kafka fortunately takes of this by design;

So, why people around the world use Kafka? To do exactly...what? The Apache official response is to _build real-time streaming data pipelines that reliably get data between systems or applications_ and to _build real-time streaming applications that transform or react to the streams of data_. Ok, but... a little bit useless, for common users. This is the main reason I used Kafka to experiment with Blockchain (more later on this boring thing). Just to be clear: I am NOT a Kafka expert and I am NOT a Blockchain fan, so please if you're a looking for a Blockchain maniac post with an accelerated course about elliptic curves, please don't waste your time XD this is probably not the right piece of Web for you.

#### Kafka: overview
So this Kafka sounds magical, it does these things, the point know is understanding _how_: first, Kafka runs as a cluster on one or more servers that can span multiple datacenters. Nothing special. Second, the Kafka cluster stores streams of records in categories called _topics_ - think about AWS SNS topic, it's really the same (shame on AWS! just kidding). Third, each record consists of a key, a value, and a timestamp. There Kafka box includes four core APIs:
- Producer APIs: to publish a stream of records to one or more Kafka topics;
- Consumer APIs: to subscribe to one or more topics and process the stream of records produced to them;
And these could be enough to implement the publish/subscribe mechanism, but there are also
- Stream APIs: to process _stream_ (?), aka consuming an input stream from one or more topics, do something on input stream, and producing an output stream to one or more output topics;
- Connector APIs: allows building and running reusable producers or consumers that connect Kafka topics to existing applications or data systems. For example, a connector to a relational database might capture every change to a table;

The communication between the clients and the servers is done with a simple, high-performance, language agnostic TCP protocol. The picture below is directly from the Kafka website - thank you guys!

<p align="center"><img src="https://image.ibb.co/juudSH/imageedit_9560608901.png" alt="perceptron" style="width: 100%; marker-top: -10px;"/></p>

#### Kafka: topics

    A topic is a category or feed name to which records are published.

Topics are always multi-subscriber, thus a topic can have zero, one, or many consumers that subscribe to the data written to it. 

__For each topic, the Kafka cluster maintains a partitioned log that looks like this__:

<p align="center"><img src="https://image.ibb.co/njQ7Mc/imageedit_1_7046918343.png" alt="perceptron" style="width: 75%; marker-top: -10px;"/></p>

Each __partition__ is an ordered, immutable sequence of records that is continually appended to a structured commit log. The records in the partitions are each assigned a sequential id number called the __offset__ that uniquely identifies each record within the partition.

The Kafka cluster durably persists all published records - whether or not they have been consumed — using a configurable retention period. For example, if the retention policy is set to 2 days, then for the 2 days after a record is published, it is available for consumption, after which it will be discarded to free up space. Kafka's performance is effectively constant with respect to data size so storing data for a long time is not a problem.

<p align="center"><img src="http://image.ibb.co/iv7ySH/imageedit_4_8265886645.png" alt="perceptron" style="width: 75%; marker-top: -10px;"/></p>

The only metadata retained on a per-consumer basis is the offset or position of that consumer in the log: this offset is controlled by the consumer. Thus, normally a consumer will advance its offset linearly as it reads records, but, in fact, since the position is controlled by the consumer it can consume records in any order it likes and also, eventually, reprocess data from the past, or skip ahead to the most recent record.

Kafka consumers are very cheap — they can come and go without much impact on the cluster or on other consumers, so you can use our command line tools to "tail" the contents of any topic without changing what is consumed by any existing consumers.

The __partitions__ in the log serve several purposes. First, they allow the log to scale beyond a size that will fit on a single server. Each individual partition must fit on the servers that host it, but a topic may have many partitions so it can handle an arbitrary amount of data. Second they act as the unit of parallelism.

#### Kafka: partitions
We already said but it's crucial to understand the meaning of the partition
  
    A partition is an ordered, immutable sequence of records that is continually appended to—a structured commit log

Ordered. Immutable. Sequence of records. Just keep these words in mind when you are thinking about a partition. Partitions are distributed over the servers in the Kafka cluster with each server handling data and requests for a share of the partitions. Each partition is replicated across a configurable number of servers for fault tolerance.

How this mechanism works and how the parallelism is accomplished? To answer this, we need the concept of _leader_ and _follower_.

Each partition has one server which acts as the "leader" and zero or more servers which act as "followers". The leader handles all read and write requests for the partition while the followers passively replicate the leader. If the leader fails, one of the followers will automatically become the new leader. Each server acts as a leader for some of its partitions and a follower for others so load is well balanced within the cluster.

### Kafka: actors
Ok, at this point you should have a _quite complete_ idea about what Kafka is, what are the features exposed, what APIs ara available, what kind of problems it try to resolve, etc etc. Let's talk about _actors_ of this distributed platform!

#### Kafka: producers
Producers publish data to the topics of their choice. The producer is responsible for choosing which record to assign to which partition within the topic. This can be done in a round-robin fashion simply to balance load or it can be done according to some semantic partition function (say based on some key in the record). More on the use of partitioning in a second!

#### Kafka: consumers
Consumers label themselves with a __consumer group name__, and each record published to a topic is delivered to one consumer instance within each subscribing consumer group. Consumer instances can be in separate processes or on separate machines. The are two rules:

- If all the consumer instances have the same consumer group, then the records will effectively be load balanced over the consumer instances: of course, this is simple to be done from Kafka side, because it can identifies more instances acting as kind of duplicated consumers;

- If all the consumer instances have different consumer groups, then each record will be broadcast to all the consumer processes: consumers are, from a Kafka perspective, different from each one, thus record are sent to everyone;

#### Kakfa: example
The picture from the Kafka doc shows a 2-server Kafka cluster hosting 4 partitions (named P0-P3) with 2 consumer groups. Consumer group A has 2 consumer instances and consumer group B has 4 four instances.

<p align="center"><img src="http://image.ibb.co/jnppZx/imageedit_2_8366438504.png" alt="perceptron" style="width: 70%; marker-top: -10px;"/></p>

A topic could be something like "news for a (fixed) neighborhood": let's say that there is a Consumer Group for those news - imagine this Consumer Group as a machine that, for instance, acts like a Telegram Bot sender on its behalf. The Consumer Group for this topic is unique so - eventually neighborhoods close to the given district may be interested too, but the reasoning I'm doing is to state _topics have a small number of consumer groups_. Of course, this dependes on the nature of the topic. In the example - actually, in real world - each Consumer Group is composed of many consumer instances, for scalability and fault tolerance. Think about this configuration as nothing more than publish-subscribe semantics where the subscriber is a cluster of consumers instead of a single process.

##### Kafka: consumption
The way consumption is implemented in Kafka is by _dividing up_ - not literally - the partitions in the server over the consumer instances so that __each instance is the exclusive consumer of a "fair share" of partitions at any point in time__. This process of maintaining membership in the group is handled by the Kafka TCP protocol dynamically. If new instances join the Consumer Group they will take over some partitions from other instances of the Consumer Group; if an instance dies, its partitions will be distributed to the remaining instances.

<span style="color:#A04279; font-size: bold;">__Note__</span>: Kafka only provides a total order over records within a partition, not between different partitions in a topic. Remember that __for each topic, the Kafka cluster maintains a partitioned log__, so records of 1 single topic are spread across 1 or more partitions. Per-partition ordering combined with the ability to partition data by key is sufficient for most applications. However, if you require a total order over records this can be achieved with a topic that has only 1 partition, though this will mean __only 1 consumer process per consumer group__. 

#### Kafka: gains and guarantees
Ok, why all these information about consumpation, topics, partitions, etc? Because Kafka provide us really interesting guarantees.

##### Guarantees

- Messages sent by a producer to a particular topic partition will be appended in the order they are sent. That is, if a record __M1__ is sent by the same producer as a record __M2__, and __M1__ is sent first, then __M1__ will have a lower offset than __M2__ and appear earlier in the log;
- A consumer instance sees records in the order they are stored in the log;
- For a topic with replication factor N, we will tolerate up to N-1 server failures without losing any records committed to the log;

##### Gains
Are Kafka streams better than classical messaging system? I would like to say "Yes, of course", but I use to think that in the end "it always depends" on what you need to do. In any case, messaging traditionally has two models: queuing and publish-subscribe.

| Paradigm | Pro | Cons |
|----------|-----|------|
| Queue   | Allows you to divide up the processing of data over multiple consumer instances, which lets you scale your processing. | Isn't multi-subscriber — once one process reads, the data it's gone. |
| Publish-subscribe   | Allows you broadcast data to multiple processes. | Has no way of scaling processing since every message goes to every subscriber. |

The Consumer Group concept in Kafka generalizes these two paradigms. As with a queue implementation, the Consumer Group allows you to divide up processing over a collection of processes (the members or instances of the Consumer Group itself). As with a publish-subscribe implementation, Kafka allows you to broadcast messages to multiple Consumer Groups.

What about the guarantees?

| Paradigm | Pro | Cons |
|----------|-----|------|
| Queue   | Retains records in-order on the server, so if multiple consumers consume from the queue then the server hands out records in the order they are stored. | The server hands out records in order, but the records are delivered _asynchronously_ to consumers, so they may arrive out of order on different consumers: this effectively means the ordering of the records is lost in the presence of parallel consumption. |

In Kafka, the notion of parallelism is defined within the topics in the partitions - so by design. Kafka is able to provide both ordering guarantees and load balancing over a pool of consumer processes. This is achieved by assigning the partitions in the topic to the consumers in the consumer group so that each partition is consumed by exactly one consumer in the group. By doing this Kafka ensures that the consumer is the only reader of that partition and consumes the data in order. Since there are many partitions this still balances the load over many consumer instances. Note however that there cannot be more consumer instances in a consumer group than partitions.

Further, Kafka allows producers to wait on acknowledgement so that a write isn't considered complete until it is fully replicated and guaranteed to persist even if the server written to fails. A retail application might take in input streams of sales and shipments, and output a stream of reorders and price adjustments computed off this data.

Some other Kafka features are Geo-Replication, Multi-tenancy and others but - I'm honest - I didn't study these topics because they are not related to the experiment I want to doing after with Blockchain.

#### Blockchain: reasons
I promise, these are the only words I want to say about Blockchain:

    A Blockchain is a continuously growing list of records, called blocks, which are linked and secured using cryptography. Each block typically contains a hash pointer as a link to a previous block, a timestamp and transaction data.

Do you see some similarities with something you just read somewhere else :)? Despite the fact that this concept are relatively simple, I read a lot about block chain but in the end I always tried to learn by doing: this is to say you can't really know how a Blockchain works without build your own. So...let's build a Blockchain!

##### Blockchain: overview
First of all, thanks a lot to [Daniel van Flymen](https://hackernoon.com/learn-blockchains-by-building-one-117428612f46) for its incredible explanation about how a Blockchain works, so please have a look at his beautiful work: you can even check out the entire code he prepared in [its repository](https://github.com/dvf/blockchain). I followed its step to understand the basics, so please put apart the Kafka part for the moment.

##### Blockchain: one file required
Daniel created a single Python file containing a Blockchain class whose constructor creates an initial empty list (to store your blockchain), and another to store transactions. The Blockchain class is responsible for managing the chain. It will store transactions and have some helper methods for adding new blocks to the chain. Ok, wait a moment: we already know what a Blockchain is, but we didn't talk about blocks.

##### Blockchain: blocks
What does a Block look like? A Block is no more than an _object_ with an index, a timestamp (in Unix time), a list of transactions, a proof (more on that later), and the hash of the previous Block. Following Daniel guide, here's an example of what a single Block looks like:
{% highlight python %}
block = {
    'index': 1,
    'timestamp': 1506057125.900785,
    'transactions': [
        {
            'sender': "8527147fe1f5426f9dd545de4b27ee00",
            'recipient': "a77f5cdfa2934df3954a5c7c7da5df1f",
            'amount': 5,
        }
    ],
    'proof': 324984774000,
    'previous_hash': "2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824"
}
{% endhighlight %}

Thus, each new block contains within itself, the hash of the previous Block. This is crucial because it's what gives blockchains immutability: if an attacker corrupted an earlier Block in the chain then all subsequent blocks will contain incorrect hashes. So what? Well, this will make sense later, but the point is that there is only a valid chain. You can notice also that a block contains a list of transactions. What is a transaction?

##### Blockchain: transactions
A transaction is a concept often related to virtual money: it could be anything, you can think about the transactions list as the always growing list of data to be validated and manipulated during time, with a specific constraint: they have to be immutable, consistent, and stored forever. The concept of money transaction fits well with these needs, because of its nature characteristics. So, the next step is to provide your class a way to a way of adding transactions to a Block. The new_transaction() method is responsible for this, and it's pretty straight-forward to implement by your-self. After new_transaction() adds a transaction to the list, it returns the index of the block which the transaction will be added to — the next one to be mined. This will be useful later on, to the user submitting the transaction.

##### Blockchain: create a block
When your Blockchain is instantiated you need to seed it with a - so called - __genesis block__ — or block with no predecessors. You also need to add a "proof" to your genesis block which is the result of mining (or Proof of Work). What does it mean mine? Wait for it :D
In addition to creating the genesis block in your constructor, you also have to provide three methods: new_block(), new_transaction() and hash().
The name are self-explanotory, you can have a look at the code directly. At this point, you must be wondering how new blocks are created, forged or mined.

##### Blockchain: Proof of Work
A __Proof of Work__ algorithm (PoW) is the way used to create or mine new Blocks on the Blockchain. The goal of PoW is to discover a number which solves a problem. The number must be difficult to find but easy to verify — computationally speaking — by anyone on the network. This is the core idea behind Proof of Work and yes, it's really similar to the concept behind RSA, that is based on prime numbers and hardly invertible functions.

Let's decide that the hash of some integer _x_ multiplied by another _y_ must end in 0. So, hash(x * y) = gdf798...0. And for this simplified example, let's fix x = 5. Implementing this in Python:

{% highlight python %}
from hashlib import sha256
x = 5
y = 0  # We don't know what y should be yet...
while sha256(f'{x*y}'.encode()).hexdigest()[-1] != "0":
    y += 1
print(f'The solution is y = {y}')
{% endhighlight %}

The solution here is y = 21. Since, the produced hash ends in 0:

{% highlight python %}
hash(5 * 21) = 1253e9373e...5e3600155e860
{% endhighlight %}

In Bitcoin, the Proof of Work algorithm is called Hashcash and it's not too different from the basic example above. It's the algorithm that miners race to solve in order to create a new block. In general, the difficulty is determined by the number of characters searched for in a string. The miners are then rewarded for their solution by receiving a coin—in a transaction. The network is able to easily verify their solution.

The implementation of the Proof of Work is really simple: the rule used by Daniel is "find a number _p_ that when hashed with the previous block's solution a hash with 4 leading 0s is produced.".

At this point, your class is almost complete and you're ready to begin interacting with it using HTTP requests.

Using the Flask microframework you can easy map HTTP endpoints to Python functions allowing you talking to your blockchain over the web using HTTP requests. This is done by providing three routes (actually, there are more than three)
- /transactions/new to create a new transaction to a block
- /mine to tell our server to mine a new block.
- /chain to return the full Blockchain.

The transaction endpoint accepts request for a transaction that looks like this:

{% highlight json %}
{
 "sender": "my address",
 "recipient": "someone else's address",
 "amount": 5
}
{% endhighlight %}

The mining endpoint is where the magic happens, and it's easy to understand what it does:

- calculate the Proof of Work;
- reward the miner (us) by adding a transaction granting us 1 coin;
- forge the new Block by adding it to the chain;

The recipient of the mined block is the address of your node. And most of what we've done here is just interact with the methods on the Blockchain class. But...a Blockchain is decentralized: at the moment, even if more than one node is running over the same network, it mantains a private version of the chain. And this is the point arised at the very beginning: how can multiple nodes share the same - actually, the unique real transactions chain? They can't without a mechanism providing a sort of consensus, implemented by an algorithm (the Consensus Algorithm, sounds good?) to let multiple node work over the same - decentralized - chain.

##### Blockchain: consensus
The first point to deal with consensus is to deal with other nodes in the network. So, before the implementation of Consensus Algorithm, you need a way to let a node know about neighbouring nodes on the network. Each node on our network should keep a registry of other nodes on the network. Thus, you'll need some more endpoints:

- /nodes/register to accept a list of new nodes in the form of URLs;
- /nodes/resolve to implement our Consensus Algorithm, which resolves any conflicts—to ensure a node has the correct chain;

The Blockchain class has also a set() to hold the list of nodes. This is a cheap way of ensuring that the addition of new nodes is idempotent—meaning that no matter how many times we add a specific node, it appears exactly once. It is really simple to implement the logic to register new nodes

As mentioned, a conflict is when one node has a different chain to another node. To resolve this, we'll make the rule that the longest valid chain is authoritative. In other words, the longest chain on the network is the de-facto one. Using this algorithm, we reach Consensus amongst the nodes in our network.

So with the valid_chain() method you can check if a chain is valid by looping through each block and verifying both the hash and the proof. With the resolve_conflicts() method you can loops through all your neighbouring nodes, downloads their chains and verifies them using the above method. If a valid chain is found, whose length is greater than your, you replace your chain with the longest and so on. And that's all.

### Let's build a Kafka-based Blockchain
Again, I want first to thanks the author of this experiment so [Luc Russell](https://hackernoon.com/a-blockchain-experiment-with-apache-kafka-97ee0ab6aefc) for its incredible explanation about how a to use Kafka as transport layer for the Blockchain, so please have a look at his beautiful work: you can even check out the entire code he prepared in [its repository](https://github.com/lucrussell/kafka-blockchain). I followed its step to understand the basics, and extend its file to provide a complete example of multiple nodes interacting between each others.

On startup, your Kafka consumer will try to do three things: initialize a new blockchain if one has not yet been created; build an internal representation of the current state of the blockchain topic; then begin reading transactions in a loop. The initialization step looks for the highest available offset on the blockchain topic. If nothing has ever been published to the topic, the blockchain is new, so it starts by creating and publishing the genesis block. 

The read_and_validate_chain() method does two things:

- first, it creates a consumer to read from the blockchain topic;
- second, it begins reading block messages from the blockchain topic;

The initialization step setup the Consumer Group to the blockchain group to allow the broker to keep a reference of the offset the consumers have reached, for a given partition and topic. The auto_offset_reset=OffsetType.EARLIEST indicates that the node begins downloading messages from the start of the topic. The auto_commit_enable=True lets periodically notify the broker of the offset we've just consumed (as opposed to manually committing). The reset_offset_on_start=True is a switch which activates the auto_offset_reset for the consumer. 

#### Docker compose
The Docker-compose















