# Lean Prover

Lean Prover is an open source Formal Verification system.

## Formal Verification

Formal Verification systems are software systems that introduce formalism to
help us offer rigorous proofs about characteristics of other systems.

### How to write Correct Systems

Techniques to write correct systems include:

* Explicit Invariants
* Detect Errors Early
* Formal Verification

#### Software Correctness

Software is correct to the extent that its implementation fulfils its contract
and it's free of software defects (bugs).

## Types

### Pi Types

A function `f` has the type `Π a : α, β a` if for each `a : α`, `f a` is an
element of type `β a`. The type of the value returned by `f` depends on its
input.

`α → β` is just notation for `Π a : α, β` when `β` does not depend on `a`.

## Strategies

To formally express a mathematical assertion in the language of dependent type
theory, we need to exhibit a term `p : Prop`. To prove that assertion, we need
to exhibit a term `hp : p`. Lean’s task, as a proof assistant, is to help us to
construct such a term, `hp`, and to verify that it is well-formed and has the
correct type.

### Implication

To prove an implication statement of the form

    p → q ,

for a given `p` and `q` of type `Prop`, implement a function that receives a
value `hp` of the antecedent type `p` and manufacturs (and returns) a value
`hq`of the consequent type `q`.

You'd use it thus:

    example : p → q :=
    λ hp : p, ... body evaluating to hq ...

### Modus Ponens

As explained in Lean Prover: Implication, a proof of a proposition of
the form `p → q` is simply a function that receives a value `hp : p` and returns
a value of type `q`.

That means that if we have both a value `hpq : p → q` and a value `hp : p` we
can obtain a value of type `q` simply by executing `hpq` on `hp`:

    hpq hp

### Double Implication

To prove a double implication proposition of the form

    p ↔ q ,

for a given `p q : Prop`, use the `iff.intro` function. It receives two
arguments, one proving `p → q` and one proving `q → p` and returns a value `hpq
: p ↔ q`.

We can use one such `hpq` value through the `iff.elim_left` or the
`iff.elim_right` functions, which return one or the other of these functions.

For example, given `hpq : p ↔ q` and `hq : q`, we can obtain `hp : p` through:

    (iff.elim_right hpq) hq

This is equivalent to:

    hpq.elim_right hq

### Negation

If `p` is a value of type `Prop`, the value `¬p` (also of type `Prop`) is
equivalent to the value `p → false`.

To prove a statement of this form (i.e., to generate a value of type `¬p`), we
implement a function that receives a value `hp` of type `p` and produce a value
of type `false` (where `false` has type `Prop`).

There's a special rule `false.elim` which encodes the principle that anything
can be proven from `false`. This principle infers the expected output type. More
specifically, it receives as an argument a value `hf : false` and can returns a
value for any given type `t : Prop`. For example:

    example : false → p := false.elim

This rule is sometimes called ex falso (short for *ex falso sequitur
quodlibet*), or the principle of explosion.

Additionally, the `absurd` keyword can be used as a short form for this. If we
have a value `hp : p` and a proof of `hnp : ¬p`, we can simply write:

    absurd hp hnp

Equivalently to `false.elim (hnp hp)`, this function will be able to create a
value for any expected type `p : Prop`.

### Conjunction

The "and" constructor is a functor that consumes two values of type `Prop` and
produces a third, such as `p ∧ q`.

To create a value with type `p ∧ q` (which has type `Prop`) given two values `hp
: p` and `hq : q` we use the `and.intro` function:

    and.intro hp hq

This is common enough that there's an alias for it:

    ⟨hp, hq⟩

The converse operation is provided through the `and.left` and `and.right`
functions: given a value `hpq` of type `p ∧ q` (for some values `p` and `q` of
type `Prop`), one can obtain values `hp : p` and `hq : q` thus:

    and.left hpq
    and.right hpq

This is equivalent to:

    hpq.left
    hpq.right

### Disjunction

The "or" constructor is a functor that consumes two values of type `Prop` and
produces a third, such as `p ∨ q`.

Given a value `hp : p`, we can create a value `hpq : p ∨ q` for any value `q`
(of type `Prop`) thus:

    or.inl hp

This infers the type `q` from the context.

We can also create a value of the same type given a value `hq : q`:

    or.inr hq

Given a value `hpq : p ∨ q` and another proposition value `r : Prop`, we can
generate a value `hr : r` if we have means (i.e., functions) to construct `hr`
from both `hp : p` and `hq : q`. To do this, we use `or.elim` and pass the two
functions:

    or.elim hpq (λ hp, ...) (λ hq, ...)

This is equivalent to:

    hpq.elim (λ hp, ...) (λ hq, ...)

### Classical Logic

You can open the classical namespace in order to use the rule of the excluded
middle:

    open classical

The rule of the excluded middle states that for every possible `p : Prop`, the
proposition `p ∨ ¬p` holds. By evaluating the `em` function (e.g., `em p`) one
can obtain a value `hpb` of type `p ∨ ¬p`.

This is useful together with `or.elim` in order to divide the proof into cases.

    or.elim (em p)
      (λ hp : p, ...)
      (λ hnp : ¬p, ...)

As an alias, one can simply use `by_cases` and let Lean infer the type of the
proposition value (`p` in our example) to be generated:

    by_cases
        (λ hp : p, ...)
        (λ hnp : ¬p, ...)

One can also use the `by_contradiction` keyword when the second of the cases
leads to a contradiction.

    by_contradiction (λ hnp : ¬p, absurd ...)

#### Examples

For example, suppose that we want to prove `q` and we
have functions `hpq : p → q` and `hnnp : ¬¬p` . We can write:

    by_cases
        (λ hp : p, hpq hp)
        (λ hnp : ¬p, absurd hnp hnnp)

We can simplify this to the equivalent:

    hpq (by_contradiction (λ hnp : ¬p, absurd hnp hnnp))

Or even:

    hpq (by_contradiction (compose false.elim hnnp))

### Quantifiers

#### ∀

Given a proposition `h` with type `∀ x : T, p x` and an object `hx` of type `x`,
we can obtain a value of type `p x` by using it as a function.

To provide a "for all" proposition like `h` above, it suffices to implement one
function that receives an object `hx` and returns an instance of the property
(`p x`).

#### ∃

Given a proposition `h` with type `∃ x : T, p x`, we can use `match` to obtain:

* An object `hx` of type `x`.
* A value of type `p x`.

Example:

    match h with ⟨hx, hxr⟩ := ... end

Alternatively, we can use `exists.elim`:

    exists.elim h (λ (hx : x) (hpx : p x), ...)

We can produce a proof of such a proposition by providing one such value `hx`
along with the proof that `p x` holds for it:

    ⟨hx, hpx : p x⟩

This is equivalent to:

    exists.intro hx hpx

### Chains of Equality

To offer proof of equality propositions we can use the `calc` keyword, beginning
with one of the sides of the equality proposition and gradually refining it,
applying previously proven theorems, until we end on the other side.

When we do this, the `rw` strategy can be very useful to avoid having to specify
a lot of lower-level details (letting Lean take care of that for us).

## Exercises

### Intuitionist Logic

The Intuitinist Logic exercises correspond to exercises 1 and 3 from chapter 3
from the Theorem Proving in Lean book. These proofs don't use the principle of
excluded middle.

#### Preface

    variables p q r : Prop

    def compose {p0 p1 p2: Prop} (g : p1 → p2) (f : p0 → p1) : p0 → p2 :=
    λ a : p0, g (f a)

    def identity {p0 : Prop} := λ a : p0, a

#### Commutativity of ∧ and ∨

The following are demonstrations of commutativity of ∧ and ∨:

    example : p ∧ q ↔ q ∧ p :=
    iff.intro (λ h, ⟨h.right, h.left⟩) (λ h, ⟨h.right, h.left⟩)

    example : p ∨ q ↔ q ∨ p :=
    iff.intro
      (λ h, h.elim or.inr or.inl)
      (λ h, h.elim or.inr or.inl)

#### Associativity of ∧

    example : (p ∧ q) ∧ r ↔ p ∧ (q ∧ r) :=
    iff.intro
      (λ h, ⟨h.left.left, ⟨h.left.right, h.right⟩⟩)
      (λ h, ⟨⟨h.left, h.right.left⟩, h.right.right⟩)

#### Associativity of ∨

    example : (p ∨ q) ∨ r ↔ p ∨ (q ∨ r) :=
    iff.intro
      (λ hpqr : (p ∨ q) ∨ r,
       hpqr.elim
         (λ hpq : p ∨ q, or.elim hpq or.inl (compose or.inr or.inl))
         (compose or.inr or.inr))
      (λ hpqr : p ∨ (q ∨ r),
       hpqr.elim
         (compose or.inl or.inl)
         (λ hqr : q ∨ r, hqr.elim (compose or.inl or.inr) or.inr))

#### Distributivity

    example : p ∧ (q ∨ r) ↔ (p ∧ q) ∨ (p ∧ r) :=
    iff.intro
      (λ hpqr : p ∧ (q ∨ r),
       have hp : p, from hpqr.left,
       hpqr.right.elim
         (compose or.inl (and.intro hp))
         (compose or.inr (and.intro hp)))
      (λ hpqpr : (p ∧ q) ∨ (p ∧ r),
       hpqpr.elim
         (λ hpq : p ∧ q, ⟨hpq.left, or.inl hpq.right⟩)
         (λ hpr : p ∧ r, ⟨hpr.left, or.inr hpr.right⟩))

    example : p ∨ (q ∧ r) ↔ (p ∨ q) ∧ (p ∨ r) :=
    iff.intro
      (λ hpqr : p ∨ (q ∧ r),
       or.elim hpqr
         (λ hp : p, ⟨or.inl hp, or.inl hp⟩)
         (λ hqr : q ∧ r, ⟨or.inr (and.left hqr), or.inr (and.right hqr)⟩))
      (λ hpqpr : (p ∨ q) ∧ (p ∨ r),
       hpqpr.left.elim
         or.inl
         (λ hq : q,
          hpqpr.right.elim or.inl (λ hr : r, or.inr ⟨hq, hr⟩)))

#### Negation

    example : ¬(p ∨ q) ↔ ¬p ∧ ¬q :=
    iff.intro
      (λ hnpq : ¬(p ∨ q), ⟨compose hnpq or.inl, compose hnpq or.inr⟩)
      (λ (hnpq : ¬p ∧ ¬q) (hpq : p ∨ q), hpq.elim hnpq.left hnpq.right)

    example : ¬p ∨ ¬q → ¬(p ∧ q) :=
    λ (hnpnq : ¬p ∨ ¬q) (hnpq : p ∧ q),
    hnpnq.elim (λ hnp : ¬p, hnp hnpq.left) (λ hnq : ¬q, hnq hnpq.right)

    example : ¬(p ∧ ¬p) := λ h : p ∧ ¬p, h.right h.left

    example : p ∧ ¬q → ¬(p → q) :=
    λ (hpnq : p ∧ ¬q) (hpq : p → q), hpnq.right (hpq hpnq.left)

    example : ¬p → (p → q) := λ (hnp : ¬p) (hp : p), absurd hp hnp

    example : (¬p ∨ q) → (p → q) :=
    λ (hpq : ¬p ∨ q) (hp : p), hpq.elim (absurd hp) identity

    example : p ∨ false ↔ p :=
    iff.intro (λ h : p ∨ false, h.elim identity false.elim) or.inl

    example : p ∧ false ↔ false := iff.intro and.right false.elim

    example : ¬(p ↔ ¬p) :=
    λ h : p ↔ ¬p,
    have hnp : ¬p, from (λ hp : p, (h.elim_left hp) hp),
    hnp (h.elim_right hnp)

    example : (p → q) → (¬q → ¬p) :=
    λ (hpq : p → q) (hnq : ¬q) (hp : p), hnq (hpq hp)

#### Other Properties

    example : (p → (q → r)) ↔ (p ∧ q → r) :=
    iff.intro
      (λ (hpqr : p → (q → r)) (hpq : p ∧ q), (hpqr hpq.left) hpq.right)
      (λ hpqr hp hq, hpqr ⟨hp, hq⟩)

    example : ((p ∨ q) → r) ↔ (p → r) ∧ (q → r) :=
    iff.intro
      (λ hpqr : (p ∨ q) → r,
       ⟨compose hpqr or.inl, compose hpqr or.inr⟩)
      (λ (hprqr : (p → r) ∧ (q → r)) (hpq : p ∨ q),
         hpq.elim hprqr.left hprqr.right)

### Classical Logic

The Classical Logic exercises correspond to exercise 2 from chapter 3 of the
Theorem Proving in Lean book. These proofs use the principle of excluded middle.

#### Proofs

    example : (p → r ∨ s) → ((p → r) ∨ (p → s)) :=
    λ hprs : p → r ∨ s,
    or.elim (em (p → r))
      or.inl
      (λ hnpr : (p → r) → false,
       or.inr
       (λ hp : p,
        (or.elim (hprs hp)
          (λ hr : r, absurd (λ hp, hr) hnpr))
          identity))

    example : (p → r ∨ s) → ((p → r) ∨ (p → s)) :=
    λ hprs : p → r ∨ s,
    or.elim (em (p → r))
      or.inl
      (λ hnpr : ¬(p → r),
       or.inr
       (λ hp : p,
        (or.elim (hprs hp)
          (λ hr : r, absurd (λ hp, hr) hnpr))
          identity))

    example : ¬(p ∧ q) → ¬p ∨ ¬q :=
    λ hnpq : ¬(p ∧ q),
    by_cases
      (λ hp : p, by_cases (λ hq : q, false.elim (hnpq ⟨hp, hq⟩)) or.inr)
      or.inl

    example : ¬(p → q) → p ∧ ¬q :=
    λ hnpq : (p → q) → false,
    by_cases
      (λ hq : q, absurd (λ hp : p, hq) hnpq)
      (λ hnq : ¬q,
       ⟨by_contradiction (λ hnp : ¬p, hnpq (λ hp : p, false.elim (hnp hp))), hnq⟩)

    example : (p → q) → (¬p ∨ q) :=
    λ hpq : p → q, or.elim (em p) (compose or.inr hpq) or.inl

    example : (¬q → ¬p) → (p → q) :=
    λ (hnpnq : ¬q → ¬p) (hp : p),
    by_contradiction (λ hnq : ¬q, absurd hp (hnpnq hnq))

Trivial:

    example : p ∨ ¬p := (em p)

This last one is quite clever. It took me a while to produce it; I had to
explore various approaches before I managed to come to the solution.

    example : ((p → q) → p) → p :=
    λ hpqp : (p → q) → p,
    by_contradiction (λ hnp : ¬p, absurd (hpqp (λ hp, absurd hp hnp)) hnp)

### Quantifiers & Equality

The Quantifiers exercises correspond to exercises from chapter 4 of the Theorem
Proving in Lean book.

#### Introduction

    variables (α : Type) (p q : α → Prop)

    example : (∀ x, p x ∧ q x) ↔ (∀ x, p x) ∧ (∀ x, q x) :=
    iff.intro
      (λ h : ∀ x, p x ∧ q x, ⟨(λ x, (h x).left), (λ x, (h x).right)⟩)
      (λ (h : (∀ x, p x) ∧ (∀ x, q x)) x, ⟨h.left x, h.right x⟩)

    example : (∀ x, p x → q x) → (∀ x, p x) → (∀ x, q x) :=
    λ (hpxqx : ∀ x, p x → q x) (hpx : ∀ x, p x) x, hpxqx x (hpx x)

    example : (∀ x, p x) ∨ (∀ x, q x) → ∀ x, p x ∨ q x :=
    λ (h : (∀ x, p x) ∨ (∀ x, q x)) x,
    h.elim (λ hpx, or.inl (hpx x)) (λ hpq, or.inr (hpq x))

#### Take Out

These exercises correspond to exercise 2 from chapter 4 of the Theorem Proving
in Lean book. They involve taking a property out of a qualifier when it doesn't
depend on the quantified variable:

    variables (α : Type) (p q : α → Prop)
    variable r : Prop

    example : α → ((∀ x : α, r) ↔ r) :=
    λ (a : α),
    iff.intro
      (λ hx : (∀ x : α, r), hx a)
      (λ (hr : r) (x : α), hr)

    open classical

    example : (∀ x, p x ∨ r) ↔ (∀ x, p x) ∨ r :=
    iff.intro
      (λ h : ∀ x, p x ∨ r,
       or.elim (em r)
         or.inr
         (λ hnr : ¬r,
          or.inl (λ x, (h x).elim (λ hpx, hpx) (λ hr, absurd hr hnr))))
      (λ h : (∀ x, p x) ∨ r,
       h.elim
         (λ (hpx : (∀ x, p x)) hx, or.inl (hpx hx))
         (λ (hr : r) hx, or.inr hr))

    example : (∀ x, r → p x) ↔ (r → ∀ x, p x) :=
    iff.intro
      (λ (h : (∀ x, r → p x)) r x, h x r)
      (λ (h : r → (∀ x, p x)) x r, h r x)

#### Barber Paradox

This is exercise 3 from chapter 4 of the Theorem Proving in Lean book:

    variables (men : Type) (barber : men)
    variable  (shaves : men → men → Prop)

    open classical

    example (h : ∀ x : men, shaves barber x ↔ ¬ shaves x x) : false :=
    have hb : shaves barber barber ↔ ¬ shaves barber barber, from h barber,
    or.elim (em (shaves barber barber))
      (λ hs : shaves barber barber, hb.elim_left hs hs)
      (λ hns : ¬shaves barber barber , hns (hb.elim_right hns))

#### Primes

These exercises correspond to exercise 4 from chapter 4 of the Theorem Proving
in Lean book. We start by defining an `infinitely_many` constructor of
propositions that both `infinitely_many_primes` and
`infinitely_many_Fermat_primes` can use.

    def infinitely_many (prop : ℕ → Prop) : Prop := ∀ n, ∃ m, m > n ∧ prop m

    def prime (n : ℕ) : Prop := n > 1 ∧ ∀ k : ℕ, k ∣ n → k = 1 ∨ k = n

    def infinitely_many_primes : Prop := infinitely_many prime

    def Fermat_prime (n : ℕ) : Prop := ∃ k : ℕ, 2^(k+1) + 1 = n

    def infinitely_many_Fermat_primes : Prop := infinitely_many Fermat_prime

    def goldbach_conjecture : Prop :=
    ∀ n, n > 2 ∧ even n → ∃ p q, prime p ∧ prime q ∧ n = p + q

    def Goldbach's_weak_conjecture : Prop :=
    ∀ n, n > 5 ∧ ¬(even n) →
      ∃ p q r, prime p ∧ prime q ∧ prime r ∧ n = p + q + r

    def Fermat's_last_theorem : Prop :=
    ∀ n, n > 2 → ¬(∃ a b c : ℕ, a^n + b^n = c^n)

#### Identities

These exercises correspond to exercise 5 from chapter 4 of the Theorem Proving
in Lean book.

    open classical

    variables (α : Type) (p q : α → Prop)
    variable a : α
    variable r : Prop

    example : (∃ x : α, r) → r :=
    λ h : ∃ x : α, r, exists.elim h (λ (hx : α) (hr : r), hr)

    example : r → (∃ x : α, r) := exists.intro a

    example : (∃ x, p x ∧ r) ↔ (∃ x, p x) ∧ r :=
    iff.intro
      (λ h : ∃ x, p x ∧ r,
       match h with ⟨hx, hpxr⟩ := ⟨⟨hx, hpxr.left⟩, hpxr.right⟩ end)
      (λ h : (∃ x, p x) ∧ r,
       match h.left with ⟨hx, hpx⟩ := ⟨hx, ⟨hpx, h.right⟩⟩ end)

    example : (∃ x, p x ∨ q x) ↔ (∃ x, p x) ∨ (∃ x, q x) :=
    iff.intro
      (assume ⟨ha, (hpaqa : p a ∨ q a)⟩,
       hpaqa.elim (λ pa, or.inl ⟨ha, pa⟩) (λ qa, or.inr ⟨ha, qa⟩))
      (assume h : (∃ x, p x) ∨ (∃ x, q x),
       h.elim (λ ⟨a, hpa⟩, ⟨a, or.inl hpa⟩) (λ ⟨a, hqa⟩, ⟨a, or.inr hqa⟩))

    example : (∀ x, p x) ↔ ¬ (∃ x, ¬ p x) :=
    iff.intro
      (λ (h : (∀ x, p x)) ⟨a, (hpa : ¬ p a)⟩, hpa (h a))
      (λ (he : ¬ (∃ x, ¬ p x)) x,
        by_contradiction (λ hnpx : ¬ p x, he (exists.intro x hnpx)))

    example : (∃ x, p x) ↔ ¬ (∀ x, ¬ p x) :=
    iff.intro
      (λ ⟨a, (hpa : p a)⟩ (h : ∀ x, ¬ p x), (h a) hpa)
      (λ h : ¬ ∀ x, ¬ p x,
       by_contradiction (λ (hne : ¬ ∃ x, p x),
       h (assume x, show ¬ p x, from
          by_cases
            (λ hpx : p x, absurd (exists.intro x hpx) hne)
            (λ hpnx : ¬ p x, hpnx))))

    example : (¬ ∃ x, p x) ↔ (∀ x, ¬ p x) :=
    iff.intro
      (λ (h : ¬ ∃ x, p x) x,
       by_cases
         (λ hpx : p x, absurd (exists.intro x hpx) h)
         (λ hpnx : ¬ p x, hpnx))
      (λ (hx : ∀ x, ¬ p x) ⟨a, (pa : p a)⟩, hx a pa)

    example : (¬ ∀ x, p x) ↔ (∃ x, ¬ p x) :=
    iff.intro
      (λ hna : ¬ ∀ x, p x,
       by_contradiction
         (assume hne : (∃ x, ¬ p x) -> false,
         hna (λ x, by_contradiction (assume hnpx : ¬ p x, hne ⟨x, hnpx⟩))))
      (λ ⟨x, (hnpx : ¬ p x)⟩ (ha : ∀ x, p x), hnpx (ha x))

    example : (∀ x, p x → r) ↔ (∃ x, p x) → r :=
    iff.intro
      (λ (ha : ∀ x, p x → r) ⟨x, (hpx : p x)⟩, (ha x) hpx)
      (λ (h : (∃ x, p x) → r) x (hpx : p x), h ⟨x, hpx⟩)

    example : (∃ x, p x → r) ↔ (∀ x, p x) → r :=
    iff.intro
      (λ ⟨x, (hpxr : p x → r)⟩ (ha : ∀ x, p x), hpxr (ha x))
      (λ (hax : (∀ x, p x) → r),
       by_cases
         (λ (hap : ∀ x, p x), ⟨a, λ _, hax hap⟩)
         (λ (hn : ¬∀ x, p x),
          by_contradiction
            (λ hne : ¬(∃ x, (p x → r)),
             have hap : ∀ x, p x, from
               (λ x, by_contradiction
                (λ hnpx : ¬ p x,
                 hne (exists.intro x (λ hpx, absurd hpx hnpx)))),
             hn hap
          )))

    example : (∃ x, r → p x) ↔ (r → ∃ x, p x) :=
    iff.intro
      (λ ⟨a, (hrpa : r -> p a)⟩ (hr : r), ⟨a, hrpa hr⟩)
      (λ (h : r → ∃ x, p x),
       by_cases
         (λ hr : r, match h hr with ⟨x, hpx⟩ := ⟨x, λ _, hpx⟩ end)
         (λ hnr : r -> false, ⟨a, λ hr, absurd hr hnr⟩))


#### Logarithmic Multiplication

The following is a proof (using Lean Prover) of the following identity
of the logarithm function:

    log (x × y) = (log x) + (log y)

Proof:

    variables (real : Type) [ordered_ring real]
    variables (log exp : real → real)
    variable  log_exp_eq : ∀ x, log (exp x) = x
    variable  exp_log_eq : ∀ {x}, x > 0 → exp (log x) = x
    variable  exp_pos    : ∀ x, exp x > 0
    variable  exp_add    : ∀ x y, exp (x + y) = exp x * exp y

    include log_exp_eq exp_log_eq exp_pos exp_add

    theorem log_mul {x y : real} (hx : x > 0) (hy : y > 0) :
    log (x * y) = (log x) + (log y) :=
    calc
    log (x * y) = log ((exp (log x)) * (exp (log y)))
                                    : by rw [exp_log_eq hx, exp_log_eq hy]
            ... = log (exp ((log x) + (log y)))
                                    : by rw exp_add
            ... = (log x) + (log y) : by rw log_exp_eq

This is exercise 6 from chapter 4 of the Theorem Proving in Lean book.

The following is a more verbose and explicit proof, avoiding the use of `rw`:

    theorem log_mul {x y : real} (hx : x > 0) (hy : y > 0) :
    log (x * y) = (log x) + (log y) :=
    calc
    log (x * y) = log ((exp (log x)) * y) :
         congr_arg log
             (congr_arg (λ p : real, (p * y)) (eq.symm (exp_log_eq hx)))
    ... = log((exp (log x)) * (exp (log y))) :
         congr_arg log
             (congr_arg (λ p : real, ((exp (log x)) * p))
               (eq.symm (exp_log_eq hy)))
    ... = log (exp ((log x) + (log y))) :
         congr_arg log (eq.symm (exp_add (log x) (log y)))
    ... = (log x) + (log y) : log_exp_eq ((log x) + (log y))

#### Multiplication by Zero

This is exercise 7 from chapter 4 of the Theorem Proving in Lean book.

    example (x : ℤ) : x * 0 = 0 :=
    calc
       x * 0 = x * (x - x) : by rw sub_self
       ... = x * x - x * x : by rw [mul_comm, sub_mul]
       ... = 0 : by rw sub_self

