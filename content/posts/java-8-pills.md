---
layout: post
title: "Java 8 Pills"
date: 2017-12-15
polly: https://madeddu.xyz/mp3/java-8-pills.mp3
categories:
- coding
- java
- theory
tags:
- java
- features
- functional-interface
- lambda-expressions
- method-references
- default-method
---

### Why Java why now
I recently followed a course[^first] on YouTube by Adib Saikali (NewCircle Instructor) about the key features introduced in Java > 8: it's an old post regarding old stuff but...I collected some notes (mainly because I need to write down what I'm listening to to stay focused and learn new concepts) that I decided to share with you. This is to say, this post is for every one that had put aside Java for a while and is now looking for a quick overview of the key aspects - exactly like me some weeks ago - to improve his abilities in coding, taking advantage of the old (n.d.r.) features introduced a few years ago. For thus who missed the footnote before and want to jump directly to the lesson, [here](https://www.youtube.com/watch?v=8pDm_kH4YKY) you can find the main source of the content of the next paragraphs.

### Content
Briefly, the basic steps of the Saikali course are:
- Lambda expressions;
- Functional interfaces;
- Method references;
- Default methods;
- Collections Enhancements;
- Streams;

I think that the arguments are really exposed in the perfect order, despite their relations and dependencies: for this reason, I will maintain this order and collect some snippets + doubts + questions arised during the course, except for the first two arguments.

#### Lambda expression
First of all, a ```lambda``` is not a method (or function) but an ```expression```: this is to say, the lambda notation is a way to say to the compiler "take this code and create a method for me". The concept of _lambda_ is different from its _implementation_: the lambda expressions exist in many different languages and are implemented in many different ways. However, lambdas have the following characteristics:
- a lambda expression _define_ an anonymous functions,
- can be assigned to variables,
- can be passed to functions,
- or returned from functions;

These features and _how the compiler process the expressions_ make lambda expressions excellent to:
- form the basis of the functional programming paradigm,
- make parallel programming easier,
- write more compact code,
- create richer data structure collections,
- and develop cleaner APIs;

The lambda expression is a _concept_, that has its own implementation flavor in different languages. But how lambdas are implemented in Java? Look at the code below:

{{< highlight java >}}
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);

integers.forEach(
    x -> System.out.println(x)
);

integers.forEach(x -> {
    int y = x + 10;
    System.out.println(y);
});
{{< / highlight >}}

```forEach``` is a method that accepts a _function_ as its input and calls the function for each value in the list. ```x -> System.out.println(x)``` is a lambda expression that defines an _anonymous function_ with one parameter named ```x``` of type ```Integer```. In the second lambda expression are defined multiple lines: moreover, you can create local variable inside the body of the lambda. You can specify (or not) the type for ```x``` parameter, because the Java compiler can do type inference. How does it work from a compiler point of view?

##### Lambda Lifecycle
More or less, the compiler _transform_ the lambda expression in a _static function_ and then call the generated function. The function should be static method, with a class wrapping around, and so on...it doesn't matter for now. Instead, we are interest in the _type_ of the lambda expressions: remember, they could be assigned to vars, passed to functions, or returned. This implies that lambda has a type. Initially Java engineers thought about a specific new _function_ type, but in the end they didn't create it. In the next paragraphs I try to cover the question "what is the type of a lambda function?" To do that, we need first to talk about _functional interfaces_.

#### Functional interface
A first simple __incomplete__ definition for the functional interface is the following:

    A functional interface is an interface with only one method.

I know what you are thinking: what kind of _feature_ is that?! Of course, you can create how many interfaces you want with one method: rather this kind of interface is the most common and used in Java. For instance, the ```Runnable```, ```Comparable``` and ```Callable``` interface are functional interfaces. They are so popular in lot of libraries, such as in [Spring](https://spring.io)'s libraries: they contains ```TransactionCallback```, ```RowMapper```, ```StatementCallback``` and others. Then, they officially decided to call this kind of interface - _functional interface_ - to formalize them inside the language. They also introduced a new optional annotation ```@FunctionalInterface``` to make the compiler able to produce an error if more than one method is added to a functional interface.

To conclude, each interface with one method is considered a functional interface by Java 8, even if it was compiled with a Java 1.0 compiler: the new lambda expressions will work with old libraries, without need to recompile. Ok wait a moment: what do lambda expressions have to do with functional interfaces?

{{< highlight java >}}
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);

Consumer<Integer> consumer = x -> System.out.print(x);

integers.forEach(consumer);
{{< / highlight >}}

In the example above I _declared_ the _type_ of the lambda expression ```x -> System.out.print(x);``` as ```Consumer<Integer>```: for how Java works, this is a ```Class``` or an ```Interface``` because they are the unique two entities that define a type in Java grammar. In fact, ```Consumer<Integer>``` is a functional interface from the package ```java.util.function package```. Why Consumer<Integer>? The answer is simple, because ```Consumer<T>``` define the type of a function that takes an argument of type T and returns void - as ```System.out.print(x)``` call used in the one-line body of the lambda expression assigned to the variable ```consumer```.

#### Back to lambda expressions
    The type of the lambda expression is the same as the type of the functional interface that the lambda expression is assigned to.

At this point you could think about lambda expressions as anonymous inner class with one method: they are __not__ (wait for it...).

#### Variable capture
Look at the code below:

{{< highlight java >}}
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);

int localVar = 10;

integers.forEach(
    x -> System.out.println(x + localVar)
);
{{< / highlight >}}

A lambda expression can interact with variables defined outside the body of the lambda: using variables outside is called _variable capture_. But... the value of the variable declared outside __must__ be ```final```. What does it mean? If you copy the code above it will work: why? Because the compiler will handle the lack of the statement final, and consider ```localVar``` as final (and also static). The signature generated from the example will be something like this:

{{< highlight java >}}
public static void generatedName(Integer x, final int var) {
    System.out.println(x + var);
}
{{< / highlight >}}

#### Private field
Look at the code below:

{{< highlight java >}}
integers.forEach(
    x -> {
        System.out.println(x + this.var);
        if (this == main) {
            System.out.println("Compiler has passed this of enclosing method as the first parameter of method generated from the lambda expression");
        }
    }
);

integers.forEach(
    new Consumer<Integer>() {
        private int state = 10;

        @Override
        public void accept(Integer x) {
            int y = this.state + Main.this.var + x;
            System.out.println("Anonymous class that implements Consumer Interface and accept overriding: " + y);
        }
    }
);
{{< / highlight >}}
The two example show a difference: in the first there is no private field. This is because there is no ability for you to define private fields inside a lambda expression: they are just the body of the signature defined in a functional interface the lambda expression will be assigned to. Remember also that a lambda expression has always a type: this because it can only be assigned to a variable, or passed as a parameter or returned by a method and, in each of these cases, you need a type. What about the ```this``` statement? Try the code to see what happens (in any it is explained in the next paragraph).

#### Lambda (_functional typed_) vs Anonymous Inner Class

Let summarize the main differences between a lambda expression and an anonymous inner class:
- inner classes can have state in the form of class level instance variables lambdas can not,
- inner classes can have multiple methods, while lambdas only have a single method body,
- ```this``` points to the object instance for an anonymous Inner class BUT points to the enclosing object for a lambda;

This is why lambda expressions are basically __different__ from anonymous inner class.

#### Available functional interfaces
Is it really necessary to create a functional interface for each lambda expression that we want to use in our code? The answer is no, because ```java.util.function.*``` package contains 43 (wow!) commonly used functional interfaces:

- ```Consumer<T>``` define a function that takes an argument of type T and returns void;
- ```Supplier<T>``` define a function that takes no argument and returns a result of Type T;
- ```Predicate<T>``` define a function that takes an argument of Type T and returns a boolean;
- ```Function<T, R>``` define a function that takes an argument of Type T and returns a result of type R;
- etc...

###### Break point
Until now we saw _how_ the functional interface provide a way to expose the lambda expression to outside world and use them in our code. Let's move one step forward, introducing method references.

### Method references
Basically a lambda is a way to define an anonymous function. The question now is: what about functions I already written? Do I have to rewrite them? Look at the following code:

{{< highlight java >}}
private static void myAlreadyWrittenFunction(Integer i) {
    System.out.println(i);
}

Consumer<Integer> consumerLambda = x -> System.out.println(x);
consumerLambda.accept(1);

Consumer<Integer> consumerOldFunction = Main::myAlreadyWrittenFunction;
consumerOldFunction.accept(1);
{{< / highlight >}}

There's a new notation with double colons that is the syntax used to reference a method. So, the short answer to the previous question is no, because method references can be used to pass an existing function in a place where a lambda is expected.

#### Reference a static method
In the ```consumerOldFunction``` we are saying to the compiler "use the ```myAlreadyWrittenFunction``` as the implementation of the ```accept``` method (signature) defined in the functional interface ```Consumer<Integer>```. Obviously, the signature of the referenced method __needs__ to match the signature of functional interface method (and yes, it will handle also overloading and generics). In this first scenario we referenced a static method, but there are four different types of method references:
- static method reference,
- constructor reference,
- specific object instance reference,
- and specific arbitrary object of a particular type reference;

#### Reference a constructor
Look at the following code:

{{< highlight java >}}
Function<String, Integer> firstMapper = x -> new Integer(x);
System.out.println(firstMapper.apply("11"));

Function<String, Integer> secondMapper = Integer::new;
System.out.println(secondMapper.apply("12"));
{{< / highlight >}}

You can use method reference to reference a constructor: this is handy when working with _streams_ (the last topics of this article). With the notation ```Function<String, Integer> secondMapper = Integer::new;``` we are asking the compiler to create for us a function that takes as angurment a string, returns an integer and in the body of that method invoque the new constructor of the Integer, passing the string parameter provided to that method. Cool right?!

__NOTE__: we use the ```Function<T, R>``` functional interface provided by the ```java.util.function.*```: it defines a function that takes an argument of Type T and returns a result of type R, as our Integer constructor;

#### Reference to a specific object instance method
Look at the following code:

{{< highlight java >}}
Consumer<Integer> firstConsumer = x -> System.out.println(x);
firstConsumer.accept(13);

Consumer<Integer> secondConsumer = System.out::println;
secondConsumer.accept(14);
{{< / highlight >}}

With the notation ```Consumer<Integer> secondConsumer = System.out::println;``` we tell the compiler that the lambda body signature should match the method println and that the lambda expression should result in a call to ```System.out.println(x)```.

#### Reference to a specific arbitrary object of a particular type
Finally, look at the following code:

{{< highlight java >}}
Function<String, String> firstMapper = x -> x.toUpperCase();
System.out.println(firstMapper.apply("abc"));

Function<String, String> secondMapper = String::toUpperCase;
System.out.println(secondMapper.apply("def"));
{{< / highlight >}}

With the notation ```Function<String, String> secondMapper = String::toUpperCase;``` we tell the compiler to invoke the ```toUpperCase``` method on the parameter that is passed to the lambda expression. So invoking ```secondMapper.apply("def")```
will call the lambda expression derived method and inside that will call the ```toUpperCase``` method on the parameter "def"

The all idea of the method reference feature is that if you already have a method ready, you can create a lambda using method reference, and then use lambda wherever you want in your code. At this point, we are ready to talk about _default methods_ and, finally, __provide a complete definition of the functional interface__.

### Default methods
Until now I wrote a little bit about lambda expressions and arrow notation -> and method references. A natural place to use lambda expressions is with the Java collections framework. The collection framework is defined with interfaces such as ```Iterable```, ```Collection```, ```Map```, ```List```, ```Set```, etc. Think about that: adding a new method such ```forEach``` we used in the previous examples to, let me say, the ```Iterable``` interface, will mean that __all__ existing implementations of ```Iterable``` will break. Why? Because they __don't__ implement the ```forEach``` method introduced in the interface: so what? All codes already compiled will not work with the new version of Java. Unacceptable. This problem is known as the __the interface evolution problem__: how can published interfaces be evolved without breaking existing implementations? Default methods provide a solution to this __big__ problem:

    A default method defined in a Java Interface<T> has an implementation (provided in the interface) and is inherited by all classes that implement the Interface<T>.

Look at the example:
{{< highlight java >}}
public class Main implements Matteo {
    public static void main(String[] args) {
        new Main().printMatteo();
    }
}

interface Matteo {
    default void printMatteo() {
        System.out.print("Matteo\n");
    }
}
{{< / highlight >}}

Question: can you override a default method? Yes, of course, simply overriding the method. Furthermore, if you a default method ```myMethod()``` defined in an interface ```A```, an interface ```B``` that extend ```A``` and override the implementation of ```myMethod()``` provided by ```A```, if you have a class ```MyClass``` that implement interface ```B```, without any implementation of ```myMethod()```, the implementation provided in interface ```B``` will be used, or

    The closest implementation in hierarchy will be used.

#### Creating a conflict
Let be ```A``` and ```B``` two different interfaces that provide two default methods with the same signature. Let be ```MyClass``` a class that implements both ```A``` and ```B```: if you create this situation the compiler arises an error that sounds like _duplicate default methods implementations_. How to solve this conflict? Of course, with overriding. Further, you can call the implementation in the interface, with notation ```A.super.myMethod()```. Instead, same level implementation implies a compiler error, and the override is mandatory.
__NOTE__: a default method should not be final...because it doesn't make so much sense.

What about our functional interface?! Do you remember the old __incomplete__ definition?

    A functional interface is an interface with only one method.

The complete definition is

    A functional interface is an interface with only one __non-default__ method.

Let's move one step forward, introducing collections enhancements.

### Collections Enhancements
Java 8 uses lambda expressions and default methods to improve the exising Java collections frameworks and add a lot of functions to them. We already seen example of these methods, such as the ```forEach``` - an example of internal iteration: it delegates the looping to a library function (such as forEach, as we said), and the loop body processing to a lambda expression. Further, this method is really important because it allows the library function we are delegating to implement the logic needed to execute the iteration on muliple cores, if desired. Some of the new methods provided by the standard library are:

- New java.lang.Iterable methods:
  - default void forEach(Consumer<? super T> action);
  - default Spliterator<T> spliterator();
- New java.util.Collection methods:
  - default boolean removeIf(Predicate<? super E> filter);
  - default Spliterator<E> spliterator();
  - default Stream<E> stream();
  - default Stream<E> parallelStream();
- New java.util.Map methods:
  - default V getOrDefault(Object key, V defaultValue);
  - putIfAbsent(K key, V value);
  - etc;

The streams related methods mentioned above are:
- default Spliterator<T> spliterator();
- default Stream<E> stream();
- default Stream<E> parallelStream();

But...what is a _stream_ exactly?

### Stream

Streams are a functional programming desing pattern for processing sequences of elements - sequentially or in parallel. When examining java programs we always run into code along the following lines:
- run a database query to get a list of objects,
- iterate over the list to compute a single result,
- iterate over the list to generate a new data structure such as another list, map, set, etc,
- or iterate over the list and ...;

Boring. Well, streams are the implementation of a concept (a design pattern) in the same way lambdas are: they can be implemented in many programming languages. In the next paragraphs I talk about - more or less - about _how_ they work in Java (without talking in details about how they are _implemented_).

{{< highlight java >}}
int sumOdd = integers.stream()
                     .filter(o -> o % 2 == 1)
                     .mapToInt(o -> o)
                     .sum();
System.out.print(sumOdd+"\n");
{{< / highlight >}}

In the example the program creates a stream instance from a __source__ (a Java collection), add a filter operation to the stream intermediate operations pipeline, add a map operation to the stream intermediate operations pipeline and add a terminal operation (sum) that kicks off the stream processing: what the hell is going on?! Let's first talk about a stream composition.

#### The stream composition
A __stream__ has three elements:
- a __source__ that the stream can pull objects from,
- a __pipeline__ of operations that will execute on the elements of the stream,
- a __terminal__ operation that pulls values down the stream;

#### The stream lifecycle
A __stream lifecycle__ has the following steps:
- a __creation__ step, ih which a stream get created from a source object such as a collection, file, or generator,
- a __configuration__ step, in which a stream get configured with a collection of pipeline operations,
- a __execution__ step, done when stream terminal operation is invoked which starts pulling objects trough the operations pipeline of the stream,
- and a __cleanup__ step, in which stream can only be used once;

It is important to remember that stream execution is __lazy__, that means that until you call a terminal operation the stream doesn't do anythings. It doesn't loop.

#### Stream Sources
There are different type of streams: in the next paragraphs I show same simple examples of different streams.

##### Number Stream Source
Look at the code:

{{< highlight java >}}
System.out.print("Long Stream Source\n");

LongStream.range(0, 5).forEach(System.out::println);
{{< / highlight >}}

The example shows a range stream from 0 to 5, starting from a ```LongSource``` from ```java.util.stream.*``` package in standard library. Remember that a stream can be anything: a stream is a design pattern. What it does? It splits out elements from a collection.

##### String Stream Source
Look at the code:

{{< highlight java >}}
List<String> cities = Arrays.asList("toronto", "ottawa", "montreal", "vancouver");

cities.stream().forEach(System.out::println);
{{< / highlight >}}

The example shows a stream from a list of String.

##### Collection Stream Source
Look at the code:

{{< highlight java >}}
long length = "ABC".chars().count();

Consumer<Long> printer = System.out::println;

printer.accept(length);
{{< / highlight >}}

A character stream source is called using chars() and then count().

##### File System Streams
Look at the code:

{{< highlight java >}}
String workDir = System.getProperty("user.dir");
Path workDirPath = FileSystems.getDefault().getPath(workDir);

System.out.println("Directory Listing Stream");
Files.list(workDirPath).forEach(System.out::println);

System.out.println("Depth First Directory Walking Stream");
Files.walk(workDirPath).forEach(System.out::println);
{{< / highlight >}}

The line ```Files.list(Path object from Java 7)``` gives a stream to work directly on list. The line ```Files.walk(Path object from Java 7)``` gives a stream to work directly on list with depth first search.

#### A first summary example
Before going on, try to understand what the code below does:

{{< highlight java >}}
String workDir = System.getProperty("user.dir");
Path workDirPath = FileSystems.getDefault().getPath(workDir);

String className = Main.class.getName().replace(".", "/") + ".java";

Files.find(workDirPath, 10,
    (fileName, attributes) -> fileName.endsWith(className)).forEach(path -> {
      try {
          Files.lines(path).forEach(System.out::println);
      } catch (Exception e) {}
    }
);
{{< / highlight >}}

Did you understand what the code does? The example simply print out it self. Briefly, the call to ```find``` method returns a stream, so it is possible to call ```forEach``` method on it: we know that the ```forEach``` terminal does something to each things that it founds. The things it founds are pointed by ```path``` variable, passed as parameter to the lambda expression in the brackets. Inside this, we use another stream of Files called ```lines``` that out each lines of the file as a stream: once again, the call to ```forEach``` print out each of the elements splitted out by the lines stream.

#### How about stream terminals?
We said that until the terminal operation is invoked, the stream doesn't do anything. You might be thinking "what kind of terminal could I use to _execute_ my stream pipeline?". There are many types of terminals for stream:

- __reduction__ terminal operations that return a single result (such as sum()),
- __mutable reduction__ terminal operations that return multiple results in a container data structure,
- __search__ terminal operations that return a result as soon as a match is found,
- __generic__ terminal operations that do any kind of processing you want on each stream element;

But __remember__: nothing happens until the terminal operation is invoked (think about updating, and so on).

##### Reduction terminal operations
Look at the following:

{{< highlight java >}}
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
System.out.println(integers.stream().count());

Optional<Integer> result;
result = integers.stream().min((x, y) -> x - y);

System.out.println(result.get());

result = integers.stream().max(Comparator.comparingInt(x -> x));
System.out.println(result.get());

Integer res = integers.stream().reduce(0, (x, y) -> x + y);
System.out.println(res);
{{< / highlight >}}

Can you explain the min? Actually, I didn't understand the ```min```: you can write a comment below if you want XD.

##### Mutable reduction operations
Look at the following:

{{< highlight java >}}
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);

Set<Integer> result = integers.stream().collect(Collectors.toSet());
System.out.println(result);

Integer[] a = integers.stream().toArray(Integer[]::new);
Arrays.stream(a).forEach(System.out::println);

Integer res = integers.stream().reduce(0, (x, y) -> x + y);
System.out.println(res);
{{< / highlight >}}

```Collectors``` are used to get all the things that came out of a stream and collect them into a structure: the ```Collectors``` class defines many useful collectors such as ```List```, ```Set```, ```Map```, groupingBy, partitioningBy, etc.

##### Search terminal operation
Look at the following:

{{< highlight java >}}
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);

Optional<Integer> result = integers.stream().findFirst();
System.out.println(result);

Boolean res = integers.stream().anyMatch(x -> x == 5);
System.out.println(res);

res = integers.stream().anyMatch(x -> x > 3);
System.out.println(res);

Optional<Integer> anyres = integers.stream().findAny();
System.out.println(anyres.get());
{{< / highlight >}}

Note the ```findAny```: result of findAny call is unpredictable. In a sense, if stream is executed, for instance, parallel, then the first call that ends will return the result so you don't know exactly a priori which will be the returned value. It just means "find any".

##### Generic terminal operation
```forEach``` is an example of generic terminal operation.

#### Streams pipeline rules
What about streams pipeline call?
- streams can process elements sequentially;
- streams can process elements in parallel;

Thus means that streams operations are not allowed to modify the stream source otherwise bad things happens XD. It is simple: a stream operate over a source. Don't modify the source! sounds like a best practise.

#### Intermediate stream operations
There are two classes of intermediate stream operations:

- __Stateless intermediate operations__: they do not need to know the history of results from the previous steps in the pipeline or keep track of how many results it have produced or seen. Example of stateless intermediate operations are ```filter```, ```map```, ```flatMap```, ```peek```.

- __Stateful intermediate operations__: they need to know the history of results from the previous steps in the pipeline or need to keep track of how many results it have produced or seen. Example of stateless intermediate operations are ```distinct```, ```limit```, ```skip```, ```sorted```.

For instance, if you want a stream to sort elements you obviously needs to know the order of sorting. Instead, if you have a parallel operations with threads that compete with each others, you are not interested in state of each elements. Let's have a look at sample example:

{{< highlight java >}}
integers.stream().filter(x -> x < 4).forEach(System.out::println);

integers = integers.stream().map(x -> x + x).collect(Collectors.toList());
integers.forEach(System.out::println);

IntSummaryStatistics intSummaryStatistics = integers.stream().mapToInt(x -> x).summaryStatistics();
System.out.println(intSummaryStatistics);
{{< / highlight >}}

A repo with all the examples in the article is [here](https://github.com/made2591/java-8-steps) and the main source is [here](https://www.youtube.com/watch?v=8pDm_kH4YKY).

Thank you everybody for reading!

[^first]: The course is entitled _Java 8 Lambda Expressions & Streams_ and is available for free [here](https://www.youtube.com/watch?v=8pDm_kH4YKY). Thank you again Adib Saikali for your lesson!