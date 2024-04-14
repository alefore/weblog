# Edge: Lessons

* Introduction
* Fix bug categories, not instances
* Use Types Effectively
* Futures & Threading
* Make Things Explicit
* Bursts & Pauses
* Conclusions

## Introduction

This document is work-in-progress.
As of 2024-04-13, this is still incomplete
and probably contains errors.
I haven't yet had the time to fully review it.

This document is an attempt to capture lessons I've learned
working on Edge
for a decade.
Edge is a Linux C++ terminal-based text editor.
I started working on
Edge in 2014.
I've used it exclusively since 2015.
As of 2024-04-14, its implementation is 67.9k lines of C++
(per `wc -l $(find src -name '*.cc' -or -name '*.h' -or -name '*.y'`).

Many of the lessons described are fairly technical
but at least one (Bursts & Pauses) is general
—observations about the life-cycle of side-projects.
Among the technical lessons,
many are specific to C++,
but they probably apply, to some extent,
to other languages.

Edge has its own extension language,
which looks like C++ with memory management,
and has logic (such as syntax highlighting)
for editing C++, Markdown, Java, Python
and a few other file types.

I use Edge for programming at work
(though this days I program relatively little at work),
to maintain my
[Zettelkasten](https://github.com/alefore/weblog/blob/master/zettelkasten.md),
and… to develop Edge.

## Fix Bug Categories

If you identify a bug,
avoid the temptation to just fix it and move on;
instead, fix the underlying issue that enabled the bug to exist.
The specific instance of the bug should just happen to disappear,
as a consequence of the underlying fix.

Sometimes this is impossible;
software *will* have bugs.
But I've found many causes where digging deeper pays off.

How can it be that I've made the mistake that
allowed this bug to exist?
What can I change such that this entire category of bugs no longer happens,
can no longer happen?

At the bare minimum,
add a unit test that fails
before fixing the bug.
But this should not be a substitute
for trying to find more robust ways
to avoid the entire category of bug.

### Examples

The following examples show how this can be done through runtime checks.
A much preferable technique is to use types effectively;
however, that's covered [a separate section](#use-types-effectively).

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
where an expression was incorrectly computing a range;
in some corner cases
the *end* `LineColumn`
preceeded the *begin* `LineColumn`
in the computed range
([fix](https://github.com/alefore/edge/commit/750f660349c5fb8b36823d77d4a39062e9fa9634)).

I decided to add an explicit requirement to `Range`:
*end* must be greater than or equal to *begin*.
I adjusted the constructor and setters to validate this explicitly
([commit](https://github.com/alefore/edge/commit/279f001740d0c1ed8fa4e29b7286d59f0d746922)).

A better solution would be to detect these issues at compile time,
but at least this approach means they won't be silently ignored
(allowing them to manifest in more subtle ways,
making debugging significantly more cumbersome).

It only took a few days for this to uncover
another instance of the same issue
([fix](https://github.com/alefore/edge/commit/14f8ed2655c09690cb86b3a7115b0efb51798398)).
Who knows for how long this bug would have gone undetected.

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
I decided to add extra validation to the `NonNull` constructors,
to ensure that a non-null input was... indeed non-null
([commit](https://github.com/alefore/edge/commit/6cb2b3e53db474849009404b751e46c6a1e3dc0b)).

It only took two days for this to find another such issue
that had been ignored silently
([commit](https://github.com/alefore/edge/commit/09586758eefa0e35d6721228aeaf3dc34b4350cc)).

## Use Types Effectively

Good use of expressive and well-designed static types
facilitates correctness and maintainability.

### Custom Types

I use semantically-rich custom types for variables and class fields
(and methods' inputs and outputs).
Many of these types are simple wrappers of primitive types
(`int`, `string`, `double`, etc.).

Primitive types are just the building blocks
with which custom types are implemented.
In general, I try to only use primitive types to define these custom types.

If a telephone number is an important concept for your application,
define a corresponding type;
don't just use `string` (or `int`).

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

Aim to convey this through types;
if you pass the wrong information
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

Using custom types enables validation of expectations
in the constructor;
the rest of the application
can readily conclude that these preconditions are met.

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
but you won't gain any static type-safety).

To define custom types, I currently use `GHOST_TYPE` expressions like these:

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

I'm not too happy with this approach, though.
I suspect a better approach may be
to do it based on template metaprogramming with an approach like this:

    struct Digits : public GhostType<Digits, std::vector<size_t>> {}

This would support significantly more customization
through template parameters for the `GhostType` class.
Specifically, I would like to make it easier to define validation functions;
with the current `GHOST_TYPE` macros this is difficult.

### Predicate Types

Defining wrapper types corresponding to predicates
is another technique I've found useful.

#### Definition

If you have a type `T` and a predicate `P`,
you would create a type `TP`
that contains an instance of `T`
that is known to match `P`.
I'll refer to `TP` as the *subtype*.

#### Rationale

This allows functions to explicitly state some of their preconditions
(*e.g.*, an input container must be sorted,
or non-empty,
or have exactly between 128 and 256 elements).
Instead of checking inputs against predicates
(or, alternatively, transforming inputs to match the predicates;
*e.g.*, sorting a container received),
functions should receive their inputs using the subtypes directly.

Judicious use of predicate types tends to:

* Ensure that all cases are handled.
  If a function receives (or returns) the general type `T`,
  it signals explicitly that its implementations
  (or customers receiving its output)
  must handle the case
  where the inputs don't match `P`
  (or else the function would have used `TP`).
  This helps ensure correctness;
  the benefits are already visible in a medium-sized code base.

* Simplify validation.
  Validation (and/or corresponding conversion to the subtype)
  tends to happen in just a few places;
  this often renders irrelevant
  a lot of assertion checks
  allow them to be safely removed.

  I've even found cases where this allowed me delete code
  that was handling situations
  that could never occur:
  all callers were always passing non-null pointers,
  but the function still contained code to handle the null case.
  Here I use the type system to ensure that these cases can't occur
  (and that I won't introduce them accidentally in the future),
  and delete the corresponding code.

#### Examples

##### SortedLineSequence

A good example of using types to represent explicitly
that an instance of a more general type matches a predicate:
`SortedLineSequence` and `SortedLineSequenceUniqueLines`.

We start with the general type, `LineSequence`.
A `LineSequence` instance holds an immutable series of lines
([implementation](https://github.com/alefore/edge/blob/master/src/language/text/line_sequence.h)).

Some modules receive a "dictionary":
a `LineSequence`, with one line for each token ("word"),
sorted in ascending order (by some comparison predicate).
For example:

* Autocomplete functionality.
* Logic that highlights typos.

The reason the `LineSequence` needs to be sorted
is to be able to run binary searches given a prefix.

One could argue that these modules should receive a `std::set`
or some other similar type;
such a discussion would be entirely orthogonal
to the point of this example.
We uses `LineSequence` because that's the structure used by Edge
to represent file contents.
This allows us to reuse Edge's file-reading logic
to read dictionaries from the file system.

How can we ensure that these modules are always given sorted sequences?

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
(which the user used to manage their dictionary)?

You can detect that the file isn't sorted and display errors.
Slightly better: at least the user will be aware that something is wrong.
But not terribly useful.

You can sort the contents in memory just after the file is read.
Sorting is ~fast
and can be done by a background thread
(possibly delaying autocomplete and similar operations
for a few milliseconds after start).

This is significantly better,
but you still need to either:

1. Validate "manually" validate that all customers of these modules
   are passing sorted contents.
   This is brittle.
   If you add a new customer and neglect to sort,
   nothing will alert you.

2. Change these modules to sort the contents as they receive them.
   This can be inefficient depending on how often this happens,
   and it doesn't entirely address the problem
   (what if some entry points in these modules neglects to sort?).

The solution is the use of `SortedLineSequence`,
a class that simply holds a `LineSequence`
where the contents are known to be sorted
([implementation](https://github.com/alefore/edge/blob/master/src/language/text/sorted_line_sequence.h)).
This is done in all its constructors:
they directly sort the contents given.
This means that it is impossible to hold a `SortedLineSequence`
where the contents aren't sorted.

Once we've done that,
we simply upgrade our autocomplete and similar modules
to require a `SortedLineSequence` input
where they previously received a `LineSequence`.

We've made it impossible for customers of these modules
to accidentally pass unsorted contents.
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
moving the boilerplate from the common case to the exceptional case.

##### LineRange

TODO.

##### OnceOnlyFunction

In many situations I pass around callables that should only be called once.
I could have simply used `std::function`
but making this explicit helps me make sure
I can safely move values around during execution
and detect reliably (though at runtime)
if I ever accidentally run the same callable twice.

### Constness

The benefits of making types immutable
often outweigh the costs.

Every type must define `const` semantics
that make it thread-compatible.
When you define a type
you're actually defining two:
the mutable `Type`
and the immutable and thread-compatible `const Type`.

When the object is exposed to multiple threads,
only `const` access should be retained.
This is sometimes enough to avoid data races,
reducing the challenge of writing concurrent code to simply
managing object lifetimes.

TODO: Move the following paragraph.

A pattern that deserves mention is balanced trees of immutable objects.
This is very specific in comparison with the previous patterns,
but this structure is so versatile that it deserves its own mention.
Mutation operations don't mutate the tree,
they return new trees with the operation applied.
The new and old trees share as much of the branches as possible.
This allows very efficient implementation of most operations.
Inserting, erasing or retriving an element at an arbitrary location
all have logarithmic runtime complexity.

#### Immutable Assignable

I often use types where all methods are `const` except for two:
the move constructor and the assignment operator.
Objects of such types aren't strictly immutable, but close.

Deeply-immutable objects can be too cumbersome:
without assignment support,
customers are very frequently forced to wrap instances
inside `NonNull<std::unique_ptr<>>`
or `NonNull<std::shared_ptr<>>`.

This is avoided by these mostly-immutable objects,
which behave as assignable pointers to a nested deeply-immutable object.
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
also allows trivial copies of the const values.

##### Prefer Immutable Types: Support assignment: Examples

The following are examples of immutable classes in Edge:

* [`LineSequence`](https://github.com/alefore/edge/blob/master/src/language/text/line_sequence.h)
* [`Line`](https://github.com/alefore/edge/blob/master/src/language/text/line.h)
* [`ConstTree`](https://github.com/alefore/edge/blob/master/src/language/const_tree.h)

#### Builder Pattern

I've found a "builder" pattern fairly versatile.
I use mutable thread-compatible builder instances
to *build* the immutable
(and thus thread-safe; and thus widely shared) instances.

To avoid cycles, the output type should not depend on the builder.
The output object should declare its constructors private,
which means it should give the builder `friend` access.

Most builder instances are used to build a single output.
As a performance optimization,
I avoid any deep copies inside `Build`.
I do this by making the `Build` method require `*this`
to be an rvalue reference (`&&`)
—so that data can be moved out of the builder—
and implementing an explicit `Copy` method
for the few customers that need it, e.g.:

    class LineBuilder { ...
      Line Build() &&;
      LineBuilder Copy() const;
    };

Unfortunately, this can be a bit cumbersome
(because of the additional `std::move` calls for some customers).

#### Prefer Immutable Types: Examples

* Line and LineBuilder

* LineSequence and LineSequenceBuilder

* LazyString (and LazyStringImpl)

### Thread safety

My `concurrent::Protected<Data>` template has worked very well
for classes that need to be thread-safe:
[protected.h](https://github.com/alefore/edge/blob/master/src/concurrent/protected.h)

Using types effectively can boost thread-safety significantly.
Before I introduced `concurrent::Protected`,
Edge only had a relativley small amount of parallelization.
I was wory of the insidious woes brought upon by multi-threading
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

The amount of concurrency in Edge has only gone up since then.
I'm defering a lot more work to background threads.
I don't think I'd have been able to get this far
if I hand't relied on types to maintain correctness.

There are other solutions to this problem, such as
[Abseil's lock annotations](https://abseil.io/docs/cpp/guides/synchronization#thread-annotations)
that probably work about as well, perhaps even better.

## Futures & Threading

I've had great success using a Futures API
to implement asynchronous requirements.

Futures-based code is not as clean as synchronous code, but close.
It is much cleaner than callback spaghetti:
code still generally reflects that a function is creating and returning a value,
and its caller is consuming the returned value.

TODO: Error handling in futures.

### Single consumer

My futures implementation supports setting only one consumer per future.
The future's value, once available, is given as a value to the consumer.
This allows me to use futures of moveable non-copyable types.

For situations where multiple listeners are desirable,
I can turn a regular future into a `ListenableFuture`
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
(for Edge that would be the `OpenBuffer` class)
to still delegate work to thread pools.
All they need to do is ensure that they only capture thread-safe values
(perhaps thread-safe snapshots of their class variables)
in the lambdas they pass to the thread pools.
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

In some cases it is important to cancel asynchronous computations
that have become irrelevant.
The pattern I've landed on is the use of a `DeleteNotification` class
that contains a `futures::ListenableValue<EmptyValue>`.

A producer that wants to support cancellation simply receives the
`futures::ListenableValue<EmptyValue>` abort notification.
The producer can either poll on this future or add a listener callback.
The customer of the producer simply creates a `DeleteNotification`,
passes its corresponding future to the producer,
and ensures that the `DeleteNotification` is retained
as long as the value is still relevant.

For example, as the user types in a prompt
(e.g., open a file, search for a regexp)
I kick off background work
(to show a preview of files or count of matches
based on what the user has already entered).
If the user types a second character before this work completes,
I just replace the `DeleteNotification` instance with a newly built instance
(which signals to the background thread that it has become irrelevant
and should just abandon whatever results it has produced)
and kick off a new operation.

### Futures in Const Objects

You can store `const` views of `ListenableFutures` inside `const` structures.
The producer will eventually give them a value, triggering consumers' execution
When parts of an object are computed asynchronously,
this allows customers to access each such part as soon as it is ready.

At first, I found this pattern somewhat counterintuitive:
It feels strange to tag as `const`
an object whose parts are still being constructed.
But I think it makes sense.
The contract of the future never changes:
customers can add callbacks that the future will run
when the value arrives (or immediately).
All callbacks will receive exactly the same value, when they run.

The alternative would be to only return the object once all its parts are ready
(i.e., rather than building an object
containing futures for its asynchronous parts,
build a future of the fully-built object).
But this would unnecessarily delay things.

#### Examples

A good example where I've used this pattern
is the `LineMetadataEntry` class
([source](https://github.com/alefore/edge/blob/master/src/language/text/line.h)).
When loading lines in a file,
Edge will attempt to compile them (to its C-like extension language)
and, if they compile to expressions free of side-effects,
evaluate them and display the evaluation results.

For example,
Edge will display "7.60416"
next to a line that contains just "365 / (52 - 4)".

Because such evaluation could take a while
(depending on the complexity of the expression),
Edge feeds these evaluations to a thread pool,
abiding by its principle of avoiding blocking the main thread.

The implementation is very natural:
due to its nested `LineMetadataEntry`,
`Line` contains:

* A future with the string representation of the evaluation result.
* The initial text that should be displayed
  if the future doesn't yet have a result.

## Make Things Explicit

Early in my career,
I made the mistake of optimizing my implementations for brevity.
If you can you say it with fewer words, why more?

> Je n’ai fait celle-ci plus longue que parce que je n’ai pas eu leadership loisir de la faire plus courte.

But simplicity and brevity are different things.
In fact, in software you often reach a point
where extra brevity complicates things.

One way I think about it is:
the implementation should reflect your thought process explicitly;
rather than only its conclusions.
Stripping away the thought process makes the software less malleable.

It bears saying it explicitly:
just because some expression is shorter (by whatever metric)
doesn't mean it's simpler.

This has obvious non-controversial implications:

* Define appropriately named constants;
  rather than using magic values directly.

* Add `const` annotations to relevant class methods.

* Use appropriate names that convey meaning; schew abbreviations.

It also has some less-obvious implications.

### Loops

Most operations on containers (either sets, sequences, maps, etc.)
can be expressed as one of a few canonical operations,
such as finding elements matching a predicate,
transforming elements,
or aggregating elements.

In those cases, I avoid writing explicit `for` or `while` loops.
I prefer standard functions (such as `std::views::transform` and related logic)
for manipulating and aggregating containers.
I'm a big fan of the recent ranges/views APIs.

Reducing my loops to canonical operations
helps me make the intent more evident:

* I am just checking if a list has at least an element matching a predicate.
* I am just copying all elements, with some transformation applied.
* I am mutating the container directly.

Seeing that I'm calling `std::views::filter | std::views::transform`
makes it immediately obvious,
even more obvious than simple range-based loops.

Consistently avoiding explicit loops has the advantage that
in those cases that can't be neatly reduced to a canonical operation,
the presence of an explicit `for` or `while` loop alerts you:
there's something unusual in this block.

### Avoid auto

TODO: Flesh out.

## Bursts & Pauses

### Bursts

My engagement with Edge is extremely bursty.
Most progress happens in bursts of activity
that happen about once per year
and usually last from two to five months,
during which I commit many changes per day.
After the bursts come long pauses of inactivity.

#### New Possibilities Open

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
  suddenly ... fall gracefully into place, effortlessly.
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
Edge's GC implementation was as efficient as I could reasonably expect
(without rewriting it drastically and making it significantly more complex),

And then, ten months later,
between 2023-09 and 2023-10,
I optimized collection
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
   Not only is work done by multiple threads,
   but the main thread now offloads some operations
   (e.g., actual deletion of objects)
   to a dedicated thread-pool entirely
   (and doesn't need to wait for their completion).

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
and made usage less error-prone
by removing the burden of calling `Ptr::Protect` from its customers.

The core implementation
([gc.h](https://github.com/alefore/edge/blob/master/src/language/gc.h),
[bag.h](https://github.com/alefore/edge/blob/master/src/concurrent/bag.h) and
[gc.cc](https://github.com/alefore/edge/blob/master/src/language/gc.cc))
dind't become significantly more complex:
LOC only grew by 3%.

* From 2022-12-24 to 2023-08-29 these files didn't change;
  they contained 1163
  (519 + 214 + 430, respectively) LOC

* In 2023-10-27,
  After all these optimizations,
  they grew by a net delta of +35 LOC,
  to 1198
  (572 + 129 + 497, respectively) LOC.

In this specific case, LOC is a reasonable proxy
for the level of complexity of the implementation.
To compute LOC, I excluded unit tests and included header documentation.

#### Ramp Down

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
   I've ran out of easy things to implement.
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
Perhaps I switch to write a short story,
or
[Zuri Shorts](https://github.com/alefore/weblog/blob/master/zurich-shorts.md).
Or learn to play the harmonica.
Or knit.
I go on a long trip.
I continue *using* Edge, but I rarely improve it.

This lasts six months on average, sometimes significantly longer.
Until suddenly I can't resist the itch
to finally fix that annoying behavior
or implement that great idea that found me
and I end getting sucked back in.
A new cycle begins.

#### Background Thinking

What triggers the periods of intense focus?
What enables the fast progress on areas I previously considered out of reach?

     ┏━━━━━━━━━━━━━━┓        ┏━━━━━━━━━━━━┓        ┏━━━━━━━━━━━━━━━━┓
     ┃  Unrelated   ┠──(+g)──> Background ┠─(+h⏳)─> Transformative ┃
     ┃ Distractions ┃        ┃  thinking  ┃        ┃      ideas     ┃
     ┗━━━━━━━━━━━━━━┛        ┗━━━━━━━━━━━━┛        ┗━━━━━━━━━━━━━━━━┛

During the pauses
my brain keeps making connections and generating and encountering ideas.
This can be alternative approaches I could try:
new underlying APIs,
different data structures,
entirely new user journeys,
etc..

This happens mostly subconsciously.
I'm not be working on Edge actively,
but I'm still passively thinking about these problems:
the places where I got stuck
and couldn't figure out a better solution;
the things that annoy me as I use Edge,
even if I don't notice them consciously.
I still read articles related with text editors
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
They (often) work,
and they (often) have far reaching implications,
and a new burst begins.

### Emotional State

My emotional state factors strongly into the timing of the cycles.
The timing of bursts is strongly associated
with important events happening in my life:

* Some bursts started when I was going through difficult personal situations
  (*e.g.*, breaking up from a serious relationships)
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
  to 17 at the beginning of 2022.
  Most of my time (at work)
  became occupied by managerial tasks,
  pushing out technical work.
  I interpreted the amount of energy I was dedicating to Edge
  as a strong sign that I was missing technical work.
  Interestingly, the burst stoped shortly after I made the decision
  and started searching for someone
  I could transfer my management responsibilities to.

### Unrelated distractions

I think the complete system diagram would be this:

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

This could be reduced to
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
  because they enables the *Transformative ideas* to manifest
  (relationships (g), (h), (a)).

### Implications

What are the main implications?

#### Acceptance

I have to understand and accept when a burst is coming to its conclusion.
Some things seem implausibly difficult today
but they may become trivial in the future.
I find this very encouraging.

#### Document Potential Improvements

Documenting my ideas for potential improvements
helps me by making it easier to find appropriately sized chunks of work,
depending on where I am in the burst-pause cycle.

I leave many "TODO" notes in the code
listing things that I think I may want to improve some day.
I try too:

* Classify them by difficulty (trivial, easy, normal)
* Give them hash tags (to help me find related changes)

As I'm reading the code working on implementing a specific feature
unrelated ideas come up.
I'm not going to work on them right away;
I'd never accomplish the task in working on.
But I'll make sure to document them right away.

As of 2023-12-21, I have 220 TODO notes in the code.
16 are marked as trivial, 80 are marked as easy.

#### Incremental Improvement

It's easy to underestimate the effect of small changes.
Changes that, on their own, don't amount to much,
can have very drastic effects as a group.
I think this is the self-reinforcing loop
of Progress (in one area) enables Progress (in others).
This is a big part of why these bursts happen:
they start as just a few small improvements,
but these small improvements precipitate
and enable many other small improvements and...
eventually they add up to large changes.

The majority of my commits are small "no-op" changes whose only purpose
is "preparation" to make follow-up changes easier or smaller.
A big change is divided into a million of baby steps
and one final simple step that enables the new functionality.
I think this is what enables
things that previously seemed ~impossible
to "suddenly ... fall gracefully into place, effortlessly".

## Conclusions

TODO.

