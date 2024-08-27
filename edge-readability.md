# Edge: Lessons: Readability



## Edge: Lessons: Preamble

This document is part of
[a series of articles](https://github.com/alefore/weblog/blob/master/edge-lessons.md)
articulating lessons I've learned
during the 10 years I've been developing my own text editor.

## Edge: Lessons: Use lambdas liberally

I've found that liberal use of lambda expressions can aid readability
in a number of ways:

* By explicitly capturing a small number of variables
  (typically by reference),
  it helps you signal that a given scope
  (the lambda body)
  only has access to a small set of all the available local variables.

* It helps signal that temporary variables are only defined
  for the specific purpose of computing the lambda's output.
  This is similar to creating nested scopes
  (without using lambda forms).

* It helps you use various `return` statements
  to yield the value that you'll assign to a given variable.
  The alternative to this would be to use assignment.
  Said differently, this lets you avoid assignment and,
  instead, use construction.

### Edge: Lessons: Use std::invoke

When I declare lambdas that must be run directly,
I use `std::invoke`.

Rather than:

    […] { … }()

I write:

    std::invoke([…] { … })

This is more wordy,
but makes it more explicit that the lambda is run immediately.

## Edge: Lessons: Use std::variant

I've found `std::variant` very helpful.
I see it as playing a similar role to interfaces,
but allowing you to more easily enumerate the entire set of subtypes.
I use variants with `std::visit` throughout.

## What's next?

I'd like to explore a few ideas:

* Replace a few uses of `std::variant` with `std::expected`.
  I only learned about `std::expected` relatively recently,
  and I think there's a few places where it'd fit slightly better.

