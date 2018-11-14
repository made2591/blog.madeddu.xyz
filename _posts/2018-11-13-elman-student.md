---
title: "How my Elman network learnt to count"
tags: [coding, golang, ann, elman, adding, neural, networks]
---

### Introduction
This is actually a sort of back-to-the-future post because it's related to something I completed one year ago: I built this Elman network and it learnt to count. What I shame, I forgot it, now it's kind of its first birthday so let's celebrate :D

<p align="center"><img src="https://static1.squarespace.com/static/550ca181e4b00ab6c2a10330/t/55afd73be4b0ba2638779743/1437587260614/boy-going-back-to-school.jpg?format=750w" alt="matrixbug" style="width: 100%; marker-top: -10px;"/></p>

This is Elman, the best in class in adding int32 numbers. For everybody who already knows what I will talk about (what?!), [here](https://github.com/made2591/go-perceptron-go)'s the Github repo. I'm sorry for the name, it's still go-perceptron-go but that repo contains my GoLang ANN.

### Let's start from
You were wondering what the f\*\*k is an Elman network: to be honest, I didn't understand exactly but [this](https://made2591.github.io/posts/neuralnetwork) post related to the perceptron could be a good starting point - at least, somehow linked cause in the end this network share a lot with multilayer perceptron. Ignored? Perfect. In one sentence: an Elmann network is a MFNN with an extra context layer. That is a Multilayer Feedforward Neural Network with an extra context layer: the point is that, unfortunately, this context layer create a closed circle in the network - thus, in the way the information is progated.

That means that Elman network are actually RNN, or Recurrent Neural Network even. That are...wait. Let's make a step back.

<p align="center"><img src="https://i.imgur.com/GbUJP5R.png" alt="matrixbug" style="width: 40%; marker-top: -10px;"/></p>

#### RNNs vs Standard ANNs
As you know an ANN can be described as a set of neuron units (read perceptron), organized in layers, linked together in several ways to achieve specific - mainly classification - jobs. What it came out is that by changing the links used to attach the neural network layers you can expect different behaviour. What does it mean changing the way the information flow?

The idea behind RNNs is to make use of *sequential information*. In a traditional neural network we assume that all inputs (and outputs) are independent between each other but, for many tasks... that's a very bad idea. For instance, if you want to predict the next word in a sentence you better know which words came before it.

RNNs are called *recurrent* because they perform the same task for every element of a sequence, with the output being depended on the previous computations. Another way to think about RNNs is that they have a *memory* which captures information about what has been calculated so far. In theory RNNs can make use of information in arbitrarily long sequences, but in practice they are limited to looking back only a few steps.

#### Base structures - [code](https://github.com/made2591/go-perceptron-go/tree/master/model/neural)
To create a neural network, the first thing you have to do is dealing with the definition of data structures. I create a ```neural``` package to collect all files related to architecture structure and elements.

##### Pattern - [code](https://github.com/made2591/go-perceptron-go/blob/master/model/neural/pattern.go)
The ```Pattern``` struct represent a single input struct. Look at the code:

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

Why patterns? Because, our goal is to teach an ANN doing something, in this case counting, so... our patterns will be our binary number expressed as slice of 0 and 1. Immagine that we are giving a children a list of operation with numbers - in binary, poor little child. Anyway, this is to say: that child in a way or in another (definitely in another) will learn how to sum integer.

Who gives these number? The function ```CreateRandomPatternArray(d, k)``` that actually return a slice of ```Pattern``` (binary number). Perfect! We have numbers!

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

Again, every neuron of Elmann is a neuron in our neural child (what?!). Next step

##### Again perceptrons?
As you know, the single perceptron schema is implemented by a single neuron. The easiest way to implement this simple classifier is to establish a threshold function, insert it into the neuron, combine the values (eventually using different weights for each of them) that describe the stimulus in a single value, provide this value to the neuron and see what it returns in output. The schema show how it works:

![perceptron](https://upload.wikimedia.org/wikipedia/commons/6/60/ArtificialNeuronModel_english.png)

We know that multilayer neural networks are a combo of element like the one shown above etc. Thus, in what is different an Elman network? Actually, as we said the only difference is the presence of a context layer - yes, the training algorithm is the back propagation as the one explained for perceptron (**almost**). Let's say that an Elmann network is a three-layer network with the addition of this set of *context units*. The middle (hidden) layer is connected to these context units fixed with a weight of one. At each time step, the input is fed-forward and a learning rule is applied. The fixed back-connections save a copy of the previous values of the hidden units in the context units (since they propagate over the connections before the learning rule is applied). Thus the network can maintain a sort of state, allowing it to perform such tasks as *sequence-prediction* that are beyond the power of a standard multilayer perceptron.

###### Back propagation - differences
Ok, the code is almost the same as defined for Perceptron, available [here](https://github.com/made2591/go-perceptron-go/blob/master/model/neural/multiLayerNetwork.go). Actually, it is because in the end the only difference is that we want the neural network to be able to store the neural hidden values at every step in the context. In fact, to preserve the MLPerceptron struct I extended the two method involved in training, ```BackPropagate``` and ```Execute```, with an optional argument (options ...int).

{% highlight golang %}
// BackPropagation algorithm for assisted learning. Convergence is not guaranteed and very slow.
// Use as a stop criterion the average between previous and current errors and a maximum number of iterations.
// [mlp:MultiLayerNetwork] input value [s:Pattern] input value (scaled between 0 and 1)
// [o:[]float64] expected output value (scaled between 0 and 1)
// return [r:float64] delta error between generated output and expected output
func BackPropagate(mlp *MultiLayerNetwork, s *Pattern, o []float64, options ...int) (r float64) {

    var no []float64;
    // execute network with pattern passed over each level to output
    if len(options) == 1 {
        no = Execute(mlp, s, options[0])
    } else {
        no = Execute(mlp, s)
    }

    ...

        // copy hidden output to context
        if k == 1 && len(options) > 0 && options[0] == 1 {

            for z := len(s.Features); z < mlp.NeuralLayers[0].Length; z++ {

                // save output of hidden layer to context
                mlp.NeuralLayers[0].NeuronUnits[z].Value = mlp.NeuralLayers[k].NeuronUnits[z-len(s.Features)].Value

            }

        }

    ...

{% endhighlight %}

and during the execution part of the network this means propagate to context:

{% highlight golang %}
// save output of hidden layer to context if nextwork is RECURRENT
if k == 1 && len(options) > 0 && options[0] == 1 {

    for z := len(s.Features); z < mlp.NeuralLayers[0].Length; z++ {

        log.WithFields(log.Fields{
            "level"				: "debug",
            "len z" 			: z,
            "s.Features"		: s.Features,
            "len(s.Features)" : len(s.Features),
            "len mlp.NeuralLayers[0].NeuronUnits" : len(mlp.NeuralLayers[0].NeuronUnits),
            "len mlp.NeuralLayers[k].NeuronUnits" : len(mlp.NeuralLayers[k].NeuronUnits),
        }).Debug("Save output of hidden layer to context.")

        mlp.NeuralLayers[0].NeuronUnits[z].Value = mlp.NeuralLayers[k].NeuronUnits[z-len(s.Features)].Value

    }

}
{% endhighlight %}

##### Where is the context
As you most probably noticed, I made a magic trick: to avoid create a new neural network struct, I used the input layer as layer to also store the context layer. That is the reason I loop with index z starting from len(s.Features) in both the ```BackPropagate``` and ```Execute```.

How to run it?

{% highlight sh %}
go get github.com/made2591/go-perceptron-go
cd $GOPATH/src/made2591/go-perceptron-go
go run main.go
{% endhighlight %}

Thank you everybody for reading!