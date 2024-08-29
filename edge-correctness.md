# Edge: Lessons: Correctness

## Edge: Lessons: Preamble

This document is part of
[a series of articles](https://github.com/alefore/weblog/blob/master/edge-lessons.md)
articulating lessons I've learned
during the 10 years I've been developing my own text editor.

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
This has worked very well generally,
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

## Use types effectively

Good use of expressive and well-designed static types
facilitates correctness and maintainability.

### Use specific types

**Use semantically-rich custom types for variables and class fields**
(and methods' inputs and outputs).
Many of these types are simple wrappers of primitive types
(`int`, `string`, `double`, etc.),
but they carry significantly more meaning.

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

### Ghost types

Unfortunately, AFAIK, C++23 provides no standard mechanism
to declare "type aliases" effectively.

With `typedef` or `using` declarations,
the compiler won't detect errors:
two aliases of the same underlying type can be used interchangably.
Because of this,
the gains of using `typedef` or `using` aliases
(code *will* be slightly more readable)
are very little (no additional static type-safety).

To define custom types, **I extend a templated `GhostType<>` class** like this:

From 
[environment.h](https://github.com/alefore/edge/blob/master/src/vm/environment.h):

    // Represents a namespace in the VM environment, where symbols can be
    // defined. For example, a reference `lib::zk::Today` is actually the symbol
    // "Today" in the namespace `{"lib", "zk"}`.
    class Namespace
        : public language::GhostType<Namespace, std::vector<Identifier>> {};

From
[file_system_driver.cc](https://github.com/alefore/edge/blob/master/src/infrastructure/file_system_driver.h):

    class FileDescriptor
        : public language::GhostType<FileDescriptor, int, FileDescriptorValidator> {
    };

    class UnixSignal : public language::GhostType<UnixSignal, int> {};
    // Why define a GhostType for `pid_t`, which is already a "specific" type?
    // Because pid_the is just a `using` or `typedef` alias, so incorrect usage
    // isn't detected by the compiler.
    class ProcessId : public language::GhostType<ProcessId, pid_t> {};

From
[naive_bayes.cc](https://github.com/alefore/edge/blob/master/src/math/naive_bayes.cc):

    class Probability
        : public GhostType<Probability, double, ProbabilityValidator> {};

`GhostType<>` is the parent class of all these custom types.
It declares appropriate constructors and operators
(*e.g.,* `std::hash`, `operator==`, `operator+=`…)
based on what the underlying type supports.

#### Extensible

One unexpected advantage of the `GhostType<>` approach
(making the specific types subclasses
of the generic `GhostType<>` class)
is that it's trivial to define custom methods on the subclasses.

My initial approach for declaring type aliases was based on macros,
like this:

    GHOST_TYPE(InputKey, std::wstring);

However, using the templated superclass works much better.

* Because I can define temporary custom methods,
  this helps me make incremental progress
  when I want to gradually adjust an underlying representation.
  I can migrate customers gradually.

* Less importantly,
  the approach based on macros had problems with complex internal types:
  the preprocessor would sometimes get confused with commas.
  There were workarounds for this, but they were cumbersome.

### Maintaing invariants

Defining wrapper types corresponding to predicates
is another technique I've found useful.

If you have a type `T` and a predicate `P`,
you would create a type `TP`
that contains instances of `T` match `P`.
I'll refer to `TP` as the *subtype*.

This allows functions to **explicitly state some preconditions**
(*e.g.*, an input container must be sorted,
or non-empty,
or have exactly between 128 and 256 elements).
Using **specific types enables validation of expectations in the constructor**.
The rest of the application
can reliably assume that these preconditions are met.
Instead of repeatedly checking inputs against predicates
(or, alternatively, transforming inputs to match the predicates;
*e.g.*, sorting a container received),
functions should receive their inputs using the subtypes directly.

This is exactly what `const` does to classes:
you define the semantics of `P`
(*i.e.*, the const predicate – what it means for a class to be `const`)
and then, given a `T` (non-`const`) instance,
you can trivially obtain a `TP` (`const`) view that implements `P`.
However, this general principle beyond just immutability.

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

#### Ghost types validators

As the `FileDescriptor` and `Probability` examples demonstrate,
`GhostType` subclasses can specify a validator.
This is used to maintain invariants on the underlying types.

For example:

    struct ProbabilityValidator {
      static PossibleError Validate(const double& input) {
        if (input < 0)
          return Error{LazyString{L"Invalid probability value (less than 0.0)."}};
        if (input > 1.0)
          return Error{
              LazyString{L"Invalid probability value (greater than 1.0)."}};
        return Success();
      }
    };

These invariants are validated at construction time
(either crashing the program or returning an `Error`,
depending on the API used).
This allows us to rest assured that all instances
will always abide by these preconditions.

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
for a few milliseconds after program start).

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

##### Probability

The `Probability` type in the Naive Bayes implementation
validates that a probability (represented using an underlying `double`)
is in the expected [1, 0] range.
Not only do I avoid having to add validation logic
to complex statements that compute probability values,
I also know that this expectation is validated
*every single time* a new probability value is computed.

##### Range and LineRange

The `Range` type
([header](https://github.com/alefore/edge/blob/master/src/language/text/range.h))
enforces that *begin* <= *end*.

`LineRange` represents a range contained entirely in a single line.

Both invariants are enforced at construction.

## Testing

I've found unit testing incredibly helpful to maintain correctness.
The cost of maintaining a reasonable set of tests
is orders of magnitude smaller than the gain.

As of 2024-08-24, I have 734 tests.
It takes roughly 10 seconds (in my laptop) to run them via `edge --tests=run`.

TODO: Describe the testing API.

TODO: Talk a bit about the history of testing.

### Why test

When I change a specific module,
did I now invalidate implicit assumptions
that customer modules are making?
Thank you, Hyrum's law!

Unavoidably, complex changes to widely-used module causes bugs.
In my experience, bugs can be sorted into two groups:

1. Those affecting modules *with* good testing coverage.
   These bugs are detected immediately,
   allowing you to inspect the relevant modules directly,
   while you still have full context about the change you're introducing.

2. Those affecting modules *without* good testing coverage.
   These bugs are detected months later,
   through strange crashes,
   after many other changes (which may misleadingly appear related)
   have been introduced,
   long after you've forgotten the context of the culprit.

Because of historical reasons,
my testing coverage varies very widely across modules.
In my experience,
the cost of maintaining reasonable tests
is much smaller than
the cost of allowing bugs to slip from the first to the second set.

### Manual testing

I've heard the argument that manual testing is good enough:
You introduced new functionality?
Just manually run the code and validate that it all works,
that you haven't broken anything.

I suppose this may work on trivial toy programs,
where each change has only a trivial set of consequences.
Once programs reach a certain complexity,
there's no substitute for being able to automatically validate
the entire set of expectations that you've articulated as tests.

### Part of the binary

Tests are always included in the Edge binary.
One can always run `edge --tests=run` to run all tests.

It would be trivial to add a compilation mode that throws away the tests.
I thought I would quickly prioritize implementing this.
Surprisingly, I never found the need to do this.

## Challenges

I consider the following unsolved challenges when it comes to correctness.
I think they continue to slow me down
and I would like to find better ways to deal with them.

* Automatically detecting read-after-move problems.
  I think they are relatively easy to miss.

* Life-cycle issues, especially with references captures in lambdas
  that may not run immediately.
  I've considered using types explicitly to at least detect this in run time,
  like a `ScopedReferenceFactory` class used thus:
  `[foo = ScopedReferenceFactory().NewReference(foo)] { … }`.
  This would at least crash directly if the reference wrapper is read after the
  `ScopedReferenceFactory` has been destroyed.
  I wish I could find a cleaner solution
  –with less boilerplate and less brittle.

