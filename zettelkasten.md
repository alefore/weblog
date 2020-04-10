# Zettelkasten

I'm using several Zettelkasten principles as a way to organize and develop my
thinking.

## Directory Structure

My Zettelkasten is a single directory containing one Markdown file for
each note. The names of the files are simply an increasing three-characters ID,
starting from `000` and using both numbers as well as lowercase characters.

As of 2020-04-10, the last note I've created is `0qs.md` (the 964th note).

I have a symbolic link `index.md` which points to my main entry point note (note
`0a9.md`). This is used by my `:zki` (Zettelkasten Index) command to take me to
the "index" into my notes.

### Git

I store my Zettelkasten in a git source code repository.

I considered using a private repository in github but, for the time being, I'm
using Google Cloud Source Repositories, which seems to work well: it even gives
me a Markdown viewer which works well enough (though I use it very rarely; I'm
much more likely to peruse my notes through my text editor, since that allows me
to trivially modify them).

### Why Not Readable Names

I considered using "human readable" names instead of my "unique ID" approach
for the files in my Zettelkasten. Perhaps I could have taken just the
titles of notes.

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

* In the history in my git repository. This will let me see not only when the
  note was created, but its entire history, down to the granularity with which
  I commit to the repository (which isn't every single time I add a note, but
  roughly, around once per day).

* In my text editor's log. This is (as of 2020-04-10) work in progress. I keep a
  log for each file of every time I've opened it and every single transformation
  I've done on it. Obviously, this grows quickly, so I expect I'll have to start
  trimming down the logs soon. And I haven't yet generated many visualizations
  for this data. However, I expect that, in the future, I will be querying these
  logs for information such as total time I've spent looking at a given note,
  frequency of access to notes by different criteria, etc..

## Note Structure

Every Zettel in my Zettelkasten is a Markdown file with the following
structure:

* Name of the idea given as the title of the document.

* Text describing the idea.

* Set of related links.

* Set of tags (optional). I haven't yet derived any actual value from the tags.
  Perhaps this means that I have fairly good structure Zettel; perhaps that
  my search functionality works very well; perhaps that my Zettelkasten hasn't
  grown enough to the point where this will become useful.

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
operations in my Zettelkasten.

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

* Go to an existing node in which it should be inserted.
* Enter the title of the new node between braces.
* Execute Edge command ":zkln" (Zettelkasten Link New). This (1) creates a new
  note with this title and a "Related" back-link to the original note, (2)
  leaves the cursor right after the title (where the contents of the note will
  begin), and (3) turns the text in the original note into a link to the new
  note (and saves the original note).

Sometimes, instead, I generate a new note not associated with any existing note.
I do this less frequently, but sometimes it helps me when the title of the new
note isn't obvious before I've written it down. To do this, I follow these
steps:

* Execute Edge command ":zkn" (Zettelkasten New). This will create a new empty
  note.
* Write down and save the note.
* Optionally, I'll navigate to existing notes, insert the ID (path) of the new
  note, and use ":zkl" (Zettelkasten Link) to turn the ID into a link to the
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

I use Mnemosyne and have a database of 8.1k cards, mostly about German, Italian,
and Swiss facts.

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

## Ideas to Explore

The following are ideas for my Zettelkasten that deserve further exploration:

* Giving presentations out of my Zettelkasten.

