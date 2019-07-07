---
layout: post
title: "The Good Employee, a story about how you can explain modern companies with graph theory"
categories: [miscellaneous, life]
tags: [learning, approach, work, thoughts, life, graph]
---

### Prelude
This is a twin post of - I guess - [a good one](https://madeddu.xyz/posts/doing-vs-learning/) that I wrote a long time ago: I was just *surfing the blog* thinking about all I would like to do, and I came to my old thoughts. I was curious, you know? so I read it, and read it once again, I reflected a bit on it - and I found it *inspiring* in a sense: even better, I would say I found myself *surprised* to agree with myself of almost 1 year and a half ago, in most of the things I wrote. For me, it was an important moment because it has been like a kind of *retrospective*. This is the reason I wanted to give this new *lifestyle-kind-of post* just the same title as his father. Like: *I'm still learning*. Or even better: *I'm still learning - Revenge of the Fallen* but I wanna warn you, once again - this is a deeply full-of-truth-and-complaints post - ok no, just kidding. This is more kind of a story, in five Chapters.

![jobs](https://i.imgur.com/HeQ1mYo.png)

### Chapter I - About companies
In the last year, I still fought against the processes. But this time, let's analyze the things from a mathematical perspective - because today if you don't speak about data and models, nobody hears you. Let's give the theoretical machine learning definition of a modern AI company: a modern AI company is a WUG - a.k.a. an Weighted Undirected Graph - that links at least three different families of nodes - Employees, Processes and Projects - in a chain of skills, troubles, goals, beers, prizes, achievements, careers, promotions, pizza, parties, lies and God only knows what else, with the overall goal to solve (or introduce) dependencies, investigate (or ignore) consequences but, by the end of the day, feed the so hungry desire to "change something" - also referred to as "bring something in production". Unfortunately, before going ahead we need to provide a formal definition of a WUG - sorry, a Company. Thus...

Formally, we define a Company as a pair:

$$C = (V, E)$$ 

where:

1. $$V = \{Pj_A, ..., Pj_Z, Pr_1, ..., Pr_n, E_1, ..., E_m\}$$ is the set whose elements are called vertices or nodes, that can be of three kind, with:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- $$\{Pj_i \mid \forall\: Pj_i \equiv Pj_{[A-Z]}\}$$ refers to Projects with cardinality $$\propto$$ the budget of the year;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- $$\{Pr_i \mid \forall\: Pr_i \equiv Pr_{[1-n]}\}$$ refers to Processes with cardinality $$\propto$$ the cost of Service Now;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- $$\{E_i \mid \forall\: E_i \equiv E_{[1-m]}\}$$ refers to Employees with cardinality $$\propto$$ the headcount allowed by HR;

1. $$E = \{[1\: of\: (Pj_{A..Z} \mid Pr_{1..n} \mid E_{1..m})] \leftrightarrow [1\: of\: (Pj_{A..Z} \mid Pr_{1..n} \mid E_{1..m})], ...\}$$ is the set of two-sets (set with two distinct elements) of vertices, whose elements are called edges, that link together the nodes (Employees, Processes and Projects). Even if there's no formal demonstration, some studies suggest the cardinality of $$E$$ is strongly influenced by the cardinality of $$V$$, the budget, the headcount, and *God only knows what else*;

Furthermore, our Company is a WUG and in a *Weighted Graph* a number, the weight, is assigned to each edge. For the moment, it's sufficient for you to have in mind that such weights, in graph theory, might represent for example costs, lengths or capacities, depending on the problem at hand. Also, we can *ignore* the formalization of the three different kinds of nodes, but not completely, because I will get back on it later. There's a name for structures that are *almost like this* and it's funny because are called *Forests*. Confused? No worries, it's because you're still not a good Employee 👀

<div class="img_container"><img src="https://i.imgur.com/irKtT5B.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Chapter II - About employees
Ok, so in a digital Forest like the one just described, you cannot move with weapons to defend yourself against pumas - or whatever else lives inside a real forest with the desire of eating you and your backpack full of energy bars. Instead, the good Employee inside the WUG. Sorry, the Forest. Sorry again, in the Company, uses his ability and applies the Prim's algorithm. Now, before going ahead, it could be useful to remind some concepts about graph exploration. 

The Prim's algorithm is a greedy algorithm that finds a minimum spanning tree for a weighted undirected graph. The fact is, *the good Employee* is really good in this kind of exploration, even if (s)he sucked in the exam of Algorithm and Data Structure. What does exactly *finding the minimum spanning tree* mean? The following is the best explanation I can provide - I hope it will be clear, if it's not please don't blame me, I'm a bad Employee.

> Finding the minimum spanning tree means finding a subset of the edges that forms a tree that includes every vertex, where the total weight of all the edges in the tree is minimized.

Clear? If I didn't lose you already, you should now ask yourself "What the hell these weights refer to now?!"

For the sake of truth, I mentioned the weights before, without explicitly saying these were the weights. I will list them with names and values. Imagine the weights as fixed distinct discrete values - with names, because it's easier to reasoning over names than values. I chose to put the values in the range [-50, +50] with a step of 10 because it's my article and I do what I want. To be also compliant, I would avoid giving value to some of the weights: it's up to every good Employee to look into his/her heart and put the value for them. There's also a Positive Value column: basically, it's built by shifting the Real Value column in the positive range [0, +100]. This is only because many of the algorithms that operate over graphs need positive weights: I keep it also the negative ones for small reasoning after the table.

| Id | Weights Enum Names       | Real Value | Positive Value |
|----|--------------------------|------------|----------------|
| 01 | Skills                   |     NaN    |       NaN      |
| 02 | Troubles                 |     +30    |       +80      |
| 03 | Goals                    |     -20    |       +30      |
| 04 | Beers                    |     -30    |       +20      |
| 05 | Prizes                   |     -50    |         0      |
| 06 | Achievements             |     -30    |       +20      |
| 07 | Careers                  |     Nan    |       Nan      |
| 08 | Promotions               |     Nan    |       Nan      |
| 09 | Responsibilities         |     +30    |       +80      |
| 10 | Pizza                    |     -15    |       +35      |
| 11 | Parties                  |     -25    |       +25      |
| 12 | Meetings                 |     +10    |       +60      |
| 13 | Lies                     |     Nan    |       Nan      |
| 14 | God only knows what else |     Nan    |       Nan      |

Of course, you can add as many weights as you want. The first thing to note, if you consider the Real Value column, is that we want to minimize the overall cost of *spanning the projects*, and this explains the positive values - something like troubles need to be avoided because who wants troubles?! - and of course prizes, goals, achievements are the things every good Employee would like to gain/reach/obtain, and this explains the negative values. Responsibility? Who wants them? It's better having pizza, parties, and meetings - ok on the last wait, it depends. Got it? 😉 Back to the algorithm, we defined what the Prim's algorithm does, but we didn't explain how it works. Well, let's see the algorithm.

<div class="img_container"><img src="https://i.imgur.com/aTwvuzi.jpg" style="width: 100%; marker-bottom: -10px;"/></div>

### Chapter III - About the algorithm
The Prim's algorithm works by building this tree one vertex at a time, from an arbitrary starting vertex, at each step adding the cheapest possible connection from the tree to another vertex. It's really good actually because it's

- **greedy**, because it evaluates each time the best local solutions without questioning the previous choices;
- **exact**, because it provides a precise solution for each instance of the problem without making rounding or inaccuracies of any other nature;
- **optimal**, because it's used by the good Employee. Just kidding, it's because it presents the best solution (or, one of them if they are more than one);

Anyway, the algorithm follows this steps:

1. Start with a Company;
2. Choose an Employee node;
3. Choose the shortest edge from this Employee and add it;
4. Choose the nearest vertex not yet in the solution;
5. Choose the nearest edge not yet in the solution, if there are multiple choices, choose one at random;
6. Repeat until you have the minimum spanning tree;

If you remember, in the beginning, I said: "We have three different kinds of nodes: forget about this for the moment". Ok, now going back to this assumption, what we gonna do is actually *change the Prim's algorithm*, in particular, the behavior of step 5, to enables priorities and be more *Company-compliant*. Thus, we wanna have a procedure that collects as much as possible `Project` nodes, without touching a lot of `Employee` nodes, and always trying to avoid as many `Processes` nodes as it can - but ATTENTION PLEASE!!! this *deeply depends* on the kind of interpretation you wanna give to the weights, the `Employee` nodes and *God only knows what else*. In the good Company, it seems that collecting the right `Project` nodes give more direct benefits than deal with the wrong `Employee` nodes, by passing through tricky `Processes` nodes: what a coincidence 🤔

Let's write the procedure to implement the Extended Prim algorithm - the good Employee variant.

{% highlight sh %}
minimum_spanning_tree = empty_set;
visited_nodes = { 1 };

# go ahead until the all the node are visited
while(visited_nodes != minimum_spanning_tree)

    # create the arch
    let new_arch (u, v) = the lowest cost edge | 
        u ∈ visited_node and \
        v ∈ minimum_spanning_tree - visited_node \
        # this is the change introduced.
        with priority given to u by following the order
        [Project, Employee, Process];

    # add it to the minimum_spanning_tree
    minimum_spanning_tree = minimum_spanning_tree ∪ {(u, v)};

    # mark the node as visited
    visited_node = visited_node ∪ {v}

{% endhighlight %}

Back to the structure, now that the Company $$C$$ is formalized, the vertices $$V$$ are defined, the links $$E$$ between them are weighted, and the algorithm is presented, I think we can make an example and hopefully we will start the light at the end of this crazy tunnel. Let's draw a Company $$C$$.

<div class="img_container"><img src="https://i.imgur.com/sQKw0pT.png" style="width: 75%; marker-bottom: -10px;"/></div>

Ok, to keep things simple, since we have a Company $$C$$, let's *isolate* some entities as shown in picture below. We choose an Employee node: let's take the Employee n.3 (`Emp\_3`). (S)he knows that work on Project B (`Pj\_B`) could change his/her career in better (*weight\_ID*: 07 $$\rightarrow$$ `Careers`. It's...how much? -30 (+20)? Or +30 (+70)? I think it depends on Employee n.3's life priorities) but (s)he has to accept (s)he will need to go through the Process n.2 (`Pr\_2`), that will cause a lot of troubles (*weight\_ID*: 02 $$\rightarrow$$ `Troubles`). (S)He also knows that *collaboration* with Employee 1 (`Emp\_1`) will require a lot of meetings (*weight\_ID*: 12 $$\rightarrow$$ `Meetings`) and *God only knows what else* will happen if (s)he will go for the Project A (`Pj\_A`) (*weight\_ID*: 14 $$\rightarrow$$ `God only knows what else`) instead of Project B (`Pj\_B`). Since each good Employee populate his/her Company with weights the overall idea should be clear. 

<div class="img_container"><img src="https://i.imgur.com/3hOJK3H.png" style="width: 75%; marker-bottom: -10px;"/></div>

### Chapter IV - About the Employee n.15
Now let's create an agnostic example and put some random weights: I also put some integer labels to all the nodes... because I'm not a good Employee and I need help in executing the Prim's Algorithm. This time, let's play the role of `Employee n.15`: (s)he has to find this *minimum spanning tree* inside this piece of Company $$C$$ to make things work.

<div class="img_container"><img src="https://i.imgur.com/vS4ZO4Q.png" style="width: 62%; marker-bottom: -10px;"/></div>

Let's follow the Prim steps one by one using the labels we put and this magic library called [the Good Employee](https://github.com/valandro/python-prim) on Github. I used it with the following `.wug` (Weighted Undirected Graph) file I prepared to get the correct *minimum spanning tree*. So fuck you, `Employee n.15` 🤟

{% highlight sh %}
1 3 20
1 6 35
3 2 20
2 4 30
2 7 60
4 7 20
4 6 1
6 5 15
6 9 15
6 10 20
9 10 55
7 9 10
7 8 80
10 11 60
{% endhighlight %}

Ok so, let's analyze the goodness of the `Employee n.15` step by step.

<table>
<colgroup>
<col width="3%" />
<col width="40%" />
<col width="57%" />
</colgroup>
<thead>
<tr>
<th markdown="span">Step</th>
<th markdown="span">The `Employee n.15`'s mind</th>
<th markdown="span">The `minimum spanning tree`'s choices</th>
</tr>
</thead>
<tbody>
<tr>
<td markdown="span">1</td>
<td markdown="span">At the very beginning, the `Employee n.15` has no choice: (s)he has to complete the `Process 1` to join the rest of the network in the Company. Only by chance, this process is not one of the worst, and it will cost to `n.15` a good +20, that is *not bad* - if (s)he doens't collect even three stars in the upper right corner of the screen.</td>
<td markdown="span"><img src="https://i.imgur.com/ejmso7X.png" style="width: 100%; marker-top: -10px;"/></td>
</tr>
<tr>
<td markdown="span">2</td>
<td markdown="span">To gain the chance on working on `Project A`, the `n.15` needs to *ask for help* to `Employee n.1`, even if (s)he will cost him/her a favor. Beside that, (s)he already has a +40 but no other paths could be taken :/</td>
<td markdown="span"><img src="https://i.imgur.com/NqpjlZk.png" style="width: 100%; marker-top: -10px;"/></td>
</tr>
<tr>
<td markdown="span">3</td>
<td markdown="span">At this point, the `n.15` can finally go straight and conquer the `Project A`. This costs him/her so much less than follow the `Process n.2`, that is based on Service Now and doesn't bring anything good because - remember the lesson - if you wait for things happening inside Service Now, you will be stucked FOREVER. Even at the same cost, (s)he would have preferred Projects in any case, because they are more valuable than Processes - remember we are running the Extended Prim we introduced before.</td>
<td markdown="span"><img src="https://i.imgur.com/mLevkP5.png" style="width: 100%; marker-top: -10px;"/></td>
</tr>
<tr>
<td markdown="span">4</td>
<td markdown="span">Unfortunately, `Project A` is a complete mess. Nobody knows anything, nobody has ownership of things and nothing is working since the beginning of time. But `n.15` is really good and (s)he knows it doesn't cost a lot *working with* `Employee n.13`: in fact, `n.13` is an intern and (s)he's given for free. In any case, for the same cost of `Employee n.13` (s)he would have preferred even paying a contractor just not to go through the `Process n.2`.</td>
<td markdown="span"><img src="https://i.imgur.com/1L1v9vg.png" style="width: 100%; marker-top: -10px;"/></td>
</tr>
<tr>
<td markdown="span">5</td>
<td markdown="span">To respect mandatory company policies, the intern `n.13` has to follow the `Process n.3` that is long, useless, and time-consuming. What a shame: (s)he will really like to join the Company $$C$$?</td>
<td markdown="span"><img src="https://i.imgur.com/rTscJq0.png" style="width: 100%; marker-top: -10px;"/></td>
</tr>
<tr>
<td markdown="span">6</td>
<td markdown="span">The fact is, working with `Employee n.13` has some side effects: in fact, since `n.13` has nothing to lose, (s)he prefers going for a cool `Project B` even only to finally start being paid for his/her services. `n.15` accepts that `n.13` is working 30% of his/her time in `Project B` because - the hell - *it* was given for free XD</td>
<td markdown="span"><img src="https://i.imgur.com/HCg5xMh.png" style="width: 100%; marker-top: -10px;"/></td>
</tr>
<tr>
<td markdown="span">7...10</td>
<td markdown="span">And so on until the last node is visited. With overall cost of 271.</td>
<td markdown="span"><img src="https://i.imgur.com/HzS8HIB.png" style="width: 100%; marker-top: -10px;"/></td>
</tr>
</tbody>
</table>

### Chapter V - About the minimum spanning tree
We spoke a lot about the minimum spanning tree, but it's actually the only entity I didn't explain inside the Company $$C$$. If you didn't get it, I already defined even this one... remember the definition of the modern AI company, that is a WUG - a.k.a. an Weighted Undirected Graph - that links at least three different families of nodes - Employees, Processes and Projects - in a chain of skills, troubles, [..] and God only knows what else, with the overall goal to solve (or introduce) dependencies, investigate (or ignore) consequences but, by the end of the day, feed the so hungry desire to "change something" - also referred to as "bring something in production". The latter is exactly the meaning of the minimum spanning tree: the "*let's work on something*", that drives every good Employee in the hard path of solving or introducing dependencies (like SAP), investigating by ignoring consequences (it works, don't touch it) but, by the end of the day, being able to say "I brought some changes" - that is translated in *(S)He brought something in production*.

And in a moment, somehow, I understood how every employee finds her/himself in this network of entities, and has to deal with all of them in a sense, better, (s)he has to take a real *direction*, building *the spanning tree*, that it's *minimum*, and this is... well, bad. I have this feeling every good Employee of every good Company is not so a good Employee by acting like this: yes, some of them apply [Djistrka](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm) (the original one, be careful) to go straight to the point, but by the end of the day they just wanna execute, avoid, survive, conquer, always by respecting the majority of the beliefs, but first always having someone or something else taking care of their shit. I think a Company should act by taking into consideration this *good Employees*' attitude. Because, the result is that nobody is able to deal with a world that is changing to fast, but is super good in graph exploration. Moreover, there is always someone who knows more than someone else, on this, on the other, who is more suitable than someone else. 

This state of confusion leads to the fear of hiring people who are not expert in something, that is not able to solve company's actual problem: that problem arose yesterday, come out months before, without nobody looking in the right direction, because the _minimum\_spanning\_tree_ rules them all. Year after year. Again, maybe I am a dreamer, but I think it's more interesting having someone with some interests and put him/her in the condition of learning how to solve problems in his/her field of competencies, instead of having people teaching or delegating all the time. Someone with the will of change (her)himself, instead of someone only able to *explore the graph* and yes, by the way, once again, this basically concerns how the people are chosen.

Ok, recruiters, you could be or not part of The company, but you are of course part of A company, I guess. Listen to me: it's not a matter of what a candidate can do, what he has done or he's actually doing: the employee should be evaluated more on how much they can learn and how fast they are in doing it than on how much they know and how much they can already do. Even my professor of Maths was not able to resolve the problems - at least, not so fast as its students - that he proposed to us after so many years of teaching. He has an incredible background, plenty of logic, theorems, fundamentals, etc. But he's not a machine: people forget things. The point is: how much fast are you in re-learning something? This what you should look for: evaluate people to hire - but also, and mostly (I guess) __already in__ the company - to find out if they are still interested in learning, in what they do for the company, and eventually why, if they are not anymore. The willingness to learn should be the ONLY choosing driver because the knowledge is ephemeral, the curiosity is not.

<div class="img_container"><img src="https://i.imgur.com/TS2ZOd9.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Epilogue
What can I say to conclude: in the age of flexibility, immutability, scalability, veganability, accountability, and everything else you can find in a job-post that doesn't coincide with the name of a Pokemon I can only wish you "Good exploration, explorers!". Unfortunately, I always hated graph theory and the exploration of data structures of any kind.

Bye bye

[^note]: Just to be clear, I think you can't learn to be intestered. It's a curse.