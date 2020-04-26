# Software Correctness

Software is correct to the extent that its implementation fulfils its contract
and it's free of software defects (bugs).

## Common and Special Errors

We can distinguish errors of two types:

* Common Errors: These correspond to the expected variation in a system. While
  we should generally aim to minimze their occurrence, specific instances are to
  be expected and shouldn't trigger any corrective interventions.

* Special Errors: These correspond to unexpected new situations and call for
  direct analysis and intervention into the system.

This distinction is useful because the two different types of errors deserve
very different treatment.

This distinction has been made for matematicians and statisticians for a very
long time. In modern times, William Edwards Deming championed this
distinction.

### William Edwards Deming

William Edwards Deming was an American engineer, statistician, professor,
author, lecturer, and management consultant.

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
state of the system and to understand why an unexpected behavior occurs. Among
these tools I include:

* Generate good logs during the regular execution. When errors that can't be
  easily reproduced occur, those logs may immediately point to the culprit.

* Logic to dump state.

* Adequate monitoring. For complicated systems, it pays tremendously to have
  separate systems constantly monitoring different operational parameters. These
  would keep track of both:

  * General parameters, such as memory or CPU used over time, number of threads
    by state, etc..

  * Application specific parameters, tracking operations or state in the
    semantics of the business logic.

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

