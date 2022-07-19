# Leadership at Google

## The Myth of Leadership

I've often heard Googlers voice what I consider a misconception about leadership: "To get promoted, I need to convince others to implement my designs, take over (a subset of) my work."
Or: "I can't display leadership in my current team: I'm the only person working on my project."
Or, perhaps: "How can it be my responsibility to influence teams (or projects) over which I have no formal authority?"
The specific formulation varies depending on the level of the Googler (and thus the concrete leadership expectations set on them), but they are all, fundamentally, the same.

This betrays, in my opinion, a fundamental misunderstanding or too lazy a simplification.
Let's put it to rest:
in order to demonstrate leadership, Google engineers don't need to be TLs (nor managers) nor to "get others to do their work".

"Delegates work to others" can be a reasonable indicator/metric of leadership.
However, we should never confuse particular forms of evidence of leadership with leadership itself.
Said differently, we should optimize for the root cause, not for specific metrics.
Furthermore, leadership can be evidenced in many other ways.

Promo committees often recognize these "gets others to implement his design" signals (and other level-specific formulations) as good indicators/metrics of leadership.
They do this because they are relatively easy to assess, compared to other signals.
Maybe this is how this misconception arose.
Regardless, I've heard this misconception so often that I figured it was time to share my opinion on this through Eng News.


## What is Leadership

The word "leadership" has many different meanings.
In my view, leadership at Google consists of two closely related skills:

1. The ability to deliver qualitative improvements of adequate (i.e., level-adjusted) scope.
2. The ability to make *good* technical (and, eventually, organizational) decisions.
   What are good decisions?
   Those that enable our software (and organizations) to thrive in the long run.

Astute readers may object that this formulation speaks more to impact and difficulty than to leadership.
Nevertheless, this serves as a robust definition, despite this overlap.
I think it is useful to assess these abilities in isolation from the two other dimensions, as part of Google's promotion process.

It's almost a tautology to claim that it's useful for Googlers to become better at this
—what is engineering if not the ability to deliver qualitative improvements and make good decisions, at least in this specific/technical context?

### Qualitative

The word "qualitative" is a key part of my definition of leadership.
This formulation requires *transformative* interventions;
incremental/quantitative improvements may provide compelling/appropriate evidence of impact, but may not satisfy leadership expectations.
You could demonstrate significant impact by driving some key business metric in the right direction but in ways that don’t necessarily show leadership;
perhaps nothing was fundamentally transformed, you just adjusted some parameters.

Leadership means, rather, structurally/fundamentally redefining the organizational/technical world around you.
It requires the creativity to see opportunities others may have missed and the initiative and drive to persevere and resolve blockers that would otherwise prevent you (and thus Google) from realizing its impact.
Leaders challenge the status quo, generate new ideas, and see them through fruition.
They take ownership of problem domains and land solutions.

"The organizational/technical world around you" is somewhat ambiguous.
The thing being transformed could be "the way other engineers work"
(e.g., how they structure their code, how they deal with outages, the team structure of a large organization, etc.).
It could also be purely technical things such as:

* The way some specific server stack works.
* The interface between two low-level systems.
* The algorithms used to balance RPC traffic by a significant fraction of the fleet.

You don't need formal authority over other engineers to execute any of this.
Formal authority certainly helps.
But formal authority (e.g., promotions and/or being formally made a manager or TL) is earned at Google by demonstrating leadership, not the other way around.
Getting promoted to a given level usually boosts your authority —or the perception that others have of the authority that you have, which is the same thing.
This happens mostly in implicit, silent ways.
To get promoted to the next level, it still is on you to demonstrate enough leadership before you earn the next-level authority.

### Social Context

The aforementioned examples of leadership can, at least theoretically, be executed by any engineer, without the need to be the TL of any other engineer;
in fact, without even having to directly collaborate with anyone.
You can come up with innovative designs or ideas that deliver good results that fundamentally/structurally transform the world.

However, as the scope grows, awareness of the context in which software exists —including all practices around its design, development and evolution— quickly becomes a necessary condition to make good decisions and enable software's success.
These decisions are often compromises across several conflicting dimensions:
maintainability, generality, usability, resource performance, fast/short-term feature delivery/execution, availability, latency, correctness, security, testability, cost of ownership…
As problem scopes grow, this awareness becomes increasingly important.

Obviously, the context in which our software exists includes, eventually, social aspects.
For example:

* What emotional responses does your software trigger on its users?
* Are other Google engineers eager to look under the hood to understand your implementation and send you CLs? Would they rather stay away?
* How much cognitive effort is necessary to make contributions?
* What is the real impact/cost for your customer of delivering/postponing a specific feature request?
* How easy is it for your manager to hire high-performing engineers motivated to develop on the software you created?
* How easy is it for your manager/director to convey the importance of the software you created and procure headcount?

Our technical decisions strongly shape these social aspects.

That means that leadership has an unavoidable social aspect.
Lone engineers can only exceptionally solve problems above a certain difficulty/scope/size.
In other words, the problem scope (difficulty, impact, …) that can be predictably expected to be solved by a good engineer working alone is small.
A good team? Much larger.
Working well in teams makes it easier to consistently deliver results that delight your users.
You say you want to change the world? Make sure you are working in the right team and you work well with your team.

However, in my view, this social aspect is an incidental rather than essential part of leadership.
This is why I omit criteria like "works well in a team" or "helps others succeed" from my definition of leadership.
Eventually, as the scope grows, this becomes a necessary condition.
Leadership does not mean "influencing others", rather "delivering qualitative improvements";
it just so happens that changing the world (or a level-adjusted smaller part thereof) for the better is increasingly difficult if you're unable to understand and/or interact with the social context.
These social aspects are a necessity of the goal, not the goal itself.

Google's goal isn't to include these social aspects in the promotion process;
instead, the goal is simply to align the promotion process (and the role profiles) with the reality in which software exists: what makes it succeed or fail.

### Leadership by Level

Even though this is a very crude simplification,
I'll offer the leadership requirements that I associate with each SWE level at Google.

* **L3**: You execute tactical subtasks that others (typically your TL or manager) give you.
  Leadership requirements are relatively minor:
  mostly, you contribute to the success of projects by tackling some of the operational/menial aspects.
  By doing this, you enable more experienced engineers to focus on the more difficult parts of these problems.

* **L4**: You start owning larger parts of a project.
  You've shown that you can manage yourself and your interactions with the rest of your team reasonably
  —e.g., show up on time, communicate adequately, learn new technologies on your own…)
  At this point, your contributions go beyond executing tasks scoped/defined by others;
  your feedback starts influencing some of the designs.
  For example, you start pointing out problems in specific parts of the larger designs —and proposing solutions.

* **L5**: You now own one medium/large sized project/domain.
  The projects that you can now execute/design/implement/lead, with minimal guidance, are both larger in scope and difficulty.
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

## Developing Leadership

You want to grow your leadership?
Your focus should be on the root cause: the leadership itself.
Don't repeat the mistake many have made of focusing on proxies for leadership.

As you get better at making decisions that enable your software to succeed, it's natural for other engineers, from your team and beyond, to start seeking your guidance;
your manager may ask you more frequently to provide guidance to other engineers.
Leadership is then evidenced indirectly by the trust other engineers place on you.
"Delegates work to others" is only one indicator of leadership; many others exist —and are visible to promotion committees.

Providing a detailed guide on how to develop leadership goes beyond the scope of this document.
Nevertheless, I'll offer a few ideas.
To grow your leadership, you could ask yourself how to get better at:

1. Making good decisions that affect your software.
2. Working well with others.
   This follows directly once you've accepted that well-functioning teams (the aforementioned "social context") are a prerequisite for delivering large solutions.
   "Others" can be your immediate team, a larger group, your entire Product Area, Google as a whole, or the entire software industry.

### Making Good Decisions

The bad news is that there are no silver bullets for how to get better at making decisions affecting your software:
it takes a lot of experience to do it well.
It's hard to give generic advice beyond trite platitudes, such as:

* Look for **second order effects** of your decisions.
  These are the indirect consequences of an event/decision.
  These are the consequences of the consequences of the consequences of the consequences of the decision.
  They are very often overlooked.
  The ability to identify them well is a very useful ability.
  Try to create positive feedback loops and/or tackle opportunities that may compound.
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
  You effectively don't have a mentor? You're likely wasting time.
* Try to **separate the essential from the incidental**.
  Get better at asking yourself:
  "Is this constraint that I'm facing a fundamental/intrinsic characteristic of the problem I'm trying to solve,
  or is it just a circumstance that, ultimately, I could change?"

### Ownership

One very important aspect of leadership is ownership.
Most leadership requirements I formulated for each level boil down to "owns larger projects".
The size is relative to both impact and difficulty.

The scope of the projects that a senior engineer owns is expected to exceed what a good engineer working alone could deliver.
In my experience, this happens towards the upper end of L5 or the lower end of L6.
As a result, a senior engineer will need to own a set of projects where a large part of the execution will be done by others.

One implication of that is that successful delegation is an important skill.
Suppose you've already demonstrated significant leadership and a group of engineers, in your team or beyond, are eager to "take over" some of your work;
even then, it can still be very hard to successfully transfer responsibilities and enable their success.
The key question is: how do you gradually scale back your direct involvement and ensure that the project expectations are still met?
Many books have been written on this, so I'm leaving it out of scope.

#### Expectations and Recognition

How do we allocate credit (for success) and blame (for failure) fairly?

Let's make this concrete.
Imagine a team consisting of:

* Three L4 engineers
* One L5 engineer
* One L6 engineer

If this team sets out to execute a specific project, level-appropriate subtasks should be reflected in the expectations of the junior engineers that will execute them.
Some important design tasks may end up listed as the expectations for the L5 engineer.
To make things interesting, assume that the project is forecasted to land with minimal involvement from the L6 engineer.

However, if the project fails, should that be considered a failure of the L6 engineer?
If so, it should be reflected in the expectations for the L6 engineer.
If we want the L6 to be responsible for the success of the project, we need to make sure that he'll receive credit for it, our forecast notwithstanding.

In doing so, we are codifying in the expectations that if something unforeseen happens, the L6 is expected to step in and support the junior engineers, seeing them through to success.

More interestingly, we're also codifying that the L6 engineer should be influencing the team significantly, increasing the probability of success for the project.
This doesn't change simply because he's not expected to participate in the project *directly*;
this influence is exerted in indirect/invisible ways, through things like career guidance, long-term planning and technical direction setting, and mentorship.
None of this assumes authority —the L6 engineer doesn't need to be a manager nor TL;
the expectations in terms of leadership on the L6 are entirely orthogonal to that.

#### Resolving Blockers

One important aspect of ownership that is sometimes neglected is focusing on proactively ensuring that blockers get addressed or escalated.

As the owner of a given project, I remain responsible for ensuring that all blockers get resolved appropriately (timely, with quality, etc.).
This is true even for blockers that depend on other engineers or teams, even after they have been assigned to them explicitly.
If they don't, blaming the project failure/delays on other engineers or teams won't change the fact that I have failed to fulfill my responsibility/expectations:
the project I own still failed.

The main skills for this all have to do with working well with other engineers (e.g., knowing when/how to use email pings, chat, meetings, etc., effectively; using tools like joint OKRs).
Another important tool is escalation —ideally starting with your manager, who may have ideas to help you unblock things.

### Working Well with Others

More unfortunate news: there are also no silver bullets for how to work well in a team:
it also takes a lot of experience to do this well.

The good news is that deliberate effort can yield drastic improvements here, which, in my experience, can have a dramatic impact on most engineer's careers.
A lot of this comes simply from paying attention to these aspects.
Work with your mentor to identify concrete steps you can take to get better at things like:

* Communicate well.
  Avoid ambiguities.
  Be explicit when it pays to be explicit.
  And succinct when it pays to be succinct.
  Adjust your writing to match your target audience.
  Identify your target audience, to begin with.
  Understand that communicating clearly requires thinking clearly;
  you can't speak very well until it becomes clear what to say.
* Foster psychological safety.
  You want open feedback and connection.
  For example, when a peer does a terrible job, you want to be able to tell them directly (and why),
  without any negative/destructive emotions/intentions on either side,
  in a way that supports them (see below).
  You want others to tell you when/why you've done a terrible job, also without any negative emotions.
* Negotiate well.
* Understand in which conversations/meetings to be "transactional" and when to be "connection oriented".
  Become effective at both.

#### Delivering Feedback Well

When you offer feedback to others, do it in such a way that you support them, rather than hamper them.
Your feedback should lift them, moving them closer to their goal.
Unfortunately, feedback carelessly given often has the opposite effect:
it encumbers its recipients, sinking them towards the starting point, away from their goal.

The reason to do this is to become a more effective team player;
this also, incidentally (and less importantly), happens to help indicators of leadership (e.g., lead others to seek your guidance more frequently).

Suppose an engineer sets out to solve a difficult technical problem.
They write a design, proposing a possible solution.
The design gets them half of the way to the ideal scenario, where the solution is fully landed.
If they come to ask you for feedback, ask yourself: "How can I engage in such a way that our interactions will advance them towards the solution?"
If you can't figure this out (and, alas, these things happen all too often), it might be even better to just hold back your feedback altogether
—that is, to just say that you don't have anything of value to offer.

One mistake that I've seen often is bringing up irrelevant aspects ("your proposal could be improved if you considered the following …") and allowing them to serve as distractions.
This often forces the author of the proposal to waste time exploring orthogonal (or less relevant) aspects, delaying their work.
Another formulation of this anti-pattern is that the perfect is the enemy of the good.
We are all too eager to see the speck in our sister’s design doc, but not notice the horrible bugs in our own.

This isn't an exhortation to avoid delivering critical or "negative" feedback.
Far from it.
"Painful" feedback can often foster long term growth, which can justify the short term pain/stress it may induce.
In any case, make sure you make a deliberate decision, based on the expected net effect.

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

