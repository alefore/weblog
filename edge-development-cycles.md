# Edge: Lessons: Development cycles

TL;DR: Things that seem impossibly difficult today
may become trivial in the future.
My engagement with Edge is extremely bursty and… that's good.
In the short-term, unrelated distractions reduce my focus
(and thus block my progress) on Edge;
but, in the medium-term, they enable transformative ideas to manifest,
which feeds new bursts of development.

This document is part of
[a series of articles](https://github.com/alefore/weblog/blob/master/edge-lessons.md)
articulating lessons I've learned
during the 10 years I've been developing my own text editor.

## Bursts & pauses

### Bursts

**My engagement with Edge is extremely bursty**.
Most progress happens in bursts of activity
that happen about once per year
and usually last from two to five months,
during which I commit many changes per day.
After the bursts come long pauses of inactivity.

#### New Possibilities Open

> Pastel crumbs on a brown mantle;
> Spring returning to the forest on the hills.

The factors that sustain these bursts are:

                         ┏━━━━━━━━━━━━━━━━┓
                         ┃ Transformative ┃
                         ┃      ideas     ┃
                         ┗━┯━━━━━━━━━━━━━━┛
                           │
                         (+a)   ╭─(+b)─╮
          ╭────(+c)─────╮  │    │      │
    ┏━━━━━V━┓         ┏━┷━━V━━━━┷━━━━━━V━┓
    ┃ Focus ┠──(+d)───>     Progress     ┃
    ┗━━━━━━━┛         ┗━━━━━━━━━━━━━━━━━━┛

This starts when "transformative ideas" crystallize
—ideas about new approaches I could try,
all the way from internal implementation details
to very visible changes.

* Per relationship (a),
  when this happens
  many features, internal improvements, or optimizations
  (for performance or correctness)
  that I've been wanting for a long time (sometimes years)
  and that previously seemed very hard or even impossible,
  suddenly ... fall gracefully into place.
  I make a lot of progress.

* Per relationship (b),
  these gains compound:
  progress feeds on itself.
  Some of the major improvements I've just landed
  enable a large set of follow-up effects.
  Many adjacent modules become simpler,
  faster,
  more general,
  easier to use,
  etc..

* Per relationships (c) and (d) respectively,
  the excitement from the fast-paced progress is very motivating,
  and the increased focus also enables faster progress.
  This causes the reinforcing loop that sustains the burst.

##### Edge: Lessons: Take Breaks: Bursts: Example GC

One tangible example
of things "that previously seemed very hard"
falling gracefully into place
are the improvements to Edge's garbage collection (GC) implementation
between 2023-09 and 2023-10.
These changes improved Edge perceptibly,
without making the implementation more complex.

A burst ending in 2022-12
made GC pauses relatively slow:
fast enough that Edge was still usable,
but slow enough that one could notice when they happened (ugh!).
However, I had the feeling that
Edge's GC implementation was as efficient as I could reasonably expect.

And then, ten months later,
between 2023-09 and 2023-10,
I optimized the collection
through a series of gradual changes
that made GC pauses imperceptible.
I did this through a combination of:

1. Implementing highly-concurrent collection.
   Most non-constant-runtime operations now happen in thread-pools
   of configurable size.
   These are things such as
   navigating the set of reachable objects
   or deleting unreachable objects.

2. Executing work asynchronously.
   Not only is work dispatched to multiple threads,
   but the main thread now offloads some operations
   (*e.g.*, actual deletion of objects)
   to a dedicated thread-pool entirely
   and no longer waits for their completion.

3. Implementing resumable (incremental) collection.
   The collection operation detects if it overruns a customer-specified timeout;
   when this happens, it just returns directly;
   The next collection resumes where we left off.
   I had assumed that I would require a very significant rewrite
   (since the user can change pointer values
   in the middle of interrupted collections).

4. Delivering profile-based optimization,
   chasing after the functions that consumed significant time
   from the main thread.

Not only did I speed collection up,
I actually managed to simplify the customer interface
([commit](https://github.com/alefore/edge/commit/42ae36687e96d2ddd2510562368f2965951f9cfd),
[commit](https://github.com/alefore/edge/commit/8eefd3a47f20350e9c18259f9c95811f478b1c25))
and make usage less error-prone
by removing the burden of calling `Ptr::Protect` from its customers.

The core implementation
([gc.h](https://github.com/alefore/edge/blob/master/src/language/gc.h),
[bag.h](https://github.com/alefore/edge/blob/master/src/concurrent/bag.h) and
[gc.cc](https://github.com/alefore/edge/blob/master/src/language/gc.cc))
didnd't become significantly more complex:
LOC only grew by 3%!

* Between 2022-12-24 and 2023-08-29 these files didn't change;
  they contained 1163
  (519 + 214 + 430, respectively) LOC

* In 2023-10-27,
  After all these optimizations,
  they grew by a net delta of +35 LOC,
  to 1198
  (572 + 129 + 497, respectively) LOC.

In this specific case, LOC is a reasonable proxy
for implementation complexity.
To compute LOC, I excluded unit tests and included header documentation.

#### Ramp Down

> Summer afternoon: a swarm of wasps dances around my Spritz.

Usually after a few months
(though the specific durations vary greatly)
comes a ramp down.
I start pursuing other things in my life.
Edge becomes a secondary endeavor.

This happens because the supply of transformative ideas is depleted,
causing progress to slow down,
which enables unrelated distractions to reduce my focus on Edge.

     ┏━━━━━━━━━━━━━━┓                              ┏━━━━━━━━━━━━━━━━┓
     ┃  Unrelated   ┃                              ┃ Transformative <─╮
     ┃ distractions ┃                              ┃      ideas     ┃ │
     ┗━━━━━━━━━━━━┯━┛                              ┗━┯━━━━━━━━━━━━━━┛ │
                  │                                  │                │
                  │                                 (+a)  ╭─(+b)─╮  (-e)
                  │                 ╭────(+c)─────╮  │    │      │    │
                  │           ┏━━━━━V━┓         ┏━┷━━V━━━━┷━━━━━━V━┓  │
                  ╰───(-f)────> Focus ┠──(+d)───>     Progress     ┠──╯
                              ┗━━━━━━━┛         ┗━━━━━━━━━━━━━━━━━━┛

The order of events is very clear:

1. Per relationship (e),
   *Progress* depletes the stock of *Transformative ideas*
   and their ripple effects.
   This dampens the effects (a) and (b).

2. *Progress* becomes visibly slower,
   dampening relationship (c).
   I commit a couple changes a day
   but this gradually drops
   to a small change or two a week.
   I'm still trying to make progress, but …
   I've run out of easy things to implement.
   everything that remains is way too challenging or impossible
   (e.g., everything has been optimized
   for performance and correctness and simplicity
   as much normal constraints allow).

3. *Focus* decreases,
   and the effect of relationship (d) disappears.
   This allows the tension depicted in relationship (f) to play out:
   *Unrelated distractions* decrease my *Focus*.

Although the tension (f) between *Focus* and *Unrelated distractions*
is present throughout the entire burst,
*Progress* drops first, in step #2.
The shift away from Edge happens afterwards.
*Focus* is necessary but not sufficient.

### Pauses

And then starts a pause.
Other things fully take precedence.
Perhaps I switch to writing a short story,
or
a [Zuri Short](https://github.com/alefore/weblog/blob/master/zurich-shorts.md).
Or learn to play the harmonica.
Or [knit](http://github.com/alefore/knit).
I go on a long trip.
I continue *using* Edge, but I rarely improve it.

This lasts six months on average, sometimes significantly longer.
Until suddenly I can't resist the itch
to finally fix that annoying behavior
or implement that great idea that found me
and I end up getting sucked back in.
A new cycle begins.

#### Background Thinking

> Powdery snow, birches washed white.
> On a twig, a twinkle: a yellow tit chirps. Was it really there?

What triggers the periods of intense focus?
What enables the fast progress on areas I previously considered out of reach?

     ┏━━━━━━━━━━━━━━┓        ┏━━━━━━━━━━━━┓        ┏━━━━━━━━━━━━━━━━┓
     ┃  Unrelated   ┠──(+g)──> Background ┠─(+h⏳)─> Transformative ┃
     ┃ Distractions ┃        ┃  thinking  ┃        ┃      ideas     ┃
     ┗━━━━━━━━━━━━━━┛        ┗━━━━━━━━━━━━┛        ┗━━━━━━━━━━━━━━━━┛

During the pauses
my brain **keeps making connections and generating and encountering ideas**.
These can be alternative approaches I could try:
new underlying APIs,
different data structures,
entirely new user journeys,
etc..

This happens mostly subconsciously.
I'm not working on Edge actively,
but I'm still passively thinking about these problems:
the places where I got stuck
and couldn't figure out a better solution;
the things that annoy me as I use Edge,
even if I don't notice them consciously.
I still read articles about text editors
and find new ideas I had never considered.

I don't know if this type of medium-term background thinking can be rushed.
Maybe
[journaling](https://github.com/alefore/weblog/blob/master/journaling.md) helps?
Maybe talking to people with different perspectives?
In my experience,
some things help but they only go so far.
This takes time.

In any case, eventually a new idea, or a group of ideas,
crystallizes and I just have to try things out and ... boom!
The new ideas (often) work,
and they (often) have far reaching implications,
and a new burst begins.

### Emotional State

My emotional state factors strongly into the timing of the cycles.
The timing of bursts is strongly associated
with important events happening in my life:

* Some bursts started when I was going through difficult personal situations
  (*e.g.*, breaking up from a serious relationship)
  and wanted some distraction from real life.
  This is the case for the long burst from 2019-12 to 2020-05.

* Many long pauses coincide
  with times when I was distracted by real life events.
  Perhaps I was doing a lot of traveling,
  starting a new relationship,
  or distracted with changing apartments.
  Being very busy with life events left time for only short-lived bursts during
  the pauses from 2015 and 2016;
  2020-05;
  and 2021-03.

The causality goes both ways:
though less frequently,
these cycles have also caused changes in my life.

* The long burst from 2022-02 to 2022-06
  influenced a difficult decision I took at work
  to switch away from managing
  (which I had been doing for about 5 years)
  to a pure engineering role.
  The team I was managing grew
  from 6 engineers in 2020
  to 17 in early 2022.
  Most of my time (at work)
  became occupied by managerial tasks,
  pushing out technical work.
  I interpreted the amount of energy I was dedicating to Edge
  as a strong sign that I was missing technical work.
  Interestingly, the burst stopped shortly after I made the decision
  and started searching for someone
  I could transfer my management responsibilities to.

## Unrelated distractions

The complete system diagram would be this:

     ┏━━━━━━━━━━━━━━┓        ┏━━━━━━━━━━━━┓        ┏━━━━━━━━━━━━━━━━┓
     ┃  Unrelated   ┠──(+g)──> Background ┠─(+h⏳)─> Transformative <─╮
     ┃ distractions <──╮     ┃  thinking  ┃        ┃      ideas     ┃ │
     ┗━━━━━━━━━━━━┯━┛  │     ┗━━━━━━━━━━━━┛        ┗━┯━━━━━━━━━━━━━━┛ │
                  │    │                             │                │
                  │    ╰──(-i)──╮                   (+a)  ╭─(+b)─╮  (-e)
                  │             │   ╭────(+c)─────╮  │    │      │    │
                  │           ┏━┷━━━V━┓         ┏━┷━━V━━━━┷━━━━━━V━┓  │
                  ╰───(-f)────> Focus ┠──(+d)───>     Progress     ┠──╯
                              ┗━━━━━━━┛         ┗━━━━━━━━━━━━━━━━━━┛

This could, however, be reduced to
the main relationships from
*Unrelated distractions* to *Progress*:

    ┏━━━━━━━━━━━━━━┓           ┏━━━━━━━━━━┓
    ┃  Unrelated   ┠───(-)─────> Progress ┃
    ┃ distractions ┠──(+⏳)────>          ┃
    ┗━━━━━━━━━━━━━━┛           ┗━━━━━━━━━━┛

* In the short-term, *Unrelated distractions* hamper *Progress*
  because they deplete focus (relationships (f) and (d)).

* In the long-term, however,
  *Unrelated distractions* are necessary for *Progress*
  because they enable *Transformative ideas* to manifest
  (relationships (g), (h), (a)).

## Implications

What are the main implications?

### Acceptance

> Der Wind nimmt die gelben Blätter des alten Ginkgo.
>
> Frei wie Libellen fliegen sie davon.

I have to understand and accept when a burst is coming to its conclusion.
**Things that seem impossibly difficult today
may become trivial in the future**.

I find this very encouraging.

### Make it easy to return

> Stars glow autumn red.
>
> The setting sun retreats from the maple tree.

Accepting that I'll take breaks means that I should
**make it easy to come back**.

There's some analogy to
[counter-cyclical fiscal policy](https://ec.europa.eu/eurostat/statistics-explained/index.php?title=Glossary:Counter-cyclical_fiscal_measures#:~:text=Counter%2Dcyclical%20fiscal%20measures%20are,to%20help%20stimulate%20economic%20recovery.).

To me this is similar to the "two-minute rule" habit formation trick:
leave yourself a lot of crumbs that you can pick up
whenever you happen to have a spare minute or two.
These million small steps tend to add up to meaningful progress.

#### Document Potential Improvements

Documenting my ideas for potential improvements
helps me: it makes it easier to find appropriately sized chunks of work,
depending on where I am in the burst-pause cycle.

I leave many "TODO" notes in the code
listing things that I think I may want to improve some day.
I try too:

* Classify them by difficulty (trivial, easy, normal)
* Give them hash-tags (to help me find related changes)

As I'm reading the code working on implementing a specific feature
unrelated ideas come up.
I'm not going to work on them right away;
I'd never accomplish the task I'm working on.
But I'll make sure to document them right away.

As of 2023-12-21, I have 220 TODO notes in the code.
16 are marked as trivial, 80 are marked as easy.

#### Diversity

I suppose I've been able to keep working on the same project for a decade
because I've been able to work on vastly different sub-projects,
all of which fall under the same umbrella, such as:

* A memory manager.
* A C++-line VM.
* Arbitrary precision numbers.
* Audio generation
  (nothing too fancy, though;
  [ref](https://github.com/alefore/edge/blob/master/src/infrastructure/audio.h)).
* A UX for interacting with my Zettelkasten.
* Futures.
* A Naive Bayes implementation
  (that I use, for example, to order search results
  by their relevance, based on history;
  [ref](https://github.com/alefore/edge/blob/master/src/math/naive_bayes.h))
* Thread pools and work queues.
* A command-line flags module
  ([ref](https://github.com/alefore/edge/blob/master/src/infrastructure/command_line.h)).
* Highly concurrent containers (for very specific use cases;
  [ref](https://github.com/alefore/edge/blob/master/src/concurrent/bag.h))
* tty integration
  (*e.g.,* parsing the escape sequences produced by an underlying shell with
  a pts;
  [ref](https://github.com/alefore/edge/blob/master/src/infrastructure/terminal_adapter.cc)).

### Incremental Improvement

> Gli aghi degli abeti in montagna abbracciano calmi i fiocchi di neve.
> Niente ferma una valanga.

It's easy to underestimate the effect of small changes.
**Changes that, on their own, don't amount to much,
can have very drastic effects as a group**.
I think this is the self-reinforcing loop
of Progress (in one area) enables Progress (in others).
This is a big part of why these bursts happen.
They start as just a few small improvements,
but these small improvements precipitate
and enable many other small improvements and...
eventually they add up to large changes.

The majority of my commits are small "no-op" changes whose only purpose
is "preparation" to make follow-up changes easier or smaller.
A big change is divided into a million baby steps
and one final simple step that enables the new functionality.
I think this is what enables
things that previously seemed ~impossible
to "suddenly… fall gracefully into place".

