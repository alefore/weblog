# Edge: Lessons: Readability

## Edge: Lessons: Preamble

This document is part of
[a series of articles](https://github.com/alefore/weblog/blob/master/edge-lessons.md)
articulating lessons I've learned
during the 10 years I've been developing my own text editor.

## Make Things Explicit

**The implementation should reflect your thought process explicitly;
not only its conclusions**.
Removing the thought process makes the software less malleable.

Early in my career,
I made the mistake of optimizing my implementations for brevity.
If you can say it with fewer words, why more?

> Je n’ai fait celle-ci plus longue que parce que je n’ai pas eu le loisir de la faire plus courte.

But simplicity and brevity are different things.
In fact, in software you often reach a point
where extra brevity complicates things.

It bears saying it explicitly:
just because some expression is shorter (by whatever metric)
doesn't mean it's simpler.

This has obvious non-controversial implications:

* Define appropriately named constants;
  rather than using magic values directly.

* Add `const` annotations to relevant class methods.

* Use appropriate names that convey meaning; eschew abbreviations.

It also has some less-obvious implications.

### Loops

Most operations on containers (either sets, sequences, maps, etc.)
can be expressed as one of a few canonical operations,
such as finding elements matching a predicate,
transforming elements,
or aggregating elements.

In those cases, I **avoid writing explicit `for` or `while` loops**.
I prefer standard functions (such as `std::views::transform` and related logic)
for manipulating and aggregating containers.
I'm a big fan of the recent ranges/views APIs.

#### Edge: Lessons: Make Things Explicit: Loops: Rationale

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
**the presence of an explicit `for` or `while` loop alerts you:
there's something unusual in this block**.

### Avoid auto

TODO: Flesh out.

## Use lambdas liberally

I've found that liberal use of lambda expressions can aid readability
in a number of ways:

* By explicitly capturing a small number of variables
  (typically by reference),
  it helps you signal that a given scope
  (the lambda body)
  only has access to a small set of all the available local variables.

* It helps signal that temporary variables
  (which you define within the lambda's scope)
  are only defined
  for the specific purpose of computing the lambda's output.
  This is similar to creating nested scopes
  (without using lambda forms).

* It helps you use various `return` statements
  to yield the value that you'll assign
  to a given (possibly `const`) variable.
  The alternative to this would be to use assignment.
  Said differently, this lets you avoid assignment and,
  instead, use construction.
  One advantage of this is helping you ensure
  that you always give the variable a value explicitly
  (since your lambda expression has to return a value).

### Edge: Lessons: Use std::invoke

When I declare lambdas that must be run directly,
I use `std::invoke`.

Rather than:

    […] { … }()

I write:

    std::invoke([…] { … })

This is more wordy,
but makes it more explicit that the lambda is run immediately.

## Use std::variant

I've found `std::variant` very helpful.
I see it as playing a similar role to interfaces,
but allowing you to more easily enumerate the entire set of subtypes.
I use variants with `std::visit` throughout.

## Constructors

I try to keep constructors as simple as possible.
Hard work must be moved to either
customer sites
(for classes that are only instantiated by few customers)
or static factory methods.

### Declaration > Initialization

I prefer to assign default values to class variables in their declaration:

    class Good { …
      LazyString value_ = LazyString{"default"}

     public:
      Good() = default;
    };

    class Bad {
      LazyString value_;

     public:
      Bad() : value_(LazyString{"default"});
    };

That's preferable mainly because
the variable's declaration is the most natural place to look for its semantics.
There are other smaller benefits
(such as avoiding possible bad declaration order,
or neglecting to assign values when there are multiple non-delegating
constructors).

This isn't always possible:
sometimes the default value depends on a parameter received by the constructor.

### Aggregate initialization

When instantiating aggregate classes,
I always use aggregate initialization explicitly
(*i.e.,* `MyClass{…}`)
rather than parentheses-based construction
(*i.e.,* `MyClass(…)`) syntax.
I also prefer always fully specifying the type
(rather than relying on type deduction).

This makes it immediately visible that
the type of the resulting expression is exactly what I've written
and, perhaps more importantly,
that no complex logic is running,
this is mostly just moving data into place
(and possibly creating a few other instances
for variables I'm not explicitly initializing).

By specifying the type fully,
I also help the compiler produce nicer error messages
when I get the type of one of the sub-expressions wrong.
This can be very helpful for long and complex initialization expressions.

## What's next?

I'd like to explore a few ideas:

* Replace a few uses of `std::variant` with `std::expected`.
  I only learned about `std::expected` relatively recently,
  and I think there's a few places where it'd fit slightly better.

