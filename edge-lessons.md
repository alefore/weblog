# Edge: Lessons

TODO: I'm still breaking down this text into separate articles.

* Introduction
* [Concurrency](https://github.com/alefore/weblog/blob/master/edge-concurrency.md)
* [Correctness](https://github.com/alefore/weblog/blob/master/edge-correctness.md)
* [Readability](https://github.com/alefore/weblog/blob/master/edge-readability.md)
* [Development cycles](https://github.com/alefore/weblog/blob/master/edge-development-cycles.md)
* [Features](https://github.com/alefore/weblog/blob/master/edge-features.md)
* Ideas that worked well
* What's Next?
* Conclusions

## Introduction

This document is work-in-progress.
As of 2024-04-13, this is still incomplete
and probably contains errors.
I haven't yet had the time to fully review it.

This document is **an attempt to capture lessons I've learned
after a decade of working on Edge** (my personal text editor).

Many of the lessons described are fairly technical
but at least one (Bursts & Pauses) is general
—observations about the life-cycle of side-projects.
Among the technical lessons,
many are specific to C++,
but they probably apply, to some extent,
to other languages.

### Background

Edge is a Linux C++ terminal-based text editor.
I started working on
Edge in 2014
–making the
[very first commit](https://github.com/alefore/edge/commit/312ecc2462315e8e0648cbd2680cc0366819df1e)
on 2014-08-09.
I've used Edge ~exclusively since 2015.

Edge has its own extension language,
which looks like C++ with memory management,
and has logic (such as syntax highlighting)
for editing C++, Markdown, Java, Python
and a few other file types.

I mainly use it…

* for programming at work
  (though this days I program relatively little at work),

* to maintain my
  [Zettelkasten](https://github.com/alefore/weblog/blob/master/zettelkasten.md),

* as my planning & productivity management system
  (which is integrated with my Zettelkasten,
  so this is really repeating the previous point), and…

* to develop Edge.

As of 2024-04-14, Edge is 67.9k lines of C++ code
(per `wc -l $(find src -name '*.cc' -or -name '*.h' -or -name '*.y'`).

### Caveats

* I believe there's significant **recency bias** in this distillation.
  I'm probably overweighing insights I've reached relatively recently
  –towards Edge's tenth anniversary–
  and not doing justice
  to those from the beginning of the journey.

* The lessons described here are fairly **subjective**.
  These are *not* universally applicable principles.
  They rest on various assumptions specific to my context,
  which may not apply to other environments or systems.
  I'm holding back from prefixing nearly every sentence
  with "in my experience" or similar qualifiers,
  but they should be assumed.

## Ideas that worked well

> Las grandes hojas del abedul:
> unas se van, fugaces, con el viento;
> otras caen solas, dando vueltas en el aire frio.

### Principles

The following are a few *ideas or principles* that
influenced my work in Edge,
which worked better than I had initially anticipated:

* The **most precious resource is your user's mental energy**.
  Computers are fast, and
  –even if Moore's law is dead–
  keep getting faster and cheaper;
  if you can spend a few million cycles to
  save the user from
  having to mentally compute something
  (that distracts them from their goal),
  do it, go the extra mile.
  Never force the user to wait for the results of an operation.

  * If the user is typing a command to be executed
    (*e.g.*, fork out an "ls" or "grep" command, kick off a regexp search),
    display a preview of the results.
    For example, as the user types the regexp,
    you can already display "this would match 26 locations".
    As the user types an "ls" or "grep" command
    (which have no side effects),
    you can already display the first lines of the output they'd get.

* **Being able to check if a program expression
  is free of side effects (without running it)
  is powerful**.
  This enables me to asynchronously evaluate such expressions
  when they appear inside a file,
  displaying the evaluation results.
  Being able to enter (and edit) such expressions as I'm editing a file,
  this sort of "evaluate-as-you-type" functionality is very useful.
  I recorded
  [a demo](https://asciinema.org/a/pCa75UZYlVXD0uboyLQl5zUbi)
  for a friend who asked about this.

* **Optimize for development speed**.
  You can't know
  the direction in which you'll take a function or module in the future,
  but you can always invest in making your software more malleable.
  Make it easier to iterate faster.
  Avoid optimizing things for performance
  in ways that leave you stuck on local maxima.

## What's Next?

> He savors a hearty brunch; takes his time.
>
> Sitting beside him, feet dangling, roller blades already on, his daughter hums eagerly.

Who knows where Edge will be in ten years?
Will I still be extending it?

This section lists some ideas I'd like to explore.

### Improve the extension language

The extension language could use some improvements.

I'd like to be able to define structures
entirely within the extension language.
This hasn't been too constraining for me
(because I can define structures within the host language
and easily expose them to extensions),
but I think it would enable extensions to grow significantly.

I would also like to support some form of generics.
Currently, for generic classes like "set" or "vector"
I have to define specific subtypes in the host language
(e.g., `SetString`, `VectorString`, `VectorInt`, etc.).
It's easy to define this in the host language, but far from ideal.

### File inspection/manipulation API

I'd like to provide friendlier ways to express operations
that filter or transform sub-trees of a document, such as:

* Find all links with the text `xyz` within a bullets list in a section
  where the header is `Tags` in this Markdown file.
* Apply <function mapping string to string>
  to all those links, transforming their text.

This API should probably be based on the parsed syntax tree.

Obviously, I have some APIs to inspect and modify file contents,
but they are line/character oriented,
which makes them too low-level
–expressing intent is difficult.

### Zettelkasten improvements

The functionality to manipulate my Zettelkasten can be refined further.

For example, I'd like to be able to add more formal annotations to my notes and…
use those annotations as I'm editing files.
Essentially, allow me to define stronger semantics in my notes,
deriving ontologies.

### CSV integration

I'd like to make it easy to use Edge to load a *medium*
(~1e5-rows range)
CSV file and interactively manipulate it.
Edge has CSV support, but it's fairly rudimentary.

For example, to quickly express things like the following
(probably based on Edge's extension language):

* "What's the average of the 5th column
  across rows where the 2nd column is greater than the 1st column?"

* "Plot a histogram of the values in the 2nd column,
  weighing each entry by the result of multiplying the 4th and 5th columns."

* "What's the Pearson correlation coefficient between two columns?"

### Audio improvements

I want to find better ways to use audio as an additional information channel.

Edge has had audio support based on libao for many years.
However, it doesn't work very well:
interruptions or quitting sometimes leaves the audio device in a strange state.
As a result, I end up almost always muting it
(by adding `--mute` to `~/.edge/flags.txt`).

For example, when you search for a regular expression,
Edge plays a different tune based on the number of results
(simplifying, something like:
"error", "zero matches", "one match", "two matches", "many matches").

This would require making the audio support more robust.
Once that's in place, I intend to explore more ideas
about how to use sound.

## Conclusions

It's crazy: I've continued working on Edge
for ten years.
I couldn't have known how far I'd go,
how many features I would have implemented.

I think the main lesson I learned
is to **resist the urge to compromise on correctness**.
This is behind a lot of the lessons in this article,
such as:

* Digging deep in order to fix bug categories
  (rather than only specific manifestations)

* Making expectations and invariants explicit
  –through judicious use of strong types,
  runtime validations,
  and unit tests.

I've invested a lot of energy into developing Edge.
I'm glad I've done so.
Starting this effort and continuing to invest in it has been very satisfying.
I've learned a lot on this journey.

