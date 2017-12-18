---
title: "Build a multilayer perceptron with Golang"
tags: [coding, golang, ann, perceptron, classifier, neural, networks]
---

### History
We can date the birth of artificial neural networks in 1958, with the introduction of Perceptron [^rosen] by Frank Rosenblatt. It was the first algorithm created to reproduce the biological neuron. Conceptually, the easier perceptron that you might think of is made of a single neuron: when it’s exposed to a stimulus, it provides a binary response, just as would a biological neuron.

![ann](https://pbs.twimg.com/media/DPtxHXKW4AEcLyc.jpg)

This model differs greatly from the neural network involving billions of neurons in a biological brain. Shortly after his birth, the researchers showed the world the problems of Perceptron: in fact, it was quickly proved that perceptrons could not be trained to recognize many classes of input patterns. To get a more powerful network, it was necessary to take advantage of multiple level of units and create a multilayers perceptron, with more intermediates neurons used to solve linearly separable[^linsep] subproblems, whose outputs were combined together by the final level to provide a concrete response to original input problem. Even though the Perceptron was just a simple but severely limited binary classifier, it introduced a great innovation: the idea to simulate the basic computational unit of a complex biological system that exists in nature. However, the need to study and understand how real neurons are bound to each other in the biological brain goes back to previous work.

### Theory
Ok, let's start by saying that no one has yet understood why neural networks work: Minsky[^minsky] himself - one of my favourite heros - had strongly criticized them (writing an entire book, [Perceptrons](https://en.wikipedia.org/wiki/Perceptrons_(book)) causing the stop in research about ANN[^ann] became important again. In this article, I describe how I built with Golang my own perceptron - and then a multilayer perceptron - I introduced in the previous paragraph. Let first talk about the representation of the input: all the example codes are from my [go-perceptron-go](https://github.com/made2591/go-perceptron-go) repository.

#### Base structures - [code](https://github.com/made2591/go-perceptron-go/tree/master/model/neural)
When I started writing my perceptron, the first thing I deal with was the definition of data structures. I create a ```neural``` package to collect all files related to architecture structure and elements.

##### Pattern - [code](https://github.com/made2591/go-perceptron-go/blob/master/model/neural/pattern.go)
The ```Pattern``` struct represent a single input to the ```Perceptron``` struct. Look at the code:
{% highlight golang %}
// Pattern struct represents one pattern with dimensions and desired value
type Pattern struct {
	Features []float64
	SingleRawExpectation string
	SingleExpectation float64
	MultipleExpectation []float64
}
{% endhighlight %}

It satisfies our needs with only four fields:
- ```Features``` is a slice of 64 bit float and this is perfect to represent input dimension,
- ```SingleRawExpectation``` is a string and is filled by parser with input classification (in terms of belonging class),
- ```SingleExpectation``` is a 64 bit float representation of the class which the pattern belongs,
- ```MultipleExpectation``` is a slice of 64 bit float and it is used for multiple class classification problems;

##### Neuron - [code](https://github.com/made2591/go-perceptron-go/blob/master/model/neural/neuronUnit.go)
The ```NeuronUnit``` struct represent a single computation unit. Look at the code:
{% highlight golang %}
// NeuronUnit struct represents a simple NeuronUnit network with a slice of n weights.
type NeuronUnit struct {
	Weights []float64
	Bias float64
	Lrate float64
	Value float64
	Delta float64
}
{% endhighlight %}

A neuron corresponds to the simple binary perceptron originally proposed by Rosenblat. It is made of:
- ```Weights```, a slice of 64 bit float to represent the way each dimensions of the pattern is modulated,
- ```Bias```, a 64 bit float that represents NeuronUnit natural propensity to spread signal,
- ```Lrate```, a 64 bit float that represents learning rate of neuron,
- ```MultipleExpectation```, a 64 bit float that represents the desired value when I load the input pattner into network in Multi NeuralLayer Perceptron,
- ```Delta```, a 64 bit float that mantains error during execution of training algorithm (later); 

##### Perceptron
As I said, the single perceptron schema is implemented by a single neuron. The easiest way to implement this simple classifier is to establish a threshold function, insert it into the neuron, combine the values (eventually using different weights for each of them) that describe the stimulus in a single value, provide this value to the neuron and see what it returns in output. The schema show how it works:

![perceptron](https://upload.wikimedia.org/wikipedia/commons/6/60/ArtificialNeuronModel_english.png)

##### Linearly separable problems
The problem with the binary perceptron made with a single neuron is the inability to handle non-linearly separable problems: these kind of problems are the ones in which, in other words, it's impossible to define an hyperplane able to separate, in the vector space of the inputs, those that require a positive output from those requiring a negative output. An example of three non-collinear points belonging to two different classes ('_+_' and '_-_') are always linearly separable in two dimensions. This is illustrated by the first three examples in the following figure:

![perceptron](https://image.ibb.co/coBkX6/linear.png)

However, not all sets of four points, no three collinear, are linearly separable in two dimensions. The fourth image would need two straight lines and thus is not linearly separable. This is the main reason scientist start working with multilayers at the very beginning. Let's move one step forward, introducing ```NeuralLayer```.

##### Neural Layer - [code](https://github.com/made2591/go-perceptron-go/blob/master/model/neural/neuralLayer.go)
The ```NeuralLayer``` struct represents a network layer with a slice of n ```NeuronUnits```.
{% highlight golang %}
type NeuralLayer struct {
	NeuronUnits []NeuronUnit
	Length int
}
{% endhighlight %}

where:
- ```NeuronUnits``` represents NeuronUnits in layer,
- ```Length``` represents number of NeuronUnit in layer;

Now we can define a multilayer perceptron.

##### Multilayer Perceptron - [code](https://github.com/made2591/go-perceptron-go/blob/master/model/neural/multiLayerNetwork.go)
{% highlight golang %}
type MultiLayerNetwork struct {
	L_rate float64
	NeuralLayers []NeuralLayer
	T_func transferFunction
	T_func_d transferFunction
}
{% endhighlight %}

where:
- ```NeuralLayers``` represents layer of neurons,
- ```Length``` represents learning rate of neuron,
- ```T_func``` and ```T_func_d``` represents the transferFunction and its derivative;

The ```MultiLayerNetwork``` struct define algorithm to create multilayer perceptron: if you pass a struct with ```NeuralLayers``` [4, 3, 3], you can define a network struct with 3 layer: input, hidden, output, with respectively 4, 3 and 3 neurons, as shown in the figure below.

![perceptron](https://image.ibb.co/j9jgpm/first_example_copia.png)

For classification problems the input layers has to be define with a number of neurons that match features of pattern shown to network. Of course, the output layer should have a number of unit equals to the number of class in training set.

#### Training Algorithm - [code](https://github.com/made2591/go-perceptron-go/blob/master/model/neural/multiLayerNetwork.go)

##### Backpropagation


Thank you everybody for reading!

[^minsky]: Marvin Lee Minsky, an American cognitive scientist - [Wikipedia](https://it.wikipedia.org/wiki/Marvin_Minsky)
[^ann]: Artificial Neural Networks, computing systems inspired by the biological neural networks - [Wikipedia](https://en.wikipedia.org/wiki/Artificial_neural_network)
[^rosen]: F. Rosenblatt. The perceptron: A probabilistic model for information storage and organization in the brain. Psychological Review, pages 65–386, 1958. (cit. a p. 5)
[^linsep]: This condition describes the situation in which there exists a hyperplane able to separate, in the vector space of the inputs, those that require a positive output from those requiring a negative output.