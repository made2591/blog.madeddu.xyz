---
layout: post
title: "GoLang vs Python: deep dive into the concurrency"
date: 2018-01-17
categories:
- coding
- golang
- python
tags:
- coding
- golang
- python
- goroutine
- algorithms
- benchmark
---

### Introduction
In the last months, I worked a lot with GoLang on several projects. Although I'm certainly not an expert, there are several things that I really appreciate about this language: first, it has a clear and simple syntax, and more than once I noticed that the style of the Github developers is very close to the style used in old C programs. From a theoretical point of view, GoLang seems to take the best of all worlds: there is the power of high-level languages, made simple by clear rules - even if sometime they are a little bit binding - that can impose a solid logic to the code. There is the simplicity of the imperative style, made of primitive types with the size in bits in their name, but without the boredom of manipulating strings as array of characters. However, two really useful and interesting features in my opinion are the goroutine and the channels.

<div class="img_container"><img src="https://i.imgur.com/MDMOE2M.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Preamble
To understand why GoLang handles concurrency better, you first need to know what concurrency exactly[^talk] is. Concurrency is the composition of independently executing computations: better, is a way to write clean code that interacts well with the real world. Often people confuse the concept of concurrency with the concept of parallelism, even if concurrency \\(\neq\\) parallelism: yes, although it _enables_ parallelism. So, if you have only one processor, your program can still be concurrent but it cannot be parallel. On the other hand, a well-written concurrent program might run efficiently in parallel on a multiprocessor[^rob]. That property could be important.
Let's talk about how GoLang let your program takes advantage of running in a multiprocessor/multithreaded environment, or, _what tools GoLang provides to write concurrent program_ because it's not about thread or core: it's all about routine.

#### Goroutine
Suppose we have a function call ```f(s)```: this is how we would call that in the usual way, running it _synchronously_. To invoke this function in a ```goroutine```, use ```go f(s)```. This new goroutine will execute _concurrently_ with the calling one. But... what is a goroutine? It's an independently executing function, launched by a ```go``` statement. It has its own call stack, which grows and shrinks as required and it's very cheap. It's practical to have thousands, even hundreds of thousands of goroutines, but it's not a thread. In fact, there might be only one thread in a program with thousands of goroutines. Instead, _goroutines are multiplexed dynamically onto threads as needed to keep all the goroutines running_. If you think of it as a very cheap thread, you won't be far off.

{{< highlight go >}}
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
{{< / highlight >}}

##### More in details[^docr]
As I said, the idea behind the coroutine is to multiplex __independently executing functions__ — coroutines — onto a set of threads. When a coroutine blocks, such as by calling a blocking system call, the __run-time automatically moves other coroutines on the same operating system thread to a different, runnable thread__ so they won't be blocked. These coroutine are called __goroutines__ and are very cheap: they have little overhead beyond the memory for the stack, which is just a few kilobytes. Further, to make the stacks small, Go's run-time uses resizable, bounded stacks. A newly minted goroutine is given a few kilobytes, which is almost always enough. When it isn't, the run-time grows (and shrinks) the memory for storing the stack automatically, allowing many goroutines to live in a modest amount of memory. The CPU overhead averages about three cheap instructions per function call, and that the reason why it's so practical to create hundreds of thousands of goroutines in the same address space. If goroutines were just threads, system resources would run out at a much smaller number.

Ok, really cool but...why!?!? Why do we write concurrent program?! To do our jobs faster (even if writing correct concurrent program could take you more time than the amount of time you would gain running your task in a parallel environment XD). A tipical threaded situation includes a main thread that allocates some shared memory and stores its location in ```p```; than main thread starts \\(n\\) worker threads, passing the pointer ```p``` to them and the workers can use ```p``` and work on the data pointed to by ```p```. But what if threads start updating the same memory address - I mean, this is one of the hardest point of computer science. Ok, let's keep it simple: from _the-operating-system-point-of-view_, some atomic system calls let you lock the access to a ```shared memory``` zone (I'm talking about semaphores, messages queues, locks, etc). From the _language-poin-of-view_, there are normally a set of primitive that - in the end - call the required system calls and let you sync the access to a shared memory zone (I'm talking about packages like multiprocessing, multithreading, pools, etc). Let's talk about a tool of GoLang that help you deal with concurrency comunication between goroutine: the channels.

#### Channels
Channels are a typed conduit through which you can send and receive values with the channel operator ```<-```. And that's all :D You only need to know that when a _main_ function executes ```<–c```, it will wait for a value to be sent. Similarly, when the _goroutined_ function executes ```c <– value```, it waits for a receiver to be ready. A sender and receiver must both be ready to play their part in the communication. Otherwise we wait until they are: you don't have to deal with semaphores, locks, etc: channels __both communicate and synchronize__. This is really important to remember and understand, and also one of the biggest difference between GoLang and other languages I know.

{{< highlight go >}}
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
{{< / highlight >}}

##### More in details[^docc]
As official documentation states, a channel _provides a mechanism for concurrently executing functions to communicate by sending and receiving values of a specified element type_. It's - quite - simple. What I didn't say yet, is that a channel as a type, different from the type of messages it admits:

    ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType

The optional ```<-``` operator specifies the channel direction, send or receive. If no direction is given, the channel is bidirectional. A channel may be constrained only to send or only to receive by conversion or assignment.

{{< highlight go >}}

    chan T          // can be used to send and receive values of type T
    chan<- float64  // can only be used to send float64s
    <-chan int      // can only be used to receive ints

{{< / highlight >}}

To help you in solving some particular sync problems, you can also create a ```buffered channel```, using the function make (```make(chan int, 100)```). The capacity, in number of elements, sets the size of the buffer in the channel. If the capacity is zero or absent, the channel is unbuffered and communication succeeds only when both a sender and receiver are ready. Otherwise, the channel is buffered and communication succeeds without blocking if the buffer is not full (sends) or not empty (receives). A nil channel is never ready for communication: I found out that by using a buffered channel you can implicit set the maximum number of go routine to have at runtime and this will be really usefull for my benchmark.

#### Summary
To summarize, you can call a function - even anonymous - in a goroutine. Then put the result in a channel and, by default, sends and receives block until the other side is ready. All these features _allows goroutines to synchronize without explicit locks or condition variables_. Ok but... how do they perform?

### GoLang vs Python
Ok, I'm a Python lover - I guess, because it's in the title and I don't remember where the .md respective source is - so I decided to make a comparision to see how these magical GoLang tricky statements really perform. To do that, I wrote a simple go-py program ([here](https://github.com/made2591/go-py-benchmark) the code) that completes the merge sort over a list of random integers and can be run in a single-core environment or multicore environment. Or, in a single-_routine_ or multi-_routine_ environment: this is because, as I said, go-routine is a - unavailable in Python - concept that goes more in depth than thread - remember that more than one go-routine could belong to one single thread. Instead, from a Python point of you, you only can work with process, threads and also semaphores, locks, rlocks and so on but it's impossible to reproduce exactly the same computation - I mean, this is normal, they are different languages but both of them in the end call a set of system calls. In any case, I think that what you can do when you are running this kind of concurrency experiments is to reproduce a computation _as much as possible_ logically equivalent. Let's start from the GoLang version.

#### GoLang Merge Sort
Both GoLang and Python version of program provide two function:
- Single _routine_;
- Multiple _prefixed number of routine_;

##### Simple Go Version
Ok, I will not talk too much about single routine methods: it's really simple. Below you can see the code of the most optimized version I was able to think about (in terms of io operations, etc) - the commented version on [Github](https://github.com/made2591/go-py-benchmark/blob/master/main.go):

{{< highlight go >}}
func msort_sort(a []int) []int {
    if len(a) <= 1 {
        return a
    }
    m := int(math.Floor(float64(len(a)) / 2))
    return msort_merge(msort_sort(a[0:m]), msort_sort(a[m:]))
}

func msort_merge(l []int, r []int) []int {
    a := []int{}
    for len(l) > 0 || len(r) > 0 {
        if len(l) == 0 {
            a = append(a, r[len(r)-1])
            if len(r) > 1 {
                r = r[:len(r)-1]
            } else {
                r = []int{}
            }
        } else {
            if len(r) == 0 || (l[len(l)-1] > r[len(r)-1]) {
                a = append(a, l[len(l)-1])
                if len(l) > 1 {
                    l = l[:len(l)-1]
                } else {
                    l = []int{}
                }
            } else {
                if len(r) > 0 {
                    a = append(a, r[len(r)-1])
                    if len(r) > 1 {
                        r = r[:len(r)-1]
                    } else {
                        r = []int{}
                    }
                }
            }
        }
    }
    return reverse(a)
}
{{< / highlight >}}

I don't think it needs explanation: if you have any questions, don't hesitate write me in the comments! I will answer as soon as possible.

##### Concurrent Go Version
Let's talk about the __concurrent version__. We could split the array and call go sub routine from the main routine, but how can we control the maximum number of concurrent go-routine - or workers - to run? Well, one way[^s1] to limit concurrency in Go is by using a buffered channel (semaphore). As I said, when you create a channel with a fixed dimension - or buffered - communication succeeds without blocking if the buffer is not full (sends) or not empty (receives), so you can implements a _semaphore_ to easily block execution based on the number of concurrent units of actions you want to have. Really cool but... there is a problem: a channel is a channel, and even if buffered, basic sends and receives on channels are ```blocking```.
Fortunately, GoLang is simply awesome and let you create __explicit non-blocking channels__, using the ```select``` statement[^nbcs]: thus, you can use the select with default clause to implement non-blocking sends, receives, and even non-blocking multi-way selects. There are some others few statement to explain, after my _prefixed-maximum-number-of-concurrent-goroutine_ version of merge sort:

{{< highlight go >}}
// Returns the result of a merge sort - the sort part - over the passed list
func merge_sort_multi(s []int, sem chan struct{}) []int {

    // return ordered 1 element array
    if len(s) <= 1 {
        return s
    }

    // split length
    n := len(s) / 2

    // create a wait group to wait for both goroutine call before final merge step
    wg := sync.WaitGroup{}
    wg.Add(2)

    // result of goroutine
    var l []int
    var r []int

    // check if passed buffered channel is full
    select {

    // check if you can acquire a slot
    case sem <- struct{}{}:

        // call another goroutine worker over the first half
        go func() {
            l = merge_sort_multi(s[:n], sem)

            // free a slot
            <-sem

            // unlock one semaphore
            wg.Done()
        }()
    default:
        l = msort_sort(s[:n])
        wg.Done()
    }

    // the same over the second half
    select {
        case sem <- struct{}{}:
            go func() {
                r = merge_sort_multi(s[n:], sem)
                <-sem
                wg.Done()
            }()
        default:
            r = msort_sort(s[n:])
            wg.Done()
    }

    // wait for go subroutine
    wg.Wait()

    // return
    return msort_merge(l, r)

}
{{< / highlight >}}

As you can see, in my default select action, I wrote a call to the single-routined version of merge sort. However, there is another interesting tool in the code: it is the ```WaitGroup``` object provided by the sync package. From the official documentations[^wg], a WaitGroup waits for a collection of goroutines to finish. The main goroutine calls ```Add``` to set the number of goroutines to wait for. Then each of the goroutines runs and calls ```Done``` when finished. At the same time, ```Wait``` can be used to block until all goroutines have finished.

#### Python Merge Sort
Ok, at this point, if you arrived here, I will be honest: I'm not a concurrency expert, actually I really hate concurrency, but writing this post and benchmarking GoLang channel learnt me a lot about this theme: the part of reproducing a computation _as much as possible_ logically equivalent in Python was really - I mean, REALLY - difficult.

##### Simple Py Version
{{< highlight python >}}
def msort_sort(array):
    n = len(array)
    if n <= 1:
        return array
    left = array[:n / 2]
    right = array[n / 2:]
    return msort_merge(msort_sort(left), msort_sort(right))

def msort_merge(*args):
    left, right = args[0] if len(args) == 1 else args
    a = []
    while left or right:
        if not left:
            a.append(right.pop())
        elif not right or left[-1] > right[-1]:
            a.append(left.pop())
        else:
            a.append(right.pop())
    a.reverse()
    return a
{{< / highlight >}}

##### Concurrent Py Version
I had to think a lot about a concurrent version: first, I thought to use an array of Threads / Processes (later on this topic) and start / joining them but then... I realized this wouldn't be so much equal to my concurrent GoLang version. First, because the call to more then one thread / process would be done _only once_ over a partition of original data - to be merged in the end, eventually in a _concurrent way_: this is not exactly the behavior of my GoLang version, that call recursively a concurrent routine until the semaphore accept new concurrent routines - and in the end call a single-routined instance of the sorting method. So I thought "I simply can't realize a multi-routined (threads or processes) of my merge sort in Python using a simple _one-shot_ split method, because it is not _computationally_ equivalent". For this reason, the first thing I tried was to replice exactly the same behavior of _Channel_ and _WaitGroup_ using the semaphores primitive in Python - and after some days of work I got it. Let's have a look at the code:

{{< highlight python >}}
def merge_sort_parallel_golike(array, bufferedChannel, results):

    # if array length is 1, is ordered : return
    if len(array) <= 1:
        return array

    # compute length
    n = len(array) / 2

    # append thread for subroutine
    ts = []

    # try to acquire channel
    if bufferedChannel.acquire(blocking=False):

        # if yes, setup call on the first half
        ts.append(Thread(target=merge_sort_parallel_golike, args=(array[:n], bufferedChannel, results,)))

    else:

        # else call directly the merge sort over the first halft
        results.append(msort_sort(array[:n]))

    # the same, in the second half
    if bufferedChannel.acquire(blocking=False):

        ts.append(Thread(target=merge_sort_parallel_golike, args=(array[n:], bufferedChannel, results,)))

    else:

        results.append(msort_sort(array[n:]))

    # start thread
    for t in ts:
        t.start()

    # wait for finish
    for t in ts:
        t.join()

    # append results
    results.append(msort_merge(results.pop(0), results.pop(0)))

    # unlock the semaphore for another threads for next call to merge_sort_parallel_golike
    # try is to prevent arise of exception in the end
    try:
        bufferedChannel.release()
    except:
        pass

if __name__ == "__main__":

    # manager to handle routine response
    manager = Manager()
    responses = manager.list()

    sem = BoundedSemaphore(routinesNumber)
    merge_sort_parallel_golike(a, sem, responses)
    a = responses.pop(0)

{{< / highlight >}}

Ok, let's start from manager. The ```Manager``` object initialized in the main provides a struct to put responses of call - more or less similar to a ```Queue```. The ```BoundedSemaphore``` plays the role of the bounded channel semaphore I talked before. A semaphore is a lock mechanism more advanced that simple lock: it has an internal counter rather than a lock flag, and it only blocks if more than a given number of threads have attempted to hold the semaphore. Depending on how the semaphore is initialized, this allows multiple threads to access the same code section simultaneously: fortunately, you can _try_ to acquire lock and go ahead in execution if you fail - this plays the ```select``` trick I mentioned before used in the GoLang version - by using the ```blocking=False``` parameter (```bufferedChannel.acquire(blocking=False)```). With ```join``` I emulated the behavior of the ```WaitGroup```, because I thought it was the standard way to sync the two threads and wait for their end before proceeding with the final merge step. Any questions?

You are wondering _"How does this perform?!"_ Ok, it SUCKS. I mean: a lot. So I try to search for something more efficient... and I found this - similar to the first solution I thought, but using the Pool object. The hell.

{{< highlight python >}}
def merge_sort_parallel_fastest(array, concurrentRoutine, threaded):

    # create a pool of concurrent threaded or process routine
    if threaded:
        pool = ThreadPool(concurrentRoutine)
    else:
        pool = Pool(concurrentRoutine)

    # size of partitions
    size = int(math.ceil(float(len(array)) / concurrentRoutine))

    # partitioning
    data = [array[i * size:(i + 1) * size] for i in range(concurrentRoutine)]

    # mapping each partition to one worker, using the standard merge sort
    data = pool.map(msort_sort, data)

    # go ahead until the number of partition are reduced to one (workers end respective ordering job)
    while len(data) > 1:

        # extra partition if there's a odd number of worker
        extra = data.pop() if len(data) % 2 == 1 else None

        # prepare couple of ordered partition for merging
        data = [(data[i], data[i + 1]) for i in range(0, len(data), 2)]

        # use the same number of worker to merge partitions
        data = pool.map(msort_merge, data) + ([extra] if extra else [])

    # return result
    return data[0]

{{< / highlight >}}

And this perform better. The question is better using Threads or Processes? Well... look at my comparative graph!

<div class="img_container"><img src="https://i.imgur.com/3Tl0YyN.png" style="width: 100%; marker-top: -10px;"/></div>

Ok, because Python version is not so good, this is a graph with only GoLang series:

<div class="img_container"><img src="https://i.imgur.com/QgZeDgt.png" style="width: 100%; marker-top: -10px;"/></div>

### Conclusion
Python sucks. GoLang rulez. I'm sorry, Python: I loved you. The complete code is available here: [go-py-benchmark](https://madeddu.xyz/posts/go-py-benchmark).

Thank you everybody for reading!

[^talk]: There are plenty of beautiful [slides](https://talks.golang.org/2012/concurrency.slide) of a GoLang talk online!
[^rob]: The lesson of [Rob Pike - Concurrency Is Not Parallelism](https://vimeo.com/49718712).
[^docr]: Directly from the the official [FAQ](https://golang.org/doc/faq) page.
[^docc]: More info [here](https://golang.org/ref/spec#Channel_types).
[^s1]: Source [here](https://medium.com/@_orcaman/when-too-much-concurrency-slows-you-down-golang-9c144ca305a)
[^nbcs]: Have a look [here](https://gobyexample.com/non-blocking-channel-operations)
[^wg]: Here more about [WaitGroup](https://golang.org/pkg/sync/#WaitGroup)







