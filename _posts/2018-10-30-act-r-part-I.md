---
layout: post
title: "ACT-R by John R. Anderson - Part I"
categories: [theory, cognitive]
tags: [theory, cognitive, architecture, reasoning, knowledge, representation]
---

### Introduction
I've always been fascinated about cognitive systems and all the theories about them. Unfortunately, I never had the chance to actively work on a cognitive architecture: making experiments over these technologies is difficult because it's difficult to me even only think about some possible toyproblem to solve. So this article is more about the basics, or at least what I found interesting about the topic.

### ACT-R
One of the most famous cognitive architecture is ACT-R: ACT-R a.k.a. "Adaptive Control of Thought—Rational" is a cognitive architecture mainly developed by John Robert Anderson at Carnegie Mellon University. If you don't know Anderson, no worries but from now on keep in mind that he obtained a B.A. from the University of British Columbia in 1968, a Ph.D. in Psychology from Stanford in 1972 to finally become an assistant professor at Yale in 1972. This in the first 25 years of his life. This is to say: if you don't understand anything about what you will read, it's most probably not your fault, neither mine's...and neither Anderson's actually - it seems there's a bug in Matrix.

<div class="img_container"><img src="https://i.imgur.com/YNkVEk8.gif" alt="matrixbug" style="width: 100%; marker-top: -10px;"/></div>

ACT-R aims to define the basic and irreducible cognitive and perceptual operations that enable the human mind. In theory, each task that humans can perform should consist of a series of these discrete operations. As a cognitive architecture, ACT-R is actually a *theory* about *how human cognition works*. On the exterior, ACT-R looks like a programming language; however, its constructs reflect *assumptions* about human cognition. These assumptions are based on numerous facts derived from psychology experiments.

Like a programming language, ACT-R is a framework: for different tasks (e.g., Tower of Hanoi, memory for text or for list of words, language comprehension, communication, aircraft controlling), researchers create models (or *programs*) that are written in ACT-R and that, beside incorporating the ACT-R's view of cognition, add their own assumptions about the particular task. These assumptions can be tested by comparing the results of the model with the results of people doing the same tasks. By "results" we mean the traditional measures of cognitive psychology:

- Time to perform the task;
- Accuracy in the task;
- More recently, neurological data such as those obtained from FMRI - yes, it will definetly burn your brain but ehy, it's for science;

One important feature of ACT-R that distinguishes it from other theories in the field is that it allows researchers to collect quantitative measures that can be directly compared with the quantitative measures obtained from human participants.

ACT-R has been used successfully to create models in domains such as:

- Learning and memory;
- Problem solving and decision making;
- Language and communication;
- Perception and attention;
- Cognitive development;
- Individual differences;

Beside its applications in cognitive psychology, ACT-R has been used in

- Human-computer interaction to produce user models that can assess different computer interfaces;
- Education (cognitive tutoring systems) to "guess" the difficulties that students may have and provide focused help;
- Computer-generated forces to provide cognitive agents that inhabit training environments;
- Neuropsychology, to interpret FMRI data;

Some of the most successful applications, the Cognitive Tutors for Mathematics, are used in thousands of schools across the country. Such "Cognitive Tutors" are being used as a platform for research on learning and cognitive modeling as part of the Pittsburgh Science of Learning Center.

<span style="color:#A04279; font-size: bold;">__CONFUSED?__</span> Just go ahead, it will hopefully be clearer in a while.

### The architecture
The entire ACT-R architecture can be summarized in a set of *elements*:

- A visual module for *identifying objects* in the *visual field*;
- A manual module for controlling the *hands*;
- A declarative module for retrieving information from *memory*;
- A goal module for keeping track of current goals and intentions;
- A production module;

Is not a coincidence that this few things resemble a very simple *Wall-e*: in the end, we have eyes, we act in the worlds with hands, we decide by using the memory - of several kinds, in different ways - and we act to achieve a goal. As a - I would say complex - *Wall-e*.

<div class="img_container"><img src="https://i.imgur.com/MbLYQ6S.png"  style="width: 100%; marker-top: -10px;"/></div>

Let's start from the tricky one: the production module.

#### The production module
I think we all agree without be cognitive experts than one of the key points of the human thoughts is the *coordination* - at every level you can think about: there are several ways ours systems coordinate themself in relations with the I/O of others, and this is something complex to model. But remember, we are describing a framework, a theory... by design, some hooks are provided to let human fullfil with what is required to have an indipendent cognitive architecture.

Coordination in the behavior of these modules is achieved through a central production system: despite the name "central", you should not imaginge this system as a hole in which all information are stored / elaborated and somehow cross-joined.

> This central production system is not sensitive to most of the activity of these modules but rather can only respond to a limited amount of information that is deposited in the *buffers* of these modules - see again the picture above.

<span style="color:#A04279; font-size: bold;">__CONFUSED?__</span> No worries: actually, it isn't difficult to imagine. Is like saying you are not aware (hopefully) of all the information are in the visual field but only about the object(s) you are currently attending to. Similarly, people are not aware of all the information in long-term memory but only the fact currently *retrieved*.

#### The buffers and the modules
As we said, the central production system can recognize patterns in *buffers*: it can even make changes to these buffers, as, for instance, when it makes a
request to perform an action in the manual buffer. For cognitive reasons we are not interested right now, the information in these modules is largely encapsulated, and the modules communicate only through the information they make available in their buffers. Actually, the theory *is not committed to exactly how many modules there are inside*, but a number have been implemented as part of the central system. The important thing to remember is that:

> The buffers of these modules hold the limited information that the production system can respond to.

#### Real-brain -> modules mapping
As part of the architecture, we said there's a *goal module* to keep track of current goals and intentions: we will return on what this means later, but for the moment imagine that you - as a common *agent* - are in a sort of equilibrium - let's say you are in statis. You don't act to change the environment around you as far as you don't have a reason, or some sort of irrational trigger to do it, that could be identified by the consciuness, soul, whatever. The GOAL module, as all the module, comunicate throught a buffer. Let call it the GOAL buffer: this keeps track of one's internal state in solving a problem, it is associated with the dorsolateral prefrontal cortex (DLPFC). The retrieval buffer is associated with the ventrolateral prefrontal cortex (VLPFC) and holds information retrieved from long-term declarative memory. There are many reasons to keep this distinction between DLPFC and VLPFC valid: for our purpose, it's fair enought know there is a certain number of neuroscience results that agree on this.

#### Perceptual–Motor and manual modules
The perceptual-motor modules' buffers are based on some cognitive theories we don't want to focus on for now. The manual buffer is responsible for control of the hands and is associated with the adjacent motor and somatosensory cortical areas devoted to controlling and monitoring hand movement. There also are rudimentary vocal and aural systems. Let's summarize these modules as modules that share the same things: they somehow interact actively (by changing it) and/or passively (by notice the changes) of the environments and put some form of information in their respective buffers, to be accessible by the central production system.

<span style="color:#A04279; font-size: bold;">__CONFUSED?__</span> Me too. But stay tuned... the exciting part is coming. AGAIN :D

### The critical cycle in ACT–R
The ```critical cycle``` in ACT–R is one in which the buffers hold representations determined by the external world and internal modules. Then patterns in these buffers are recognized, a production fires, and the buffers are then updated for another cycle. The assumption in ACT–R is that this cycle takes about 50ms to complete. The conditions of the production rule specify a pattern of activity in the buffers that the rule will match, and the action specifies changes to be made to buffers.

<span style="color:#A04279; font-size: bold;">__NOTE__</span> This somehow resembles how an expert system works: have you ever worked with CLIPS, or any other logical language like Prolog? Hold on :)

<div class="img_container"><img src="https://i.imgur.com/3FPMeoC.jpg" alt="quantum" style="width: 100%; marker-top: -10px;"/></div>

### Mixture of parallel and serial
This is the coolest part of the system: within each module, there is a great deal of *parallelism*. For instance, the visual system is simultaneously processing the whole visual field, and the declarative system is executing a parallel search through many memories in response to a retrieval request. Also, the processes within different modules can go on in parallel and asynchronously.

This give to the architecture some sort of non-deterministic behaviour that to me keeps open the chance to model intentions. I mean, not only the way an Intention Module can be implemented: actually, a distribution over the way intentions are represented in there. Let's think about this: how much fast can be someone to avoid parallel - let's say - dangerous stimuli coming in the their visual field? It depends: if s/he's an athlete maybe faster and in a more efficient way, otherwise in a - kinda - *casual* way, guided by many other deterministic information - like how many hours you dreamt, or whatever, but still things - let's say - you are not interested at all in modeling. Using other words, let's imagine your actions are somehow a results of one or more production rule(s), i.e. in this context, encoded arbitrary behaviours that are part of your perceptual-motor modules (but even your emotional, etc). Then, the way they are picked up and fired, if subject to parallel access is eventual consistent by design. One of the interpretation over a way some casual can happen is to study the probability distribution, let call it the likelyhood this will happen. And actually, it's something we control because as we gonna discover in a while, is that the equations that guide ACT-R are strictly related to probability.

From an high level perspective, actually, it's a matter of chances being able to *do something* - even the definition of better is relative - instead of something else. I think this is really cool cause somehow keep open the possibilty of ACT-R as a model of not only *one expert systems*, but actually one system that occasionally act as an expert system. Like humans, that are not equally (and of course, I mean, even punctually) able to achieve each of them the goals their able to achieve if you think about them as a whole but as a single subject.

However, there are also two levels of serial bottlenecks in the system:

- First, the content of any buffer is limited to a single declarative unit of knowledge, called a ```chunk``` in ACT–R. Thus, only a single memory can be retrieved at a time or only a single object can be encoded from the visual field;
- Second, only a single production is selected at each cycle to fire: that is not such a big limiti if you think about it because we can somehow accept you're not gonna take exactly at the same moment two different decisions. To be a physic for a while, this even respects the theory of relativity that should state the conventional concept of simultaneity doesn't exist, if I'm not wrong 🤔

### A declarative Chunk and Activation function
The declarative memory system and the procedural system constitute the cognitive core of ACT–R. Let's have a look at a declarative chunk - or, a single declarative unit of knowledge. In the picture below there's a visual presentation of a declarative (again, see later) chunk with its subsymbolic quantities

<div class="img_container"><img src="https://i.imgur.com/vNj5pSe.png"  style="width: 75%; marker-top: -10px;"/></div>

where:
- $$W_j$$ is the ```attentional weights```;
- $$S_{ji}$$ is the ```strenghts of association```;
- $$B_i$$ is the ```base level activation```;

These are no more than values: keep them apart for a minute. Imagine that access to information in declarative memory is not instantaneous and **an important component of the ACT–R theory concerns the activation processes that control this access**. A set of equations and parameters have been devised in ACT–R that controls this activation process.

> The activation of a chunk is a sum of a base-level activation, reflecting its general usefulness in the past, and an associative activation, reflecting its relevance to the current context.

#### Activation function
For the base-level activation, it rises and falls with practice and delay according to the equation:

$$B_i = ln(\sum_{j=1}^{n}t_j^{-d})$$

where $$t_j is the time since the j_{th} practice of an item.$$ This equation reflects the log odds an item will reoccur as a function of *how it has appeared in the past*. This is to say, each presentation has an impact on odds that decays away as a power function (production the power law of forgetting) and different presentations add up (it turns out producing the power law of practice). Fair enough? Fri, in the ACT-R community, $$.5$$ has emerged as the default value for the parameter *d over a range* of applications.

There are two equations mapping activation onto probability of retrieval and latency. With respect to probability of retrieval, the assumption is

> The chunks will be retrieved only if their activation threshold is over a threshold.

This is because activation values are noisy: there is only a certain probability that any chunk will be above threshold. This is somehow resemble ANN, am I wrong? Have a look [here](https://madeddu.xyz/posts/neuralnetwork), I implemented and discussed a little bit a simple one of them here.

The probability that the activation will be greater than a threshold is given by the following equation:

$$P_i = \frac{1}{1 + e^{\frac{\tau-A_i}{s}}}$$

where $$$$ controls the noise in the activation levels and is typically set at about $$.4$$. If a chunk is successfully retrieved, the latency of retrieval will reflect the activation of a chunk. The time to retrieve the chunk is given as:

$$T_i = Fe^{-A_i}$$

with $$F \approx 0.35e^{\tau}$$

Although we have a narrow range of values for the noise parameter $$s$$, the retrieval threshold, and latency factor, $$F$$, are parameters that have varied substantially from model to model.

#### Procedural Memory
As we already said, the production system can detect the patterns that appear in the buffers and decide what to do next to achieve coherent behavior: in a sense, the production system achieves the control and adaptiveness of thought. The key idea is that at any point in time multiple production rules might apply, but because of the seriality in production rule execution, only one can be selected, and this is the one with the highest utility - that is somehow related to experience even. Production rule utilities are noisy, continuously varying quantities just like declarative activations and play a similar role in production selection as activations play in chunk selection.

The utility of a production rule $i$ is defined as:

$$U_i = P_iG - C_p$$

where:
- $$P_i$$ is an estimate of the probability that if production rule $$i$$ is chosen the current goal will be achieved;
- $$G$$ is the value of that current goal;
- $$C_i$$ is an estimate of the cost (typically measured in time) to achieve that goal;

Both $$P_i$$ and $$C_i$$ are learned from experience with that production rule.

### Conclusion

I want to go more in depth with some more details about knowledge representation, so stay tuned :D

UPDATE: [ACT-R - Part II](https://madeddu.xyz/posts/act-r-part-II) available now

Thank you everybody for reading!