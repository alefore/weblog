# Edge: Lessons: Features

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

I've spent some effort improving the input prompts
(i.e., when Edge has to ask you for a value,
such as the path of the file you want to open
or the value you want to give to a buffer's variable).
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

