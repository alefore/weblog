# Detect Errors Early

Systems should be implemented using approaches that enable they users and
developers to detect errors as early as feasible.

## Early Detection Makes Errors Easier To Fix

The sooner that an error is detected, the easier it'll be to fix. This
is because:

* The programmer will have more context around the error, having recently
  modified the module in which the error occurs.

* The error will have less time to affect the system, such as by corrupting data
   or by spreading false assumptions to other layers (both of which may require
  expensive clean up operations).

## Early Detection: Smaller Impact

The earlier errors are detected, the smaller their impact.

* Errors detected before software makes it to production won't affect users.
* Early detection of errors in production decreases the opportunities they'll
  have to do things such as corrupt data or cause larger outages.

## Types of Errors: Common and Special

According to William Edwards Deming, it's useful to distinguish errors
of two types:

* Common Errors: These correspond to the expected variation in a system. While
  we should generally aim to minimze their occurrence, specific instances are to
  be expected and shouldn't trigger any corrective interventions.

* Special Errors: These correspond to unexpected new situations and call for
  direct analysis and intervention into the system.

### William Edwards Deming

William Edwards Deming was an American engineer, statistician, professor,
author, lecturer, and management consultant.

### Expect Common Errors

> “By failing to prepare, you are preparing to fail.” –Benjamin Franklyn

You should accept that common errors are unavoidable. No matter how much care
engineers put into their craft–no matter how much passion–, software will have
bugs. By accepting that this will happen and creating meta-systems to deal with
these issues you'll vastly improve the success of your software system.

In other words, at some point the cost of minimizing the occurrence of common
errors will grow so much that it will be a significantly better investment to
improve the processes such that, when these unavoidable errors occur, they are
handled gracefully.

## How to Detect Errors Early?

The following are some techniques to detect errors early:

* Explicit Invariants
* Redundancy Detects Errors
* Design Docs
* Iteration: Errors in Mental Models
* User Interfaces
* Vanish Invalid States
* Unit Tests
* Code Reviews
* Proactively Check Invariants
* Release Often
* Detect Errors in Production

### Explicit Invariants

Software modules should make their internal and external invariants explicit,
in order to:

* Enable maintainability: make them easier to understand, extend and use.
* Enable correctness.

Questions:
* What's the ideal format to express invariants?

### Redundancy Detects Errors

Increasing the amount of redundant information can be a valid technique to
detect and avoid errors.

#### Longer Names

A user interface that receives commands may yield fewer operation errors by
using slightly longer command names than one that uses the shortest possible
sequences that unambiguously identifies each command.

### Design Docs: Detect Errors Early

As a mechanism to articulate a model at the right level of detail, design docs
can be an invaluable tool for detecting conceptual errors.

The effects of detecting errors during a design phase are large. Because they
allow the model to be expressed in a semi-formal manner (e.g., using pseudo
code) where irrelevant aspects can be omitted, it enables the author and
reviewers to focus directly in the most relevant parts of the model, eliminating
distractions. This allows errors or suboptimal decisions to be detected that
may otherwise go undetected for a very long time.

#### Design Docs

I believe strongly in the value of writing brief design docs for even
medium-sized systems.

#### Operate at the Right Level of Abstraction

When you're implementing a model, make sure you operate at the right level of
abstraction, allowing you to analyze the main points of the model, without
getting too distracted by irrelevant details.

In the right level of abstraction, you can analyze the main parts of the model.
You can rely on lower-level abstractions to solve sub-parts of the model, but
they should be clearly separated into different lower-level layers.

### Iteration: Detect Errors in Models Earlier

By allowing us to validate emphirically the mental models we have about a
reality for which we're building an information system (e.g., the
requirements for the system, the entities involved, the interactions between
those entities, etc.), iteration foments early detection of
errors.

### User Interfaces: Real-time Feedback

A good user interface provides real-time feedback in order to allow its users to
detects errors early.

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
help development by enabling earlier detection of errors.

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

###### Domain Types

In general, prefer to define and use domain-specific "ghost" types that are
aliases (or simple extensions) of primitive types, rather than using these
primitive types directly; the ghost types will more readibly reflect the
semantics and may enable the compiler to detect type errors.

Tags: software

Questions:
* What's the best way to define these types in C++?

####### Validate on Construction

When defining domain-specific types, apply as much runtime validation
as feasible during construction, thus ensuring that all successfully created
instances are valid (as long as there's no mutable state).

####### Interface: Vanish Invalid States

To Vanish Invalid States from Software Types, Domain Types
should deliberately constrain their interfaces to only permit semantically valid
operations.

For example, a type that contains a "state" representation that only has some
valid transitions should deliberately only permit valid transitions (ideally
failing at compilation time if an invalid transition is attempted).

##### C++: Function Inputs: References Beat Pointers

In C++, functions that receive non-optional objects that they operate on should
prefer to receive them by reference rather than by pointer since that eliminates
the possibility of causing null pointer dereferences inside them.

### Unit Tests

You are only finished creating a software component when you have a set of tests
that can be executed automatically to reasonably validate its correctness.

#### Unit Testing beats REPL Testing

Unit testing is significantly superior to REPL testing,
because it is automatically repeatable.

##### REPL Testing

Some programming language implementations offer a "read, eval, print loop"
(REPL) interface, where the programmer may load his code and use it to evaluate
expressions. REPL testing refers to using this interface to evaluate expressions
in order to validate that code units behave as the programmer expects.

### Code Reviews: Detect Errors Earlier

Code Reviews can be a very useful mechanism to detect errors earlier (and,
beyond that, drive up code quality in a team).

The general idea is that an engineer can levarge the experience of a more senior
engineer (at least relative to the problem domain) to identify errors earlier.
This should capture both low-level "implementation" errors as well as
higher-level conceptual errors that were not detected during the modeling
phase.

### Runtime Invariants Checks

This principle states that, in order to ensure its correctness,
software should be repeatedly and explicitly validating at runtime the
invariants that it can't prove with static analysis (e.g., those that can't be
validated by the type system), in order to detect errors as early as
possible.

Questions:
* How to handle invariant failures detected at runtime?

#### Validate Invariants at the Boundaries

It can be specially useful to validate runtime invariants at the
boundaries connecting layers.

##### Why Validate at the Boundaries?

Since different modules tend to be modified independently, it pays to check
invariants at the connecting points, since that will help detect mismatches in
the expectations and contracts.

#### Validate Invariants when State Changes

In modules that maintain state, it can be particularly helpful to validate
runtime invariants immediately before and immediately after state is
modified.

### Release Often

In order to detect errors early, software should be released to
production often. Put differently, the time after a code change has been made
before it reaches production should be minimized.

#### Automatic Canarying

It pays to automate the process of making new releases, including creating
meta-systems that monitor these releases and alert the software owners if any
anomalies are detected. Your release procedures should take into account that,
thanks to common errors, things *will* go wrong.

The following are desirable characteristics of such a system:

* Releases only require human involvement when things go wrong.

* Isolate failures

##### Isolate failures

Canarying procedures should be designed such that, when things do go wrong, they
are isolated into a small subset of the system (and the release is stopped).

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

### Detect Errors in Production

Applications running in production should be constantly monitoring their
internal state to detect errors as early as feasible.

#### Degrade Gracefully

In order to degrade gracefully, applications should be constantly
monitoring their internal state as they execute and warn its customers
about any localized problems as early as possible, so that the errors
can be investigated and remedied before their impact continues to grow,
eventually putting the entire system at risk.

## Handle Errors

### Recover And Alert

If a system detects an error (e.g., an internal invariant doesn't hold) it
should recover if this is feasible, as long as it can reasonably well establish
that this is unlikely to lead to larger issues in the future.

Larger issues include things like:

* Triggering cascading failures in other systems.
* Corrupting data.
* Serving the wrong data, potentially with privacy violations.

When this happens, special care must be taken to alert the system owners about
the condition, rather than allowing it to go unnoticed. This implies the need
for an alerting or logging system.

### Crashing May be Best

Sometimes crashing is the best reaction to detecting certain errors. This occurs
in the following situations:

* Some corruption in memory has been detected that could cause corruption of
  persistent data.

* An invariant has been violated that may put the security or privacy of the
  system's users at risk. In this case, complete unavailability may be the best
  option.

* The error detected is very likely to just cause a crash further down the
  execution of the program. In this case, it's best to crash as soon as
  possible, to help the user identify the root cause.

