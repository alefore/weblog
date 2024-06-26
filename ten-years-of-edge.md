# Edge: Lessons

* Introduction
* Fix bug categories
* Concurrency
* Use Types Effectively
* Bursts & Pauses
* Ideas that worked well
* What's Next?
* Conclusions

## Introduction

This document is work-in-progress.
As of 2024-04-13, this is still incomplete
and probably contains errors.
I haven't yet had the time to fully review it.

This document is **an attempt to capture lessons I've learned
after a decade of working on Edge** (my personal text editor).

Many of the lessons described are fairly technical
but at least one (Bursts & Pauses) is general
—observations about the life-cycle of side-projects.
Among the technical lessons,
many are specific to C++,
but they probably apply, to some extent,
to other languages.

### Background

Edge is a Linux C++ terminal-based text editor.
I started working on
Edge in 2014
–making the
[very first commit](https://github.com/alefore/edge/commit/312ecc2462315e8e0648cbd2680cc0366819df1e)
on 2014-08-09.
I've used Edge ~exclusively since 2015.

Edge has its own extension language,
which looks like C++ with memory management,
and has logic (such as syntax highlighting)
for editing C++, Markdown, Java, Python
and a few other file types.

I mainly use it…

* for programming at work
  (though this days I program relatively little at work),

* to maintain my
  [Zettelkasten](https://github.com/alefore/weblog/blob/master/zettelkasten.md),

* as my planning & productivity management system
  (which is integrated with my Zettelkasten,
  so this is really repeating the previous point), and…

* to develop Edge.

As of 2024-04-14, Edge is 67.9k lines of C++ code
(per `wc -l $(find src -name '*.cc' -or -name '*.h' -or -name '*.y'`).

### Caveats

* I believe there's significant **recency bias** in this distillation.
  I'm probably overweighing insights I've reached relatively recently
  –towards Edge's tenth anniversary–
  and not doing justice
  to those from the beginning of the journey.

* The lessons described here are fairly **subjective**.
  These are *not* universally applicable principles.
  They rest on various assumptions specific to my context,
  which may not apply to other environments or systems.
  I'm holding back from prefixing nearly every sentence
  with "in my experience" or similar qualifiers,
  but they should be assumed.

## Fix Bug Categories

If you identify a bug,
resist the temptation to just fix it and move on.
Instead, **fix the underlying issue that enabled the bug to exist**.
The specific instance of the bug should just happen to disappear,
due to the underlying fix.

Sometimes this is impossible;
software *will* have bugs.
But I've found many cases where digging deeper pays off.

How can it be that I've made the mistake that
allowed this bug to exist?
What can I change such that this entire category of bugs no longer happens,
*can no longer happen*?

At the bare minimum,
add a unit test that fails
before fixing the bug.
But don't let "adding a unit test" replace
trying to find more robust ways
to avoid the entire category of bug.
At least, aim to have a defensible answer to the question
"why didn't I have a unit test that would have detected this bug?"

### Examples with runtime validation

The following examples show how this can be done through runtime checks.
A much preferable technique is to use types effectively;
that's covered in [a separate section](#use-types-effectively).

#### Range

Edge uses the concept of *ranges* in text:

* A `LineColumn` represents a position in a buffer
  –*e.g.*, "line 5, column 2".

* A `Range` is a pair of `LineColumn` instances
  –*e.g.*, "from line 5, column 2;
  to line 7, column 21").

Ranges are used frequently
by operations that only affect a part of a buffer
(*e.g.*, "search only in this section").

I had a bug
where an expression was incorrectly computing a range.
In some corner cases
the *end* `LineColumn`
preceeded the *begin* `LineColumn`
in the computed range
([fix](https://github.com/alefore/edge/commit/750f660349c5fb8b36823d77d4a39062e9fa9634)).

I decided to **add an explicit requirement to `Range`**:
*end* must be greater than or equal to *begin*.
I adjusted the constructor and setters to validate this explicitly
([commit](https://github.com/alefore/edge/commit/279f001740d0c1ed8fa4e29b7286d59f0d746922)).

A more robust solution would detect these issues at compile time,
but at least they won't be silently ignored
(allowing them to manifest in more subtle ways,
making debugging significantly more cumbersome).

It only took a few days for this to uncover
unrelated instance of the same issue
([fix](https://github.com/alefore/edge/commit/14f8ed2655c09690cb86b3a7115b0efb51798398)).
Who knows how long this bug would have gone undetected.

#### Validate NonNull

Edge uses templated `NonNull<>` wrappers for various pointer types
([source](https://github.com/alefore/edge/tree/master/src/language/safe_types.h)).
This has worked very well generally (more on that later),
but has issues with read-after-move:
when I move out of a `NonNull<std::unique_ptr<>>`
or `NonNull<std::shared_ptr<>>`,
the old instance is actually left with a `nullptr`,
which will lead to all sorts of strange issues if I accidentally reuse it
(*i.e.,* if I have read-after-move bugs).

When I caught a bug where that was happening
([commit](https://github.com/alefore/edge/commit/2635806022c6fe8609ca81e4ff06f9801670bd0f)),
I decided to add **extra validation to the `NonNull` methods**,
to ensure that a non-null input was... indeed non-null
([commit](https://github.com/alefore/edge/commit/6cb2b3e53db474849009404b751e46c6a1e3dc0b)).

It only took two days for this to find another such issue
that had been ignored silently
([commit](https://github.com/alefore/edge/commit/09586758eefa0e35d6721228aeaf3dc34b4350cc)).

## Concurrency

Concurrency: difficult, but necessary.
Some techniques help.

### Necessary

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

#### Limits of non-parallel concurrency

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

#### Concurrency: Threads vs other approaches

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
([header](https://github.com/alefore/edge/blob/master/src/infrastructure/file_system_driver.h),
which receives a thread-pool.

### Constness

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

#### Immutable Assignable

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

##### Examples

The following are examples of immutable classes in Edge:

* [`LineSequence`](https://github.com/alefore/edge/blob/master/src/language/text/line_sequence.h)
* [`Line`](https://github.com/alefore/edge/blob/master/src/language/text/line.h)
* [`ConstTree`](https://github.com/alefore/edge/blob/master/src/language/const_tree.h)

#### Builder Pattern

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

#### Examples

TODO: Link. Add more.

* [`Line`](https://github.com/alefore/edge/blob/master/src/language/text/line.h)
  and
  [`LineBuilder`](https://github.com/alefore/edge/blob/master/src/language/text/line_builder.h)

### Thread safety

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

### Futures

I've had **great success using a Futures API**
to implement asynchronous requirements.

Futures-based code is not as clean as synchronous code, but close.
It is much cleaner than callback spaghetti.
Code still generally reflects that a function is creating and returning a value,
and its caller is consuming the returned value.

TODO: Error handling in futures.

#### Single consumer

My futures implementation **supports setting only one consumer per future**.
Once available, the **future's value is passed as a value to the consumer**
(rather than, for example, handing out a `const` reference).
This allows me to use futures of moveable non-copyable types.

For situations where multiple listeners are desirable,
turn a regular future into a `ListenableFuture`
([implementation](https://github.com/alefore/edge/blob/master/src/futures/listenable_value.h))
which holds the value and allows setting multiple listeners,
which receive `const` references (rather than the value itself).

#### WorkQueues

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

##### Examples

I use `ThreadPoolWithWorkQueue` in many places.
For example, my `FileSystemInterface`
schedules all blocking operations in a background thread,
but uses a `ThreadPoolWithWorkQueue` to ensure
that their results are received in the main thread.
Customers of `FileSystemInterface` just adopt the futures interface;
they will continue to run in a single thread
(and don't even need to support single-thread reentrancy).

#### Cancellation

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

#### Futures in Const Objects

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

##### Examples

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

## Use Types Effectively

Good use of expressive and well-designed static types
facilitates correctness and maintainability.

### Custom Types

I **use semantically-rich custom types for variables and class fields**
(and methods' inputs and outputs).
Many of these types are simple wrappers of primitive types
(`int`, `string`, `double`, etc.).

Primitive types are just the building blocks
with which custom types are implemented.
In general, I try to only use primitive types to define these custom types.

If a telephone number is an important concept for your application,
define a corresponding type.
Don't use `string` (or `int`).

A good function receives:

* a telephone number,
* a number of call attempts,
* a file descriptor, and
* a location description.

A bad function receives:

* a string (with a telephone number),
* an int (with a number of call attempts),
* another int (with a file descriptor), and
* another string (with a location description).

Aim to convey this through types.
If you pass the wrong information
(or the right information in the wrong order),
the compiler should reject the expression.

When you receive the underlying representation of the object
as a primitive type (*e.g.*, `int`),
convert it immediately to your custom type.
In my experience,
most application logic operates entirely using the custom types,
and it only needs to convert from and to the underlying type
in relatively few places.

#### Validation

Using **custom types enables validation of expectations
in the constructor**.
The rest of the application
can reliably assume that these preconditions are met.

Concrete examples in Edge are:

* The `Range` type
  ([header](https://github.com/alefore/edge/blob/master/src/language/text/range.h)),
  which enforces that *begin* <= *end*.

* The `Probability` type in the Naive Bayes implementation.
  This validates that a probability (represented using `double`)
  is in the expected [1, 0] range.
  Not only do I avoid having to add validation logic
  every time I compute a new probability
  (e.g., when I evaluate complex formulas that yield a new probability),
  I also know that this expectation is validated
  every single time a new probability is defined.

TODO: The above (about validation of Probability) isn't currently true.

#### Ghost types

Unfortunately, afaik, there's no standard way to achieve this in C++23.
With `typedef` or `using`, the compiler won't detect errors,
so there are very little gains
(code *will* be slightly more readable,
but you won't gain static type-safety).

To define custom types, **I currently use `GHOST_TYPE` expressions** like these:

From 
[environment.h](https://github.com/alefore/edge/blob/master/src/vm/environment.h):

    // Represents a namespace in the VM environment, where symbols can be
    // defined. For example, a reference `lib::zk::Today` is actually the symbol
    // "Today" in the namespace `{"lib", "zk"}`.
    GHOST_TYPE_CONTAINER(Namespace, std::vector<Identifier>)

From
[file_system_driver.cc](https://github.com/alefore/edge/blob/master/src/infrastructure/file_system_driver.h):

    GHOST_TYPE(FileDescriptor, int);
    GHOST_TYPE(UnixSignal, int);
    // Why define a GHOST_TYPE for `pid_t`, which is already a "specific" type?
    // Because pid_the is just a `using` or `typedef` alias, so incorrect usage
    // isn't detected by the compiler.
    GHOST_TYPE(ProcessId, pid_t);

`GHOST_TYPE` are a group of macros
that declare a new struct for a custom type,
including  appropriate constructors and operators
(things like std::hash, operator==, operator+=, etc.)
based on what the underlying type supports.

I'm not too happy with this approach.
I suspect a **better approach may be
to do it based on template metaprogramming**,
something like:

    struct Digits : public GhostType<Digits, std::vector<size_t>> {}

This would support significantly more customization
through template parameters for the `GhostType` class.
Specifically, I would like to make it easier to define validation functions.
With the current `GHOST_TYPE` macros this is difficult.

### Maintaing invariants with static types

Defining wrapper types corresponding to predicates
is another technique I've found useful.

#### Definition

If you have a type `T` and a predicate `P`,
you would create a type `TP`
that contains instances of `T` match `P`.
I'll refer to `TP` as the *subtype*.

This is exactly what `const` does to classes:
you define the semantics of `P`
(*i.e.*, what it means for a class to be `const`)
and then, given a `T` instance,
you can trivially obtain a `TP` view that implements `P`.
However, this general principle goes way beyond just immutability.

This allows functions to **explicitly state some preconditions**
(*e.g.*, an input container must be sorted,
or non-empty,
or have exactly between 128 and 256 elements).
Instead of checking inputs against predicates
(or, alternatively, transforming inputs to match the predicates;
*e.g.*, sorting a container received),
functions should receive their inputs using the subtypes directly.

TODO: Give an example.

#### Rationale

Judicious use of predicate types tends to:

* **Ensure that all cases are handled.**
  If a function receives (or returns) the general type `T`,
  it signals explicitly that its implementations
  (or customers receiving its output)
  must handle the case
  where the objects passed don't match `P`
  (or else the function would have used `TP`).
  This helps ensure correctness;
  the benefits are already visible
  in Edge's medium-sized code base.

* **Simplify validation.**
  Validation (and/or corresponding conversion to the subtype)
  tends to happen in just a few places;
  this often renders irrelevant
  a lot of assertion checks,
  allowing them to be safely removed.

* **Delete dead code in a robust way.**
  I've even found cases where this allowed me to delete code
  that was handling situations
  that could never occur:
  all callers were always passing non-null pointers,
  but the function still contained code to handle the null case.
  Here I use the type system to ensure that these cases can't occur
  –and that *I won't introduce them accidentally in the future*–
  and delete the corresponding code.

#### Examples

##### SortedLineSequence

A good example of using types to express explicitly
that an instance of a more general type matches a predicate:
`SortedLineSequence` and `SortedLineSequenceUniqueLines`.

The general type `LineSequence` holds an finite immutable sequence of lines
([implementation](https://github.com/alefore/edge/blob/master/src/language/text/line_sequence.h)).

Some functions receive a "dictionary":
a `LineSequence`, with one line for each token ("word"),
*sorted in ascending order* (by some comparison predicate).
For example, functions implementing:

* Autocomplete functionality.
* Logic that highlights typos.

The reason the `LineSequence` needs to be sorted
is to be able to run binary searches given a prefix.

One could argue that these functions should receive a `std::set`
or some other similar type.
Such a discussion would be entirely orthogonal
to the point of this example.
We use `LineSequence` because that's the structure used by Edge
to represent file contents.
This allows us to reuse Edge's file-reading logic
to read dictionaries from the file system.

How can we ensure that these functions are always given sorted sequences?

You can ignore the problem:
if the customer passes an unsorted file,
they'll get strange results.
They should know better.

This is brittle;
you need to align the sorting predicate,
which can be tricky with unicode characters and such.
In other words,
can you be sure that C++'s `operator<`
(which may depend on your locale's settings)
matches the behavior of the `sort` command-line program
(with which the user manages their dictionary)?

You can *detect* that the file isn't sorted and display errors.
Slightly better: at least the user will be aware that something is wrong.
But not terribly useful.

You can *sort* the contents in memory just after the file is read.
Sorting is ~fast
and can be done by a background thread
(possibly delaying autocomplete and similar operations
for a few milliseconds after start).

This is significantly better,
but you still need to either:

1. Validate "manually" that all customers of these functions
   are passing sorted contents.
   This is brittle.
   If you add a new customer and neglect to sort,
   nothing will alert you.

2. Change these functions to sort the contents as they receive them.
   This can be inefficient depending on how often this happens,
   and doesn't entirely address the problem
   –what if some entry points in these functions neglects to sort?

The solution is to use `SortedLineSequence`,
a class that simply holds a `LineSequence`
where the contents are known to be sorted
([implementation](https://github.com/alefore/edge/blob/master/src/language/text/sorted_line_sequence.h)).
This is done at construction:
its **constructors directly sort the contents** they receive.
This means that it is impossible to hold a `SortedLineSequence`
where the contents aren't sorted.

We simply upgrade our autocomplete and similar functions
to require a `SortedLineSequence` input
where they previously received a `LineSequence`.

This **makes it impossible for customers of these functions
to accidentally pass unsorted contents**.
Customers can just create the `SortedLineSequence`
right after reading (or generating) contents.

##### NonNull

In Edge, I've defined `NonNull<>` wrappers for `std::unique_ptr<>`,
`std::shared_ptr<>`
and raw pointer types.
This is useful to convey explicitly
that a pointer given to a function
(or just held in some variable)
can't be null.

Most of the times functions receive values by reference,
but there are genuine cases to use `NonNull<>` pointers
such as:

* When storing them inside an `std::vector`
  (since vectors can't contain references).

* When you want to share ownership
  (or transfer unique ownership of non-moveable types).

* When you want to have assignable values
  referencing instances of non-moveable types.

If a function receives a regular (whether raw or smart) pointer
(*i.e.*, *not* a `NonNull<>`-wrapped pointer)
it *must* be prepared to handle a `nullptr` value.
This is relatively rare.

Consistent application of `NonNull<>` helps ensure
that functions that receive a regular pointer
handle the `nullptr` case.
This is much more robust than specifying this through comments.

As an aside, `NonNull<>` pointers are much more common than nullable ones.
In
[my GC module](https://github.com/alefore/edge/blob/master/src/language/gc.h),
pointer types (`gc::Ptr<>`) can't contain null objects;
a nullable pointer must be explicitly marked
(typically as `std::optional<gc::Ptr<>>`),
moving the boilerplate from the common to the infrequent case.

##### LineRange

TODO.

##### OnceOnlyFunction

In many situations I pass around callables that should only be called once.
I could have simply used `std::function`
but making this explicit helps me make sure
I can safely move values around during execution
and detect reliably (though at runtime)
if I ever accidentally run the same callable twice.

This is implemented in
[`src/language/once_only_function`](https://github.com/alefore/edge/blob/master/src/language/once_only_function.h)
(as a wrapper of `std::move_only_function`).

## Bursts & Pauses

### Bursts

**My engagement with Edge is extremely bursty**.
Most progress happens in bursts of activity
that happen about once per year
and usually last from two to five months,
during which I commit many changes per day.
After the bursts come long pauses of inactivity.

#### New Possibilities Open

> Pastel crumbs on a brown mantle;
> Spring returning to the forest on the hills.

The factors that sustain these bursts are:

                         ┏━━━━━━━━━━━━━━━━┓
                         ┃ Transformative ┃
                         ┃      ideas     ┃
                         ┗━┯━━━━━━━━━━━━━━┛
                           │
                         (+a)   ╭─(+b)─╮
          ╭────(+c)─────╮  │    │      │
    ┏━━━━━V━┓         ┏━┷━━V━━━━┷━━━━━━V━┓
    ┃ Focus ┠──(+d)───>     Progress     ┃
    ┗━━━━━━━┛         ┗━━━━━━━━━━━━━━━━━━┛

This starts when "transformative ideas" crystallize
—ideas about new approaches I could try,
all the way from internal implementation details
to very visible changes.

* Per relationship (a),
  when this happens
  many features, internal improvements, or optimizations
  (for performance or correctness)
  that I've been wanting for a long time (sometimes years)
  and that previously seemed very hard or even impossible,
  suddenly ... fall gracefully into place.
  I make a lot of progress.

* Per relationship (b),
  these gains compound:
  progress feeds on itself.
  Some of the major improvements I've just landed
  enable a large set of follow-up effects.
  Many adjacent modules become simpler,
  faster,
  more general,
  easier to use,
  etc..

* Per relationships (c) and (d) respectively,
  the excitement from the fast-paced progress is very motivating,
  and the increased focus also enables faster progress.
  This causes the reinforcing loop that sustains the burst.

##### Take Breaks: Bursts: Example GC

One tangible example
of things "that previously seemed very hard"
falling gracefully into place
are the improvements to Edge's garbage collection (GC) implementation
between 2023-09 and 2023-10.
These changes improved Edge perceptibly,
without making the implementation more complex.

A burst ending in 2022-12
made GC pauses relatively slow:
fast enough that Edge was still usable,
but slow enough that one could notice when they happened (ugh!).
However, I had the feeling that
Edge's GC implementation was as efficient as I could reasonably expect.

And then, ten months later,
between 2023-09 and 2023-10,
I optimized the collection
through a series of gradual changes
that made GC pauses imperceptible.
I did this through a combination of:

1. Implementing highly-concurrent collection.
   Most non-constant-runtime operations now happen in thread-pools
   of configurable size.
   These are things such as
   navigating the set of reachable objects
   or deleting unreachable objects.

2. Executing work asynchronously.
   Not only is work dispatched to multiple threads,
   but the main thread now offloads some operations
   (*e.g.*, actual deletion of objects)
   to a dedicated thread-pool entirely
   and no longer waits for their completion.

3. Implementing resumable (incremental) collection.
   The collection operation detects if it overruns a customer-specified timeout;
   when this happens, it just returns directly;
   The next collection resumes where we left off.
   I had assumed that I would require a very significant rewrite
   (since the user can change pointer values
   in the middle of interrupted collections).

4. Delivering profile-based optimization,
   chasing after the functions that consumed significant time
   from the main thread.

Not only did I speed collection up,
I actually managed to simplify the customer interface
([commit](https://github.com/alefore/edge/commit/42ae36687e96d2ddd2510562368f2965951f9cfd),
[commit](https://github.com/alefore/edge/commit/8eefd3a47f20350e9c18259f9c95811f478b1c25))
and make usage less error-prone
by removing the burden of calling `Ptr::Protect` from its customers.

The core implementation
([gc.h](https://github.com/alefore/edge/blob/master/src/language/gc.h),
[bag.h](https://github.com/alefore/edge/blob/master/src/concurrent/bag.h) and
[gc.cc](https://github.com/alefore/edge/blob/master/src/language/gc.cc))
didnd't become significantly more complex:
LOC only grew by 3%!

* Between 2022-12-24 and 2023-08-29 these files didn't change;
  they contained 1163
  (519 + 214 + 430, respectively) LOC

* In 2023-10-27,
  After all these optimizations,
  they grew by a net delta of +35 LOC,
  to 1198
  (572 + 129 + 497, respectively) LOC.

In this specific case, LOC is a reasonable proxy
for implementation complexity.
To compute LOC, I excluded unit tests and included header documentation.

#### Ramp Down

> Summer afternoon: a swarm of wasps dances around my Spritz.

Usually after a few months
(though the specific durations vary greatly)
comes a ramp down.
I start pursuing other things in my life.
Edge becomes a secondary endeavor.

This happens because the supply of transformative ideas is depleted,
causing progress to slow down,
which enables unrelated distractions to reduce my focus on Edge.

     ┏━━━━━━━━━━━━━━┓                              ┏━━━━━━━━━━━━━━━━┓
     ┃  Unrelated   ┃                              ┃ Transformative <─╮
     ┃ distractions ┃                              ┃      ideas     ┃ │
     ┗━━━━━━━━━━━━┯━┛                              ┗━┯━━━━━━━━━━━━━━┛ │
                  │                                  │                │
                  │                                 (+a)  ╭─(+b)─╮  (-e)
                  │                 ╭────(+c)─────╮  │    │      │    │
                  │           ┏━━━━━V━┓         ┏━┷━━V━━━━┷━━━━━━V━┓  │
                  ╰───(-f)────> Focus ┠──(+d)───>     Progress     ┠──╯
                              ┗━━━━━━━┛         ┗━━━━━━━━━━━━━━━━━━┛

The order of events is very clear:

1. Per relationship (e),
   *Progress* depletes the stock of *Transformative ideas*
   and their ripple effects.
   This dampens the effects (a) and (b).

2. *Progress* becomes visibly slower,
   dampening relationship (c).
   I commit a couple changes a day
   but this gradually drops
   to a small change or two a week.
   I'm still trying to make progress, but …
   I've run out of easy things to implement.
   everything that remains is way too challenging or impossible
   (e.g., everything has been optimized
   for performance and correctness and simplicity
   as much normal constraints allow).

3. *Focus* decreases,
   and the effect of relationship (d) disappears.
   This allows the tension depicted in relationship (f) to play out:
   *Unrelated distractions* decrease my *Focus*.

Although the tension (f) between *Focus* and *Unrelated distractions*
is present throughout the entire burst,
*Progress* drops first, in step #2.
The shift away from Edge happens afterwards.
*Focus* is necessary but not sufficient.

### Pauses

And then starts a pause.
Other things fully take precedence.
Perhaps I switch to writing a short story,
or
a [Zuri Short](https://github.com/alefore/weblog/blob/master/zurich-shorts.md).
Or learn to play the harmonica.
Or [knit](http://github.com/alefore/knit).
I go on a long trip.
I continue *using* Edge, but I rarely improve it.

This lasts six months on average, sometimes significantly longer.
Until suddenly I can't resist the itch
to finally fix that annoying behavior
or implement that great idea that found me
and I end up getting sucked back in.
A new cycle begins.

#### Background Thinking

> Powdery snow, birches washed white.
> On a twig, a twinkle: a yellow tit chirps. Was it really there?

What triggers the periods of intense focus?
What enables the fast progress on areas I previously considered out of reach?

     ┏━━━━━━━━━━━━━━┓        ┏━━━━━━━━━━━━┓        ┏━━━━━━━━━━━━━━━━┓
     ┃  Unrelated   ┠──(+g)──> Background ┠─(+h⏳)─> Transformative ┃
     ┃ Distractions ┃        ┃  thinking  ┃        ┃      ideas     ┃
     ┗━━━━━━━━━━━━━━┛        ┗━━━━━━━━━━━━┛        ┗━━━━━━━━━━━━━━━━┛

During the pauses
my brain **keeps making connections and generating and encountering ideas**.
These can be alternative approaches I could try:
new underlying APIs,
different data structures,
entirely new user journeys,
etc..

This happens mostly subconsciously.
I'm not working on Edge actively,
but I'm still passively thinking about these problems:
the places where I got stuck
and couldn't figure out a better solution;
the things that annoy me as I use Edge,
even if I don't notice them consciously.
I still read articles about text editors
and find new ideas I had never considered.

I don't know if this type of medium-term background thinking can be rushed.
Maybe
[journaling](https://github.com/alefore/weblog/blob/master/journaling.md) helps?
Maybe talking to people with different perspectives?
In my experience,
some things help but they only go so far.
This takes time.

In any case, eventually a new idea, or a group of ideas,
crystallizes and I just have to try things out and ... boom!
The new ideas (often) work,
and they (often) have far reaching implications,
and a new burst begins.

### Emotional State

My emotional state factors strongly into the timing of the cycles.
The timing of bursts is strongly associated
with important events happening in my life:

* Some bursts started when I was going through difficult personal situations
  (*e.g.*, breaking up from a serious relationship)
  and wanted some distraction from real life.
  This is the case for the long burst from 2019-12 to 2020-05.

* Many long pauses coincide
  with times when I was distracted by real life events.
  Perhaps I was doing a lot of traveling,
  starting a new relationship,
  or distracted with changing apartments.
  Being very busy with life events left time for only short-lived bursts during
  the pauses from 2015 and 2016;
  2020-05;
  and 2021-03.

The causality goes both ways:
though less frequently,
these cycles have also caused changes in my life.

* The long burst from 2022-02 to 2022-06
  influenced a difficult decision I took at work
  to switch away from managing
  (which I had been doing for about 5 years)
  to a pure engineering role.
  The team I was managing grew
  from 6 engineers in 2020
  to 17 in early 2022.
  Most of my time (at work)
  became occupied by managerial tasks,
  pushing out technical work.
  I interpreted the amount of energy I was dedicating to Edge
  as a strong sign that I was missing technical work.
  Interestingly, the burst stopped shortly after I made the decision
  and started searching for someone
  I could transfer my management responsibilities to.

### Unrelated distractions

The complete system diagram would be this:

     ┏━━━━━━━━━━━━━━┓        ┏━━━━━━━━━━━━┓        ┏━━━━━━━━━━━━━━━━┓
     ┃  Unrelated   ┠──(+g)──> Background ┠─(+h⏳)─> Transformative <─╮
     ┃ distractions <──╮     ┃  thinking  ┃        ┃      ideas     ┃ │
     ┗━━━━━━━━━━━━┯━┛  │     ┗━━━━━━━━━━━━┛        ┗━┯━━━━━━━━━━━━━━┛ │
                  │    │                             │                │
                  │    ╰──(-i)──╮                   (+a)  ╭─(+b)─╮  (-e)
                  │             │   ╭────(+c)─────╮  │    │      │    │
                  │           ┏━┷━━━V━┓         ┏━┷━━V━━━━┷━━━━━━V━┓  │
                  ╰───(-f)────> Focus ┠──(+d)───>     Progress     ┠──╯
                              ┗━━━━━━━┛         ┗━━━━━━━━━━━━━━━━━━┛

This could, however, be reduced to
the main relationships from
*Unrelated distractions* to *Progress*:

    ┏━━━━━━━━━━━━━━┓           ┏━━━━━━━━━━┓
    ┃  Unrelated   ┠───(-)─────> Progress ┃
    ┃ distractions ┠──(+⏳)────>          ┃
    ┗━━━━━━━━━━━━━━┛           ┗━━━━━━━━━━┛

* In the short-term, *Unrelated distractions* hamper *Progress*
  because they deplete focus (relationships (f) and (d)).

* In the long-term, however,
  *Unrelated distractions* are necessary for *Progress*
  because they enable *Transformative ideas* to manifest
  (relationships (g), (h), (a)).

### Implications

What are the main implications?

#### Acceptance

> Der Wind nimmt die gelben Blätter des alten Ginkgo.
>
> Frei wie Libellen fliegen sie davon.

I have to understand and accept when a burst is coming to its conclusion.
**Things that seem impossibly difficult today
may become trivial in the future**.

I find this very encouraging.

#### Make it easy to return

> Stars glow autumn red.
>
> The setting sun retreats from the maple tree.

Accepting that I'll take breaks means that I should
**make it easy to come back**.

There's some analogy to
[counter-cyclical fiscal policy](https://ec.europa.eu/eurostat/statistics-explained/index.php?title=Glossary:Counter-cyclical_fiscal_measures#:~:text=Counter%2Dcyclical%20fiscal%20measures%20are,to%20help%20stimulate%20economic%20recovery.).

To me this is similar to the "two-minute rule" habit formation trick:
leave yourself a lot of crumbs that you can pick up
whenever you happen to have a spare minute or two.
These million small steps tend to add up to meaningful progress.

##### Document Potential Improvements

Documenting my ideas for potential improvements
helps me: it makes it easier to find appropriately sized chunks of work,
depending on where I am in the burst-pause cycle.

I leave many "TODO" notes in the code
listing things that I think I may want to improve some day.
I try too:

* Classify them by difficulty (trivial, easy, normal)
* Give them hash-tags (to help me find related changes)

As I'm reading the code working on implementing a specific feature
unrelated ideas come up.
I'm not going to work on them right away;
I'd never accomplish the task I'm working on.
But I'll make sure to document them right away.

As of 2023-12-21, I have 220 TODO notes in the code.
16 are marked as trivial, 80 are marked as easy.

##### Diversity

I suppose I've been able to keep working on the same project for a decade
because I've been able to work on vastly different sub-projects,
all of which fall under the same umbrella, such as:

* A memory manager.
* A C++-line VM.
* Arbitrary precision numbers.
* Audio generation
  (nothing too fancy, though;
  [ref](https://github.com/alefore/edge/blob/master/src/infrastructure/audio.h)).
* A UX for interacting with my Zettelkasten.
* Futures.
* A Naive Bayes implementation
  (that I use, for example, to order search results
  by their relevance, based on history;
  [ref](https://github.com/alefore/edge/blob/master/src/math/naive_bayes.h))
* Thread pools and work queues.
* A command-line flags module
  ([ref](https://github.com/alefore/edge/blob/master/src/infrastructure/command_line.h)).
* Highly concurrent containers (for very specific use cases;
  [ref](https://github.com/alefore/edge/blob/master/src/concurrent/bag.h))
* tty integration
  (*e.g.,* parsing the escape sequences produced by an underlying shell with
  a pts;
  [ref](https://github.com/alefore/edge/blob/master/src/infrastructure/terminal_adapter.cc)).

#### Incremental Improvement

> Gli aghi degli abeti in montagna abbracciano calmi i fiocchi di neve.
> Niente ferma una valanga.

It's easy to underestimate the effect of small changes.
**Changes that, on their own, don't amount to much,
can have very drastic effects as a group**.
I think this is the self-reinforcing loop
of Progress (in one area) enables Progress (in others).
This is a big part of why these bursts happen.
They start as just a few small improvements,
but these small improvements precipitate
and enable many other small improvements and...
eventually they add up to large changes.

The majority of my commits are small "no-op" changes whose only purpose
is "preparation" to make follow-up changes easier or smaller.
A big change is divided into a million baby steps
and one final simple step that enables the new functionality.
I think this is what enables
things that previously seemed ~impossible
to "suddenly… fall gracefully into place".

## Ideas that worked well

> Las grandes hojas del abedul:
> unas se van, fugaces, con el viento;
> otras caen solas, dando vueltas en el aire frio.

### Principles

The following are a few *ideas or principles* that
influenced my work in Edge,
which worked better than I had initially anticipated:

* The **most precious resource is your user's mental energy**.
  Computers are fast, and
  –even if Moore's law is dead–
  keep getting faster and cheaper;
  if you can spend a few million cycles to
  save the user from
  having to mentally compute something
  (that distracts them from their goal),
  do it, go the extra mile.
  Never force the user to wait for the results of an operation.

  * If the user is typing a command to be executed
    (*e.g.*, fork out an "ls" or "grep" command, kick off a regexp search),
    display a preview of the results.
    For example, as the user types the regexp,
    you can already display "this would match 26 locations".
    As the user types an "ls" or "grep" command
    (which have no side effects),
    you can already display the first lines of the output they'd get.

* **Being able to check if a program expression
  is free of side effects (without running it)
  is powerful**.
  This enables me to asynchronously evaluate such expressions
  when they appear inside a file,
  displaying the evaluation results.
  Being able to enter (and edit) such expressions as I'm editing a file,
  this sort of "evaluate-as-you-type" functionality is very useful.
  I recorded
  [a demo](https://asciinema.org/a/pCa75UZYlVXD0uboyLQl5zUbi)
  for a friend who asked about this.

* **Optimize for development speed**.
  You can't know
  the direction in which you'll take a function or module in the future,
  but you can always invest in making your software more malleable.
  Make it easier to iterate faster.
  Avoid optimizing things for performance
  in ways that leave you stuck on local maxima.

### Features

The following are a few *features* that worked better than I anticipated.
I hope this helps other people working on their own text editors.

* **Linear undo**/redo history,
  as explained in
  [`src/undo_state.cc`](https://github.com/alefore/edge/blob/master/src/undo_state.cc).
  If you undo a few transformations and apply a new transformation …
  you don't lose the redo history: you can still apply it through undo.

* The ability to **jump to any position in the file that's currently visible**
  simply by pressing `f` +
  three characters ~matching the text you want to jump to
  (with disambiguation when the prefix would match multiple positions) + return.
  I'm using this fairly frequently,
  comparing with "scrolling" up to the position (or a full regexp search).

* Native support for **multiple cursors**.
  For example, a regexp-search
  just creates a cursor in every position with a match.
  Being able to quickly say
  "set the cursors to: a cursor at the beginning of every line
  in the current paragraph"
  and then just say "add four spaces at each cursor" is powerful.
  So is saying "set a cursor on every character (in the current file)
  where the compiler reported an error".

* The **preview buffer** at the bottom of the screen is helpful.
  If the cursor is over an existing file (based on some search paths),
  previewing its contents
  (remembering where the cursor was when that file was last opened)
  can be helpful.
  Similarly, previewing the output of commands as you type them
  (for commands that don't have side-effects)
  can also be very flexible.

* Native support for **"compiler" buffers**.
  Automatically tagging certain command buffers as a compiler
  (*e.g.*, "if the command is an invocation to `make` or `bazel` or …")
  and then (1) parsing the output to detect references to open files
  and (2) rerunning the command whenever I save a file,
  works well.
  Saving a source file suffices to kick off compilation;
  I continue editing but, as soon as the compiler outputs errors in the file,
  I get overlays summarizing the errors.

* **clang-format integration**.
  Whenever I save a file of some specific formats (*e.g.,* C++, Java,
  Javascript, SQL…), I just pipe it through `clang-format` or similar commands
  ([implementation](https://github.com/alefore/edge/blob/master/rc/editor_commands/lib/clang-format.cc)).
  Freeing you from caring about spaces as you're editing
  speeds you up.

## What's Next?

> He savors a hearty brunch; takes his time.
>
> Sitting beside him, feet dangling, roller blades already on, his daughter hums eagerly.

Who knows where Edge will be in ten years?
Will I still be extending it?

This section lists some ideas I'd like to explore.

### Improve the extension language

The extension language could use some improvements.

I'd like to be able to define structures
entirely within the extension language.
This hasn't been too constraining for me
(because I can define structures within the host language
and easily expose them to extensions),
but I think it would enable extensions to grow significantly.

I would also like to support some form of generics.
Currently, for generic classes like "set" or "vector"
I have to define specific subtypes in the host language
(e.g., `SetString`, `VectorString`, `VectorInt`, etc.).
It's easy to define this in the host language, but far from ideal.

### File inspection/manipulation API

I'd like to provide friendlier ways to express operations
that filter or transform sub-trees of a document, such as:

* Find all links with the text `xyz` within a bullets list in a section
  where the header is `Tags` in this Markdown file.
* Apply <function mapping string to string>
  to all those links, transforming their text.

This API should probably be based on the parsed syntax tree.

Obviously, I have some APIs to inspect and modify file contents,
but they are line/character oriented,
which makes them too low-level
–expressing intent is difficult.

### Zettelkasten improvements

The functionality to manipulate my Zettelkasten can be refined further.

For example, I'd like to be able to add more formal annotations to my notes and…
use those annotations as I'm editing files.
Essentially, allow me to define stronger semantics in my notes,
deriving ontologies.

### CSV integration

I'd like to make it easy to use Edge to load a *medium*
(~1e5-rows range)
CSV file and interactively manipulate it.
Edge has CSV support, but it's fairly rudimentary.

For example, to quickly express things like the following
(probably based on Edge's extension language):

* "What's the average of the 5th column
  across rows where the 2nd column is greater than the 1st column?"

* "Plot a histogram of the values in the 2nd column,
  weighing each entry by the result of multiplying the 4th and 5th columns."

* "What's the Pearson correlation coefficient between two columns?"

### Audio improvements

I want to find better ways to use audio as an additional information channel.

Edge has had audio support based on libao for many years.
However, it doesn't work very well:
interruptions or quitting sometimes leaves the audio device in a strange state.
As a result, I end up almost always muting it
(by adding `--mute` to `~/.edge/flags.txt`).

For example, when you search for a regular expression,
Edge plays a different tune based on the number of results
(simplifying, something like:
"error", "zero matches", "one match", "two matches", "many matches").

This would require making the audio support more robust.
Once that's in place, I intend to explore more ideas
about how to use sound.

## Conclusions

It's crazy: I've continued working on Edge
for ten years.
I couldn't have known how far I'd go,
how many features I would have implemented.

I think the main lesson I learned
is to **resist the urge to compromise on correctness**.
This is behind a lot of the lessons in this article,
such as:

* Digging deep in order to fix bug categories
  (rather than only specific manifestations)

* Making expectations and invariants explicit
  –through judicious use of strong types,
  runtime validations,
  and unit tests.

I've invested a lot of energy into developing Edge.
I'm glad I've done so.
Starting this effort and continuing to invest in it has been very satisfying.
I've learned a lot on this journey.

