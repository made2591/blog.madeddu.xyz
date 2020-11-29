---
layout: post
title: "My first UniKernel image for sequence prediction"
date: 2018-11-30
polly: https://madeddu.xyz/mp3/compact-prediction-tree.mp3
categories:
- coding
- golang
- algorithms
tags:
- coding
- golang
- unik
- cpt
- ml
- sequence
- prediction
---

### Introduction
Predicting the next item of a sequence over a finite alphabet has important applications in many domains. Since I always wanted to implemented something like that, while I was looking for an interesting approach I found this interesting idea based on tree. And you don't deal with trees since a lot, be prepared because as usual it seams simple, but it not. Moreover, since I like Golang and I always wanted to try [UniK](https://github.com/solo-io/unik), I decided to implement my version of the CPT using Golang and use this exercise as a source to build my first unikernel image.

<div class="img_container">
    <img src="https://i.imgur.com/nVJ5wka.png" alt="golang" style="width: 28%; marker-top: -10px;"/>
    <img src="https://i.imgur.com/uwHt83s.png" alt="sequenceprediction" style="width: 25%; marker-top: -10px; margin-left:15px"/>
    <img src="https://i.imgur.com/fsx2vqS.png" alt="unik" style="width: 20%; marker-top: -10px;"/>
</div>

The entire code is available in the Github repo [here](https://github.com/made2591/go-cpt).

#### Too much all together
I knew just a little bit of Golang, almost anything about the algorithm and nothing at all about unik. Let's start from the algorithm.

#### Compact Prediction Tree
A **CPT** (*C*ompact *P*rediction *T*ree) losslessly compress the training data so that all relevant information is available for each prediction. Nice. The approach originally proposed[^op] by the T. Gueniche and P. F. Viger is incremental, offers a low time complexity for its training phase and it is easily adaptable for different applications and contexts. The performance of **CPT** with state of the art techniques, namely PPM (*P*rediction by *P*artial *M*atching), DG (*D*ependency *G*raph) and *all-K-th-order Markov chain. The results show that **CPT** yield higher accuracy on most datasets (up to 12% more than the second best approach), has better training time than DG and PPM, and is considerably smaller than all-K-th-Order Markov.

#### The structure
The entire code is based on two foundamental structure: a trie[^trie] or ```PredictionTree``` ([code](https://github.com/made2591/go-cpt/blob/master/model/predictionTree/PredictionTree.go) of the package) and n ```InvertedIndexTable``` ([code](https://github.com/made2591/go-cpt/blob/master/model/invertedIndexTable/InvertedIndexTable.go)).

#### Prediction Tree
A Prediction Tree is a struct composed by 3 element:
- Item – the actual item stored in the tree, that in our case represent a 32bit int;
- Children – the children of the tree, a slice of ```PredictionTree``` (see the code);
- Parent – A reference to the Parent tree of the tree

This is the reason a Prediction Tree is basically a trie data structure which compresses the entire training data into the form of a tree. Let's say you have 4 difference sequence that contains a set of symbol predefined, like the one below:

- \[A, C, F, E\]
- \[A, C, B\]
- \[F, D, A\]
- \[F, E, D\]

Then, the respective Prediction Tree will be like the one below:

<div class="img_container"><img src="https://i.imgur.com/SHCcMzE.png"  style="width: 30%; marker-top: -10px;"/></div>

#### Inverted Index Table
The Inverted Index Table maintain a reference for each symbol to respective sequences it belongs to. Let's say you have 4 difference sequence that contains a set of symbol predefined, like the one below:

- \[A, C, F, E\]
- \[A, C, B\]
- \[F, D, A\]
- \[F, E, D, C\]

Then, the respective Inverted Index Table will be like the one below:

|       | seq_1 | seq_2 | seq_3 | seq_4 |
| :---: | :---: | :---: | :---: | :---: |
|   A   |   1   |   1   |   0   |   0   |
|   B   |   0   |   1   |   0   |   0   |
|   C   |   1   |   1   |   0   |   1   |
|   D   |   0   |   0   |   1   |   1   |
|   E   |   1   |   0   |   0   |   1   |
|   F   |   1   |   0   |   1   |   1   |

In the code a list of sequence references is maintained instead of a table.

#### Training (build the structs)
The training step consists in fullfill the structs by scanning a list of training sequences (list of list of symbols).

#### Testing (prediction)
The prediction step involves making predictions for each testing sequence in an iterative manner. For a single row, the sequences similar to that row are found thanks to the Inverted Index Table. The consequent of the similar sequences are isolated and maintained in a dictionary with their scores. In the end, this dictionary is used to return the item with the highest score as the final prediction.

- The first step consists in finding the sequences *similar* to the target sequence \[A,C\]. These similar sequences are identified by finding the unique items in the target sequence, finding the set of sequence IDs in which a particular unique item is present and then, taking an intersection of the sets of all unique items. So the sequences in which A is present are the 1 and 2. C also is present in 1, 2 and 4. So the sequences somehow similar to \[A,C\] - our target sequence - is the intersection of set \[1,2\] and \[1,2,4\], thus \[1,2\] - or \[A, C, F, E\] and \[A, C, B\];

- The second step consists in finding the *consequent* of each similar sequence to the target sequence - still \[A,C\]. For each similar sequence, consequent is defined as the sub-sequence after the last occurrence of the last item of the target sequence in the similar sequence, without the items present in the target sequence. As we said the similar sequences are \[1,2\] - \[A, C, F, E\] and \[A, C, B\].

\[A, C, F, E\]: the subsequence after last occurence of C is \[F, E\]. Both of them are not present in \[A,C\], thus the consequent of this similar sequence is \[F, E\]. If you encountered in this set element part of the original target sequence, remove them;
\[A, C, B\]: the subsequence after last occurence of C is \[B\]. Again, B is not present in \[A,C\], thus the consequent of this similar sequence is \[B\];

- The third step consists in adding scoring all the consequents of all the similar sequences for the target sequence in a dictionary along with their score. Let be the dictionary empty at the beginning - the score for the items in the Consequent \[F, E\] is calculated by following this rule: if the item is not present in the dictionary, then the score = 1 + (1 / number of similar sequences) + (1 / number of items currently in the countable dictionary + 1) * 0.001. Otherwise, score = the same multiplied by the oldscore.

So for element E, i.e. the first item in the consequent of the first similar sentence, the score will be
score[F] = 1 + (1/3) + 1/(0+1) * 0.001 = 1.3343
score[E] = 1 + (1/3) + 1/(1+1) * 0.001 = 1.3338
score[B] = 1 + (1/3) + 1/(2+1) * 0.001 = 1.3336

Finally, \[A,C\] the key is returned with the greatest value in the dictionary of scores as the prediction. In the case of the above example, F is returned as a sequence prediction.

### UniKImage

>True, linux is monolithic, and I agree that microkernels are nicer... As has been noted (not only by me), the linux kernel is a minuscule part of a complete system: Full sources for linux currently runs to about 200kB compressed. And all of that source is portable, except for this tiny kernel that you can (probably: I did it) re-write totally from scratch in less than a year without having /any/ prior knowledge.
>
>Linus Torvalds, 1992

[UniK](https://github.com/solo-io/unik) is a tool for compiling application sources into unikernels (lightweight bootable disk images) and MicroVM rather than binaries. UniK runs and manages instances of compiled images across a variety of cloud providers as well as locally: you can utilize it with a simple docker-like command line interface, that let you make and build unikernels and MicroVM as easy as building containers. UniK is built to be easily extensible, allowing (and encouraging) adding support for unikernel/MicroVM compilers and cloud providers. [here](https://github.com/solo-io/unik/blob/master/docs/architecture.md) more details about the architecture.

#### Steps to build image
To have the sequence prediction engine available as a bootable image, I used as said in the beginning. You can have a look at the repository on how to install everything is required. In the next paragraph I will provide someinsight you can find the original repository.

##### Install and configure UniK
Install unik by following instruction in official repository from [here](https://github.com/solo-io/unik/blob/master/docs/install.md).

{{< highlight sh >}}
git clone https://github.com/solo-io/unik.git
cd unik
make
{{< / highlight >}}

then follow configuration step [here](https://github.com/solo-io/unik/blob/master/docs/configure.md).

##### Golang server
Again, taken from unik repository. You have to ensure the project is cloned in $GOPATH, then:

- Go installed and your $GOPATH configured (see getting started with Go)
- Your project should be located within your system's $GOPATH (if you're unfamiliar with Go and the $GOPATH convention, read more here)
- There should be a main package in the root directory of your project
- Godeps installed (run go get github.com/tools/godep once Go is installed)
- Run GO15VENDOREXPERIMENT=1 godep save ./... from the root of your project.

This will create a Godeps/Godeps.json file as well as place all dependencies of your project in the ./vendor directory. This will allow UniK to compile your application entirely using only the root directory of your project.

Open a shell and run the daemon - then keep it running:
{{< highlight sh >}}
unik daemon --debug
{{< / highlight >}}

To build and run the image - remember to use godep to let unik include dependencies first!! - run:
{{< highlight sh >}}
unik build --name go-cpt-image --path ./ --base rump --language go --provider virtualbox --force
unik run --instanceName go-cpt-instance --imageName go-cpt-image
{{< / highlight >}}

To retrieve the running instances:
{{< highlight sh >}}
unik instances
{{< / highlight >}}

You should get something like this:
<div class="img_container"><img src="https://github.com/made2591/go-cpt/blob/master/unik.png?raw=true" alt="golang" style="width: 100%; marker-top: -10px;"/></div>

You can see IP assigned to instances in the last column of the output. To see the logs of the running instances run:

{{< highlight sh >}}
unik logs --instance go-cpt-instance
{{< / highlight >}}

What this image does is actually expose the different endpoint to initialize training and make prediction by rest api - ```it's only a draft```:

A sample file are already uploaded into the upload folder: you can modify the ```main.go``` root of the project to avoid cutting the training and testing set. Otherwise, to see the run you can both execute the code locally or
{{< highlight sh >}}
curl http://<YOUR_RUNNING_INSTANCES>:8080/initcpt
{{< / highlight >}}

You should see predictions for the first 10 sequences :-)
<div class="img_container"><img src="https://github.com/made2591/go-cpt/blob/master/predictions.png?raw=true" alt="golang" style="width: 100%; marker-top: -10px;"/></div>

#### Conclusion
Many thanks to [UniK contributors](https://github.com/solo-io/unik/graphs/contributors) for sure, to [NeerajSarwan](https://github.com/NeerajSarwan) for his work over CPT and all who want to contribute

Thank you everybody for reading!

[^op]: The original paper is available at [here](https://www.researchgate.net/publication/263696690_Compact_Prediction_Tree_A_Lossless_Model_for_Accurate_Sequence_Prediction).
[^trie]: [Trie](https://en.wikipedia.org/wiki/Trie) (Wikipedia).