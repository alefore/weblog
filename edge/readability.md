# Edge: Lessons: Readability

## Edge: Lessons: Preamble

This document is part of
[a series of articles](https://github.com/alefore/weblog/blob/master/edge-lessons.md)
articulating lessons I've learned
during the 10 years I've been developing my own text editor.

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

