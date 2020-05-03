# Software Correctness

Software is correct to the extent that its implementation fulfils its contract
and it's free of software defects (bugs).

## Common and Special Errors

We can distinguish errors of two types:

* Common Errors
* Special Errors

### Common Errors

Common errors (called "chance causes") are those corresponding to the expected
variation in a system. While we should generally aim to minimze their
occurrence, specific instances are to be expected and shouldn't trigger any
corrective interventions into the system.

In fact, for these errors, intervention (to correct a single instance) tends to
be harmful as it may introduce additional risk and tends to destabilize the
system.

### Special Errors

Special errors correspond to unexpected new situations that call for direct
analysis and intervention into the system.

These errors are often said to have "assignable causes". In Shewhart analysis,
they are attributed to "a low-level worker". The indicate that there is
something unexpected that "we can point the finger at".

### Why Distinguish Common and Special Errors

The distinction of errors into common and special errors is useful because the
two different types deserve very different treatment.

### History

The distinction of errors into common and special types has been made by
matematicians and statisticians for a very long time.

In modern times, William Edwards Deming championed this distinction.

#### William Edwards Deming

William Edwards Deming was an American engineer, statistician, professor,
author, lecturer, and management consultant.

## Catalogue of Software Errors

This is a catalogue of software errors, including both types of software bugs as
well as runtime/operational conditions that may cause software to malfunction.

This section is work in progress. It's still wildly incomplete.

### Software Errors: Threading Issues

Threading errors include:

* Shared mutable state
* Priority inversion

#### High Contention

High contention happens on systems with a large number of threads running in
multi-core machines when multiple threads compete to acquire the same mutex.

One technique that can be useful to avoid contention is the use of reader locks,
allowing multiple "reader" threads to make progress concurrently.

Strictly speaking, high contention is more performance issue than a software
bug.

##### Effects of High Contention

Contention serializes execution. In the extreme case, several threads are
constantly waiting on the mutex. This can drastically impact the latency and
throughput of large systems, as many requests are blocked waiting for the hot
mutex.

Additionally, many mutex implementations apply optimizations that may backfire
when there's high contention. For example, spin-lock based implementations will
waste a large amount of CPU. Even hybrid approaches (such as Abseil) will still
end up wasting significant CPU, while doing less actual work.

#### Race conditions

Race conditions are errors caused by multiple threads that interfere with one
another. This happens often because reasoning about code that executes under
multiple threads is significantly more difficult than reasoning about single
threaded code.

They typically happen because of multiple threads (possibly running in different
machines) that attempt to perform sequences of operations on a shared object
that changes state because of these operations.

##### Race Conditions: Example: Counter

A good example of these conditions is a server that maintains a counter, which
can be read or set. If a client does read-modify-write operations (read the
counter, increment it by one, write the incremented counter), even if both the
client and the server are correct, a combination of two clients will start
losing increments.

#### Deadlocks

Deadlocks are a type of software error where careless use of locks results in a
situation where two or more operations become blocked, waiting for other
operations to complete.

The two most common cases of this are:

* Careless acquision of mutex locks
* Executor exhaustion

##### Careless acquision of mutex locks

When critical sections protected by different mutexes overlap, it's possible for
two threads to each acquire one of the mutex instances and then block trying to
acquire the other.


###### Ordering Mutex

The solution to deadlocks is to define a complete order for any set of
mutex instances that overlap (i.e., any two mutexes that a thread may attempt to
acquire concurrently). We then make threads always follow this order when they
acquire mutexes.

This isn't always easy to do; it may cause significant complexity to a class.

##### caused by callbacks

A judicious programmer will isolate mutexes within a single class (i.e., never
expose them beyond the small class that they protect) and, in those rare cases
where a single instance of the class needs more than a single mutex, ensure that
they are always acquired in a given order. However, one particularly nasty
source of deadlocks remains: when classes execute callbacks that their customers
provided while holding locks. In this case, the customer callback may trigger
some action that may call back into the original class and cause it to attempt
to reacquire the lock.

We've seen this happen in practice in our team at Google. To avoid this, our
coding principle mandates to avoid running customer-provided callbacks while
holding an internal lock; in the rare cases where this is necessary, the
contract must be very explicit about the semantics of which methods of a given
class can be called by callbacks that the class receives.

We try hard to avoid this (even if it costs us additional performance) because
we see it as a very poor form of interface design (where an internal detail, the
use of a mutex, leaks to its customers).

One technique to avoid this, in some cases, is to schedule the execution of the
callbacks asynchronously, in a separate executor. This may be feasible in cases
where the function that wants to execute the callback is itself called under a
lock (so that the function can't just unlock and execute the callback).

##### Executor exhaustion

In executors (e.g., thread pools) that have a fixed size, it may happen that all
slots become occupied (i.e, all threads become busy) but no progress can be made
because all threads may block waiting for completion of scheduled work that
hasn't started (and that won't start, because the executor is exhausted).

To avoid this, one must either use dynamically growing (unbounded) executors,
or use libraries that ensure that threads never block waiting for other threads
(in other words, threads that would block will instead adopt and execute
scheduled work).

###### Generic Completion Handler

At Google, we had to add custom logic to one of the libraries that my team owns
that has its custom implementation of settable futures, to ensure that executor
exhaustion doesn't happen: a thread that wants to block until a future receives
its value will potentially execute pending work that is needed in order for the
future to receive its value.

Before we implemented this, we received a few (very occasional!) reports of
deadlocks in our system. We were already aware of the potential for these
deadlocks to happen, but we thought we had eliminated the possible causes. This
was particularly tricky because we had to look very closely at a few of our
classes and reason about all the ways in which they could trigger executor
exhaustion.

#### Thread Safety Analysis (C++)

[Thread Safety Analysis](https://clang.llvm.org/docs/ThreadSafetyAnalysis.html)
can be invaluable to avoid thread safety issues.

In our team, it is standard practice that every field in a class that aims to be
thread-compatible must be either:

* Declared as `const` and thread-compatible.
* Thread-safe.
* Protected by a mutex and use the `GUARDED_BY` annotation.

This has been very helpful for the development of large multi threaded
libraries.

### Software Errors: Arithmetic errors

Many arithmetic errors happen because programmers tend to incorrectly assume
that number types represent "perfect" numbers, with no precision limits. In
practice, the vast majority of numbers are represented using types with very
significant limitations (in terms of expressivity at corner cases), such as
32-bit signed or unsigned integers or floating-point numbers. This is a very
good trade-off for performance (it would be prohibitively expensive to use
arbitrary precision numbers in most situations), but can be a source of errors.

#### Numerical Overflow

Numberical overflow happens when arithmetic operations produce values that are
too large or too small (i.e., too far away from zero) for what the underlying
types can represent.

Unfortunately, many implementations just ignore these errors silently, violating
our principle of detecting errors early.

#### Boundary Conditions

Incorrect boundary conditions are a common source of errors. Two subtypes are
"off by one" errors, where a programmer incorrectly uses a "greater than"
comparison instead of a "greater than or equal to" comparison (or vice versa).

This tends to happen because, similar to handling corner cases, careful
inspection of the boundary conditions tends to be difficult and is often not
given as much attention as it deserves.

#### Loss of Precision

This error happens when the underlying number types can't represent numbers with
adequate enough precision.

For example, in many languages, `0.3 == 0.1 * 3` evaluates to false.

#### Unsigned Types

Use of unsigned types is a source of common errors.

This probably happens because unsigned types are often silently promoted to
signed types. For example, the difference of two unsigned values is a signed
value (i.e., the subtrahend may be larger than the minuend), but, due to its
context, it may often be implicitly converted to an unsigned value.

An example of this would be:

    for(size_t i = my_vector.size(); i >= 0; --i) ...

This loop would never stop: an unsigned number is always greater than or equal
to 0.

To avoid this, programmers should:

* Avoid unsigned types.

* When subtracting two numbers and storing the result in an unsigned type, make
  sure to always confirm that the subtrahend is less than or equal than the
  minuend, which can be quite cumbersome.

##### Only Make Errors Undetectable

The temptation to use unsigned types is that one may incorrectly expect that
this makes invalid states impossible to represent (for variables that are
expected to never be negative). However, what ends up happening in practice is
that these errors still occur, but the use of unsigned types just makes it
impossible to detect them.

##### Gosling Quote about Unsigned Types

[According to Gosling](http://www.gotw.ca/publications/c_family_interview.htm):

> Quiz any C developer about unsigned, and pretty soon you discover that almost
> no C developers actually understand what goes on with unsigned, what unsigned
> arithmetic is. Things like that made C complex. The language part of Java is,
> I think, pretty simple.

##### Bjarne Stroustrup: Subscripts and sizes should be signed

In [Subscripts and sizes should be
signed](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1428r0.pdf),
Bjarne Stroustrup makes a very compelling case against the use of unsigned
integers even for subscripts and sizes.

##### Scott Meyers: Signed and Unsigned Types in Interfaces

In [Signed and Unsigned Types in
Interfaces](https://www.aristeia.com/Papers/C++ReportColumns/sep95.pdf), Scott
Meyers argues against the use of unsigned types:

> One problem is that unsigned types tend to decrease your ability to detect
> common programming errors. Another is that they often increase the likelihood
> that clients of your classes will use the classes incorrectly.

His conclusion:

> In many cases, it’s better to simply return an int directly [...]. If you do
> that, of course, you reduce the range of values you can return, because you
> must pay for a sign bit you don’t really need. In many cases, however, the
> increase in software robustness more than compensates for the reduced range of
> values.

### Software Errors: Algorithmic Errors

#### Runaway Algorithmic Complexity

Runaway algorithmic complexity occurs when algorithms are used that
have an inadequate runtime or memory complexity for the inputs that they are
given. Technically, it may be that the software *is* correct, but this won't
help: the software will still fail to produce any useful results.

When this happens, this is typically much worse than just some slightly bad
performance; rather, it tends to manifest in programs that seemingly run
forever. This is especially common when algorithms that have exponential
complexity are used.

##### Avoid n² Algorithmic Complexity

In practice, for the types of sets that we often manipulate, algorithms of
n² or worse runtime complexity tend to not be acceptable.

Of course, the specifics will depend on the context and this is a gross
simplification.

###### Why n² Algorithmic Complexity Breaks

Given that a variable that grows exponentially can only be in one of
two states, systems with n² runtime complexity (or worse) transition
very suddenly from a state where there's no palpable cause for alarm (the
threshold where things fail is very far from the current value) to a state where
they cease too work by a large margin (e.g., take disproportionately long to
complete).

Furthermore, most algorithms we encounter in practice (e.g., around containers)
tend to consume values of n that are large enough to make this very problematic.
For example, a container with only ten thousand elements will already require
10 million operations, which is often unacceptable.

####### Exponential Growth: Binary

When analyzing the effects of variables that experience exponential
growth, most of the time we can think of them as binary values that can
only be in one of these states:

* Zero (or "very low").
* Infinite (or "ridiculously high").

This is because the transition between the two happens very quickly, almost
instantaneously.

####### Once Inflection Point Arrives, we Underestimate Speed of Change

Once an inflection point arrives, people commonly underestimate the speed with
which change governed by power laws will occur.

Source: Paul Saffo, HBR: Six Rules for Effective Forecasting

#### Unhandled Corner Cases

Corner cases are a common source of software failures. Because, by definition,
they happen rarely, programmers will sometimes cut corners and not handle them.

This doesn't simply happen out of lazyness: there's a point of diminishing
returns where we must make a pragmatic decision not to handle every possible
corner case. When we make the decision inappropriately (for example, because
reality changes), these corner cases will appear. As software gains more users,
these will bring new demands and more stringent requirements. Perhaps they'll
apply the software in unanticipated situations. When this happens, it's likely
that maintainers of complex software will need to extend it, so that it can
continue to correctly meet the demands of its customers.

#### Null pointer dereference

Null pointer dereferencing happens when an attempt is made to read a pointer
that doesn't point to a valid object but, rather, to the null address. Because
the null address is guaranteed to be different than the address of any actual
object, this is indicative of errors: some piece of code expected an object but
didn't receive one.

Because null is often used to indicate the absence of an object (for code that
needs to handle two separate cases: when an object is present and when an object
is absent), mismatches between modules makes null pointer dereference errors
more common than one might expect.

##### Effects

In most modern language implementations (at least for Java, C, and C++) attempts
to read from the null address will raise a runtime error:

* In Java, this throws a NullPointerException.

* In C++ in Linux, the null pointer is mapped to the address 0; reading from
  this address causes a segmentation fault. I believe this happens simply
  because the first page (and possibly a few additional ones) are reserved and
  never used, precissely to detect this situation.

These runtime errors will abort the execution of the application (unless the
application deliberately consumes them).

##### Old Unix

In 2006 I worked in a project to port applications written in C and a few other
languages from old Unix implementations to Linux. Most of these were the
applications that run the datacenters and shops in the largest retail
corporation in the world.

One of the challenges we faced was that the old Unix implementations that we
were migrating from allowed null pointer dereferences, effectively ignoring
these logical errors. Instead, the first few pages of memory (I don't recall
how many) would just be filled with zeros.

Consider code such as this:

    char* value = strcpy(getenv("SETTING_FOO"));

In the old Unix systems, this would create a new char buffer of size 1 with the
0 character (i.e., `{'\0'}`), as the programmers expected. In Linux, this yields
a segmentation fault.

Another example of things that worked in the old Unix systems was the following,
where `bar` is a pointer to a `struct` with a pointer `foo` (possibly among
many other objects):

    Foo* foo = bar->foo;

In the old systems, `foo` is now a null pointer; in Linux ... segmentation
fault.

We had to adjust many expressions that, like this one, assumed that reading the
first page of memory was OK, it just returned zeros.

##### Crash

When a null pointer is dereferenced, raising a runtime error, as most modern
language implementations do, is the best behavior.

One could instead suggest that the first few pages should simply be readable and
mapped to a memory area filled with zeros. This would perhaps simplify a few
expressions by saving the programmer from the need to explicitly check against
null in some situations (see examples in Null pointer dereference: Old
Unix), but it would have the unfortunate effect of hiding actual
errors, where some code *requires* an actual object but doesn't receive it. This
goes against the principle of detecting errors early.

### Software Errors: Life Cycle Errors

#### Use After Free

Use-after-free is a type of software error where a memory position is
deallocated (i.e., returned to the application-level allocator or the operating
system for reuse) while structures in the application still reference it. When
the memory is reused (reached through those remaining references), it may have
already been reused, leading to data corruption.

This type of errors can be difficult to identify in cases where the patterns in
the application only reuse the deallocated memory very shortly after it is
deallocated and in a way where it is very unlikely to have been corrupted.

This error is only possible in language implementations that don't manage memory
automatically. In those, memory debuggers (such as valgrind, Bruce Peren's
[Electric Fence](https://en.wikipedia.org/wiki/Electric_Fence) library, or
Google's AddressSanitizer) may be very useful to track down these errors.

#### Read Before Initialized

This error occurs in languages (such as C++) that allow variables to be created
without being fully initialized. In this case, a programmer may declare a
variable and try to read it before it has been fully initialized.

Nowadays most compilers will detect these errors and warn programmers about
them. A judicious programmer would always initialize variables where they are
declared (or in class constructors, for class variables that can't be
initialized where they are declared because they depend in parameters to the
constructors).

#### Memory Leaks

Memory leaks are a type of memory problem where parts of the address space of a
given process become unreachable yet don't get released to the underlying
allocations manager (or operating system). This renders this memory
inaccessible, reducing the effective amount of memory that the process can use.

This can be divided in two subcases:

* Bounded leaks. In this case, the application leaks a constant amount of memory
  per process. This makes the application less efficient.

* Unbounded (or ongoing) leaks. In this case, the application continues to leak
  memory during its execution. This is significantly worse than the previous
  situation because the application will eventually run out of memory.

#### & Garbage Collection

Memory Leaks affect systems with garbage collection. In those systems, memory
leaks are less common than in systems that manage memory explicitly, but they
still occur. In systems with garbage collection, this happens when structures
maintain unnecessary references to large structures.

For example, a "substring" function may receive a very large string and return a
view over it. Depending on the implementation, the view may be a very small
string that references the original view. This may be a very valid design choice
for performance (since it avoids having to explicitly copy contents, thus making
the substring operation execute in constant time). However, this means that even
if the application relinquishes every other reference to the original string,
the garbage collector will still be unable to deallocate the very large portions
of the original string, even though they'll never be used.

#### File Descriptor Leaks

File descriptor leaks are similar to memory leaks but happen in a relatively
different manner. In this case, applications may not judiciously close file
descriptors, eventually filling up their file descriptor tables and becoming
unable to create new file descriptors (e.g., open new files or create or receive
new connections).

We've seen this happen during the development of gRPC or other low-level
interprocess communication libraries that leaked inactive connections in some
corner cases. Our standard monitoring systems will track the number of open file
descriptors per process and alert if this exceeds a threshold. Unfortunately,
this doesn't necessarily indicate a file descriptor leak.

#### Memory Fragmentation

Memory Fragmentation is a condition where the largest contiguous region that
a memory allocator can reserve is significantly smaller than the total amount of
memory available to the allocator.

Fragmentation is a result of the inherent randomness in the
allocation/deallocation process, where the application returns memory in random
fashion (from the perspective of the allocator).

Platforms use several strategies to deal with fragmentation, including executing
"compaction" procedures (where memory is moved, and references are updated) as
part of garbage collection (which has other effects beyond reducing
fragmentation).

In long-running systems, it may be important to factor in the effect of
fragmentation when estimating the memory capacity of a system. In some
distributed systems, periodically dumping the state to persistent storage and
restarting the process may be necessary.

##### Example: Authentication

I had to troubleshoot some issues related to fragmentation around 2012, where we
had a few localized outages with some authentication systems at Google: some C++
servers were ocassionally losing significant amounts of memory to fragmentation.
Exposing statistics about the state of the allocators helped us recognize the
problem. The solution at the time (I don't know what the current state is) was
to force those systems to dump state and restart (with some redundant servers
that could continue to serve, so no observable loss of availability during the
restart), and to ensure this happened periodically.

### Software Errors: Out of Capacity Errors

#### Stack Overflow

This error happens when a thread runs out of stack space. This typically
indicates a very long chain of possibly recursive calls in a language or set of
functions that don't support proper tail recursion.

This happens relatively sooner than one may naively assume, specially in highly
threaded systems, since each stack must be contiguous. There's a trade off
between the maximum number of threads that can be supported and the maximum size
that their stacks can grow to.

To avoid this, programmers must be careful when using recursion. One
particularly nasty case where we encountered this at Google was in a library
that creates "stacks" of delegating behaviors. Each behavior in our library
would just call a method of the next behavior in line. Even if we didn't have
recursion, the chain of delegations often got deep enough that this became a
problem. To avoid this, we inserted "break" points where the delegating calls
would schedule a callback (to call the next in line) in an executor, thus
allowing the stack to unwind.

#### Computer Out of Memory

This error occurs when a machine doesn't have enough physical memory to hold all
the pages that processes running on it need. Most of the time this will be
caused by a single "abusive" process (or small set of related processes) that
upset the usual balance.

Multi-tenant systems will often impose a limit on the amount of process each
user or "job" is allowed to consume.

This can happen because of:

* Life-cycle issues such as memory leaks, where an underlying issue causes a
  process to continue reserving more and more pages, unbounded.

* Runaway Algorithmic Complexity. This is relatively rare but it may happen for
  algorithms with super-linear memory complexity, where small changes in the
  input sizes may have a disproportionate effect on memory requirements.

* Temporary overload conditions. Perhaps the program simply needs slightly more
  memory than currently available for a brief period of time. This may happen
  because of overload (e.g., a server receives a spike of large requests).

##### Out of Memory: Effects

When a computer runs out of memory for its processes, it will typically start
aggressively flushing pages down to a disk cache (if available). since disks are
several orders of magnitude slower than main memory, this will tend to cause
very noticeable latency degradation, as processes get interrupted due to page
faults. The system may technically still make progress (and it may eventually
free enough memory and recover), but for practical purposes it may become
effectively unusable.

After a while (e.g., if the disk cache is full), the operating system will start
killing processes, typically starting with the abusive one.

#### Process Out of Memory Space

TODO.

#### Out of File Descriptors

This error happens when a process opens too many file descriptors and, unlike
the case of file descriptor leaks, these are all in use. This is a
problem because, like file descriptor leaks, it prevents the process from
opening new files and establishing new connections (either inbound or outbound).

I encountered this a few times in my career. In 2020-04, one of the systems
under my purview, a gateway for a push component, started alerting because
several of its tasks are regularly creating more than 50k file descriptors. I
strongly suspect this is not due to leaks but rather because some specific
configurations are asking the gateway to connect to way too many server tasks.

Unfortunately, in multi tenant systems (such as the aforementioned gateway), a
single problematic configuration can put the entire stability of the system at
risk (i.e., rarely are the well-behaved tenants isolated from the problems
caused by the abusive ones).

### Software Errors: Distributed Systems

#### Lack of Flow Control

In distributed systems, a client may initiate work in a server at a given rate
that doesn't observe the capacity of the server. For example, the client may
simply initiate new requests at a constant rate, regardless of the rate at which
they complete in the server.

This is bad because it makes overflow likely: more in-flight work may accumulate
than either side has capacity to process.

The solution to this is relatively simple: one should put limits in the client
processes in terms of active work that they are allowed to initiate. For
example, each client process may be allowed to initiate up to a thousand active
requests and, once that point is reached, only initiate new requests as existing
requests complete.

## Avoid Errors

### User Interfaces: Real-time Feedback

A good user interface provides real-time feedback in order to allow its users to
avoid errors.

#### Value of Autocomplete: Feedback about Correctness

Somewhat counterintuitively, the value of autocompletion
isn't only in saving the user the time required to type the token that is
autocompleted, but also in giving feedback to the user about the correctness of
the input ahead of time.

Said differently, an autocomplete system that takes as long to activate as it
takes for the user to enter the autocompleted token can still be useful.

#### Syntax Highlighting

One of the ways in which a text editor can help its users detect errors
early is through syntax highlighting.

#### Context in Real-time (User Interfaces)

User interfaces should analyze tokens as soon as they are entered, displaying
relevant context about them.

* Whether the user enters a path, the interface should validate whether it
  exists and provide visual queues.

* Whenever the user enters an identifier (e.g., a symbol name in software source
  code), the editor should provide a visual queue confirming that it is
  correctly defined.

##### User Interfaces: Why Display Context in Real-time

By displaying context in real-time, user interfaces allow users to
detect errors earlier and lower the friction of various
operations by enabling users to operate on higher-level semantic
abstractions.

### Formal Verification

Formal Verification systems are software systems that introduce formalism to
help us offer rigorous proofs about characteristics of other systems.

### Vanish Invalid States

Vanish invalid states from your vocabulary: make it impossible to represent
states that are semantically invalid.

This principle applies to object types in source code, configurations, file
formats, database schemas, networking protocols, etc..

#### Software Types

This principle states that you should choose the types you use when writing
software to dimish the number of semantically invalid states that they can
represent.

##### Use the Type System Effectively

Making good use of compile-type/static type checks can drastically
help development by avoiding errors.

###### Types vs Implementation

Using static type checks tends to be very useful because it divides a
piece of software into two separate models:

* The type declarations/signatures.
* The type definitions/implementations.

This is useful because the first, the declarations, tends to be a significantly
simpler model than second. This enables programmers to focus entirely on the
first model first and, once they are satisfied that it is correct, to use the
compiler to proove that the entire implementation is consistent.

Instead, in languages where this separation doesn't happen, inconsistencies in
the implementation will sometimes only be detected much later.

##### Domain Types

In general, prefer to define and use domain-specific "ghost" types that are
aliases (or simple extensions) of primitive types, rather than using these
primitive types directly; the ghost types will more readibly reflect the
semantics and may enable the compiler to detect type errors.

Tags: software

Questions:
* What's the best way to define these types in C++?

###### Interface: Vanish Invalid States

To Vanish Invalid States from Software Types, Domain Types
should deliberately constrain their interfaces to only permit semantically valid
operations.

For example, a type that contains a "state" representation that only has some
valid transitions should deliberately only permit valid transitions (ideally
failing at compilation time if an invalid transition is attempted).

###### Validate on Construction

When defining domain-specific types, apply as much runtime validation
as feasible during construction, thus ensuring that all successfully created
instances are valid (as long as there's no mutable state).

##### C++: Function Inputs: References Beat Pointers

In C++, functions that receive non-optional objects that they operate on should
prefer to receive them by reference rather than by pointer since that eliminates
the possibility of causing null pointer dereferences inside them.

### Unit Tests

You are only finished creating a software component when you have a set of tests
that can be executed automatically to reasonably validate its correctness.

#### Unit Testing beats REPL Testing

REPL testing has the advantage over Unit testing of having
very low friction: tests are executed and thrown away. Nevertheless, unit tests
are is significantly superior because they are automatically repeatable.

While it is beneficial to be able to use REPL testing during development, it
doesn't remove the need to accompany modules with an adequate set of tests.

##### REPL Testing

Some programming language implementations offer a "read, eval, print loop"
(REPL) interface, where the programmer may load his code and use it to evaluate
expressions. REPL testing refers to using this interface to evaluate expressions
in order to validate that code units behave as the programmer expects.

### Code Reviews: Avoid Errors

Code Reviews can be a very useful mechanism to avoid errors (and, beyond that,
drive up code quality in a team).

The general idea is that an engineer can levarge the experience of a more senior
(at least relative to the problem domain) engineer to identify and correct
errors before they are released. The expectation is that this helps identify
both low-level "implementation" errors as well as higher-level conceptual errors
that were not detected during the modeling phase.

## Detect Errors Early

Systems should be implemented using approaches that enable they users and
developers to detect errors as early as feasible.

### Detect Errors Before Context Disappears

The sooner that an error is detected, the more context will be
available around it. Whoever introduced the error will have more context about
the module that contains the error. In an organization, the team behind the
subcomponent is more likely to still have knwoledge about the subcomponent.

### Early Detection: Smaller Impact

The earlier errors are detected, the smaller their impact.

The error will have less time to affect the system, such as by corrupting data
or by spreading false assumptions to other layers (both of which may require
expensive clean up operations).

#### Hoare: Checking Errors in Production

C. A. R. Hoare, Turing award lecture in 1981:

> "A consequence of this principle is that every occurrence of every subscript
> of every subscripted variable was on every occasion checked at run time
> against both the upper and the lower declared bounds of the array. Many years
> later we asked our customers whether they wished us to provide an option to
> switch off these checks in the interests of efficiency on production runs.
> Unanimously, they urged us not to--they already knew how frequently subscript
> errors occur on production runs where failure to detect them could be
> disastrous. I note with fear and horror that even in 1980 language designers
> and users have not learned this lesson. In any respectable branch of
> engineering, failure to observe such elementary precautions would have long
> been against the law."

### Detect Errors in Mental Models

#### Design Docs: Avoid Errors

As a mechanism to articulate a model at the right level of detail, design docs
can be an invaluable tool for detecting and avoiding conceptual errors.

The effects of detecting errors during a design phase are large. Because they
allow the model to be expressed in a semi-formal manner (e.g., using pseudo
code) where irrelevant aspects can be omitted, it enables the author and
reviewers to focus directly in the most relevant parts of the model, eliminating
distractions. This allows errors or suboptimal decisions to be detected that
may otherwise go undetected for a very long time.

##### Operate at the Right Level of Abstraction

When you're implementing a model, make sure you operate at the right level of
abstraction, allowing you to analyze the main points of the model, without
getting too distracted by irrelevant details.

In the right level of abstraction, you can analyze the main parts of the model.
You can rely on lower-level abstractions to solve sub-parts of the model, but
they should be clearly separated into different lower-level layers.

#### Iteration: Detect Errors in Models Earlier

By allowing us to validate emphirically the mental models we have about a
reality for which we're building an information system (e.g., the
requirements for the system, the entities involved, the interactions between
those entities, etc.), iteration foments early detection of
errors.

### Detect Errors in Production

Applications should proactively monitoring their internal state in order to
detect errors as early as feasible.

#### Redundancy Detects Errors

Increasing the amount of redundant information can be a valid technique to
detect and avoid errors.

##### Longer Names

A user interface that receives commands may yield fewer operation errors by
using slightly longer command names than one that uses the shortest possible
sequences that unambiguously identifies each command.

#### Explicit Invariants

Software modules should make their internal and external invariants explicit,
in order to:

* Enable maintainability: make them easier to understand, extend and use.
* Ensure correctness.

##### Runtime Invariants Checks

In order to ensure its correctness, software should repeatedly and
explicitly validate at runtime the invariants that it can't prove with static
analysis (e.g., those that can't be validated by the type system), in
order to detect errors as early as possible.

###### Validate Invariants at the Boundaries

It can be specially useful to validate runtime invariants at the
boundaries connecting layers.

####### Why Validate at the Boundaries?

Since different modules tend to be modified independently, it pays to check
invariants at the connecting points, since that will help detect mismatches in
the expectations and contracts.

###### Validate Invariants when State Changes

In modules that maintain state, it can be particularly helpful to validate
runtime invariants immediately before and immediately after state is
modified.

#### Release Often

In order to detect errors early, software should be released to
production often. Put differently, the time after a code change has been made
before it reaches production should be minimized.

##### Automatic Canarying

It pays to automate the process of making new releases, including creating
meta-systems that monitor releases and alert the software owners if any
anomalies are detected.

The following are desirable characteristics of such a system:

* Releases only require human involvement when things go wrong.

* Isolate failures.

## Degrade Gracefully

Rather than fail catastrophically, systems should degrade gracefully.

> “By failing to prepare, you are preparing to fail.” –Benjamin Franklyn

You should accept that errors are unavoidable. No matter how much care engineers
put into their craft–no matter how much passion–, software *will* have bugs. By
accepting that bugs will happen and creating meta-systems to deal with them
you'll vastly improve the success of a software system.

At some point the cost of minimizing the occurrence of errors will grow so much
that it will be a significantly better investment to improve the processes
around a system such that, when errors unavoidably occur, they are handled
gracefully. By being deliberate about tolerating failures in subcomponents, a
system may increase its robustness.

### Quiet Dependence

We often don't notice the systems we depend on until they fail.

This makes graceful degradation particularly important.

### Make the Environment Resilient

The environment in which a system operates is part of the system. This includes
things like the social context in which it is built, released, used, and
maintained. Optimizing the environment to be more resilient towards failures in
the system (reduce the effect of errors) can be a great way to increase the
robustness of a system.

This is because there's a point after which the cost of avoiding common errors
grows very steeply.

### Isolate Components into Failure Domains

Isolating components into separate failure domains that are managed
independently can drive up the overall reliability of a distributed system by
decreasing the likelihood of single events that cause multiple components to
fail. Among these events are things such as data corruption (as long as it can
be isolated into a single region) or incorrect updates (as long as there's a
reasonable canarying procedures that detect failures early).

Unfortunately, some types of events remain against which it is impossible (read:
prohibitively expensive) to divide the system into failure domains. Among these
are software bugs (e.g., a new vulnerability is discovered in the software and
exploited against all domains) or some types of overload situations (e.g., a
surge in traffic to the system due to unexpected and sudden demand increase is
likely to overwhelm multiple subdomains).

#### Probability of Failure

If the probability of failure in components is p, it would be expected that the
probabilty of N components failing concurrently is pⁿ. However, in practice, the
probability of joint failure tends to be much higher because the probabilities
tend to not be statically independent: a single event may cause multiple
components to fail.

#### Canarying: Isolate failures

Canarying procedures should be designed such that, when things do go wrong, they
are isolated into a small subset of the system (and the release procedure
halts).

This is desirable in order to allow the system's owners to react to the issue at
their leisure, rather than having to wake up in the middle of the night. Of
course, you should accept that sometimes, depending on the nature of your
system, you *will* have emergencies, but you should aim to minimize their
occurence.

The main mechanism to achieve this is to use separate "development",
"canarying", and "production" stacks, and release gradually; only release to one
after you've successfully released to the previous. "Canarying" typically will
be a small but representative subset of the regular production deployments
(perhaps 5% of the production capacity).

One important reason to have a "development" instance that mimics production
somewhat closely (but doesn't really affect end users) is so that errors with
very bad effects (data corruption) will be isolated from the real system.

### Atomic Operations

A sequence of operations that have side-effects is implemented atomically when
at any point in time either none of the operations or all of the operations have
executed successfully.

### & Graceful Degradation

Atomic operations yields a form of graceful degradation: when some unusual
conditions causes one of the operations in the sequence to fail, the entire
sequence is aborted and yields no visible changes to the world. This is a very
strong form of robustness against errors, since it prevents them from corrupting
shared state.

#### Example: Overwrite a File

When a large set of changes have to be applied to a file, it may be a better
strategy to create an entirely independent copy of the file with the mutations
applied. Once the mutations have been successfully applied, the new file can be
atomically moved to the original location (overwriting the old file).

This is because for most systems it's preferable to leave the old file
unmodified than to only apply a subset of the operations, corrupting the file.
This approach ensures that the file won't be modified when errors occur, making
the system more robust against this type of errors:

* Out of disk errors: the system doesn't have enough disk space to store the
  modified file.

* Hardware failures: the system was shutdown while the operation was executing.

* Software bugs: an invariant violation is detected during the process of
  writing the file. This is specially likely in systems that execute complex
  logic as they generate the modified file.

This is typically how most text editors "save file" functionality works.

#### Example: Update a Directory

The same technique described in Example: Overwrite a File can be
applied for systems that have to periodically regenerate an entire directory.
Those systems could generate the entire directory in a new location and only (1)
update a symbolic link, and (2) delete the old directory when they have
succeeded. Failures during the generation process (i.e., before the symbolic
link has been updated) are invisible.

#### Partial Atomic

Even though this isn't fully atomic, it may be convenient to separate operations
in a sequence into two phases:

* A preparation phase where the majority of the logic runs but the results
  aren't yet "committed". This phase is encouraged to attempt any parts of the
  operation that are likely to fail. That means mainly two things:

  * Run any complex logic that may trigger invariant violations in the data
    being produced.

  * Run validations on the state of the world, such that operational problems
    are likely to be detected here.

* A "commit" state that communicates the effect of the operation with the
  world.

A sequence of operations that abides by this principle will first execute the
preparation phase of all operations and, only if they have all succeeded, start
the commit phase.

Such a sequence won't be atomic: as the sub-operations are committed, some of
their results, but not others, may be observable; a failure half-way will
corrupt the state. Besides, this may still allow (just discourage) the software
to abort during the commit phase.

However, this is still valuable because it drastically decreases the odds of
failure during the commit phase: it makes it likely for most failures to occur
during the preparation phase.

#### Atomic Read-Modify-Write

Read-Modify-Write are operations that first read some data (from a database,
a file, a distributed system), then apply a logical change, and then write the
resulting data back to storage.

In order for these operations to be atomic, the Write operation must atomically
confirm that the data hasn't been modified (by a racing operation) since it was
read (and, obviously, the underlying storage system must allow the write to be
atomic). Several caching systems or distributed file systems allow this.

To implement this, it's enough if:

* Along with the data, the Read operation returns an opaque identifier which
  changes whenever the data is mutated. Some systems use a hash over the entire
  data; others use a version counter or the timestamp of the last write.

* The Write operation allows the user to pass back the opaque identifier, with
  the semantics that the write must only succeed if the identifier is still
  current. When such a write fails (i.e., the data has changed since it was
  read), the Read-Modify-Write operation is restarted.

### Recover And Alert

If a system detects an error (e.g., an internal invariant doesn't hold) it
should recover if this is feasible, as long as it can reasonably well establish
that this is unlikely to lead to larger issues in the future.

Larger issues include things like:

* Triggering cascading failures in other systems.
* Corrupting data.
* Serving the wrong data, potentially with privacy violations.

In these situations, special care must be taken to alert the system owners about
the condition, rather than allowing it to go unnoticed. This implies the need
for an alerting or logging system.

### Crashing May be Best

Sometimes crashing is the most desirable behavior for a system that detects an
internal error. This occurs in the following situations:

* Some corruption in memory has been detected that could cause corruption of
  persistent data.

* An invariant has been violated that may put the security or privacy of the
  system's users at risk. In this case, complete unavailability may be the best
  option.

* The error detected is very likely to just cause a crash further down the
  execution of the program. In this case, it's best to crash as soon as
  possible, to help the user identify the root cause.

### Tolerate Overload

For shared systems, a very important characteristic is to handle overload
adequately: a system provisioned to process traffic at a given rate should
continue to process traffic at that rate even when it receives traffic at a
greater rate.

#### Reject Excess

Because capacity can't be infinite, we must accept that there's a limit (to the
traffic rate that a system can process, typically a function of its available
resources) above which we will have to serve errors.

A robust system continues to serve the nominal rate with adequate latency and
explicitly rejects additional traffic. A naive system attempts to accept all
traffic to avoid serving errors and, as a result, fails catastrophically.

#### Avoid Cascading Failures

When engineering a system, take special care to make it possible for the system
to survive failure of specific subcomponents.

##### Babysitter Process

To decrease the impact of programming errors causing memory corruption, it often
pays to wrap the majority of the complexity of an application with a simple
"baby sitter" parent process whose only job is to (1) launch the main process
for the application, and (2) react to crashes in the main process by
restartarting the application (ideally in the last "known good" state).

This is especially useful for server-type processes.

## Engineers: Reacting to Errors

### Observability Tools

It pays off to invest in observability tools, that make it easy to inspect the
state of the system and to understand why an unexpected behavior occurs. The
goal is to increase the number of questions we can ask of the behavior of the
system.


Of course, these techniques are only useful to the extent that good systems are
available to analyze the information they provide. A giant log is useless if we
lack mechanisms to analyze it (to answer questions from it) at the correct
semantic level.

#### Logic to dump state

Make it easy (i.e., low friction) to expose the state of a system, with the
right level of details.

This can include interactive interfaces where the administrator can dig down to
the state of specific subsystems under investigation.

##### Example

As an example of logic to dump state, we've developed a simple
"debuginfo" library in my team at Google that allows us to annotate classes with
a simple method to expose through an HTTP interface the values of their fields,
traversing complex type hierarchies recursively. This library also keeps track
of anomalies (e.g., some condition occurred that a given class identified as
potentially problematic) and helps surface them.

When a system behaves unexpectedly, we browse that interface, which often
enables us to quickly validate hypothesis about the state of instances of our
servers.

A key aspect here is that adding debuginfo support to a new class is trivial. If
it was even slightly difficult, it's likely that significantly fewer classes in
our systems would have adopted this library.

#### Monitoring

For distributed systems, adequate monitoring tools is a crucial observability
tool. For these systems, it pays tremendously to have separate systems
constantly monitoring operational parameters. These would keep track of both:

* General parameters, such as memory or CPU used over time, number of threads by
  state, etc..

* Application specific parameters, tracking operations or state in the semantics
  of the business logic.

#### Execution Logs

Generating good logs during the regular execution of a program can be very
useful as an observability tool, especially for errors that are difficult to
reproduce.

### Debugging

#### Reproducing

When debugging an error it may be very useful to invest first in making the
failure easy to reproduce. That will enable a faster iteration where the
engineer validates their hypotheses.

#### Bisect the Problem

When trying to find the root cause of a problem, formulate a hypothesis that
divides and simplifies the problem, and attack that hypothesis. A good
hypothesis is one that balances these two criteria:

* Is easy to validate (so as to make progress fast).

* Is expected to have a probability (of being true) close to 50% (so as to
  maximize the amount of uncertainty eliminated by its validation).

Iterate doing this until you zoom in on the root cause.

For example, if a system has five different modules that may be at fault, an
ideal hypothesis may pick a subset of these modules that you estimate close to
having a 50% probability of containing the problem.

### Postmortems

Postmortems are a very useful technique for introspection
useful to understand why a system didn't work as expected and take
corrective action.

