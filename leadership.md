# Leadership at Google

TL;DR:
Leadership doesn’t require delegating work to others nor having authority over them (section "The Myth of Leadership").
While leadership has important social aspects, these are incidental, not essential.
Instead, leadership means delivering qualitative improvements of adequate (level-specific) scope and making good decisions on the realm of software (section "What is Leadership").
To improve your leadership, focus on expanding your sphere of ownership, and improving your decision-making and collaboration skills (section "Developing Leadership").

## The Myth of Leadership

I've often heard engineers voice what I consider a misconception about leadership:

* *To get promoted, I need to convince others to implement my designs, take over (a subset of) my work.*
* *I can't display leadership in my current team: I'm the only person working on my project.*
* *How can it be my responsibility to influence teams (or projects) over which I have no formal authority?*

The specific formulation varies depending on the seniority of the engineer (and thus the concrete leadership expectations set on them), but they are all, fundamentally, an expression of the idea that leadership is simply about delegating work to others.
They all betray, in my opinion, a fundamental misunderstanding or too lazy a simplification.
Let's put it to rest:
in order to demonstrate leadership, engineers *don't* need to be TLs (nor managers) *nor* to "get others to do their work".

*Delegates work to others* may be a reasonable indicator/metric of leadership.
However, we should never confuse particular forms of evidence of leadership with leadership itself.
With that in mind, we should optimize for actual leadership behaviors, not for specific metrics.
Furthermore, leadership can be evidenced in many other ways.

Promo committees have often recognized these "gets others to implement their design" signals (and other level-specific formulations) as good indicators/metrics of leadership.
They've done this because they are relatively easy to assess, compared to other signals.
Maybe this is what bore the misconception that leadership is only about work delegation.

These days, the role that leadership plays in our performance evaluation process is less explicit.
Regardless, (1) I've heard this misconception very often and
(2) I'm persuaded that leadership will continue to play an important role in our careers.

## What is Leadership

The word "leadership" has many different meanings.
In my view, leadership, at least at Google, consists of two closely related skills:

1. The ability to **deliver qualitative improvements of adequate scope** (*i.e.*, corresponding to the level).
   These may be improvements to existing systems or to the "status quo"—for example, building new products that improve engineers' or end users' lives.
2. The ability to **make *good* decisions**, both technical and, eventually (at higher levels), organizational.
   What are good decisions?
   Those that enable our software (and organizations) to thrive in the long run.

Other signals such as giving presentations, facilitating training sessions, and coaching/mentoring others are often associated with leadership.
I believe there's no need to include them in my definition explicitly, as I explain below, in [Social Context](1n3.md#skip).

Astute readers may object that this formulation speaks more to impact and difficulty than to leadership.
Nevertheless, despite this arguable overlap with impact and difficulty, it is a robust definition:
it is useful to assess the aforementioned skills (make qualitative improvements and make good decisions) isolated from the dimensions of impact and difficulty.

By my definition, it's almost a tautology to claim that it's useful for engineers to become better at leadership—what is engineering if not the ability to deliver qualitative improvements and make good decisions in specific technical contexts?

### Qualitative

The word "qualitative" is a key part of my definition of leadership.
This formulation requires *transformative* interventions;
incremental or quantitative improvements may provide compelling and appropriate evidence of impact, but may not be evidence of leadership.
You could demonstrate significant impact by driving some key business metric in the right direction but in ways that don’t necessarily show leadership;
perhaps nothing was fundamentally transformed, you just adjusted some parameters and collected a low hanging fruit.

Leadership means, rather, structurally and fundamentally redefining the organizational, technical world around you.
It requires the creativity to see opportunities others may have missed and the initiative and drive to persevere and resolve blockers that would otherwise prevent you (and thus your organization) from realizing its impact.
Leaders challenge the status quo, generate new ideas and see them through fruition.
Leaders persevere.
They take ownership of problem domains and land solutions.

As Chuck Close puts it, “*Inspiration is for amateurs. The rest of us just show up and get to work. If you wait around for the clouds to part and a bolt of lightening to strike you in the brain, you are not going to make an awful lot of work. All the best ideas come out of the process; they come out of the work itself.*”

At some point, a sequence of the aforementioned "just adjusted some parameters" events can start to produce transformative results.
A roofshot becomes a moonshot.
That's the point where interventions become transformative and start indicating leadership.

*Transforming* the world isn't the goal; the goal is transformative *improvements*.
This isn't an exhortation to seek change simply for the sake of change.
The *impact* of your changes must significantly offset their costs.
For example, when replacing a prior lead, a new lead should consider carefully the ideal pace at which to introduce interventions:
the goal is to maximize the positive transformational impact on the organization, not just to create churn.

"The organizational/technical world around you" is somewhat ambiguous.
The thing being transformed could be "the way other engineers work"
(*e.g.*, how they structure their code, how they deal with outages, the team structure of a large organization).
It could also be purely technical things such as:
the way some specific server stack works,
the interface between two low-level systems,
the algorithms a significant fraction of the fleet uses to balance RPC traffic.

### Social Context

The aforementioned examples of leadership can, at least theoretically, be executed by any engineer, without the need to be the TL of any other engineer.
In fact, without even having to directly collaborate with anyone.
Any individual can come up with innovative designs or ideas that deliver good results that fundamentally/structurally transform the world.

However, as the scope grows, awareness of the context in which software exists—including all practices around its design, development and evolution—quickly becomes a necessary condition to make good decisions and lead software to success.
These decisions are often compromises across several conflicting dimensions:
maintainability, generality, usability, resource performance, fast/short-term feature delivery/execution, availability, latency, correctness, security, testability, cost of ownership…
This awareness of the context becomes increasingly important as problem scopes grow.

Obviously, the context in which our software exists includes social aspects.
These social aspects are highly influenced by our technical decisions and vice versa.
Bringing in external perspectives, from many different roles, can be very useful during product development.
To understand what I mean, consider questions such as:

* What emotional responses does your software trigger on its users?
* Are other engineers eager to look under the hood to understand your implementation and send you CLs? Would they rather stay away?
* How much cognitive effort is necessary to make contributions?
* What is the real impact/cost for your customer of delivering/postponing a specific feature request?
* How easy is it for your manager to hire high-performing engineers motivated to develop on the software you created?
* How easy is it for your manager/director to convey the importance of the software you created and procure headcount?

The answers to these and similar questions shape the journey of our software and its odds of success.
It follows that leadership has an unavoidable social aspect.
A problem scope of high enough impact and difficulty has only a very small chance of being solved by a good engineer *working alone*.
But by a good team? Much larger.
Working well in teams makes it easier to consistently deliver results that delight your users.
You say you want to change the world? Make sure you are working in the right team and you work well with your team.

Walter Gropius wrote something similar to this in the context of architecture.
In order for an architect to realize his ideas
"*would require a whole staff of collaborators and assistants: men who would
work, not automatically as an orchestra obeys its conductor's baton, but
independently, although in close cooperation, to further a common cause.*"

However, in my view, this social aspect is an incidental rather than essential part of leadership.
This is why I omit criteria like "works well in a team" or "helps others succeed" from my definition of leadership.
Eventually, as the scope grows, this becomes a necessary condition.
Leadership means delivering qualitative improvements, not impressing your influence on others;
it just so happens that changing the world (or a level-adjusted smaller part thereof) for the better is increasingly difficult if you're unable to understand and/or interact with the social context.
These social aspects are a necessity of the goal, not the goal itself.

Google's goal isn't to include these social aspects in the promotion process;
instead, the goal is simply to align the promotion process (and the role profiles) with the reality in which software exists: what makes it thrive or fail.

### Leadership by Level

Even though this is a very crude simplification,
I'll offer the leadership requirements that I associate with each SWE level at Google.

* **L3**: You execute tactical subtasks that others (typically your TL or manager) give you.
  Leadership requirements are relatively minor:
  mostly, you contribute to the success of projects by tackling some of the operational/menial aspects.
  By doing this, you learn more about the software, freeing up time for more experienced engineers to focus on advanced aspects of the work.

* **L4**: You start owning larger parts of a project.
  You've shown that you can manage yourself and your interactions with the rest of your team reasonably
  (*e.g.*, show up on time, communicate adequately, learn new technologies on your own…).
  At this point, your contributions go beyond executing tasks scoped/defined by others;
  your feedback starts influencing some of the designs.
  For example, you start pointing out problems in specific parts of the larger designs—and proposing solutions.

* **L5**: You now own one medium to large sized project or domain.
  The projects that you can now execute/design/implement/lead, with minimal guidance, are larger in both scope and difficulty.
  In other words, you can now take open ended problems and come up with (and deliver) a good solution.
  "Open ended" may mean that there are many possible solutions that all seem plausible and significant analysis may be needed to evaluate them.
  It may also mean that there is no apparent solution (and significant analysis may be needed to identify one).
  At this point, you've established enough credibility that other engineers, in your team and possibly beyond, are beginning to gravitate towards you for advice for things like design choices.

* **L6**: You've shown significant influence across various medium sized or larger projects.
  Accordingly, you're now beginning to get recognition among a larger organization, beyond just your immediate team, as an expert.
  Typically, other engineers will be executing a medium number of projects under your technical supervision.
  They may (and often will) not be all in your teams, but also in peripheral teams.
  They may be expected to do the majority of the work (including the design work, especially if they are L5 engineers),
  but you'll be the backup that they'll come to when they hit harder problems.
  You are responsible for the success of these projects.
  At this point the sum of the scope of all projects to which you contribute begins to exceed what a single engineer, no matter how brilliant, would be likely to be able to deliver working alone.

* **L7**: You are becoming widely recognized at your Product Area (and/or Google) as the main owner for a medium size domain.
  There is a large number of L5-scope projects that you are overseeing, even if very indirectly;
  when any of these projects begin to stall, you get involved and help the engineers/team.
  At this point you very likely have a very significant influence on the medium- to long-term success of many engineers.
  This influence very likely wielded in a large number of indirect ways.

### Leadership and Authority

You don't need authority over other engineers to demonstrate the conditions of leadership.

Formal authority certainly helps.
But formal authority (*e.g.*, promotions and/or being formally made a manager or TL) is earned, at least at Google, by demonstrating leadership, not the other way around.

Getting promoted to a given level usually boosts your authority—or [the perception that others have of the authority that you have, which is the same thing](https://www.youtube.com/watch?v=4sJY7BTIuPY).
This happens mostly in implicit, silent ways.
To get promoted to the next level, it still is on you to demonstrate enough leadership before earning the next-level authority.

This is true even if you're a manager:

* To be hired into a management position, Google's hiring committee will require evidence of leadership.
* To earn more formal authority (*e.g.*, get promoted, take on more HC), the manager demonstrates leadership, not the other way around.

As you get better at making decisions that enable your software to succeed, it's natural for other engineers, from your team and beyond, to start seeking your guidance;
your manager may ask you more frequently to provide guidance to other engineers.
Leadership is then evidenced indirectly by the trust other engineers place on you.
"Delegates work to others" is only one indicator of leadership;
many others exist—and are visible to promotion committees.
Essentially, going back to my proposed definition, there are many signals that show that you are directly responsible for qualitative improvements of adequate scope.
Promotion committees can get a good sense of this from things like (depending on your level) CL reviews, document comments or email threads, calibration notes, etc..

## Developing Leadership

You want to grow your leadership?
Your focus should be on the underlying driver: the leadership itself.
Don't repeat the mistake of focusing on proxies for leadership.

Providing a detailed guide on how to develop leadership goes beyond the scope of this document.
Nevertheless, I'll offer a few ideas.
To grow your leadership, you could ask yourself how to get better at:

1. **Making good decisions** about your software, in order to boost its positive qualitative impact.
2. **Working well with others**.
   This seems obvious once we accept that well-functioning teams are a prerequisite for delivering large solutions (as described in Social Context).
   "Others" may include your immediate team, a larger group, your entire Product Area, all of Alphabet, or the entire software industry.

### Making Good Decisions

Unfortunately, there are no magic solutions for how to get better at making decisions that affect your software;
it takes a lot of experience to do it well.
It's hard to give generic advice beyond trite platitudes, such as:

* Look for **second order effects** of your decisions.
  These are the indirect consequences of an event/decision.
  These are the consequences of the consequences of the consequences of the consequences of the decision.
  They are very often overlooked.
  The ability to identify them well is a very useful ability.
  Try to create positive feedback loops and/or tackle opportunities that may compound
  (*e.g.*, make investments that will continue to yield dividends over time;
  solve problems today that make it easier for you to solve other problems tomorrow).
* Don't underestimate the **non-linearity of causes and effects**.
  Be aware of situations where small changes in one causal variable may produce disproportionate changes in others.
* Get better at **forecasting**.
  Exploring this is obviously beyond the scope of this text.
* Consult with a **mentor**.
  Experience, by definition, can't be obtained simply through reading and/or conversations;
  however, being able to consult with a coach or advisor can vastly accelerate your thinking process.
  You are unique in a million ways but odds are the challenges you're solving are variations of patterns other engineers have attacked before you.
  As the saying goes, in China, if you're one in a million, there's still thousands of people just like you.
  A good mentor that has already solved challenges of the scope you're facing can quickly help you avoid pitfalls by showing you your blind spots and helping you refine your guiding principles.
  You effectively don't have a mentor? You're likely wasting time;
  ask your manager to help you find a mentor.
* Try to **separate the essential from the incidental**.
  Get better at asking yourself:
  "Is this constraint that I'm facing a fundamental/intrinsic characteristic of the problem I'm trying to solve,
  or is it just a circumstance that, ultimately, I could change?"
* Master the **time aspects** of making a decision.
  How urgent is it to make the decision?
  Make sure you make decisions at the right time.
  When you can afford to wait, don't rush to decide before you've gathered enough information.
  On the other hand, don't postpone making nor implementing a decision when the costs (*e.g.*, opportunity costs) exceed the gains (which often have diminishing returns).

Additional information:

* ["Decision Time" by Laurence Alison & Neil Shortland](https://www.penguin.co.uk/books/442687/decision-time-by-shortland-laurence-alison-and-neil/9781785043611)

### Ownership

One very important aspect of leadership is ownership.
Most leadership requirements I formulated for each level boil down to "owns larger projects".
The size is relative to both impact and difficulty.

The scope of the projects that a senior engineer owns is expected to exceed what a good engineer working alone could deliver.
In my experience, this happens towards the upper end of L5 or the lower end of L6.
As a result, a senior engineer will need to own a set of projects where a large part of the execution will be done by others.

One implication of that is that **successful delegation matters**.
Suppose you've already demonstrated significant leadership and a group of engineers, in your team or beyond, are eager to "take over" some of your work;
even then, it can still be very hard to successfully transfer responsibilities and enable their success.
The key question is: how do you gradually scale back your direct involvement and ensure that the project expectations are still met?
Many books have been written on this, so I'm leaving it out of scope.

Additional information:

* [To Be a Great Leader, You Have to Learn How to Delegate Well](https://hbr.org/2017/10/to-be-a-great-leader-you-have-to-learn-how-to-delegate-well)

#### Expectations and Recognition

How do we allocate credit (for success) and blame (for failure) fairly?

To make this concrete, imagine you are an L6 engineer in a team consisting of:

* Three L4 engineers
* One L5 engineer
* One L6 engineer (you!)

If this team sets out to execute a specific project, level-appropriate subtasks should be reflected in the expectations of the junior engineers that will execute them.
Some important design tasks may end up listed as the expectations for the L5 engineer.
To make things interesting, assume that the project is forecasted to land with minimal involvement from you.

However, if the project fails, should that be considered your failure?
There are two options:

* The project is *not* your responsibility.
  We don't expect you to be involved anyway.
  If the project fails, you shouldn't even bat an eye…
  This may be the outcome of the prioritization/planning (*i.e.*, we'd rather let the project fail than distract you from more important work).

* The project is your responsibility.
  If the project stalls, you are expected to step in and support the junior engineers, seeing them through to success;
  if the project fails, *you* are failing (to meet your expectations).
  If, on the other hand, the project succeeds, you are succeeding and deserve recognition, *even if you ended up not having anything to do with the project directly*.

The choice should be codified in your expectations.

Regardless of this choice, your L6 expectations likely reflect that you should be influencing the team significantly.
You'll be increasing the probability of success (impact, delivery, quality, etc.) for this and other projects the team invests in.
This doesn't change simply because you're not expected to participate in some projects *directly*;
your influence is exerted, often in indirect/invisible ways, through things like career guidance, long-term planning and technical direction setting, and mentorship.
In this sense, even in that case where the project is not your *direct* responsibility, it's likely that you'll still deserve some recognition for its success.

None of this assumes authority—you don't need to be a manager nor TL;
your expectations in terms of leadership are entirely orthogonal to that.

#### Resolving Blockers

One important aspect of ownership that is sometimes neglected is focusing on proactively ensuring that blockers get addressed or escalated.

As the owner of a given project, you are responsible for ensuring that all blockers get resolved appropriately (timely, with quality, etc.).
This is true *even for blockers that depend on other engineers or teams*, even after they have been assigned to them explicitly.
When blockers don't get resolved, blaming the project failure/delays on other engineers or teams…
doesn't change the fact that you have failed to fulfil your responsibility/expectations:
the project you own still failed/stalled.

The main skills for this all have to do with working well with other engineers (*e.g.*, knowing when/how to use email pings, chat, meetings, etc., effectively; using tools like joint OKRs).
Another important tool is escalation—ideally starting with your manager, who may have ideas to help you unblock things.

### Working Well With Others

More unfortunate news…
there are also no silver bullets for how to work well in a team.
It takes a lot of experience to do this well.

The good news is that deliberate effort can yield drastic improvements here, which, in my experience, can have a dramatic impact on most engineer's careers.
A lot of this comes simply from paying attention to the following aspects and developing corresponding skills.
Work with your mentor to identify concrete steps you can take to get better at things like:

* **Foster psychological safety**.
  You want open feedback and connection.
  For example, when a peer drops the ball, you want to be able to tell them directly how it affected you and the project,
  without any negative/destructive emotions/intentions on either side,
  in a way that supports them (see below).
  You want others to fell comfortable telling you when you could have done better.
* **Negotiate well**.
  Seek to understand others and find common ground with them, rather than "win" the argument.
  The way I think of this is:
  (1) understand well the needs of others
  (2) understand well your own needs (and don't confuse them with proposed solutions)
  (3) understand the possibilities your technical/organizational reality affords you, and
  (4) be creative enough to build solid and elegant bridges that satisfy both.
* **Communicate well**.
  Communication is our main portal to conveying ideas and connecting with others.
  Try these:
  * Avoid ambiguities and use consistent terminology.
  * Be explicit when it pays to be explicit
    (*e.g.*, call out AIs in meeting notes);
    and succinct when it pays to be succinct
    (*e.g.*, don't include unnecessarily long "Background" sections in your design docs).
  * Adjust your writing to match your target audience.
    Start by identifying your target audience.
  * Understand that communicating clearly requires thinking clearly;
    you can't speak very well if you don't know what you're trying to say.
  * Be deliberate about when to be "transactional" and when to be "connection oriented";
    this will help you become more effective at both.

Additional information:

* [Debugging Teams](https://www.debuggingteams.com/), in particular the Humility, Respect, Trust model for building effective teams.

#### Delivering Feedback Well

When you offer feedback to others, do it in such a way that you support them, rather than hamper them.
Your feedback should lift them, moving them closer to their goal.
Unfortunately, feedback carelessly given often has the opposite effect:
it encumbers its recipients, sinking them towards the starting point, away from their goal.

The reason to do this is to become a more effective team player;
this also, incidentally (and less importantly), happens to help indicators of leadership (*e.g.*, lead others to seek your guidance more frequently).

Suppose an engineer sets out to solve a difficult technical problem.
They write a design, proposing a possible solution.
The design gets them half of the way to the ideal scenario, where the solution is fully landed.
If they come to ask you for feedback, ask yourself: "How can I engage in such a way that our interactions will advance them towards the solution?"
If you can't figure this out (and, alas, these things happen all too often), it might be even better to just hold back your feedback altogether—that is, to just say that you don't have anything of value to offer.

One mistake that I've seen often is bringing up irrelevant aspects ("your proposal could be improved if you considered the following …") and allowing them to serve as distractions.
This often forces the author of the proposal to waste time exploring orthogonal (or less relevant) aspects, delaying their work.
Another formulation of this anti-pattern is that the perfect is the enemy of the good.
We are all too eager to see the speck in our sister’s design doc, but not notice the horrible bugs in our own.

This isn't an exhortation to avoid delivering critical or "negative" feedback.
Far from it.
Constructive feedback often fosters long term growth, even when it's painful.
This can justify the short term pain/stress it may cause.
In any case, make sure you:

* Make a deliberate decision, based on the expected net effect.
* Do what you can to deliver your feedback in a way that promotes psychological safety.

Additional information:

* [For better commenting, avoid PONDS](https://www.lesswrong.com/posts/k5TTsuHovbeTWgszD/for-better-commenting-avoid-ponds)
* [Situation-Behavior-Impact Feedback Framework](https://medium.com/pm101/the-situation-behavior-impact-feedback-framework-e20ce52c9357)
* [Crucial Conversations: Tools for talking when stakes are high](https://www.mhprofessional.com/crucial-conversations-tools-for-talking-when-stakes-are-high-9780071415835-usa)

##### Feedback: Humility

There's a logical fallacy where, when we're unable to explain, justify, or establish a proposition, we often assume incorrectly that the proposition must be false.
But absence of evidence is not evidence of absence.

When we can't explain an action/behavior by a peer, this fallacy often makes us *incorrectly* assume that our peer must be making a mistake.
Therefore, when giving feedback to our peers, perhaps to help them notice a blind spot in their behavior or a flaw in a technical proposal, we must do so with humility;
we must remember that we are likely missing information.

##### Express Gratitude

If you see someone produce something good, take the time to let them know you appreciate their work.
The effects this tends to have on the recipients tend to be disproportionately larger than the costs.

Make sure you are using peer bonuses and similar tools adequately.
In my experience, they are severely underused in most organizations.

#### Fostering Growth

One important component of working well with others, especially for senior engineers, is to enable more junior engineers to grow and succeed in the long term.
I consider this is a reasonable expectation for L6 engineers (not only for managers).
The set of skills required to do this effectively are far from trivial.

I mentioned earlier the importance of working in the right team.
If you are *not* working in the perfect team, you can keep on searching for the perfect team (as if that existed!), or you can… perfect your already pretty-good team!
Over time, the team you're in becomes a reflection of who you are, especially as your leadership grows.

I've covered delegation briefly (in section Ownership), but this goes further.
One framing that I've often found helpful is that the goal of the senior engineer is that the junior engineer (or group thereof) successfully delivers a specific project.
This means that if the project fails, the senior engineer is failing in their goal;
however, it also means that, to the extent that the senior engineer becomes involved in the execution of the project, they are also failing in their goal (whether the project fails or succeeds is irrelevant).

The senior engineers must carve enough space for the junior engineers to experiment on their own.
This means allowing the junior engineers to take risks, which implies accepting failure.
This is necessary in order for the junior engineers to learm (possibly from their own mistakes) and… start demonstrating leadership.

Venturing safely beyond the confines of our comfort zone fosters growth.
As a senior engineer you have to identify the comfort zone of junior engineers in your team and match them with projects (and provide them the right level of support) that stretches them and fosters growth.
You have to avoid matching engineers with overly ambitious assignments that lay too far beyond their current capabilities;
this stretches them too thinly and causes failure, stress, frustration and, possibly, burn out.

Additional information:

* [Situational Leadership Theory](https://en.wikipedia.org/wiki/Situational_leadership_theory)

### Beginning your Journey

A few specific ideas to begin your leadership journey:

* Host an intern.
* Become a mentor or help a junior engineer ramp up.
* Become a readability reviewer.
* Offer presentations on upcoming conferences.
* Claim ownership over important but possibly neglected work that you could execute in your team.

## Summary

I hope I've offered a compelling articulation for why signals like delegating work to others are only that: signals of leadership, not to be confused with the leadership itself.
I hope I’ve persuaded you that formal authority should follow, not precede, leadership itself;
and that the social context in which our software exists and the ability to collaborate well doesn't need to be part of our definition of leadership but that it is, nevertheless, a necessary condition, at least above a certain scope.

It is also my hope that following the concrete proposals I've made will help you on your journey to develop leadership;
that it’ll bring you closer to being able to deliver larger qualitative improvements to the lives of the users of your software.

Good luck!

