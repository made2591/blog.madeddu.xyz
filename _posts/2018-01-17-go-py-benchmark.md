---
title: "GoLang vs Python: a single-multicore comparison"
tags: [coding, golang, python, algorithms, benchmark]
---

### Introduction
In the last months I have worked a lot with GoLang on different projects. Although I'm certainly not an expert, there are several things that I really appreciate about this language: first, it has a clear and simple syntax, and more than once I noticed that the style of the Github developers is very close to the style that I saw in the old programs written in C. From a theoretical point of view, GoLang seems to take the best of all worlds: there is the power of high-level languages, made simple by clear rules - even if sometime they are a bit binding - that can impose a solid logic to the code. There is the simplicity of the imperative style, made of primitives of which you specify the size in bits, but without the boredom of having to manipulate strings as an array of characters. However, a really useful and interesting feature in my opinion are the goroutine and the channels.

<p align="center"><img src="https://ksr-ugc.imgix.net/assets/013/579/935/cd53c61559974d1fa22a094ecff1f8a3_original.jpg?crop=faces&w=1552&h=873&fit=crop&v=1472824649&auto=format&q=92&s=e7e49d2e6d5bcf4ef3a486facef11cc3" style="width: 100%; marker-top: -10px;"/></p>

### Preamble
To understand why GoLang handles concurrency better, you first need to know what concurrency exactly[^talk] is. Concurrency is the composition of independently executing computations. In Computer Science, concurrency is a way to structure software, particularly is a way to write clean code that interacts well with the real world: yes, this means that concurrency $$\neq$$ parallelism, although it _enables_ parallelism. So, if you have only one processor, your program can still be concurrent but it cannot be parallel. On the other hand, a well-written concurrent program might run efficiently in parallel on a multiprocessor[^rob]. That property could be important.

#### Goroutine
Suppose we have a function call ```f(s)```: this is how we'd call that in the usual way, running it _synchronously_. To invoke this function in a goroutine, use ```go f(s)```. This new goroutine will execute _concurrently_ with the calling one. But... what is a goroutine? It's an independently executing function, launched by a go statement. It has its own call stack, which grows and shrinks as required and it's very cheap. It's practical to have thousands, even hundreds of thousands of goroutines, but it's not a thread. In fact, there might be only one thread in a program with thousands of goroutines. Instead, goroutines are multiplexed dynamically onto threads as needed to keep all the goroutines running. But if you think of it as a very cheap thread, you won't be far off.

{% highlight go %}
package main

import "fmt"

func f(from string) {
    for i := 0; i < 3; i++ {
        fmt.Println(from, ":", i)
    }
}

func main() {

    // Suppose we have a function call `f(s)`. Here's how
    // we'd call that in the usual way, running it
    // synchronously.
    f("direct")

    // To invoke this function in a goroutine, use
    // `go f(s)`. This new goroutine will execute
    // concurrently with the calling one.
    go f("goroutine")

    // You can also start a goroutine for an anonymous
    // function call.
    go func(msg string) {
        fmt.Println(msg)
    }("going")

    // Our two function calls are running asynchronously in
    // separate goroutines now, so execution falls through
    // to here. This `Scanln` code requires we press a key
    // before the program exits.
    var input string
    fmt.Scanln(&input)
    fmt.Println("done")
}
{% endhighlight %}

#### Channels
Channels are a typed conduit through which you can send and receive values with the channel operator ```<-```. And that's all :D You only need to know that when a _main_ function executes ```<–c```, it will wait for a value to be sent. Similarly, when the _goroutined_ function executes ```c <– value```, it waits for a receiver to be ready. A sender and receiver must both be ready to play their part in the communication. Otherwise we wait until they are.
Thus channels __both communicate and synchronize__.

{% highlight go %}
package main

import "fmt"

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)
}
{% endhighlight %}

#### Summary
You can call a function - even anonymous - in a goroutine. Then put the result in a channel an, by default, sends and receives block until the other side is ready. This allows goroutines to synchronize without explicit locks or condition variables. Ok but... how do they perform?

### GoLang vs Python
Ok, I'm a Python lover - I guess, because it's in the title and I don't remember where the .md respective source is - so I decided to make a comparision to see how these magical GoLang tricky statements really perform. To do that, I decided to write a simple task: a merge sort, that can be run in a single-core environment or multicore environment. This is decided by passing the number of cores you want to use - the number of subprocess/goroutines call - 



Thank you everybody for reading!

[^talk]: There are plenty of beautiful [slides](https://talks.golang.org/2012/concurrency.slide) of a GoLang talk online!
[^rob]: The lesson of [Rob Pike - Concurrency Is Not Parallelism](https://vimeo.com/49718712)











