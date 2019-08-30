---
layout: post
title: "Build a multilayer perceptron with Golang"
categories: [coding, golang, theory, algorithms]
tags: [golang, ann, perceptron, classifier, neural, networks]
---

### History
We can date the birth of artificial neural networks in 1958, with the introduction of Perceptron [^rosen] by Frank Rosenblatt. It was the first algorithm created to reproduce the biological neuron. Conceptually, the easier perceptron that you might think of is made of a single neuron: when it's exposed to a stimulus, it provides a binary response, just as would a biological neuron.

![ann](https://i.imgur.com/uUZ5vyF.jpg)

This model differs greatly from the neural network involving billions of neurons in a biological brain. Shortly after his birth, the researchers showed the world the problems of Perceptron: in fact, it was quickly proved that perceptrons could not be trained to recognize many classes of input patterns. To get a more powerful network, it was necessary to take advantage of multiple level of units and create a multilayers perceptron, with more intermediates neurons used to solve linearly separable[^linsep] subproblems, whose outputs were combined together by the final level to provide a concrete response to original input problem. Even though the Perceptron was just a simple but severely limited binary classifier, it introduced a great innovation: the idea to simulate the basic computational unit of a complex biological system that exists in nature.

### Theory
Fundamentally, a neural network is nothing more than a really good function approximator — I mean, you give a trained network as an input vector, it performs a series of operations, and it produces an output vector. To train an ann to estimate an unknown function, the process is really simple: you have to get a training set - a collection of data points - that the network will learn from and generalize on to make future inferences. In a multilayer perceptron data points are forwarded through the network layer-by-layer until they reach the final layer. The final layer's activations are the predictions that the network actually makes. In this article, I describe how I built with Golang my own perceptron - and then a multilayer perceptron. Let first talk about the representation of the input: all the example codes are from my [go-perceptron-go](https://github.com/made2591/go-perceptron-go) repository.

#### Base structures - [code](https://github.com/made2591/go-perceptron-go/tree/master/model/neural)
To create a neural network, the first thing you have to do is dealing with the definition of data structures. I create a ```neural``` package to collect all files related to architecture structure and elements.

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

![perceptron](https://i.imgur.com/uu0iNCC.png)

##### Metric
Why _weights_? What does it mean the expression _dimension modulation_ of the the input? Well, training conceptually is "the process of learning the skills you need to do a particular job or activity". But how do you know if you're getting better, or if you are learning the skills you need? Of course, you need a metric of how good or bad you're doing. Also in ANN there's a metric generally called _cost function_. Suppose we want to change a certain _wi_ weight of the network. More or less, the cost function looks at the function the network has inferred and uses it to estimate values for the data points in the training set. The difference between the outputs of the network and the training set data points are the main values for the cost function. When training your network, the goal is to get the value of this cost function as low as possible. The most basic of the training algorithms is the _gradient descent_.
Suppose we can calculate the error _E_ according to the variation of the weight value _wi_: we are therefore able to draw the graph in a graph like the one in the figure.

<div class="img_container"><img src="https://i.imgur.com/W5zQiw7.png"  style="width: 250px; marker-top: -10px;"/></div>

Therefore, if we calculate the derivative of this function, we can understand how the variation of the weight makes a positive or negative contribution to the error. In practice, whatever the derived value, we can use a single weight correction function that decrease the involved weight of derived quantity (modulated by learning rate). Despite the fact that it's quite impossible, for any network or cost function, to be truly convex, the gradient descent follows the derivatives computed for each neuron unit to essentially "roll" down the slope until it finds its way to the center - as close as possible to the _global minimum_. Before continuing, let's take a step back.

##### Why multilayer? The linearly separable problems
The problem with the binary perceptron made with a single neuron is the inability to handle non-linearly separable problems: these kind of problems are the ones in which, in other words, it's impossible to define an hyperplane able to separate, in the vector space of the inputs, those that require a positive output from those requiring a negative output. An example of three non-collinear points belonging to two different classes ('_+_' and '_-_') are always linearly separable in two dimensions. This is illustrated by the first three examples in the following figure:

<div class="img_container"><img src="https://i.imgur.com/jFC0NZR.png"  style="width: 250px; marker-top: -10px;"/></div>

However, not all sets of four points, no three collinear, are linearly separable in two dimensions. The fourth image would need two straight lines and thus is not linearly separable. This is the main reason scientist start working with multilayers at the very beginning. Let's move one step forward, introducing the ```NeuralLayer``` struct.

##### Neural Layer - [code](https://github.com/made2591/go-perceptron-go/blob/master/model/neural/neuralLayer.go)
The ```NeuralLayer``` struct represents a network layer with a slice of _n_ ```NeuronUnits```.
{% highlight golang %}
type NeuralLayer struct {
	NeuronUnits []NeuronUnit
	Length int
}
{% endhighlight %}

where:
- ```NeuronUnits``` represents NeuronUnits in layer,
- ```Length``` represents number of NeuronUnit in layer;

Now that we are able to build layers of neurons, we can define the ```MultiLayerNetwork``` struct.

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

Inside the ```MultiLayerNetwork``` struct there's an algorithm to create multilayer perceptron: if you pass a struct with ```NeuralLayers``` [4, 3, 3], you can define a network struct with 3 layer: input, hidden, output, with respectively 4, 3 and 3 neurons, as shown in the figure below.

![perceptron](https://i.imgur.com/vcxtFKD.png)

The piece of code that handle network creation is the following:

{% highlight golang %}

// ... the following is in the neuralLayer.go

// PrepareLayer create a NeuralLayer with n NeuronUnits inside
// [n:int] is an int that specifies the number of neurons in the NeuralLayer
// [p:int] is an int that specifies the number of neurons in the previous NeuralLayer
// It returns a NeuralLayer object
func PrepareLayer(n int, p int) (l NeuralLayer) {

	l = NeuralLayer{NeuronUnits: make([]NeuronUnit, n), Length: n}

	for i := 0; i < n; i++ {
		RandomNeuronInit(&l.NeuronUnits[i], p)
	}

	log.WithFields(log.Fields{
		"level":   "info",
		"msg":     "multilayer perceptron init completed",
		"neurons": len(l.NeuronUnits),
		"lengthPreviousLayer": l.Length,
	}).Info("Complete NeuralLayer init.")

	return

}

// ... the following is in the multiLayerNetwork.go

// PrepareMLPNet create a multi layer Perceptron neural network.
// [l:[]int] is an int array with layers neurons number [input, ..., output]
// [lr:int] is the learning rate of neural network
// [tr:transferFunction] is a transfer function
// [tr:transferFunction] the respective transfer function derivative
func PrepareMLPNet(l []int, lr float64, tf transferFunction, trd transferFunction) (mlp MultiLayerNetwork) {

	// setup learning rate and transfer function
	mlp.L_rate = lr
	mlp.T_func = tf
	mlp.T_func_d = trd

	// setup layers
	mlp.NeuralLayers = make([]NeuralLayer, len(l))

	// for each layers specified
	for il, ql := range l {

		// if it is not the first
		if il != 0 {

			// prepare the GENERIC layer with specific dimension and correct number of links for each NeuronUnits
			mlp.NeuralLayers[il] = PrepareLayer(ql, l[il-1])

		} else {

			// prepare the INPUT layer with specific dimension and No links to previous.
			mlp.NeuralLayers[il] = PrepareLayer(ql, 0)

		}

	}

	log.WithFields(log.Fields{
		"level":     "info",
		"msg":       "multilayer perceptron init completed",
		"layers":  len(mlp.NeuralLayers),
		"learningRate: ": mlp.L_rate,
	}).Info("Complete Multilayer Perceptron init.")

	return

}

{% endhighlight %}

For classification problems the input layers has to be define with a number of neurons that match features of pattern shown to network. Of course, the output layer should have a number of unit equals to the number of class in training set.

__NOTE__: from the architectural point of view an interesting theorem guarantee that _given a sufficient number of hidden units, everything that can be solved by a multilayer network at n levels can also be solved by a two-level network_. Therefore in examples we will limit ourselves to using only two levels.

#### BackPropagation Algorithm - [code](https://github.com/made2591/go-perceptron-go/blob/master/model/neural/multiLayerNetwork.go)
The learning algorithm can be divided into two phases: propagation and weight update.

##### Propagation - 1 of 2
Each propagation involves the following steps:

- the _propagation_ forward through the network to generate the output value(s) is done by ```Execute``` function,
- the calculation of the cost (error term) is done here at the very beginning of the ```BackPropagate``` function,
- the propagation of the output activations __back__ through the network, using the training pattern target in order to generate the deltas (the difference between the targeted and actual output values) of all output and hidden neurons, done of coure ```BackPropagate``` function;

First, let's have a look to the ```Execute``` function.

{% highlight golang %}
// Execute a multi layer Perceptron neural network.
// [mlp:MultiLayerNetwork] multilayer perceptron network pointer, [s:Pattern] input value
// It returns output values by network
func Execute(mlp *MultiLayerNetwork, s *Pattern, options ...int) (r []float64) {

	// new value
	nv := 0.0

	// result of execution for each OUTPUT NeuronUnit in OUTPUT NeuralLayer
	r = make([]float64, mlp.NeuralLayers[len(mlp.NeuralLayers)-1].Length)

	// show pattern to network =>
	for i := 0; i < len(s.Features); i++ {

		// setup value of each neurons in first layers to respective features of pattern
		mlp.NeuralLayers[0].NeuronUnits[i].Value = s.Features[i]

	}

	// execute - hiddens + output
	// for each layers from first hidden to output
	for k := 1; k < len(mlp.NeuralLayers); k++ {

		// for each neurons in focused level
		for i := 0; i < mlp.NeuralLayers[k].Length; i++ {

			// init new value
			nv = 0.0

			// for each neurons in previous level (for k = 1, INPUT)
			for j := 0; j < mlp.NeuralLayers[k - 1].Length; j++ {

				// sum output value of previous neurons multiplied by weight between previous and focused neuron
				nv += mlp.NeuralLayers[k].NeuronUnits[i].Weights[j] * mlp.NeuralLayers[k - 1].NeuronUnits[j].Value

				log.WithFields(log.Fields{
					"level":     "debug",
					"msg":       "multilayer perceptron execution",
					"len(mlp.NeuralLayers)":  len(mlp.NeuralLayers),
					"layer:  ": k,
					"neuron: ": i,
					"previous neuron: ": j,
				}).Debug("Compute output propagation.")

			}

			// add neuron bias
			nv += mlp.NeuralLayers[k].NeuronUnits[i].Bias

			// compute activation function to new output value
			mlp.NeuralLayers[k].NeuronUnits[i].Value = mlp.T_func(nv)

			log.WithFields(log.Fields{
				"level":     "debug",
				"msg":       "setup new neuron output value after transfer function application",
				"len(mlp.NeuralLayers)":  len(mlp.NeuralLayers),
				"layer:  ": k,
				"neuron: ": i,
				"outputvalue" : mlp.NeuralLayers[k].NeuronUnits[i].Value,
			}).Debug("Setup new neuron output value after transfer function application.")

		}

	}


	// get ouput values
	for i := 0; i < mlp.NeuralLayers[len(mlp.NeuralLayers)-1].Length; i++ {

		// simply accumulate values of all neurons in last level
		r[i] = mlp.NeuralLayers[len(mlp.NeuralLayers)-1].NeuronUnits[i].Value

	}

	return r

}
{% endhighlight %}

Basically, what ```Execute``` function does is computing the result of execution for each _output_ ```NeuronUnit``` in _output_ ```NeuralLayer```. In order, it first _inserts input_ to _input_ ```NeuralLayer``` of the network, assigning the values of the dimensions (```Features``` field) of each pattern to values (```Value``` field) of each ```NeuronUnit``` in input layer (```mlp.NeuralLayers[0]```); after that, for each layers from first hidden to output, and for each neurons in the previous level and the current, execution algorithm computes the sum of multiplication between the weight that links two involved neurons and the (output) computed in the step before of the previous neuron - this is the meaning of most internal for. Then, the bias - natural propension to activation - of the neuron is added to the quantity _nv_, and output value of the current neuron in the current neural layer is _updated_ with the activation function computed passing this quantity _nv_ as parameter. The last for simply accumulate values of all neurons in last level and return the result. To summarize, this algorithm makes the input flow through the network, using weights to modulate the various dimensions that describe it and the activation functions to calculate the response of each neuron. In the end, the values accumulated in the neurons of the last level are returned.

Back to the ```BackPropagate```, we already said it starts executing the network. The idea is to get the value accumulated in the neurons of the last level, to compute the error accumulated retracing the various steps backwards. With the (__uncorrect__) assumption of a convex function, we can imagine that _solving the weight update task backwards, by calculating the derivative of the activation function_, is a good way to _go down towards the global optimum_. In reality, there is no guarantee of not being _stuck in a false minimum_, and this depends on the characteristics of the function and (most likely) also on the architecture chosen for our ann.

<div class="img_container"><img src="https://i.imgur.com/sR50plx.png"  style="width: 300px; marker-top: -10px;"/></div>

Weights update:

##### Weight update - 2 of 2
For each weight in the network, the following steps must be followed:

- the weight's output delta and input activation are multiplied to find the gradient of the weight,
- a percentage (modulated by learning rate) of the weight's gradient is subtracted from the weight;

The learning rate _influences_ the speed and quality of learning. The greater it is, the faster the neuron trains, but the lower it is, the more accurate the training is. The sign of the gradient of a weight indicates whether the error varies directly with, or inversely to, the weight. Therefore, the weight must be updated in the opposite direction - this is the reason of the name _gradient descent_.

{% highlight golang %}
// BackPropagation algorithm.
// [mlp:MultiLayerNetwork] input value		[s:Pattern] input value (scaled between 0 and 1)
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

	// init error
	e := 0.0

	// compute output error and delta in output layer
	for i := 0; i < mlp.NeuralLayers[len(mlp.NeuralLayers)-1].Length; i++ {

		// compute error in output: output for given pattern - output computed by network
		e = o[i] - no[i]

		// compute delta for each neuron in output layer as:
		// error in output * derivative of transfer function of network output
		mlp.NeuralLayers[len(mlp.NeuralLayers)-1].NeuronUnits[i].Delta = e * mlp.T_func_d(no[i])

	}

	// backpropagate error to previous layers
	// for each layers starting from the last hidden (len(mlp.NeuralLayers)-2)
	for k := len(mlp.NeuralLayers)-2; k >= 0; k-- {

		// compute actual layer errors and re-compute delta
		for i := 0; i < mlp.NeuralLayers[k].Length; i++ {

			// reset error accumulator
			e = 0.0

			// for each link to next layer
			for j := 0; j < mlp.NeuralLayers[k + 1].Length; j++ {

				// sum delta value of next neurons multiplied by weight between focused neuron and all neurons in next level
				e += mlp.NeuralLayers[k + 1].NeuronUnits[j].Delta * mlp.NeuralLayers[k + 1].NeuronUnits[j].Weights[i]

			}

			// compute delta for each neuron in focused layer as error * derivative of transfer function
			mlp.NeuralLayers[k].NeuronUnits[i].Delta = e * mlp.T_func_d(mlp.NeuralLayers[k].NeuronUnits[i].Value)

		}

		// compute weights in the next layer
		// for each link to next layer
		for i := 0; i < mlp.NeuralLayers[k + 1].Length; i++ {

			// for each neurons in actual level (for k = 0, INPUT)
			for j := 0; j < mlp.NeuralLayers[k].Length; j++ {

				// sum learning rate * next level next neuron Delta * actual level actual neuron output value
				mlp.NeuralLayers[k + 1].NeuronUnits[i].Weights[j] +=
					mlp.L_rate * mlp.NeuralLayers[k + 1].NeuronUnits[i].Delta * mlp.NeuralLayers[k].NeuronUnits[j].Value

			}

			// learning rate * next level next neuron Delta * actual level actual neuron output value
			mlp.NeuralLayers[k + 1].NeuronUnits[i].Bias += mlp.L_rate * mlp.NeuralLayers[k + 1].NeuronUnits[i].Delta

		}

	}

	// compute global errors as sum of abs difference between output execution for each neuron in output layer
	// and desired value in each neuron in output layer
	for i := 0; i < len(o); i++ {

		r += math.Abs(no[i] - o[i])

	}

	// average error
	r = r / float64(len(o))

	return

}

{% endhighlight %}

After execution step, ```BackPropagate``` function starts computing output error and delta for the output level. The delta for a given neuron can be calculated as follows:

	delta = (expected - output) * transfer\_derivative(output)

where expected is the expected output value (_o[i]_) for the neuron and output is the output value for the neuron (_no[i]_) computed by the Execution step (the first operation is done in the code by _e = o[i] - no[i]_ operation). Then, the _transfer\_derivative()_ calculates the slope of the neuron's output value and the algorithm save this value to the delta fields of each of the neurons (not only in the oupput layers): this is done because the layers of the network are iterated in reverse order - or _backwards_, as it is shown by the _k-for_ (_k--_) - starting at the output and working backwards. This ensures that the neurons in the output layer have errors values calculated first that neurons in the hidden layer can use in the subsequent iteration.

In the hidden layer, things are a little more complicated. The error signal for a neuron in the hidden layer is computed as the _weighted error of each neuron in the output layer_. Think of the error traveling back along the weights of the output layer to the neurons in the hidden layer: the back-propagated error signal __is accumulated__ and then __used to determine the error for the neuron in the hidden layer__, as follows:

	delta\_i = accumulated(weight\_i * delta\_j) * transfer\_derivative(output)

where _delta\_j_ is the error signal from the _j\_th_ neuron in the output layer, _weight\_i_ is the weight that connects the _i\_th_ neuron of the output layer to the current neuron, and output is the output of the current neuron[^details]. After that there is the network layers weights update, that follow this rules

	weight\_i = weight\_i + (learning_rate * delta\_j * input)

Finally, the errors (as the abs difference between expcted minus computed) accumulated in the neurons of the last level are returned. Wait a minute: where is the training algorithm?

#### Training Algorithm
Look at the code below! Basically, what it does is running for a fixed amount of epochs the BackPropagate function.
{% highlight golang %}
// MLPTrain train a mlp MultiLayerNetwork with BackPropagation algorithm for assisted learning.
func MLPTrain(mlp *MultiLayerNetwork, patterns []Pattern, mapped []string, epochs int) {

	epoch := 0
	output := make([]float64, len(mapped))

	// for fixed number of epochs
	for {

		// for each pattern in training set
		for _, pattern := range patterns {

			// setup desired output for each unit
			for io, _ := range output {
				output[io] = 0.0
			}
			// setup desired output for specific class of pattern focused
			output[int(pattern.SingleExpectation)] = 1.0
			// back propagation
			BackPropagate(mlp, &pattern, output)

		}

		log.WithFields(log.Fields{
			"level":             "info",
			"place":             "validation",
			"method":            "MLPTrain",
			"epoch":        	 epoch,
		}).Debug("Training epoch completed.")

		// if max number of epochs is reached
		if epoch > epochs {
			// exit
			break
		}
		// increase number of epoch
		epoch++

	}

}
{% endhighlight %}

Thank you everybody for reading!

[^minsky]: Marvin Lee Minsky, an American cognitive scientist - [Wikipedia](https://it.wikipedia.org/wiki/Marvin_Minsky).
[^ann]: Artificial Neural Networks, computing systems inspired by the biological neural networks - [Wikipedia](https://en.wikipedia.org/wiki/Artificial_neural_network).
[^rosen]: F. Rosenblatt. The perceptron: A probabilistic model for information storage and organization in the brain. Psychological Review, pages 65–386, 1958. (cit. a p. 5).
[^linsep]: This condition describes the situation in which there exists a hyperplane able to separate, in the vector space of the inputs, those that require a positive output from those requiring a negative output.
[^details]: I do not want to bore you with maths, if you want to read more maths details [here](https://en.wikipedia.org/wiki/Backpropagation).