---
title: 'Probability Notes'
date: '2021-10-21'
tags: ['probability', 'class', 'berkeley']
draft: false
summary: 'Notes from EECS 126. Work in progress'
---

# Probability in Electrical Engineering and Computer Science

Real applications of probability

Introduction to Probability, 2nd Edition
Follows the course, can borrow from engineering library

TODO:
Set calendar for homework self grades and resubmission time
Set midterm time on calendar
Choose discussion time
Countable vs uncountable

## Probability: Math + a way of thinking about uncertainty

Humans are naturally good at thinking about probability. For example, you can easily walk down a crowded street and estimate what path allows you to avoid bumping into people. However, humans are pretty awful at turning those innate estimates into quantifiable numbers. The goal is to take the natural probability estimation into something that a robot can understand.

**Quantify + Model**

Inference. Humans are also really good at taking in noisy observations and infer what is really happening.

How do I generate some image from AI? Take a natural image and repeatedly add noise to it. You then end up with an image of pure noise. The idea behind stable diffusion and dalle-2 is to reverse this process. We generate a bunch of noise, then reduce noise until we end up with a image.

Modern theory of probability has an axiomatic starting point. Before the 1930s, probability was a heuristic field.
All of probability are logical deductions from the starting point.

_Probability space:_ A probability space (Ω, F, P) is a triple.
Ω is a set, the sample space.
F is a family of subsets of Ω, called events
P is a probability measure

Rules:

- F is a σ-algebra, it contains Ω itself. F is closed under complements and countable unions.
  - If A ∈ F, then any of A's complements belong to F
  - countable unions: If A1, A2, ... ∈ F, then the union of Ai ∈ F
  - By DeMorgan's Laws, this implies that F is also closed under countable intersections
- P is a function that maps events to a number between 0 and 1. `P: F -> [0,1]`
  - Probability measures must obey Komogorov Axioms
    1. `P(A) >= 0` Probability of any event must be non-negative
    2. `P(Ω) = 1` The probability of any outcome occuring must be equal to 1
    3. σ-additivity aka countable additivity. If A1, A2, ... is a countable sequence of events and disjoint, then `P(U) = ΣP(Ai)`
    - counterexample: `P = Unif(0,1)`, then P({x}) = 0 for x ∈ [0,1]. If `1 = P([0,1])`, then `Σ P({x}) = 0` Dive into this later

Examples of the Above Rules

1. Flip a coin with bias P(Heads with probability p, Tails with probability 1-p).

   - Ω = {H, T} The sample space is just heads and tables
   - F = 2 ^ σ = {H, T, Ø, {H, T}} These are all subsets of σ
   - P(H) = p, P(T) = 1-p, P(Ø) = 0, P({H, T}) = 1 Why are Ø and {H, T} here?
   - We then want to create an experiment of n "independent" flips. Is our vocabulary rich enough for this? no
   - Ω = {H, T}^n, F = 2^Ω, P(f1, f2, ... , fn) = p^{number of flips fi = H} \* (1-p)^{number of flips fi = T}
   - There can be multiple answers for a probability space. It just needs to be rich enough to capture all the probabilities

2. Ω now equals all possible configurations of atoms in the universe. How do we represent this?
   - Let A = {set of configurations that lead to flip heads}
   - Let B = {set of configurations that lead to flip tails}
   - F = {Ø, A, B, Ω = {A U B}}
   - P(A) = p, P(B) = 1-p, P(Ø) = 0, P(Ω) = 1

Note: The probability space is usually implicitly described, except for HW 1.
Lol

## All of the rules of probability you are likely familiar with are consequences of the three axioms.

Ex: If `A ∈ F`, `P(A^C) = 1 - P(A)`. The probability of A's complement is 1 - probability of A.

- Since F is σ algebra, then A^C ∈ F.
  - A U A^C. A and A complement is a disjoint union equal to Ω
  - 1 = P(Ω) Using axiom 2
  - P(A U A^C) = P(A) + P(A^C) Using Axiom 3

Ex: If A and B are events and A is a subset of B, then `P(A) = P(B)`
B = A U (B \ A). B is equal to the disjoint union of A and B minus the elements of A
P(B) = P(A U (B \ A)) => P(A) + P(B\A) Using axiom 3
P(A) + P(B\A) >= P(A) Using axiom 1

Ex: If A and B are events, then P(A U B) = P(A) + P(B) - P(A ∩ B). Aka inclusion exclusion principle
Proof: `AUB, A∩B ∈ F`
A U B = B U (A \ B)
P(A U B) = P(B) + P(A\\(A∩B)) Using axiom 3
P(B) + P(A\\(A∩B)) = P(B) + P(A) - P(A ∩B) Using Axiom 3

Ex: Ω = countable set
F = 2 ^ Ω, each individual is an event {w} ∈ F, for all w ∈ Ω
P(A) = Σ for all A ⊆ Ω
This is a discrete sample space.

Ex: Law of total probability. If A1, A2, ..., partition Ω, then (Ω = U Ai). Mutually exclusive, collectively exhaustive.
Then, for any B ∈ F, then `P(B) = Σ P(B ∩ Ai)`
B = U(Ai ∩ B) then apply axiom 3

The goal of probability is to take complicated things and use that to approximate them into multiple simpler events.

Mathematicians: Given a probability space(model), what can I say about outcomes of experiments that derive from this model?
Statistician: Given outcomes, how do I choose a good model(probability space)?
Engineers: Given a real world problem, how do I choose a model that captures the essence of the model and then use the model to draw insight.
