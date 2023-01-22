---
title: 'Probability Discussion Notes'
date: '2021-10-21'
tags: ['probability', 'class', 'berkeley']
draft: false
summary: 'Notes from EECS 126. Work in progress'
---

## Subadditivity

P(∪<sub>i>=1</sub>A<sub>i</sub>) = Σ<sub>i>=1</sub> P(A<sub>i</sub>)

A<sub>1</sub> U A<sub>2</sub> ... ∪ A<sub>n</sub> =

Let B<sub>k</sub> := ∪<sub>i=1</sub><sup>k</sup> A<sub>i</sub>

= ∪<sub>i=1</sub><sup>n</sup> (A<sub>i</sub> \ B<sub>i-1</sub>)

Use Third Axiom of Probability

= Σ<sub>i>=1</sub><sup>n</sup> P (A<sub>i</sub> \ B <sub>i-1</sub>)

Use Axiom of Probability D<sub>1</sub> ⊆ D<sub>2</sub> => P(D<sub>1</sub>) ⊆ P(D<sub>2</sub>)

<= Σ<sub>i>=1</sub><sup>n</sup> P (A<sub>i</sub>)

## Continuity from Below

P(∪<sub>i>=1</sub>A<sub>i</sub>) = lim<sub>i->∞</sub> P(A<sub>i</sub>)

Reuse from Subadditivity portion

Let B<sub>k</sub> := ∪<sub>i=1</sub><sup>k</sup> A<sub>i</sub>

= ∪<sub>i=1</sub><sup>n</sup> (A<sub>i</sub> \ B<sub>i-1</sub>)

Use third axiom of probability

= Σ<sub>i>=1</sub><sup>n</sup> P (A<sub>i</sub> \ B <sub>i-1</sub>)

Use the Hint for monotone convergence

lim<sub>n->∞</sub>Σ<sub>i>=1</sub><sup>n</sup> P (A<sub>i</sub> \ B <sub>i-1</sub>)

lim<sub>n->∞</sub>U<sub>i>=1</sub><sup>n</sup> P (A<sub>i</sub> \ B <sub>i-1</sub>)

Use U<sub>i>=1</sub><sup>n</sup> P (A<sub>i</sub>) = U<sub>i>=1</sub><sup>n</sup> P (A<sub>i</sub> \ B<sub>i-1</sub>) which is true if they are disjoint sets

lim<sub>n->∞</sub>P(U<sub>i>=1</sub><sup>n</sup> P (A<sub>i</sub>))

Use U<sub>i>=1</sub><sup>n</sup> P (A<sub>i</sub>) = A<sub>n</sub>

lim<sub>n->∞</sub>P(A<sub>n</sub>)

## Balls and Bins

### How do you create a probability space for throwing n balls to n bins, where each ball can land in any bin uniformily at random.

Way 1:

ω = (ball 1: bin 3, ball 2: bin 1, ...)

Pr(ω) = 1 / n<sup>n</sup>

Ω = all possible ω

Way 2

ω = (# of balls in bin 1: 5, # of balls in bin 2: 0, ...)

Pr(ω) = pretty hard to calculate

Ω = all possible ω

What are the tradeoffs? Formulation 1 has more information, Formulation 1 also has which ball goes to which bin, also Pr(ω) is much harder to calculate

### Compute the probability of the event {all empty bins sit to the left of all the bins containing at least one ball} in terms of P(A<sub>i</sub>)

b is the event of interest, but we need to write its probability in terms of A<sub>i</sub>

Use law of total probability: P(B) = Σ <sub>i=0</sub><sup>n</sup> P(B ∩ A<sub>i</sub>)

B ∩ A<sub>i</sub> means there are i empty bins and all of them are on the left side

Since all the bins need to be on the left, there are (n C i) ways for the bins to be placed

P(B) = Σ <sub>i=0</sub><sup>n</sup> P(A<sub>i</sub>) / (n C i)

### Compute P(A<sub>1</sub>)

How can there be 1 empty bin if I throw n balls?

There must be 1 bin with 0 balls, 1 bin with 2 balls, rest with 1 ball each

P(1 empty bin) = (n (n-1) (n C 2) (n-2)!) / n<sup>n</sup>
