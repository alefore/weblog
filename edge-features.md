# Edge: Lessons: Features

## Edge: Lessons

TODO: I'm still breaking down this text into separate articles.

* Introduction
* [Concurrency](https://github.com/alefore/weblog/blob/master/edge-concurrency.md)
* [Correctness](https://github.com/alefore/weblog/blob/master/edge-correctness.md)
* [Readability](https://github.com/alefore/weblog/blob/master/edge-readability.md)
* [Development cycles](https://github.com/alefore/weblog/blob/master/edge-development-cycles.md)
* [Features]((https://github.com/alefore/weblog/blob/master/edge-features.md)
* Ideas that worked well
* What's Next?
* Conclusions

### Introduction

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

#### Background

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

#### Caveats

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

### Ideas that worked well

> Las grandes hojas del abedul:
> unas se van, fugaces, con el viento;
> otras caen solas, dando vueltas en el aire frio.

#### Principles

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

### What's Next?

> He savors a hearty brunch; takes his time.
>
> Sitting beside him, feet dangling, roller blades already on, his daughter hums eagerly.

Who knows where Edge will be in ten years?
Will I still be extending it?

This section lists some ideas I'd like to explore.

#### Improve the extension language

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

#### File inspection/manipulation API

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

#### Zettelkasten improvements

The functionality to manipulate my Zettelkasten can be refined further.

For example, I'd like to be able to add more formal annotations to my notes and…
use those annotations as I'm editing files.
Essentially, allow me to define stronger semantics in my notes,
deriving ontologies.

#### CSV integration

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

#### Audio improvements

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

### Conclusions

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

## Edge: Lessons: Preamble

This document is part of
[a series of articles](https://github.com/alefore/weblog/blob/master/edge-lessons.md)
articulating lessons I've learned
during the 10 years I've been developing my own text editor.

## What worked well

The following are a few *features* that worked better than I anticipated.
I hope this helps other people working on their own text editors.

### Linear undo

**Linear undo**/redo history,
as explained in
[`src/undo_state.cc`](https://github.com/alefore/edge/blob/master/src/undo_state.cc).
If you undo a few transformations and apply a new transformation …
you don't lose the redo history: you can still apply it through undo.

While the undo history can grow exponentially, this has never mattered in
practice.

### Jump in screen

The ability to **jump to any position in the file that's currently visible**
simply by:

* Typing `f` (to activate this mode).
* Typing three characters ~matching the text you want to jump to
  (with disambiguation when the prefix would match multiple positions).
* Typing return to confirm the selection.

I'm using this fairly frequently,
comparing with "scrolling" up to the position (or a full regexp search).

I guess the general principle is to avoid long key presses
(that generate the same key repeatedly);
instead, always replace that with better semantics.

### Mutiple cursors

Native support for **multiple cursors** works well.

For example, a regexp-search
creates a cursor in every position with a match.
Being able to quickly say
"set the cursors to: a cursor at the beginning of every line
in the current paragraph"
and then just say "add four spaces at each cursor" is powerful.
So is saying "set a cursor on every position (in the current source code file)
where the compiler reported an error".

### clang-format

clang-format integration is a must-have.

Whenever I save a file of some specific formats
(*e.g.,* C++, Java, Javascript, SQL…),
Edge just pipes it through `clang-format` or similar commands
([implementation](https://github.com/alefore/edge/blob/master/rc/editor_commands/lib/clang-format.cc)).

Freeing you from caring about spaces as you're editing
speeds you up significantly.

Edge has some logic to indent things
roughly predicting what it thinks you want,
but it's very rudimentary
(it predates the integration with `clang-format`).

### Preview buffer

The preview buffer at the bottom of the screen is helpful.

For example:

* If the active cursor is on a path to an existing file
  (based on some search paths),
  Edge gives a preview of its contents
  (remembering where the cursor was when that file was last opened).

* Similarly, Edge lets you preview the output of commands as you type them
  (for commands that don't have side-effects).
  This means that you can "search as you type"
  for commands like like "look", "ls", or "grep".

* For "destructive" commands (e.g, "git commit…"),
  the preview buffer shows you useful data
  (depending on the command,
  things like a help screen or the output of "git status").

### Compiler buffers

Native support for "compiler" buffers has worked very well.

Edge automatically tags certain command buffers as a compiler
(*e.g.*, "if the command is an invocation to `make` or `bazel` or …").
This causes a few things such as:

* Parse the contents to detect references to open files.
  Edge maintains a database with all these references.
  If the compiler references an open file
  (which typically means you have errors),
  Edge will display an overlay next to the line.

* Rerun the command whenever a file is saved.

Saving a source file suffices to kick off compilation;
I continue editing but, as soon as the compiler outputs errors in the file,
I get overlays summarizing the errors.

### Git auto-commit

My Zettelkasten directory contains a `.edge-git-push.txt` file.
Because of this, every time I save a file in my Zettelkasten directory,
Edge automatically commits the change (to the underlying git repository)
and runs `git push`.

I wouldn't do this for source code repositories:
for those I want to be able to save multiple files
that I can commit as a single logical unit
after significant validations
with a meaningful commit message.

But for my low-friction Zettelkasten…
this has worked very well,
helping me keep multiple copies in sync.

### Good prompts

I've spent some effort improving the prompts.
Computers are so fast today that there's no reason
not to kick-off processing on every key press.

What kind of processing?

* As I'm entering a regular expression to search in the file,
  show me how many positions it would match.

* As mentioned elsewhere,
  show me a preview of the output of the side-effects free command I'm typing.
  Suppose I want to use `look` to lookup the word "immediately";
  typing `look immed` suffices to narrow it down and show me what I want.
  I don't even need to type Enter (and then close the buffer).

* If I'm typing a path (e.g., to open a file),
  show me with colors and additional information:

  * That what I've typed so far matches something
    (or show me the first character where it no longer matches something).

  * That what I've typed so far matches exactly 8 files.
    If what I've typed so far matches a directory exactly,
    show me how many files it contains.

  * That if I pressed Tab right now, this would advance the match
    –perhaps to a full match,
    perhaps to the point where I have to type more characters to disambiguate.

* Show me how many "similar" commands I've entered before.
  This allows me to quickly search in the history of commands,
  simply by entering prefixes and pressing the "up" arrow.

I've also adjusted some key-bindings to behave optimally
depending on the contents of the prompt.
Specifically, Ctrl+U usually deletes to the beginning of the current line,
but in a path (inside a prompt) it will delete only until the previous slash.

I've found that, because of this, I'm often running Edge and then,
within Edge, entering the path of the file I want to open
(rather than just using the default shell completion).

## Experiments

This is an attempt to document a few experiments that…
I expected would have worked a bit better than they have.

These are ideas that held a lot of promise but haven't fully panned out.
It could be that they are bad ideas or it could be that they need more work
before they become useful.

* Audio integration

* Dictionary completion

* Edge supports applying changes to multiple files simultaneously.
  For example, I could open 50 files, search for a given identifier
  (which puts a cursor in every occurrence),
  and then delete the identifier.
  While I often do this with multiple cursors within a single file…
  I only very rarely edit multiple files at once.

