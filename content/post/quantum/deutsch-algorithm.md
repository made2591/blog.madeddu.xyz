---
layout: post
title: "The Deutsch Algorithm"
date: 2019-01-23
categories:
- theory
- quantum
- algorithms
tags:
- theory
- quantum
- parallelism
- q
- informative
- ibm
- deutsch
- algorithm
---

### Much more than a post (again)
What is the quantum theory? As said by [quantumexperience](https://quantumexperience.ng.bluemix.net/) official site by IBM, it's _an elegant mathematical theory able to explain the counterintuitive behavior of subatomic particles, most notably the phenomenon of entanglement_. In the late twentieth century it was discovered that quantum theory applies not only to atoms and molecules, but to bits and logic operations in a computer. This realization has been bringing about a revolution in the science and technology of information processing: I decided to write some notes to better explain, from a physics-agnostic computer scientist's point of view XD, __what I understood__ - and it is certainly wrong - about Q until now and why I think it's an amazing field for computer science. More on this story in [my previous post](https://madeddu.xyz/posts/quantum-computing).

So - back to the origins - why am I writing this post? Because I recently came over my quantum notes again and YES, I'M CONTINUING THEM (clap clap clap), even if, unfortunately, I don't have a lot of time to dedicate to it - you know, the always-valid excuse of life *I don't have time*.
This post is about a specific algorithm - one of the basic reasoning to be done about *quantum parallelism* (more on this in a few lines): I'm gonna talk about the Deutsch Algorithm, the reason behind it, how it works and I will literally vomit what I collected (a sort of preview XD) in the last crazy Sunday of study as a mathematical demonstration of its component.

But... before going into details, let's make some reasoning over classical computation first.

<div class="img_container"><img src="https://i.imgur.com/qmFWYIi.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Agenda

- Classic computation: reversible and irreversible functions + some mentalist tricks to engage you
- Toffoli classic gate: aka... a gate *to rule them aaaall* 😂 + no other abuse of this sentence, I swear
- Toffoli quantum gate: aka... what the hell is going on dude here!??!?! -> as [Eminem said](https://www.youtube.com/watch?v=9dcVOmEQzKA&feature=youtu.be&t=145)
- The Deutsch Algorithm: the basic fundation of quantum parallelism - fact and proof (what I got)
- Conclusion: just some random thoughts about the topic

Unfortunately, the first three sections are needed to go throught the demonstration of the Deutsch algorithm. At least, I tried to give a little bit of context to better understand the reasons behind the algorithm. Let's start this journey and sorry if it will take some time :/

<span style="color:#FF8C00; font-size: bold;">EASTER EGG</span>: And for the very first time, there's an easter egg (kind of - at least) in the blog post!

### Classic computations
A fundamental difference between classical and quantum circuits is that theclassical logic gates could be irreversible (for example ```AND```, ```XOR```, ```NAND```), while the quantum logic gates are always unitary and therefore  reversible. On the other hand, it would be desirable for an alternative computation model to beable to express at least all computations that can be expressed with the classical model. So the first objective to talk about quantum computation is therefore to represent the classical computationsas unitary transformations, i.e. as quantum computations.

#### Reversible vs Irreversible
Since unitary transformations are invertible (i.e. reversible), the first step is to transform any irreversible classical computation into a reversible one. In order to operate in a reversible way it is necessary that the function to be evaluated is a bjection (i.e. [injective](https://en.wikipedia.org/wiki/Injective_function) and [surjective](https://en.wikipedia.org/wiki/Surjective_function)). In this case we can in fact unequivocally trace from each output to the value of the input that generated it, that is, operate in reverse. Any irreversible computation can be transformed into an equivalent reversible computation, making the corresponding function to be biunivocally evaluated.

For example, given any function

\\(f : \{0, 1\}^{k} \mapsto \{0, 1\}^{m}\\)

it is possible to construct

\\(\widetilde{f} : \{0, 1\}^{k+m} \mapsto \{0, 1\}^{k+m}\\)

such that \\(f\\) is biunivocal and calculates \\((x,f(x))\\) by acting on the input \\((x,0^m)\\), where \\(0^m\\) denotes \\(m\\) bits initialized with value 0. Each biunivocal function:

\\(f : \{0, 1\}^{n} \mapsto \{0, 1\}^{n}\\)

can be actually seen as a permutation on the \\(n\\) bits in input or, equivalently, on integers \\(0,1, ...,2^{n−1}\\). Accordingly, it describes a classical reversible computation. Take a moment to reflect on this - *mentalist trick n°1*.

#### Toffoli gate
Any irreversible classical computation can be transformed into an equivalent *but reversible* computation using the [Toffoli gate](https://en.wikipedia.org/wiki/Toffoli_gate). This is a classic reversible operation, represented by the circuit shown below, which operates on three input bits: two are *control bits* and the third is the target bit that is exchanged if the control bits are both 1, as show in the truth table.

<div class="img_container"><img src="https://i.imgur.com/XKgkF81.png"  style="width: 40%; marker-top: -10px;"/></div>

|     | In  |     |     | Out |     |
| --- | --- | --- | --- | --- | --- |
| a   | b   | c   |     | a'  | b'  | c' |
| 0   | 0   | 0   |     | 0   | 0   | 0  |
| 0   | 0   | 1   |     | 0   | 0   | 1  |
| 0   | 1   | 0   |     | 0   | 1   | 0  |
| 0   | 1   | 1   |     | 0   | 1   | 1  |
| 1   | 0   | 0   |     | 1   | 0   | 0  |
| 1   | 0   | 1   |     | 1   | 0   | 1  |
| 1   | 1   | 0   |     | 1   | 1   | 1  |
| 1   | 1   | 1   |     | 1   | 1   | 0  |

The reversibility of this operation is easily verified by observing that by applying the Toffoli gate twice in a row the same starting result is obtained (two value are ported as they are, the third one is a ```XOR``` that is reversible by design thus is verified):

\\((a, b, c) \rightarrow{} (a, b, c \oplus ab) \rightarrow{} (a, b, c)\\)

So the operation itself coincides with its inverse. It is equally easy to verify that the Toffoli gate represents the permutation \\(\pi = (67)\\) on integers \\(0, 1, \ldots , 7\\) (exchanges the two sequences \\(110\\) and \\(111\\)).

#### NAND and FANOUT operation
Toffoli's gate is universal for the classic reversible computations, that is, every classical computation can be built in a reversible way through the Toffoli gate. This result follows from the universality of the operations of ```NAND``` and ```FANOUT``` (the operation of copying a classic bit) for the classical computations and from the fact that both these operations can be expressed through the Toffoli circuit. In fact, by applying the operation with \\(c = 1\\), we obtain \\(a^{'} = a\\), \\(b^{'} = b\\) and \\(c^{'} = 1 \oplus ab = \neg ab\\), i.e. we obtained the simulation of ```NAND``` and it is also a reversible operation because Toffoli port is. The reversible ```FANOUT``` is instead obtained as shown in the picture above: by applying the Toffoli gate with \\(a = 1\\) and \\(c = 0\\) the result is the copy of bit \\(b\\) (remember that this copy operation is not possible for a qubit!!!).

<div class="img_container"><img src="https://i.imgur.com/Rhk2aKZ.png"  style="width: 30%; marker-top: -10px;"/></div>

As for ```NAND}``` and ```FANOUT``` the construction of a reversible circuit for any classical operation \\(f\\) by means of the Toffoli port involves the use of some service bits in input (or *ancilla bits*) and in output (or *garbage*). After deleting these service bits, the resulting circuit performs the transformation:

\\((x, y) \mapsto (x, y \oplus f(x))\\)

(where \\(x\\) is the input of \\(f\\) and \\(y\\) is the register intended to contain the output) and can be considered as the *standard reversible circuit* for the evaluation of \\(f\\).

#### Classical computations on quantum circuits
As already observed, a classical reversible computation corresponds to a permutation on the sequences of the input bits. This guarantees the possibility of constructing a complex unitary matrix that represents it.

In particular, the Toffoli gate can be implemented as quantum circuit. In this case the input is given by three qubits and the transformation, analogous to the classical case, consists in the exchange of the third qubit if the first two are \\(1\\). For example the quantum Toffoli gate applied to the state \\(\vert 110\rangle\\) produces the state \\(\vert 111\rangle\\). Thus...

> The quantum Toffoli port can then be used to simulate all the classical computations on a quantum computer, ensuring that a quantum computer is able to perform any computable computation on a classic computer.

.... BOOOOOM

<div class="img_container"><img src="https://i.imgur.com/OsZZs0h.gif" style="width: 100%; marker-top: -10px;"/></div>

Let's go ahead by exploring how a quantum Toffoli gate can be used.

#### Probabilistic computations on quantum circuits
*Randomized* algorithms are algorithms that are executed using a random number generator (the launch of a coin) to decide one of the possible branches of execution. The first randomized algorithm was introduced by Solovay and Strassen in the 1970s to determine whether a number is prime or not. The algorithm produces a correct answer only with a certain probability. This probability can be increased by repeating the execution for an appropriate number of times.

These algorithms can also be efficiently simulated by quantum circuits. In fact, to simulate a random bit it is sufficient to prepare a qubit in the \\(\vert 0\rangle\\) state and then apply the Hadamard port. You will get the status \\(\frac{\vert 0\rangle + \vert 1\rangle}{\sqrt{2}}\\) that measured will give \\(0\\) or \\(1\\) each with probability \\(1/2\\). It should also be noted that in this way a *really random number* is obtained, *something that a classic computer can not do*... (yes, this should let you think).

### Quantum parallelism
On a quantum computer, a function \\(f(x)\\) can be evaluated on different values of \\(x\\) at the same time. This is known as *quantum parallelism* and is a fundamental characteristic of quantum circuits. Consider a boolean function of the form:

\\(f : \{0, 1\} \mapsto \{0, 1\}\\)

To calculate \\(f(x)\\) by means of a quantum computation the transformation \\(f(x)\\) must be defined as a unit transformation \\(U_f\\). As seen previously, this can be done by applying on the input state \\(\vert x,y\rangle\\), let's say our data register[^dataregister], an appropriate sequence of quantum logic gates (which we will indicate with a black box called \\(U_f\\)) that transform \\(\vert x,y\rangle\\) into the state \\(\vert x,y \oplus f(x)\rangle\\), called the target register. If \\(y = 0\\) then the final state of the second qubit will accurately contain the value of \\(f(x)\\), because of the \\(\oplus\\)'s (```XOR```) true table.

<div class="img_container"><img src="https://i.imgur.com/pFzHTwK.png"  style="width: 60%; marker-top: -10px;"/></div>

In the circuit in shown above, the input is

\\(\frac{\vert 0\rangle + \vert 1\rangle}{\sqrt{2}} \otimes \vert 0\rangle\\)

that is[^recall], the value of \\(x\\) is an overlap of \\(0\\) and \\(1\\) that can be obtained by applying Hadamard to \\(\vert 0\rangle\\). Applying \\(U_f\\) to this data register is obtained[^expl1]:

\\(\frac{\vert 0, f(0)\rangle + \vert 1, f(1)\rangle}{\sqrt{2}}\\)

This state contains information both on the value \\(f(0)\\) and on the value \\(f(1)\\).

> We just evaluated  \\(f\\) simultaneously on \\(x\\) = 0 and \\(x = 1\\).

This type of parallelism is deeply different from the classical one where multiple circuits (each of which calculates \\(f(x)\\) for a single value of \\(x\\)) are executed simultaneously.

<div class="img_container"><img src="https://i.imgur.com/pcbb5sv.jpg" style="width: 100%; marker-top: -10px;"/></div>

Please take some time to reflect on this if you are not convinced before going ahead - *mentalist trick n°2*.

#### One step more
This procedure can be generalized to calculate functions on an arbitrary number of bits using a generalization of the Hadamard gate known as the **Walsh-Hadamard** transform. This operation consists of \\(n\\) Hadamard ports acting in parallel on \\(n\\) qubits. For example, for \\(n = 2\\), the Walsh-Hadamard transform is indicated with \\(H^{\otimes 2} = H \otimes H\\) and applied to two qubits prepared in the state \\(\vert 0\rangle\\) gives as a result:

\\(\frac{\vert 0\rangle + \vert 1\rangle}{\sqrt{2}} \otimes \frac{\vert 0\rangle + \vert 1\rangle}{\sqrt{2}} = \frac{\vert 00\rangle + \vert 01\rangle + \vert 10\rangle + \vert 11\rangle}{2}\\)

In general, the result of \\(H^{\otimes n}\\) applied to \\(n\\) qubits in the \\(\vert 0\rangle\\) state is:

\\(\frac{1}{\sqrt{2^n}}\sum\limits_{x}\vert x\rangle\\)

where \\(x\\) is the binary representation of the numbers from \\(0\\) to \\(2^n - 1\\). Thus...

> The Walsh-Hadamard transform produces an equiprobable overlap of all the states of the \\(n\\) qubits computational basis.

Note: <span style="color:#FF8C00; font-size: bold;">to obtain an overlap of \\(2^n\\) states only \\(n\\) logical ports are needed.</span> We are getting closer...

The parallel evaluation of a function \\(f(x)\\), with input \\(x\\) of \\(n\\) bits and \\(1\\) bit as output, can therefore be performed by a circuit similar to last one shown before, with \\(n+1\\) qubit in input prepared in the \\(\vert 0\rangle^{\otimes n}\vert 0\rangle\\). Then Hadamard applies to the first \\(n\\) qubits and then the \\(U_f\\) circuit is applied. The result will be:

\\(\frac{1}{\sqrt{2}}\sum\limits_{x}\vert x\rangle\vert f(x)\rangle\\)

#### Unfortunately...
Quantum parallelism is not directly usable in the sense that it is not possible to obtain all the values calculated with a single measurement: the measurement of the state above will give the value of \\(f(x)\\) for a single value of \\(x\\). To exploit the hidden information in this parallelism, we have to, somehow, make better use of the information contained in the overlap.

For example, by exploiting in an appropriate manner the interference between the states in the overlap. By combining quantum parallelism with this property that comes from quantum mechanics, results like the one exemplified by *the Deutsch algorithm* can be obtained. And FINALLY...

### The Deutsch Algorithm
The Deutsch algorithm shows how, through the parallel evaluation of a function on all its inputs, global properties of the function can be determined, such as, for example, that of being a constant or balanced function[^expl2]. Using a classical algorithm, in the worst case we need to evaluate the function on at least \\(2^{n-1} + 1\\) (am I wrong?) values in order to be able to establish with certainty whether \\(f\\) is constant or balanced.

<div class="img_container"><img src="https://i.imgur.com/5sjQuH5.png"  style="width: 60%; marker-top: -10px;"/></div>

The implementation of the Deutsch algorithm is shown in the quantum circuite above. The input of the circuit that calculates the function \\(f\\) is now the qubits resulting from the application of Hadamard to the \\(\vert 0\rangle\\) and \\(\vert 1\rangle\\) states. This input is therefore:

\\(\vert \psi_1\rangle = \vert x, y\rangle = \frac{\vert 0\rangle+\vert 1\rangle}{\sqrt{2}} \otimes \frac{\vert 0\rangle-\vert 1\rangle}{\sqrt{2}} = \frac{\vert 00\rangle - \vert 01\rangle + \vert 10\rangle - \vert 11\rangle}{\sqrt{2}}\\)

For simplicity, let's mantain the two initial qbits separated. Let's apply \\(U_f\\) to the state \\(\vert \psi_1\rangle\\) where

\\(x = \frac{\vert 0\rangle+\vert 1\rangle}{\sqrt{2}}\\)

and

\\(y = \frac{\vert 0\rangle-\vert 1\rangle}{\sqrt{2}}\\)

We already know that \\(U_f\\) doesn't change \\(x\\) and map the quantum system (our quantum register) \\(\vert x, y\rangle\\) to \\(\vert x, y \oplus f(x)\rangle\\).

Thus, applying \\(U_f\\) to \\(\vert x, y\rangle\\) means apply \\(U_f\\) to

\\(\vert x\rangle \frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}}\\)

whatever \\(\vert x\rangle\\) will be. The result of this application will be

\\(\frac{\vert x\rangle\vert y \oplus f(x)\rangle}{\sqrt{2}}\\)

where \\(\vert y \oplus f(x)\rangle\\) - once we measure \\(y\\) by collapsing to value \\(0\\) or \\(1\\) - is

\\(\frac{\vert 0 \oplus f(x)\rangle - \vert 1 \oplus f(x)\rangle}{\sqrt{2}}\\)

Thus,

\\(\frac{\vert x\rangle\vert y \oplus f(x)\rangle}{\sqrt{2}} = \vert x\rangle\frac{\vert 0 \oplus f(x)\rangle - \vert 1 \oplus f(x)\rangle}{\sqrt{2}}\\)

Take a moment to understand this step before going ahead.

Now, remember that \\(0 \oplus f(x) = f(x)\\) because of the nature \\(\oplus\\), thus the result of \\(U_f\\) applied to \\(\vert \psi_1\rangle\\) - always by keeping away \\(\vert x\rangle\\) for a while - is

\\(\vert x\rangle\frac{\vert f(x)\rangle - \vert 1 \oplus f(x)\rangle}{\sqrt{2}}\\)

Thus,

\\(\frac{\vert x\rangle\vert y \oplus f(x)\rangle}{\sqrt{2}} = \vert x\rangle\frac{\vert 0 \oplus f(x)\rangle - \vert 1 \oplus f(x)\rangle}{\sqrt{2}} = \vert x\rangle\frac{\vert f(x)\rangle - \vert 1 \oplus f(x)\rangle}{\sqrt{2}}\\)

Since \\(U_f\\) doesn't modify \\(\vert x\rangle\\) we only need to evaluate \\(y \oplus f(x)\\) - or \\(\vert f(x)\rangle - \vert 1 \oplus f(x)\rangle\\).

- If \\(f(x) = 0\\) this is simply \\(y\\) - in fact:

\\(\vert y \oplus f(x)\rangle = \frac{\vert f(x)\rangle - \vert 1 \oplus f(x)\rangle}{\sqrt{2}} = \frac{\vert 0\rangle - \vert 1 \oplus 0\rangle}{\sqrt{2}} = \frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}} = H\vert 1\rangle = y\\)

Realize that by replacing the value of \\(f(x)\\) with \\(0\\) we obtain exactly \\(y\\) that is our desiderata, thus

\\(\vert x\rangle\frac{\vert f(x)\rangle - \vert 1 \oplus f(x)\rangle}{\sqrt{2}} = \vert x\rangle\frac{\vert y\rangle}{\sqrt{2}}\\)

Take some moment to convince about this step.

- Otherwise, if \\(f(x) = 1\\) then the result is

\\(\vert x\rangle\frac{\vert f(x)\rangle - \vert 1 \oplus f(x)\rangle}{\sqrt{2}} = \vert x\rangle\frac{\vert 1\rangle - \vert 1 \oplus 1\rangle}{\sqrt{2}} = \vert x\rangle\frac{\vert 1\rangle - \vert 0\rangle}{\sqrt{2}} = \vert x\rangle\frac{-\vert y\rangle}{\sqrt{2}}\\)

Now, let's keep a part \\(1/\sqrt{2}\\(: we can rewrite

\\(\vert x\rangle\vert f(x)\rangle - \vert 1 \oplus f(x)\rangle), f(x) \in \{0, 1\}\\)

in

\\((-1)^{f(x)}\vert x\rangle(\vert 0\rangle - \vert 1\rangle), f(x) \in \{0, 1\}\\)

<span style="color:#FF8C00; font-size: bold;">Note</span>: if someone is able to convince me about this, please comment it below or feel free to contact me at <a href="mailto:matteo.madeddu@gmail.com">matteo.madeddu@gmail.com</a> because I didn't find a real good explanation to this. Anyway, let's assume it's true because of some trick (I have a theory, that is replacing \\(((-1)^{f(x)}\\) with \\((-f(x)^{f(x)}\\)) over signs, and go ahead because the rest it seems *ok* imho.

Thus, by applying \\(U_f\\) to \\(\vert \psi_1\rangle\\) we obtain a result \\(\vert \psi_2\rangle\\) that varies over two possibilities

\\(f(0) = f(1) \rightarrow{} \pm \left[\frac{\vert 0\rangle + \vert 1\rangle}{\sqrt{2}} \otimes \frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}}\right]\\)

\\(f(0) \neq f(1) \rightarrow{} \pm \left[\frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}} \otimes \frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}}\right]\\)

Note that in the second alternative, we have that \\((-1)^{f(1)} = -(-1)^{f(0)}\\). Note also that \\(\vert \psi_1\rangle\\) can be written as

\\(\frac{1}{\sqrt{2}}\left(\vert 0\rangle\frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}} + \vert 1\rangle\frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}}\right)\\)

Thus \\(U_f\\) applied to \\(\vert \psi_1\rangle\\) can be written as

\\(\frac{1}{\sqrt{2}}\left((-1)^{f(0)}\vert 0\rangle\frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}} +  (-1)^{f(1)}\vert 1\rangle\frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}}\right)\\)

or, equally, as

\\(\frac{1}{\sqrt{2}}\left((-1)^{f(0)}\vert 0\rangle\frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}} -  (-1)^{f(0)}\vert 1\rangle\frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}}\right)\\)

(because in the second case or \\((-1)^{f(1)} == -(-1)^{f(0)}\\) since \\(f(0) \neq f(1)\\)) or, even, as

\\(\pm\frac{1}{\sqrt{2}}\left(\vert 0\rangle\frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}} + \vert 1\rangle\frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}}\right)\\)

that is

\\(\pm\left[\frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}} \otimes \vert \frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}}\right]\\)

Now we apply Hadamard to the first qubit and we obtain \\(\vert \psi_3\rangle\\) which results in

\\(f(0) = f(1) \rightarrow{} \pm\vert 0\rangle \left[\frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}}\right]\\)

\\(f(0) \neq f(1) \rightarrow{} \pm\vert 1\rangle \left[\frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}}\right]\\)

At this point we observe that \\(f(0) \oplus f(1) = 0\\) if \\(f(0) = f(1)\\), otherwise \\(f(0) \oplus f(1) = 1\\). We can therefore write the result in a more concise way

\\(\vert \psi_3\rangle = \pm \vert f(0) \oplus f(1)\rangle \left[\frac{\vert 0\rangle - \vert 1\rangle}{\sqrt{2}}\right]\\)

Through a measurement of the first qubit we can then determine with certainty (the probability associated with the first qubit is 1) the value of \\(f(0) \oplus f(1)\\) and therefore if the function \\(f\\) is constant or balanced. To do this we had to evaluate \\(f(x)\\) only once.

### Conclusion
The Deutsch algorithm can be extended to Boolean functions on \\(n\\) bits. Let us consider a function \\(f:\{0,1\}^n \rightarrow{} \{0,1\}\\) and suppose to know that \\(f\\) can be either constant or balanced. The quantum algorithm of Deutsch-Jozsa allows us to establish it in one step. The quantum circuit that implements this algorithm is the same Deutsch algorithm described with input \\(x\\) of the function of \\(n\\) qubits prepared in the \\(\vert 0\rangle\\) state, which we will call the data register. The qubit target, intended to contain the result of \\(f(x)\\), is instead prepared in the \\(\vert 1\rangle\\) state.

To be continue...

Thank you everybody for reading!

[^dataregister]: A quantum system of two qbits.
[^recall]: If this is not clear, recall the quantum register definition.
[^expl1]: I applied \\(y \oplus f(x)\\) and the distribution property.
[^expl2]: That is, it takes value \\(0\\) on exactly half of the inputs and value \\(1\\) on the remaining half.