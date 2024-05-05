# Load Shedding: Surprises

## Introduction

This article lists a few counterintuitive or surprising things
I've learned during fifteen years working on load shedding at Google.

This is an early draft (as of 2024-05-05).

## Retries are difficult

As a mechanism to handle spurious overload problems,
retries are very difficult to implement correctly;
their apparent simplicity is deceptive.
Engineers consistently get it wrong,
deploying implementations that make overload problems significantly worse.

For example, engineers often assume that exponential back-off
is a good strategy for clients to dynamically reduce the load they send
to an overloaded server.
Exponential back-off is, in fact, a terrible strategy in many situations.

TODO: Explain why Exponential back-off is terrible.

## Overloaded and mostly idle

Sufficiently large distributed systems are *simultaneously*
mostly idle and overloaded.

It is difficult to increase utilization
without causing some sub-components to run out of capacity.
Perfect load balancing does not exist.
Once they become large and complex enough,
even systems with low average utilization will have overloaded sub-components.

## Overload errors are undesirable; protections, necessary

Nobody likes overload errors:
we should do what we can to avoid them
(improve load balancing, dynamically provision systems, etc.).

None of that changes the fact that **all
subcomponents of large distributed systems
must degrade gracefully when overloaded**.
The desire to eliminate overload errors
should never justify rolling back overload protections.

The only way to ensure that a system (or a part thereof)
never returns "overloaded!" errors
is to make sure it has access to infinite capacity.
But, in our universe, there's no such thing as infinite capacity!
Once you accept a maximum bound for the capacity you can afford,
you are forced to face the question:
What should the system do in the unlikely situation that demand
(e.g., incoming traffic) exceeds capacity?
The answer should be obvious: Return "overloaded!" errors.

This is analogous to erroneously believing that "software should not have bugs"
implies "we shouldn't create mechanisms to deal with software bugs."
We should strive to do both: avoid bugs in software,
and implement adequate mechanisms to deal with them,
when they inevitably occur.

