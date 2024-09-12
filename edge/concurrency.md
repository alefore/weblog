# Edge: Lessons: Concurrency

TL;DR:
My experience with Edge is that threads-based concurrency,
when used carefully,
is worth the additional complexity it unavoidably brings.
Shifting work off to separate threads
that communicate results back to the main thread
helps make the editor significantly more responsive.

The following techniques helped:

* Making objects immutable (or, at least, immutable-assignable)
* Using types that explicitly ensure correct access to shared data
* Using a futures API for async work.

## Edge: Lessons: Preamble

This document is part of
[a series of articles](https://github.com/alefore/weblog/blob/master/edge-lessons.md)
articulating lessons I've learned
during the 10 years I've been developing my own text editor.

## Necessary

Why is concurrency necessary?
I expect this will be obvious to most developers;
I don't think I have much new to say here.

You can only get so far with a single thread/process.
Parallelism lets you offload work from the main thread;
this allows you to do significantly more expensive operations
than when you're constrained to the amount of time between refreshes
(*i.e.,* before the user perceives lag).

I think I first added a background thread in
[91c1cfe](https://github.com/alefore/edge/commit/91c1cfe99ee0e67cd875806abdd0cf1314b6a9fd), in 2017-04-15,
relatively early in Edge's history.
I avoided this type of concurrency
for what felt (but, looking at the data, wasn't!) a long time.
I feared the unavoidable complexity it would bring.

### Limits of non-parallel concurrency

Depending on your requirements, you may be able to stay single-threaded
and just make all your operations interruptible and resumable.
I still do that in some parts of Edge, for example:

* The VM uses a `Trampoline` class when evaluating expressions
  ([header](https://github.com/alefore/edge/blob/master/src/vm/expression.h)).
  After a certain amount of "evaluation steps"
  (deliberately defined somewhat vaguely),
  it yields back to the caller.

* The GC implementation is both highly concurrent
  (both parallelizing various operations
  and deferring work to background threads)
  and interruptible:
  if a call to `Pool::Collect` outlasts a timeout, the operation is aborted
  (but a subsequent call will resume where we had left off).

This lets you achieve concurrency without parallelism.
But, at some point, as your logic grows,
the complexity of this approach
–explicitly maintaining all the state machinery
required to be able to stop and resume at arbitrary points–
becomes worse than just carefully allowing concurrency in.
You are also limited to a single processor.

### Concurrency: Threads vs other approaches

Of course, you can use processes (*e.g.,* `fork`) rather than threads.
And, of course, you can also use non-parallel concurrency.
When it suffices, you really should.

My experience with Edge is that threads,
when used carefully,
go further (though they can also bring more headaches).

While some functions
(such as file-related and many other system calls)
offer async versions,
they tend to be cumbersome and brittle
–*e.g.*, you can't use types
to ensure that you don't accidentally call a blocking version.
Using the regular sync versions in separate threads tends to be easier.

As an example,
I do this in my file-system wrapper
([header](https://github.com/alefore/edge/blob/master/src/infrastructure/file_system_driver.h)),
which receives a work queue.
The work queue helps it ensure
that the*results* of all async calls
are always received and processed
by the original thread
(which explicitly handles the work queue).

## Constness

The **benefits of making types immutable
often outweigh the costs**.

Every type should define `const` semantics
that make it thread-compatible.
When you define a type
you're actually defining two:
the mutable `Type`
and the immutable and thread-compatible `const Type`.

Unless a class is explicitly marked as thread-safe,
before exposing an instance to multiple threads,
you should drop all non-`const` references.
This is often (depending on the class) enough to avoid data races,
reducing the challenge of writing concurrent code to simply
managing object lifetimes.

This section talks about how Edge implements `const`ness.
The motivation goes significantly beyond just managing concurrency,
but that's a very important part.

TODO: Move the following paragraph:

A pattern that deserves mention is balanced trees of immutable objects.
This is very specific compared to the previous patterns,
but this structure is so versatile that it deserves its own mention.
Mutation operations don't mutate the tree,
they return new trees with the operation applied.
Trees new and old share as many branches as possible.
This allows very efficient implementation of most operations.
Inserting, erasing or retriving an element at an arbitrary location
all have logarithmic runtime complexity.

### Immutable Assignable

**Assignable references to deeply-immutable objects are very versatile**.
These types are classes where all methods are `const` except for two:
the move constructor and the assignment operator.
Objects of such types aren't strictly immutable, but… close.

Deeply-immutable objects can be too cumbersome:
without assignment support,
customers are very frequently forced to wrap instances
inside `NonNull<std::unique_ptr<>>`
or `NonNull<std::shared_ptr<>>` explicitly.

This is avoided by these mostly-immutable objects,
which behave as assignable pointers to nested deeply-immutable objects.
I've read that this roughly matches the behavior of immutable structs in C#.

To implement such types,
I usually nest all data fields in a private `Data` struct
and give a single field to the class,
of type `NonNull<std::shared_ptr<const Data>>`.

    class Line { ...
      struct Data {
        LazyString contents;
      };

      NonNull<std::shared_ptr<const Data>> data_;
    };

This ensures that the state is immutable while
allowing references to be updated, something like this:

    Line line = GetFirstLine(...);
    while (line.IsEmpty())
      line = GetNextLine(line, ...);
    line = AddSuffix(line, ...);

Using `NonNull<std::shared_ptr<>>`
also allows trivial shallow copies of the const values.

The resulting classes are thread-compatible, though not thread-safe;
assignment needs to be protected.

#### Examples

The following are examples of immutable classes in Edge:

* [`LineSequence`](https://github.com/alefore/edge/blob/master/src/language/text/line_sequence.h)
* [`Line`](https://github.com/alefore/edge/blob/master/src/language/text/line.h)
* [`ConstTree`](https://github.com/alefore/edge/blob/master/src/language/const_tree.h)

### Builder Pattern

I've found a "builder" pattern fairly versatile.
I use **mutable thread-compatible builder instances
to *create* immutable
(and thus thread-safe; and thus widely shared) instances**.

To avoid dependency cycles,
the output type should not depend on the builder.
The output object should declare constructors private,
which means it should give the builder `friend` access.

Most builder instances are used to build a single output.
As a performance optimization,
I avoid deep copies in `Build`.
To do this, I make the `Build` method
require `*this` to be an rvalue reference (`&&`)
—so that data can be moved out of the builder—
and implement an explicit `Copy` method
for the few customers that need it, e.g.:

    class LineBuilder { ...
      Line Build() &&;
      LineBuilder Copy() const;
    };

Unfortunately, this can be a bit cumbersome
(because of the additional `std::move` calls for some customers).

### Examples

TODO: Link. Add more.

* [`Line`](https://github.com/alefore/edge/blob/master/src/language/text/line.h)
  and
  [`LineBuilder`](https://github.com/alefore/edge/blob/master/src/language/text/line_builder.h)

## Thread safety

My **`concurrent::Protected<Data>` template has worked well**
for classes that need to be thread-safe (and stay non-const):
[src/concurrent/protected.h](https://github.com/alefore/edge/blob/master/src/concurrent/protected.h)

Using types effectively can boost thread-safety significantly.
Before I introduced `concurrent::Protected`,
Edge only had a relatively small amount of parallelization.
I was wary of the insidious woes brought upon by multi-threading
and wanted to only allow it in a controlled and careful manner.

And yet, even though `concurrent::Protected` isn't 100% fool-proof
(one doesn't have to try too hard to escape its protections),
adopting it
—starting to rely on types
to ensure that locks are acquired
before the data they protect can be accessed—
already helped me detect bugs
([example, where `WorkQueue::NextExecution` was neglecting to lock a
mutex](https://github.com/alefore/edge/commit/69fafea1d557bbb16ca579067dfc3f68a7712c0d)).

The amount of concurrency in Edge has only grown since.
I'm deferring a lot more work to background threads.
I don't think I'd have been able to get this far
if I had not relied on types to maintain correctness.

There are other solutions to this problem, such as
[Abseil's lock annotations](https://abseil.io/docs/cpp/guides/synchronization#thread-annotations)
that probably work about as well, perhaps even better.

## Edge: Lessons: Futures

I've had **great success using a Futures API**
to implement asynchronous requirements.

Futures-based code is not as clean as synchronous code, but close.
It is much cleaner than callback spaghetti.
Code still generally reflects that a function is creating and returning a value,
and its caller is consuming the returned value.

TODO: Error handling in futures.

### Single consumer

My futures implementation **supports setting only one consumer per future**.
Once available, the **future's value is passed as a value to the consumer**
(rather than, for example, handing out a `const` reference).
This allows me to use futures of moveable non-copyable types.

For situations where multiple listeners are desirable,
turn a regular future into a `ListenableFuture`
([implementation](https://github.com/alefore/edge/blob/master/src/futures/listenable_value.h))
which holds the value and allows setting multiple listeners,
which receive `const` references (rather than the value itself).

### WorkQueues

One pattern I've found useful is to schedule computations in background threads
and use futures in a way that
ensures that the output values
are received by the original thread
(that scheduled the computation).

This allows objects that aren't yet thread-safe
(for Edge the main case would be the `OpenBuffer` class)
to still delegate work to thread pools.
All they need to do is ensure that
all values they capture in lambdas they pass to thread pools
are thread-safe.
The outputs of those asynchronous computations
will be received in the main thread.

The customer interface is that the `ThreadPoolWithWorkQueue::Run` method
returns a `futures::Value` that will be notified in the calling thread:

    thread_pool
        .Run([...] -> Output {
          // This runs in a background thread in the thread-pool.
          return ExpensiveComputation(...);
        })
        .Transform([this](Output output) {
          // Capturing `this` is fine, even if `this` isn't thread-safe: this
          // lambda will run in the main thread, thanks to the work queue.
          ...
        });

This works because `ThreadPoolWithWorkQueue`
schedules notification of these futures
through the `WorkQueue`.
All we need to do is ensure that these `WorkQueue` instances
are advanced in the appropriate thread.

#### Examples

I use `ThreadPoolWithWorkQueue` in many places.
For example, my `FileSystemInterface`
schedules all blocking operations in a background thread,
but uses a `ThreadPoolWithWorkQueue` to ensure
that their results are received in the main thread.
Customers of `FileSystemInterface` just adopt the futures interface;
they will continue to run in a single thread
(and don't even need to support single-thread reentrancy).

### Cancellation

Being able to **cancel asynchronous computations that have become irrelevant**
can be fairly important.
The pattern I've landed on is the use of a `DeleteNotification` class
that contains a `futures::ListenableValue<EmptyValue>`.

A producer that wants to support cancellation
simply receives among its inputs
a `futures::ListenableValue<EmptyValue>` abort notification.
The producer can either poll on this listenable future
or add a listener callback.
The producer's customer simply creates a `DeleteNotification`,
passes its corresponding future to the producer,
and ensures that the `DeleteNotification` is retained
as long as the future value being produced is still relevant.

For example, as the user types in a prompt
(e.g., open a file, search for a regexp…)
I kick off background work
(to show a preview of files;
or a count of matches based on the query the user has already entered).
If the user types a second character before this work completes,
I just replace the `DeleteNotification` instance with a newly built instance
(which signals to the background thread that its output has become irrelevant
and it should just abandon whatever partial results it has computed)
and kick off a new operation.

### Futures in Const Objects

You can store `const` views of `ListenableFutures` inside `const` structures.
The producer will eventually give them a value, triggering consumers' execution.
When parts of an object are computed asynchronously,
this allows customers to access each such part as soon as it is ready.

At first, I found this pattern somewhat… counterintuitive.
It feels strange to tag as `const`
an object whose parts are still being constructed.

But I think it makes sense.
The contract of the future never changes:
customers can add callbacks that will run
when the value arrives (or immediately).
All callbacks will receive exactly the same value when they run.

The alternative would be to only return the object once all its parts are ready
(i.e., rather than building an object
containing futures for its asynchronous parts,
build a future of the fully-built object).
But this would unnecessarily delay things.

#### Examples

A good example of this pattern
is the `LineMetadataEntry` class
([source](https://github.com/alefore/edge/blob/master/src/language/text/line.h)).
When loading lines in a file,
Edge tries to compile them
(as expression of Edge's extension language)
and –if they compile and the resulting expressions are free of side-effects–
evaluate them and display the evaluation results.

For example,
Edge will display "7.60417"
next to a line that contains "365 / (52 - 4)".

Such evaluation could take a while
(depending on the complexity of the expression),
so Edge feeds these evaluations to a thread pool,
upholding its principle of avoiding blocking the main thread.

The implementation is very natural:
due to its nested `LineMetadataEntry`,
`Line` contains:

* A future with the string representation of the evaluation result.
* The initial text that should be displayed
  if the future doesn't yet have a result.

## Challenges

I consider the following unsolved challenges when it comes to concurrency.
I think they continue to slow me down
and I would like to find better ways to deal with them.

* Mangled stacks.
  The use of asynchronous techniques such as futures or callback spaghetti
  results in very confusing stacks,
  which makes debugging difficult.

