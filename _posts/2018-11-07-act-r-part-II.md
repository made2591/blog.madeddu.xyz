---
layout: post
title: "ACT-R by John R. Anderson - Part II"
categories: [theory, cognitive]
tags: [theory, cognitive, architecture, reasoning, knowledge, representation]
---

### Introduction
In my [previous post](https://madeddu.xyz/posts/act-r-part-I) I wrote about the cognitive architecture ACT-R, mainly putting together what I learnt by research over the topic. In this post, I would like to go more in depth about how ACT-R works, the concepts behind and try to provide my interpretation of some technical examples, regarding coding of the modeling and everything related.

#### What really is ACT-R
ACT-R is a production system theory that tries to explain human cognition by developing a model of the knowledge structures that underlie cognition. There are two types of knowledge representation in ACT-R:

- Declarative knowledge;
- Procedural knowledge;

Declarative knowledge corresponds to things we are aware we know and can usually describe to others.  Examples of declarative knowledge include sentence like:

George Washington was the first president of the United States.
An atom is like the solar system.

Procedural knowledge is knowledge which we display in our behavior but which we are not conscious of. For instance, no one can describe the rules by which we speak a language and yet we do. In ACT-R declarative knowledge is represented in structures called ```chunks``` whereas procedural knowledge is represented in ```productions```. Thus, chunks and productions are the basic building blocks of an ACT-R model.

This blog post aims to go more in depth about the formal notation used for specifying chunks and production rules and to describe how the two types of knowledge interact to produce cognition.

#### Chunks a.k.a. declarative knowledge
In ACT-R, elements of declarative knowledge are called chunks. Chunks represent knowledge that a person might be expected to have when they solve a problem. A chunk is defined by two elements:
- its **type**: you can think of types as categories (e.g., birds);
- its slots: you can think of slots as category attributes (e.g., color or size);

Look at the example:
{% highlight prolog %}
Action023:
    isa chase
    agent dog
    object cat
Fact3+4:
    isa addition-fact
    addend1 three
    addend2 four
    sum seven
{% endhighlight %}

Below are chunks that encode the facts that the dog chased the cat and that 4+3=7. The type of the first chunk is chase and its slots are agent and object. The isa slot gives the type of the chunk. The type of the second chunk is addition-fact and its slots are addend1, addend2, and sum.

#### Production rules a.k.a. procedural knowledge
There no a simple definition of a procedural rule. As we saw in previous post, they represent some how behaviour, *procedure*. Formally:
> A production rule is a statement of a particular contingency that controls behavior.

Look at the example:
{% highlight prolog %}
IF the goal is to classify a person
    and he is unmarried
THEN classify him as a bachelor

IF the goal is to add two digits d1 and d2 in a column
    and d1 + d2 = d3
THEN set as a subgoal to write d3 in the column
{% endhighlight %}

The condition of a production rule (the IF part) consists of a specification of the chunks in various buffers.    The action of a production rule (the THEN part) basically involves the modifications of those chunks or requests for other chunks.  The above are informal English specifications of production rules.  They give an overview of what the production does in the context of the declarative memory structures used, but do not necessarily detail everything that needs to happen within the production. You will learn the syntax for precise production specification within the ACT-R system.

### Formalism
Since ACT-R is even a language, then let's start with production rules.

#### Production rules: the format
A production rule is a condition-action pair. The condition (also known as the left-hand side) specifies a pattern of chunks that must be present in the buffers for the production rule to apply. The action (right-hand side) specifies some actions to take.

The buffers are the interface between the procedural memory system and the other components (modules) of the ACT-R architecture. For instance, the goal buffer is the interface to the goal module. Each buffer can hold one chunk at a time, and the actions of a production affect the contents of the buffers. In according to KISS principle, let's start by only concerning two buffers - one for holding the current goal and one for holding information retrieved from the declarative memory module.

The general form of a production rule is:

{% highlight prolog %}
(p Name
  list of buffer tests
==>
  list of buffer changes
)
{% endhighlight %}

The buffer tests consist of a set of patterns to match against the current buffers' contents. If all of the patterns correctly match, then the production is said to match and it can be selected. It is possible for more than one production to be selected, and from all the selected productions one will be chosen to fire and that production's actions will be performed. The process of choosing a production from those that are selected is call *conflict resolution*, and it will be discussed in detail in later units. For now, what is important is that

> Only one production may fire at a time.

After a production fires, selection and conflict resolution will again be performed and that will continue until the model has finished.

{% highlight prolog %}
(P example-counting                |    English Description
   =goal>                          |      If the goal is
      isa         count            |          to count
      state       counting         |          the current state is counting
      number       =num1           |          there is a number we will call =num1
   =retrieval>                     |         and a chunk has been retrieved
      isa         count-order      |          of type count-order
      first       =num1            |          where the first number is =num1
      second      =num2            |          and it is followed by another number
                                   |          we will call =num2
==>                                |      Then
   =goal>                          |          change the goal
      number       =num2           |          to continue counting from =num2
   +retrieval>                     |        and request a retrieval
      isa         count-order      |          of a count-order fact
      first       =num2            |          for the number that follows =num2
)
{% endhighlight %}

#### Production rules: the format
The condition of the preceding production specifies a pattern to match in the goal buffer and a pattern to match in the retrieval buffer:

{% highlight prolog %}
=goal>
    isa         count
    state       counting
    number      =num1
=retrieval>
    isa         count-order
    first       =num1
    second      =num2
{% endhighlight %}

A pattern starts by naming which buffer is to be tested followed by $$>$$. The names *goal* and *retrieval* specify the goal buffer and the retrieval buffer. It is also required to prefix the name of the buffer with $$=$$ - more details on this later. After naming a buffer, the first test must specify the chunk-type using the *isa* test and the name of a chunk-type. That may then be followed by any number of tests on the slots for that chunk-type. A slot test consists of an optional modifier (which is not used in any of these tests), the slot name and a specification of the value it must have. The value may be either a specific constant value or a variable.

Thus, this part of the first pattern:

{% highlight prolog %}
=goal>
    isa         count
    state       counting
{% endhighlight %}

means that the chunk in the goal buffer must be of the chunk-type count and the value of its state slot must be the explicit value counting. The next slot test in the goal pattern involves a variable:

{% highlight prolog %}
number      =num1
{% endhighlight %}

The *=* prefix in a production is used to indicate a variable. Variables are used in productions to test general conditions. They can be used to test that a slot holds any value, that two slots hold the same value or that two slots hold different values. The name of the variable can be any symbol and should be chosen to help make the purpose of the production clear. A variable is only meaningful within a specific production. The same variable name used in different productions does not have any relation between the two uses.

The first time a variable is used in a production it gets assigned (bound to) the value of the specified slot from the chunk in the buffer. If the slot does not have a value, then the pattern does not match. Further uses of that variable within the production will be tests against the specific value to which it is bound.

So, this slot test from the goal pattern:

{% highlight prolog %}
number      =num1
{% endhighlight %}

causes the variable called =num1 to be bound to the current value of the number slot from the chunk in the goal buffer, if it has a value.

Now, we will look at the retrieval buffer's pattern in detail:

{% highlight prolog %}
=retrieval>
    isa         count-order
    first       =num1
    second      =num2
{% endhighlight %}

First it tests that the chunk is of type count-order. Then it tests the first slot of the chunk with the variable =num1. Since that variable was bound in the goal test this is testing that this slot has that same value. Finally, it tests the second slot which will bind its value to the =num2 variable.

In summary, this production will match if the goal is of type count, the chunk in the retrieval buffer is of type count-order, the chunk in the goal buffer has the value counting in its state slot, the value in the number slot of the goal and the first slot of the retrieval buffer match, and there is a value in the second slot of the retrieval buffer.

One final thing to note is that =goal and =retrieval, as used to specify the buffers, are also variables. They will be bound to the chunk that is in the goal buffer and the chunk that is in the retrieval buffer respectively.

#### Action side
The right-hand side (RHS - the part after the arrow) or action side of a production consists of a small set of actions. The typical actions are to change the contents of the buffers as in our example:

{% highlight prolog %}
=goal>
    start       =num2
+retrieval>
    ISA         count-order
    first       =num2
{% endhighlight %}

The actions are specified similarly to the conditions. They start with the name of a buffer followed by ">" and then any number of slot and value specifications.

If the buffer name is prefixed with "=" then the action is to modify the chunk currently in that buffer. Thus this action on the goal buffer:

{% highlight prolog %}
=goal>
    start       =num2
{% endhighlight %}

changes the value of the start slot of the chunk in the goal buffer to the value of the =num2 variable.

If the buffer name is prefixed with "+" then the action is a request to the buffer's module. Typically this results in the module replacing the chunk in the buffer with a different one. Requests to the declarative memory module (the module for which the retrieval buffer is the interface) are always a request to retrieve a chunk from declarative memory that matches the specification provided and to place that chunk into the retrieval buffer. Different modules may handle different types of requests and may respond in other ways.

Thus, this request:

{% highlight prolog %}
+retrieval>
    ISA         count-order
    first       =num2
{% endhighlight %}

is asking the declarative memory module to retrieve a chunk which is of type count-order and with a first slot that has the value bound to =num2 and place it into the retrieval buffer. If there exists such a chunk, then it will be placed into the retrieval buffer.

### Conclusion

In Part III I will talk about how to install and run a model in ACT-R. Stay tuned!

Thank you everybody for reading!