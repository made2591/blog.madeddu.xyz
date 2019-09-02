---
layout: post
title: "A Golang Turing machine library"
date: 2019-01-19
categories:
- coding
- golang
- miscellaneous
- algorithms
tags:
- coding
- golang
- busy-beaver
- tibor
- computational
- theory
katex: true
markup: "mmark"
---

### Preamble
In 1962, Hungarian mathematician Tibor Radó introduced the Busy Beaver competition for Turing machines: in a class of machines, find one which halts after the greatest number of steps when started on the empty input. Even if it could seem trivial, the Busy Beaver competition has implications in computability theory, the halting problem, and complexity theory.

I decided to use GoLang to implement a Turing machine library and accomplish three goals: first, having a Turing Machine model to play with for learning purpose; second, learning how to use interfaces and the factory pattern, other then testing package to test my code and let it be more flexible for future enhancement (at least I hope!); third, implement some Busy Beaver setup and verify that the model works with well known executions. If you want to discover more about Golang, 60's math games and beavers, go ahead with reading :D!

<div class="img_container"><img src="https://i.imgur.com/DJeK9E4.png" style="width: 70%; marker-top: -10px;"/></div>

### Theory first
Before starting, let's define a Turing machine. A Turing machine is a mathematical model of computation that defines an abstract machine which manipulates symbols on a strip of tape according to a list of rules. Formally, we can image a infinite tape of 0 with a pointer (identified by square bracket) to one specific zero,

$$ ... 0 0 0 0 0 [0] 0 0 0 0 0  ... $$

a set of states identified by letters (or numbers),

$${A, B, C}$$

and a list of transactions, like the one

$$(A, 0, B, 1, R);$$

where a single transaction like *(A, 0, B, 1, R)* has to been read as

> *Given a Turing machine in state A with the pointer over a 0, write 1, evolve to state B and move the pointer by one position in right direction over the tape*.

The pointer can only be moved by one position at time, left, right, or stay where it is. Everything's clear?

### Implementation
To implement my Turing machine, I choose Golang as language and, to be more flexible, I coded by using the factory pattern. What is a factory pattern?

#### The factory pattern
The factory pattern is a commonly used pattern in object oriented programming: the main reason you decide to use factory pattern is that it provide to thirds a way to better consume your struct. Instead of initializing instances using something like ```myStruct := &MyStruct{}```, a factory pattern provide a function signature that return your struct, by ensuring that everyone will supply the required attributes.

Now, the cool thing is that in Golang functions *can return interfaces instead of structs*: interfaces allow you to define behaviour without exposing internal implementation. As in other programming languages, you define method to be implemented inside an interface, and every struct that implement them is considered *implementing* the interface. This means we can make private structs, while only exposing the interface outside our package, and let user interact with the struct with the only available method inside.

#### Transactions
The first lines of the package ```Transaction``` look like the one shown below:

{{< highlight golang >}}
var ACTIONS = [...]string{"R", "L", "N"}

// Transaction interface
type Transaction interface {
	Validate(m TuringMachine) bool
	Simulate() (state.State, symbol.Symbol, string)
	GetCurrentState() state.State
	GetSymbolScanned() symbol.Symbol
	GetNewState() state.State
	GetSymbolWritten() symbol.Symbol
	GetMoveTape() string
	Print() string
}

// transaction struct
type transaction struct {
	currentState  state.State
	symbolScanned symbol.Symbol
	newState      state.State
	symbolWritten symbol.Symbol
	moveTape      string
}
{{< / highlight >}}

where ```state.State``` and ```symbol.Symbol``` refer respectively to the state, symbol packages and the State, Symbol interfaces defined inside them. The ```Transaction``` interface defines which methods have to be exposed outside: the ```transaction``` struct implements the Transaction interface. Let's have a look at a couple of methods after that:

{{< highlight golang >}}
// NewTransaction() Create a new Transaction with given
// currentState, symbolScanned, symbolWritten and moveTape action
func NewTransaction(currentState state.State, symbolScanned symbol.Symbol,
    newState state.State, symbolWritten symbol.Symbol, moveTape string) Transaction {

	t := &transaction{}
	t.currentState = currentState
	t.symbolScanned = symbolScanned
	t.newState = newState
	t.symbolWritten = symbolWritten
	t.moveTape = moveTape

	return t

}
{{< / highlight >}}

The ```NewTransaction(...)``` functions solve the problem of exposing the ```transaction``` struct, by return a pointer to a struct as the Transaction interface: in fact, since the transaction struct is private we wouldn't be able to interfact with it. That's the reason the signature actually returns an interface, that is exposed: this is possible because the pointers can also implement interfaces.

{{< highlight golang >}}
// Validate(m TuringMachine) Validate the transaction t with respect
// to the TuringMachine m. A Transaction to be considered valid
// need to be defined with a valid ACTIONS, the current state of the
// TuringMachine must be equal to the state in which the transaction
// is activated and the head of the TuringMachine must point to the
// the same symbol scanned by the transaction. If all of this three
// condition are verified, Validate(m TuringMachine) over t returns
// true; it returns otherwise
func (t *transaction) Validate(m TuringMachine) bool {

	// check if moveTape action is allowed
	for _, a := range ACTIONS {
		if strings.EqualFold(a, t.moveTape) {
			// check if actual state and scanned
                        // symbol match with transaction
			return t.currentState.Equal(m.GetActualState())
                            && t.symbolScanned.Equal(m.GetActualSymbol())
		}
	}

	return false

}
{{< / highlight >}}

The ```Validate``` function return true if a transaction is valid inside the Turing Machine m. A Transaction is valid if

- the action to be execute is a valid action identifier - it's contained in the set of {"L","R","N"};
- the current state of the Turing machine is equal to the state in which the transaction is activated;
- the current head of the Turing machine points to the the same symbol scanned by the transaction;

Similarly, you can implement ```Symbol```, ```State``` and ```TuringMachine```. Since Symbol and State packages are pretty easy to implement, let's have a look the Turing machine definition and the execution method defined inside it.

#### Turing Machine
The Turing machine interface is defined as shown below:

{{< highlight golang >}}
// TuringMachine interface
type TuringMachine interface {
	Run()
	Step() state.State
	Computed() bool
	GetActualSymbol() symbol.Symbol
	GetActualState() state.State
	MoveHeadPointer(s symbol.Symbol, m string) int
	Print() string
}

// turingMachine struct
type turingMachine struct {
	initialStates set.Set
	finalStates   set.Set
	transactions  set.Set
	actualState   state.State
	headPointer   int
	tape          []symbol.Symbol
}
{{< / highlight >}}

The key methods are ```Run()```, ```Step()``` and ```Computed()```. The Run() method is the simplest one: it runs the Step() function indefinitely until the Turing machine is computed completely.

{{< highlight golang >}}
func (tm *turingMachine) Run() {

	for !tm.Computed() {
		tm.Step()
	}
	return

}
{{< / highlight >}}

A Turing machine is computed completely when it reaches a Final State, that is exactly what the Computed method said about our Turing machine.

{{< highlight golang >}}
func (tm *turingMachine) Computed() bool {

	return tm.actualState.IsFinal()

}
{{< / highlight >}}

What about the Step() function?

{{< highlight golang >}}
func (tm *turingMachine) Step() state.State {

	for _, t := range tm.transactions.Iterator() {
		if t.(Transaction).Validate(tm) {
			return tm.Execute(t.(Transaction))
		}
	}
	return tm.actualState

}
{{< / highlight >}}

For the execute part, you can have a look directly [here](https://github.com/made2591/go-tm/blob/master/turing/machine/machine.go).

### Before going ahead
Testing! I learn how much you gain by testing your code playing with returns, default, pointers and indexes. Fortunately, build testing in Golang is quite easy, specially for simple functions like the one we shown before. For instance, the following test function is defined in a ```symbol_test.go``` file - at the same level of the ```symbol```. Then

{{< highlight golang >}}
func TestErase(t *testing.T) {

	// test erase a Symbol
	s := NewSymbol(uint8(1))
	s.Erase()
	if v := s.GetValue(); v != BLANK {
		t.Errorf("Erase was incorrect, got: %d, want: %d.", v, uint8(BLANK))
	}

}
{{< / highlight >}}

The testing functions are always named like ```[Test]OriginalFunctionName``` and always accept only one parameter ```t *testing.T```[^gotesting]. After that, from inside your project folder run this in a shell:

{{< highlight sh >}}
go test ./...
{{< / highlight >}}

Just prepend GOCACHE=off if you want to ignore caching. For the entire code, follow the [link](https://github.com/made2591/go-tm).

Now it's time to build our Busy Beaver!!

### The Busy Beaver competition
The Busy Beaver game consists of designing a halting, binary-alphabet Turing machine which writes the most *1*s on the tape, using only a limited set of states. The rules for the 2-state game are as follows:

- The machine must have two states in addition to the halting state, and
- The tape starts with 0s only

As the player, you should conceive each state aiming for the maximum output of 1s on the tape while making sure the machine will halt eventually.

The *n-th* Busy Beaver, BB-n or simply "Busy Beaver" is the Turing machine that wins the n-state Busy Beaver Game. That is, it attains the maximum number of 1s among all other possible *n*-state competing Turing Machines. The BB-2 Turing machine, for instance, achieves four *1*s in six steps.

You can easily build the BB-2 Beaver with 4 simple transaction plus 1 to move from the initial state to the "A" state.

{{< highlight golang >}}

tr0 := transaction.NewTransaction(
	state.NewInitialState(),
	symbol.NewSymbol(uint8(0)),
	state.NewState(uint8(21)),
	symbol.NewSymbol(uint8(0)), "N")

tr1 := transaction.NewTransaction(
	state.NewState(uint8(21)),
	symbol.NewSymbol(uint8(0)),
	state.NewState(uint8(22)),
	symbol.NewSymbol(uint8(1)), "R")

tr2 := transaction.NewTransaction(
	state.NewState(uint8(21)),
	symbol.NewSymbol(uint8(1)),
	state.NewState(uint8(22)),
	symbol.NewSymbol(uint8(1)), "L")

tr3 := transaction.NewTransaction(
	state.NewState(uint8(22)),
	symbol.NewSymbol(uint8(0)),
	state.NewState(uint8(21)),
	symbol.NewSymbol(uint8(1)), "L")

tr4 := transaction.NewTransaction(
	state.NewState(uint8(22)),
	symbol.NewSymbol(uint8(1)),
	state.NewFinalState(),
	symbol.NewSymbol(uint8(1)), "R")

{{< / highlight >}}

Or, in a more user friendly format:

|     | 21   | 22      |
| --- | ---- | ------- |
| 0   | 1R22 | 1L21    |
| 1   | 1L22 | 1RFINAL |

with an initial transaction from INITIAL to 21 with symbol 0 scanned, that does nothing and ports to state 21.
Let's have a look at the evolution of the BB-2 Turing machine: first apply first transaction from INITIAL to A - it a sort of 0° Step that just change the state of the machine from INITIAL to 21, in such a way that the init tape looks like this

$$ ... 0 0 0 0 0 [0] 0 0 0 0 0 0 0 ... $$

### BB-2 Beaver Execution

<span style="color:#A04279; font-size: bold;">1° Step</span>: apply 1R22 transaction from A in 0 to B by writing 1, evolve in state B and move the cursor one step in the right direction

$$ ... 0 0 0 0 0 1 [0] 0 0 0 0 0 0 ... $$

<span style="color:#A04279; font-size: bold;">2° Step</span>: apply 1L21 transaction from B in 0 to A by writing 1, evolve in state A and move the cursor one step in the left direction

$$ ... 0 0 0 0 0 [1] 1 0 0 0 0 0 0 ... $$

<span style="color:#A04279; font-size: bold;">3° Step</span>: apply 1L22 transaction from A in 1 to B by writing 1 (this will result in an overwrite) and move the cursor one step in the left direction

$$ ... 0 0 0 0 [0] 1 1 0 0 0 0 0 0 ... $$

<span style="color:#A04279; font-size: bold;">4° Step</span>: apply 1L21 transaction from B in 0 to A by writing 1, evolve in state A and move the cursor one step in the left direction

$$ ... 0 0 0 [0] 1 1 1 0 0 0 0 0 0 ... $$

<span style="color:#A04279; font-size: bold;">5° Step</span>: apply 1R22 transaction from A in 0 to B by writing 1, evolve in state B and move the cursor one step in the right direction

$$ ... 0 0 0 1 [1] 1 1 0 0 0 0 0 0 ... $$

<span style="color:#A04279; font-size: bold;">6° Step</span>: finally, apply 1RFINAL transaction from B in 1 to FINAL by writing 1 (this will result in an overwrite), evolve in state B and move the cursor one step in the right direction

$$ ... 0 0 0 1 1 [1] 1 0 0 0 0 0 0 ... $$

And the execution is finished! Four *1*s, six steps, as promised!

To be continued with Non Deterministic experiment...

### Conclusion
For the most curious people, the original paper is Tibor Radó - *On Non-Computable Functions* - [here](http://infoteorica.weebly.com/uploads/1/7/8/9/17895653/rado_on_non-computable_functions_bell_system_technical_journal_41_1962_pp._877-884.pdf).
My Github repo with my implementation of the Busy Beaver game is available [here](https://github.com/made2591/go-tm).

[^gotesting]: [Here](https://github.com/golang/go/blob/master/src/testing/testing.go) the code