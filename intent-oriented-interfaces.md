# Intent-Oriented Interfaces

## Introduction

This is a very early and incomplete draft.

This article:

* Introduces terminology to describe system interfaces
  as being intent-oriented,
  offering a concrete definition for "intent".

* Describes the advantages of intent-oriented interfaces.

* Offers a few principles that may help
  design interfaces that are more intent oriented.

This article is based on many conversations with
Chris Nokleberg,
Ivan Metlushko, and
Tudor-Ioan Salomie.

## What is intent

### Definition

A customer adopts a shared infrastructure system
to achieve some goals.
These goals, along with any requirements or expectations,
are what we call "intent".

### Interfaces

These shared infrastructure systems
offer an interface
that the customer uses to communicate with them.

Among others, these interfaces can be:

* A configuration language
* A command-line interface
* An application programming interface

#### Interface Abstractions

The interfaces expose a set of abstractions
that customers must learn and use
to create artifacts that express or implement the customers' intent.

Unfortunately, these abstractions
rarely allow the customer to express the intent in the most natural way.
Instead, the customer must learn an arbitrary (form their perspective) language
to which he needs to translate the intent.

We use *mechanics* to refer to the artifacts where intent is expressed
indirectly, using arbitrary primitives that are arcane to the intent.
Mechanics map customers' intent to primitives offered
by the interface.

### Terminology

Throughout this document we'll take the view
of the owner of a *shared system*
(sometimes just a *system*)
that exposes an *interface*.
*Artifacts* created and maintained by *customers*
use this interface to *configure* (interact with) the system.

#### Artifacts

Artifacts come in many forms:

* Configuration files (adhering to the system's configuration schema)
* Source code (calling into the system's API)

They can be *very* complex,
often orders of magnitude more complex than the underlying systems.
As Larry Wall
[wrote](https://www.tuhs.org/Usenet/comp.unix.shell/1991-January/002464.html),
"*it is easier to port a shell than a shell script.*"

### Intent-oriented interfaces

We say that interfaces are intent-oriented
to the extent the abstractions or primitives they expose
match the mental model
in which the interfaces' customers reason about their intent.
That is, to the extent that the logical distance
between the mental models of the interfaces
and of the system's customers is small.

Intent-oriented systems
implement methods of fulfilling intent
based on declarations of intent,
rather than force customers' to specify and maintain this logic.

We use "a system is intent-oriented" as short-hand for
"a system is used through intent-oriented interfaces".

#### Subjective

Not only do different customers pursue different goals,
they also think about their goals in different ways.
The extent to which an interface is intent-oriented
can change as customers come and go.
It can also change as customers' mindsets change.
The extent to which an interface is intent-oriented
can only be measured relative to its current user base.

## Why intent-oriented interfaces

Gaps between customers' mental models and
systems' interfaces
force the use of constructs that are only loosely related to the intent.
When this happens,
customers build artifacts
using primitives that obscure what truly matters to them.

### Making the actual goal explicit

Configuring the system with arbitrary concepts brings false negatives:
customers' actual goals are obscured.
You don't know *why* they configured things a specific way:
what was the fundamental reason for a given choice?

### Don't emphasize irrelevant aspects

Conversely, configuring the system with arbitrary primitives
causes false positives:
irrelevant aspects are over-emphasized.
Since customers had to express intent using arbitrary concepts,
you can't easily know which of these arbitrary concepts are just incidental.
Did the user set up a certain parameter because they really care about it?
Or was this just the mechanism through which they can achieve their goal?

### Allow evolution

Enabling intent to surface explicitly,
rather than being lost in complexity of mechanics,
enables evolution.
This happens because it moves complexity
from layers above the interface
–the artifacts your customers maintain–
directly into your system.
In your system, you can manage it more easily.
This makes it easier …

* … to roll out new features
  that enable the system to better serve its customers.
  You can roll out new features you implement
  without having to adjust many complex customer artifacts.
  Otherwise, you may need to conditionally rewrite customer configurations'
  to apply an optimization that better achieves customer's goals.

* … to understand
  the relative importance of different potential improvements.
  How much would customers' benefit from a given optimization?
  How many have an intent to which the optimization is actually relevant?

* … to identify and remove features
  that your customers' don't actually care about.
  When no customer really cares about the feature
  (they only depend on it due to mechanics,
  because they had no way to express their intent),
  you won't be able to retire it
  until you've rewritten all customer artifacts
  that depend on it.

## Designing intent-oriented interfaces

There's no silver bullet
for how to align interfaces with their customers' intent.

This document attempts to offer, nevertheless,
a few techniques or principles that may help.

TODO: Explicit representation of intent.

### Customer User Journeys

Explicitly listing your user journeys can be of great help.
That means spending time answering the questions of:

* Who are your target customers?

* What are the most valuable ways your system can be useful to your customers?
  What are their goals?
  Can you articulate their goals in terms of value dimensions?

An ongoing process of iteratively refining these journeys,
putting them at the center of your interface design,
can go a long way.

### Seek feedback

Regularly collect feedback from your customers.
Seek to identify and understand common challenges they have with your system.
This may highlight places where there may be mismatches.

Be aware of the [Einstellung effect](https://en.wikipedia.org/wiki/Einstellung_effect).
This implies that you need to spend time with your customers
to really be able to understand their needs.
In a broader sense,
this is what [déformation professionnelle](https://en.wikipedia.org/wiki/D%C3%A9formation_professionnelle)
implies:
the longer and sharper your focus on shared infrastructure
(or a specific part of a complex layered system),
the more difficult it may be for you
to understand the perspective of your customers
(or of those on other layers).

### Refine your abstractions

Review the abstractions your interfaces offer
and improve them iteratively.
Do they capture the fundamental attributes your customers care about?
Can you introduce more semantically rich primitives that could replace them?

This is difficult and time-consuming work.
It moves complexity from your customers' artifacts
into your system –the lower layers that implement the interface.

## Conclusion

TODO

