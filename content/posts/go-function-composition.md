---
layout: post
title: "A monadic reasoning around function composition in Golang"
date: 2020-01-16
polly: https://madeddu.xyz/mp3/go-function-composition.mp3
categories:
- coding
- golang
tags:
- coding
- golang
markup: "mmark"
---

### Introduction

Function composition is something we as developers do every day, more or less. This concept come from Mathematics: if you search on Wikipedia, you find out that *function composition is an operation that takes two functions $$f$$ and $$g$$ and produces a function $$h$$ such that* $$h(x) = g(f(x))$$. In this operation, the function $$g$$ is applied to the result of applying the function $$f$$ to the input $$x$$. That is, the functions $$f: X \rightarrow Y$$ and $$g: Y \rightarrow Z$$ are composed to yield a function that maps $$x$$ in $$X$$ to $$g(f(x))$$ in $$Z$$.

There's an entire way to develop software using this concept of composition: in this post, I will discuss how we can leverage this concept in Golang, trying to provide some pros and cons as well about this approach.

<div class="img_container"><img src="https://i.imgur.com/1Ccnab7.jpg" style="width: 100%; marker-top: -10px;"/></div>

#### Monads

Let's start by saying that function composition patterns exist for a while: an ex-colleague of mine (thank you *martiano*) recently bet me on implementing the Monad in Golang, and that's the reason I recently worked on this. So, before going into details about function composition and how this is used by, let's start by giving a definition of Monad.

> A monad is just a monoid in the category of endofunctors

Ok...let's introduce some concept required to understand this sentence.

##### Monoids

In abstract algebra, a `monoid` is an algebraic structure with a single associative binary operation and an identity element. Suppose that $$S$$ is a set and $$\cdot$$ is some binary operation $$S \times S \rightarrow S$$, then $$S$$ with $$\cdot$$ is a monoid if it satisfies the following two axioms:

- **Associativity**: for all $$a, b$$ and $$c$$ in $$S$$, the equation $$(a \cdot b) \cdot c = a \cdot (b \cdot c)$$ holds.
- **Identity element**: there exists an element $$e$$ in $$S$$ such that for every element $$a$$ in $$S$$, the equations $$e \cdot a = a \cdot e = a$$ hold.

You should have already realized that many of the things you've seen in your past respect these two axioms. Keep this concept in mind for a while. Let's talk go ahead with the concept of endofunctors, that means...let's talk about functors first.

##### Functors

A `functor` is a very simple but powerful idea coming from category theory. Let's give a simple definition:

> A functor is a mapping between categories

Thus, given two categories, $$C$$ and $$D$$, a functor $$F$$ maps objects in $$C$$ to objects in $$D$$ — it's a function on objects. If $$a$$ is an object in $$C$$, we'll write its image in $$D$$ as $$F\;a$$ (no parentheses). 

But a category is not just objects — it's objects and morphisms[^morphism] that connect them. A functor also maps morphisms — it's a function of morphisms. But it doesn't map morphisms willy-nilly — it preserves connections. So if a morphism $$f$$ in $$C$$ connects object $$a$$ to object $$b$$, like this

$$
f \colon a \to b
$$

the image of $$f$$ in $$D$$, $$F f$$, will connect the image of $$a$$ to the image of $$b$$:

$$
F\;f \colon F\;a \to F\;b
$$

<div class="img_container"><img src="https://i.imgur.com/drM4RW8.png" style="width: 50%; marker-top: -10px;"/></div>

So a functor preserves the structure of a category: **what's connected in one category will be connected in the other category**. At this point, let's just define an `endofunctor`:

> An endofunctor is a functor that maps a category to that same category

Let's make no other assumptions on endofunctors: just keep in mind they are functor acting on a single category.

##### Once more thing

There's something more to the say. The first is about the `composition of morphisms`. If $$h$$ is a composition of $$f$$ and $$g$$:

$$
h \colon g \cdot f
$$

we want its image under $$F$$ to be a composition of the images of $$f$$ and $$g$$:

$$
F\;h \colon F\;g \to F\;f
$$

<div class="img_container"><img src="https://i.imgur.com/b15jh0w.png" style="width: 30%; marker-top: -10px;"/></div>

Second, we want all identity morphisms in $$C$$ to be mapped to identity morphisms in $$D$$:

$$
F id_a = id_{Fa}
$$

Here, $$id_a$$ is the identity at the object $$a$$, and $$id_{F\;a}$$ a the identity at $$F\;a$$.

<div class="img_container"><img src="https://i.imgur.com/0IrUv5j.png" style="width: 30%; marker-top: -10px;"/></div>

That is, functors must preserve **composition of morphisms** and **identity morphisms**. And now we have all the instrument to better understand the real definition of a Monad.

### Back to monad

I actually gave the shorter version of the definition of a Monad above, to keep things simple as much as possible. We said that *a monad is a monoid in the category of endofunctors*. Since we introduced some more concepts, all told:

> A monad in $$X$$ is a monoid in the category of endofunctors of $$X$$, with product $$\times$$ replaced by a composition of endofunctors and unit set by the identity endofunctor

Ok, at this point in time you should start feeling a bit more confident about the terms and the idea behind monad than before. While hoping you're feeling this, I just wanna present the problem from a programming perspective since, in the end, we are replacing the same concepts of identity and composition, with two standard operations and nothing more.

> A monad is composed by a type constructor ($$M$$) and two operations, `unit` and `bind`

The `unit` operation (sometimes also called `return`) receives a value of type $$a$$ and wraps it into a monadic value of type $$m\;a$$, using the type constructor. The `bind` operation receives a function $$f$$ over type $$a$$ and can transform monadic values $$m\;a$$ applying $$f$$ to the unwrapped value $$a$$.

We can also say the Monad pattern is a design pattern for types, and a monad is a type that uses that pattern. With these elements, we can compose a sequence of function calls (a "pipeline") with several bind operators chained together in an expression. Each function call transforms its input plain type value, and the bind operator handles the returned monadic value, which is fed into the next step in the sequence.

### Coding a Monad

Let's assume we have three functions $$f1$$, $$f2$$ and $$f3$$, each of which returns an increment of its integer parameter. Additionally, each of them generates a readable log message, representing the respective arithmetic operation. Like the ones shown below.

{{< highlight go >}}
...

func f1(x int) (int, string) {

    return x + 1, fmt.Sprintf("%d+1", x)

}

func f2(x int) (int, string) {

    return x + 2, fmt.Sprintf("%d+2", x)

}

func f3(x int) (int, string) {

    return x + 3, fmt.Sprintf("%d+3", x)

}

...
{{< / highlight >}}

Imagine now that we would like to chain the three functions $$f1$$, $$f2$$ and $$f3$$ given a parameter $$x$$ - i.e. we want to compute $$x+1+2+3$$. Additionally, we want to have a readable description of all applied functions. We would do something like this.

{{< highlight go >}}
...

x := 0
log := "Ops: "

res, log1 := f1(x)
log += log1 + "; "

res, log2 := f2(res)
log += log2 + "; "

res, log3 := f3(res)
log += log3 + "; "

fmt.Printf("Res: %d, %s\n", res, log)

...
{{< / highlight >}}

Nice. Buuuut... this solution sounds a lot like 80s code, isn't it? 

### What's the problem?

The problem is that we have repeated the so-called *glue code*, which accumulates the overall result and prepares the input of the functions. If we add a new function $$f4$$ to the sequence, we have to repeat this *glue code* again. Moreover, manipulating the state of the variables `res` and `log` makes the code less readable, and is not essential to the program logic. 

Ideally, we would like to have something as simple as the chain invocation $$f3(f2(f1(x)))$$. Unfortunately, the return types of $$f1$$ and $$f2$$ are incompatible with the input parameter types of $$f2$$ and $$f3$$. 

### A Monad approach

To solve the problem we introduce two new functions:

{{< highlight go >}}
...

func unit(x int) *Log {

    return &Log{Val: x, Op: ""}

}

func bind(x *Log, f Increment) *Log {

    val, op := f(x.Val)
    return &Log{Val: val, Op: fmt.Sprintf("%s%s; ", x.Op, op)}

}

...
{{< / highlight >}}

This is kind of similar to the functions we described before - yes... you might think "the names, maybe". Actually, not only: lets first introduce the other new elements required to let these two functions compile correctly.

{{< highlight go >}}
...

type Increment func(y int) (int, string)

type Log struct {
    Val int
    Op string
}

...
{{< / highlight >}}

We first introduced a new function type. The syntax is pretty simple:

`type MyFunctionTypeName func(<params>) <returns>`

This is a useful and common way to define a function as a `type` and identify a pattern without rewriting every time the entire signature defining the function inputs and parameters. We also introduced a new structure, the `Log` that actually contains nothing more than two fields, `Val` and `Op`: this is a wrap around the results returned by our initial functions, and this is not a coincidence. Let's focus on the two new functions, `unit` and `bind`.  

#### Unit

{{< highlight go >}}
...

func unit(x int) *Log {

    return &Log{Val: x, Op: ""}

}

...
{{< / highlight >}}

The `unit` function returns a new `*Log` (pointer of a `Log` instance) for a given `x` value. This is the step zero needed in the composition step we are gonna implement soon, thanks to the `bind` function. 

#### Bind

{{< highlight go >}}
...

func bind(x *Log, f Increment) *Log {

    val, op := f(x.Val)
    return &Log{Val: val, Op: fmt.Sprintf("%s%s; ", x.Op, op)}

}

...
{{< / highlight >}}

The `bind` function returns a `*Log` (pointer of a `Log` instance) for a given `x` value and function $$f$$ of type `Increment`. That means, **a function that has the Increment type signature**. The `bind` operation applies the provided function to the given input, of type `*Log`, and return a `*Log` instance with the computed result, and... see? What we called *glue-code* before, i.e. the operations of logs concat

{{< highlight go >}}
...

// no more required and not repeated
log += log2 + "; "
log += log3 + "; "

...
{{< / highlight >}}

is no more required and not anymore repeated. Hence, we can solve the problem with a single chained function invocation:

{{< highlight go >}}
...

x := 0
fmt.Printf("%s\n", bind(bind(bind(unit(x), f1), f2), f3).ToString())

...
{{< / highlight >}}

where `ToString()` is just a util method to pretty print the `*Log` object instance:

{{< highlight go >}}
...

func (l *Log) ToString() string {
	
    return fmt.Sprintf("Res: %d, Ops: %s", l.Val, l.Op)

}

...
{{< / highlight >}}

Let's summarize the `bind` chain: we actually created a common value (the first `*Log` instance created with the `unit(x)` call) starting from our primitive value `x`. Then, we started to delegate the each `bind` to use the result of the internal `bind` call as input (back from `unit` and up to the most external bind), literally *chaining the three functions* $$f1$$, $$f2$$ and $$f3$$.

Doing like this avoids the shortcomings of our previous approach because the bind function implements all the *glue-code* and we don't have to repeat it. We can add a new function $$f4$$ by just including it in the sequence as `bind(f4, bind(f3, ... ))` and we won't have to do other changes. 

### Generalize the idea behind

Let's assume we want to generically compose the functions $$f1$$, $$f2$$, ... $$fn$$. If all input parameters match all return types, we can simply use $$fn(... f2(f1(x)) ...)$$. However, often this approach is not applicable. For instance, in the `*Log` example, the types of the input parameters and the returned values are different. In the example we wanted to "inject" additional logic between the function invocations and wanted to aggregate the interim values.

Before calling $$f1$$, we execute some initialization code. We initialized variables to store the aggregated log and interim values. After that, we call the functions $$f1$$, $$f2$$, ... $$fn$$ and between the invocations we put some glue code - in the example we aggregate the log and the interim values.

#### *Log as MonadicType

In order to compose the bind and unit functions, the return types of unit and bind, and the type of bind's first parameter must be compatible. This is called `a Monadic Type`. 

Last but not least, since repeating the calls to `bind` again and again can be tedious, we can even define a `pipeline` support method like the one below:

{{< highlight go >}}
...

func pipeline(x *Log, fs ...Increment) *Log {

    for _, f := range fs {
        x = bind(x, f)
    }
    return x

}

...
{{< / highlight >}}

and finally, compose function with a really *functional style* like this:

{{< highlight go >}}
...

x := 0
res := pipeline(unit(x), f1, f2, f3)
fmt.Printf("%s\n", res.ToString())

...
{{< / highlight >}}

### Conclusion 

I truly hope this opened a bit your mind around how you can leverage function passed as parameters in Golang, and use this approach to implementation function composition as well your code! 

If you like this post, please upvote it on HackerNews [here](https://news.ycombinator.com/submitted?id=made2591).

You can find the entire gist [here](https://gist.github.com/made2591/bdcab4cc710789b546d500480ac7dd68).

[^morphism]: A category $$C$$ consists of two classes, one of objects and the other of morphisms. There are two objects that are associated to every morphism, the source and the target. A morphism $$f$$ with source $$X$$ and target $$Y$$ is written $$f\colon X \to Y$$, and is represented by an arrow from $$X$$ to $$Y$$.