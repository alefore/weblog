# Zettelkasten

I'm using several Zettelkasten principles as a way to organize and develop my
thinking.

## What are Zettelkästen?

A Zettelkasten (German for "notes box" or "notes cabinet") is a method to take
and organize notes about multiple topics. This has been used by many writers for
centuries and is being "rediscovered" recently by many bloggers and rationalists
(myself included).

### Plurals

In German, the word Zettel is both the singular and plural form for "a note"
("notes"). Kasten means "box" or "cabinet", so the compound noun Zettelkasten
means "box of notes". The plural, "boxes of notes", would be Zettelkästen.

### Notorious Users

Many articles online associate the Zettelkasten system with the German
sociologist Niklas Luhmann, who left behind a very large (90.000) collection of
notes and who described a few techniques for organizing paper Zettel ("notes").

However, many other notorious writers have used Zettelkästen. For example, an
[article in Der Spiegel from
1966](https://www.spiegel.de/spiegel/print/d-46407320.html) mentions how Jules
Verne, who died two decades before Luhmman's birth, used a Zettelkasten
consisting of 25.000 notes.

## Digital Zettelkasten

I decided to create my Zettelkasten as a set of files, rather than following the
time-tested approach of writing physical notes. This mostly reflects the fact
that I'm very comfortable viewing and editing text files and prefer the
mobility, durability and efficiency (of searching, navigating, etc.) aspects of
the digital medium.

I'm somewhat curious as to what advantages a physical Zettelkasten would have
that I'm missing out on. I suppose I'll never know.

### Git

I store my Zettelkasten in a git source code repository.

I considered using a private repository in GitHub but, for the time being, I'm
using Google Cloud Source Repositories to back up my repository. This has worked
well so far. I even get a repository viewer that renders Markdown files well
enough, though I use it very rarely; I'm much more likely to peruse my notes
through my text editor, since that allows me to modify them directly.

## Directory Structure

My Zettelkasten is a single directory containing one Markdown file for
each note. The names of the files are simply an increasing three-characters ID,
starting from `000` and using both numbers as well as lowercase characters.

As of 2020-04-10, the last note I've created is `0qs.md` (the 964th note).

I have a symbolic link `index.md` which points to my main entry point note (note
`0a9.md`). This is used by my `:zki` (Zettelkasten Index) command to take me to
the "index" into my notes.

### Why Not Readable Names

I considered using "human readable" names instead of my "unique ID" approach for
the file names (paths) in my Zettelkasten. Perhaps I could have taken
just the titles of notes.

I'm not sure this would have been better; it might have saved me from
implementing some automation in Edge (e.g., I would be able to load a file just
by remembering its title, without having to ... do the same through the
automation in Edge).

In the beginning, I created various symbolic links like `bitcoin.md` for a few
topics. However, I ended up getting rid of them; instead, I just always go
through the index (with `:zki`) or search directly for articles that have a
token in their title (e.g., `:zkt bitc`), which turned out to be more
comfortable for me.

### Why Not Dates

I considered using date-based schemas for the paths, such as `2020-04-10-0` or
just `2004100`. Proponents of this approach emphasize the advantage of knowing
when a note was created.

I decided to opt for the simpler 3-characters approach because I thought it
would keep the friction (of refering to notes) slightly lower.

Furthermore, I felt that metainformation about a note's creation and access is
important enough to deserve additional mechanisms and, once these
mechanisms are in place, embedding this in the note's title is somewhat
superfluous.

#### Notes: Metadata

I keep metadata about my notes in the following ways:

* Embedded directly in the notes contents. The obvious example of this are the
  links.

* In the history in my git repository. This lets me see not only when the note
  was created, but its entire history, down to the granularity with which I
  commit to the repository (which isn't every single time I add a note, but
  around once per day).

* In my text editor's log. This is (as of 2020-04-10) work in progress. I keep a
  log for each file of every time I've opened it and every single transformation
  I've done on it. Obviously, this grows quickly, so I expect I'll have to start
  trimming down the logs soon. And I haven't yet generated many visualizations
  for this data. However, I expect that, in the future, I will be querying these
  logs for information such as total time I've spent looking at a given note,
  frequency of access to notes by different criteria (e.g., day of the week,
  time of the day), etc..

## Note Structure

Every Zettel in my Zettelkasten is a Markdown file with the following
structure:

* Name of the idea given as the title of the document.

* Text describing the idea.

* Set of related links.

* Set of tags (optional). I haven't yet derived any actual value from the tags.
  Perhaps this means that I have fairly good structure Zettel; perhaps that my
  search functionality works very well; perhaps that my Kasten hasn't grown
  enough to the point where this will become useful.

### Good Names

Choose carefully the names you use in your models, making them as
self-explanatory as possible.

### Make Patterns Memorable

Giving good names to patterns we can observe and reason about makes them more
memorable and makes us more likely to be aware of them and to notice them when
they occur.

#### Consistent Names

Avoid using different names for the same entity; always refer to a given entity
with the same name.

Using multiple names is confusing because it introduces ambiguity - it isn't
clear to the reader if the two names actually refer to exactly the same entity,
or to entities that differ in subtle ways.

#### Short Names

A good name is short; it only contain tokens that refer to fundamental parts of
the thing being named.

### Links

Each note in my Zettelkasten contains links to other related
notes.

Links can occur directly in the text or in a separate "Related" section given
after the text.

The "Related" section contains just a list of bullets with one item per link.
Sometimes I annotate links by prefixing them with one of the following words:

* Why - The link points to a related note that explains the reasons behind the
  idea.

* How - The link points to a related note that augments the current idea by
  providing guidance on how to apply it.

* Counterpoint - The related note can be seen in conflict.

#### Counterpoints

Notes in my Zettelkasten link to ideas that they can be considered to
be in conflict with.

To do this, I annotate such links as "Counterpoint:".

##### Models Grow From Conflicting Thoughts

You may have ideas or thoughts that conflict with a model that you're exploring;
by exploring the conflicts you gain deeper understanding of the context, which
often enables you to strengthen (rather than discard) the model.

The goal of analyzing contradictions in our thinking shouldn't necessarily be to
resolve them (by adjusting or removing one of the ideas in conflict) but rather
to enrich our understanding.

## Low Friction

I am proactive about lowering the friction for common
operations in my Zettelkasten. I do this by defining
operations in the [zettelkasten
extension](https://github.com/alefore/edge/blob/master/rc/editor_commands/lib/zk.cc)
of [my text editor](https://github.com/alefore/edge).

### Common Operations

The following are common operations for my Zettelkasten:

* Reading:
  * Follow links
  * Navigating to a note
* Writing:
  * Collect ideas
  * Register a new note
  * Find pages that should link to a new note
  * Add a note to a given page

#### Follow links

A common operation in my Zettelkasten is to follow a link,
which I do just by scrolling to the file portion (the part between parentheses)
and pressing return.

This has acceptable friction. It could be lower if I could also make the text
of the link (the part between the braces react).

#### Collect Ideas

I separate the process of collecting new ideas that I want to insert into my
Zettelkasten from the process of actually registering them as notes. I
do this because the friction of entering notes into my Zettelkasten is high
enough (due mainly to the expectations of quality that I maintain for notes)
that it would become too distracting and slow to do this as I'm engaging in
other activities where ideas occur.

##### Collecting Ideas: External Files

To collect ideas for my Zettelkasten, I maintain external
(i.e., disconnected from my Zettelkasten) "scratch pad" files where I jot ideas
down.

These files are optimized to allow me to capture thoughts as quickly as
possible, with little concern for the quality with which I represent—just enough
that I'll be able to remember the idea in the next few days.

I use mostly Google Keep for these lists, and I do it mainly because of its ease
of use and the fact that I can seamlessly use it from my laptop as well as my
phone.

#### Register a new note

To register a new note, I typically use the following process:

* Go to an existing node from which the new note should be referenced.
* Enter the title of the new node between braces.
* Execute Edge command `:zkln` (Zettelkasten Link New). This (1) creates a new
  note with this title and a "Related" back-link to the original note, (2)
  leaves the cursor right after the title (where the contents of the note will
  begin), and (3) turns the text in the original note into a link to the new
  note (and saves the original note).

Sometimes, instead, I generate a new note not associated with any existing note.
I do this less frequently, but sometimes it helps me when the title of the new
note isn't obvious before I've written it down. To do this, I follow these
steps:

* Execute Edge command `:zkn` (Zettelkasten New). This will create a new empty
  note.
* Write down and save the note.
* Optionally, I'll navigate to existing notes, insert the ID (path) of the new
  note, and use `:zkl` (Zettelkasten Link) to turn the ID into a link to the
  note (which reads the title).

#### Examples

This recording gives examples of how I perform some of these operations through
my text editor:

[![asciicast](https://asciinema.org/a/314506.png)](https://asciinema.org/a/314506)

## Spaced Repetition and Notes

I intend to generate flashcards from the notes I'm adding to my
Zettelkasten in order to use Spaced Repetition to memorize
them.

I have started by annotating notes with what I hope will allow me to
programatically generate flashcards (and, ideally, update them, though that
seems harder) but that don't clutter the Markdown visualization very much. To do
this, I add a sequence of "Cloze: TEXT" annotations at the end of the notes,
with the semantics that a flashcard should be generated from the title and text
of the note with a cloze deletion of the selected text.

Unfortunately, I haven't yet automated the generation of cards, and it isn't
very clear to me how well this will work.

### Spaced Repetition

Spaced repetition is a very good way to learn. It requires consistency.

I use [Mnemosyne](https://en.wikipedia.org/wiki/Mnemosyne_(software)) and have a
database of 8.1k cards (as of 2020-04-10), mostly about Swiss facts (e.g., how
the parlament works), German and Italian vocabulary and grammar, and a few other
random topics. I've been using this database since around 2014 (possibly
earlier).

## Extracting articles

I am extracting articles directly from my Zettelkasten and publishing them in
github. For each article that I want to extract, I provide:

* A starting Zettel for the article, representing the main entry point into the
  topic.

* A blacklist of Zettel with "remote" topics that are reachable from the links
  in the starting Zettel but that shouldn't be included in the article because
  they are only too vaguely related with the main topic.

I have some automation that:

* Traverses the graph in depth-first order, appending their contents to the
  output.

* Removes all local links (to other Zettel).

* Tracks the depth (the distance from the starting Zettel) and adjusts the
  titles, nesting them accordingly.

* Omits the "Related" sections (since all links there will be expanded or
  ignored).

TODO: The automation should be improved to cap the amount of nesting for the
subtitles.

### Results

I imagined that the results of concatenating Zettel would be incredibly choppy
and I would have to significantly rework the output of this technique. I was
ready to maintain a set of patches and rebase them on top of new dumps.

However, much to my surprise, concatenation of notes seem to have worked fairly
well. The resulting articles feel perhaps a little bit "dense" (which I suppose
is good). I haven't had to edit any of these dumps just yet.

### Example

As an example of how I extract articles from my Zettelkasten, I use
the following code to generate my article about Zettelkästen:

    zke("../weblog/zettelkasten", "0bh",
        "01m"         // Voicing Conflicting Thoughts
            + " 01k"  // Dualism
            + " 0jp"  // Long Names: Code Smell
            + " 023"  // Function Names: Mention Side-Effects
            + " 01a"  // Choose Variable Semantics Carefully
            + " 07d"  // Tools for Thinking
            + " 0a7"  // Brains: Small Stack
            + " 06j"  // Non-linear Impact of Changes
            + " 0f2"  // Software Notifications
            + " 09z"  // Reduce Friction
            + " 01u"  // Be Present
            + " 003"  // To-Do Lists
            + " 01v"  // Ideal Environment
            + " 0fn"  // Only Functional Objects, Objets d'art & Plants
            + " 0pw"  // Spaced Repetition: Topics
            + " 0px"  // Spaced Repetition: Effective Practices
    );

The arguments to the `zke` function are:

* The path on which the article (output) should be saved
  (`../weblog/zettelkasten`; the function will append the `md` extension).
* The ID of the starting Zettel (`0bh`).
* The space-separated list of blacklisted Zettel. These are Zettel that are
  transitively reachable from the starting Zettel, but that I don't want to
  include in the article (because they are a bit too far away from the main
  topic).

This is executed by [Edge](http://github.com/alefore/edge)'s VM language (the
implementation is itself [defined as an Edge
extension](https://github.com/alefore/edge/blob/master/rc/editor_commands/lib/zk.cc)).

## Ideas to Explore

The following are ideas for my Zettelkasten that deserve further exploration:

* Giving presentations out of my Zettelkasten.

* Using a Zettelkasten for fiction. I've been working on a novel and I'm curious
  as to whether a Zettelkasten, perhaps with significant metadata, may be a good
  medium to structure it and produce the final manuscript (similar to how I
  produce articles).

