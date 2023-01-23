---
title: 'Probability Notes'
date: '2021-10-21'
tags: ['probability', 'class', 'berkeley']
draft: false
summary: 'Notes from EECS 126. Work in progress'
---

# Lecture 1: Probability space, Three Axioms of Probability

## TLDR:

Probability space function: (Ω set , F events, P probability measure)
Axioms:

1. P(A) >= 0
2. P(Ω) = 1
3. If A1, ..., An are disjoint events => P(UA) = Σ (A)
   Real applications of probability

Introduction to Probability, 2nd Edition
Follows the course, can borrow from engineering library

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
    - counterexample: `P = Unif(0,1)`, then P(\{x\}) = 0 for x ∈ [0,1]. If `1 = P([0,1])`, then `Σ P(\{x\}) = 0` Dive into this later

Examples of the Above Rules

1. Flip a coin with bias P(Heads with probability p, Tails with probability 1-p).

   - Ω = \{H, T\} The sample space is just heads and tables
   - F = 2 ^ σ = \{H, T, Ø, \{H, T\}\} These are all subsets of σ
   - P(H) = p, P(T) = 1-p, P(Ø) = 0, P(\{H, T\}) = 1 Why are Ø and \{H, T\} here?
   - We then want to create an experiment of n "independent" flips. Is our vocabulary rich enough for this? no
   - Ω = \{H, T\}^n, F = 2^Ω, P(f1, f2, ... , fn) = p^\{number of flips fi = H\} \* (1-p)^\{number of flips fi = T\}
   - There can be multiple answers for a probability space. It just needs to be rich enough to capture all the probabilities

2. Ω now equals all possible configurations of atoms in the universe. How do we represent this?
   - Let A = \{set of configurations that lead to flip heads\}
   - Let B = \{set of configurations that lead to flip tails\}
   - F = \{Ø, A, B, Ω = \{A U B\}\}
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
F = 2 ^ Ω, each individual is an event \{w\} ∈ F, for all w ∈ Ω
P(A) = Σ for all A ⊆ Ω
This is a discrete sample space.

Ex: Law of total probability. If A1, A2, ..., partition Ω, then (Ω = U Ai). Mutually exclusive, collectively exhaustive.
Then, for any B ∈ F, then `P(B) = Σ P(B ∩ Ai)`
B = U(Ai ∩ B) then apply axiom 3

The goal of probability is to take complicated things and use that to approximate them into multiple simpler events.

Mathematicians: Given a probability space(model), what can I say about outcomes of experiments that derive from this model?
Statistician: Given outcomes, how do I choose a good model(probability space)?
Engineers: Given a real world problem, how do I choose a model that captures the essence of the model and then use the model to draw insight.

TODO:
Set calendar for homework self grades and resubmission time
Set midterm time on calendar
Choose discussion time
Countable vs uncountable

# Lecture 2: Conditional Probability, Independence, Random Variables

**Conditional Probability:** If `B ∈ F` and `P(B) > 0`, then conditional probability P(A|B) = P(A ∩ B) / P(B)

_Intuition_: Probability A occurs given we know that B occurred

Formal Definition: P(.|B) gives a restriction of our model (Ω, F, P) to those samples in B
If `(B, F|B, P(.|B))` is a probability space itself, where `F|B = \{A ∩ B: A ∈ F\}`

_Ex._ If A1, A2, ... ∈ F, P(Ai) > 0, A partion Ω
`P(B) = Σ P(B ∩ Ai) = Σ P(B|Ai) * P(Ai)`

## Bayes Rule

Sometimes, it's easy to express P(B|A), but we are really interested in P(A|B)
Let A be the state of the experiment, B be the observation data

P(A|B) - given the observation, we want the state. This is the task of inference.

Suppose we forgot Bayes Rule, let's try to recreate it from definition of conditional probability
`P(A|B) = P(B ∩ A) / P(B) = P(A) * P(B|A) / P(B)`

_Ex._ Suppose we have the following model: 85% of students got a pass, 15% of students got a no pass. 60% students that got passes went to lecture, 40% of students did not go to lecture. 10% of students that got no passes went to lecture, 90% didn't go to lecture.

We want P(NP | N) / P(NP | Y). How much more likely are you to NP when you don't attend lecture vs when you do attend lecture?
P(NP ∩ N) / P(NP ∩ Y) \* P(Y) / P(N)

_Ex._ Suppose we roll two dice and the sum is 10. What is the probability roll 1 was = 4?

```
B = \{Sum of rolls = 10\}
A = \{first roll is 4\}

P(A|B) = P(A ∩ B) / P(B)
       = P({First roll is 4, sum of rolls is 10}) / P({sum of rolls is 10})
       = P({first: 4, second: 6}) / P({(4,6), (5,5), (6,4)})
       = (1/36) / (3/36)
       = 1/3
```

### Conditioning also allows us to usefully decompose intersections of events.

_Ex._ Consider event A1 ... An
P(∩ A) = P(A<sub>1</sub> | ∩<sup>n</sup><sub>i = 2</sub> A<sub>i</sub>) P (∩ A<sub>i</sub>)

_Ex._ Given n people in a room, what is the probability that more than 2 people share a birthday?

A<sub>i</sub> = \{person i does not share a birthday with any of the people j = 1, ..., i-1\}

P(A<sub>i</sub> | ∩ <sup>i-1</sup><sub>j=1</sub> A<sub>j</sub>) = (365 - (i-1)) / 365

This is because the person needs to land in one of the days not in the 365.
P(no shared birthdays) = P(∩ A<sub>i</sub>)

Use 1-x <= e^-x.
Approximate using taylor series

Let n = 23 - there are 23 kids in a class.
P(shared birthday) >= 1 - e^((i-1)/365))

## Independence

Events A, B are independent if P(A ∩ B) = P(A) * P(B)
Disjoint events are not independent
*Note:\* in a special case where P(A) > 0: A,B: dependent <=> P(B|A) = P(B)

In general, collection A<sub>1</sub>, A<sub>2</sub>, ... ∈ F are independent events if P(∩<sub>i ∈ S</sub> A<sub>i</sub>) = ∏ <sub>i ∈ S</sub> P(A<sub>i</sub>) for all finite sets of indices S

If A<sub>1</sub>, A<sub>2</sub>, ... ∈ F are independent, then B<sub>1</sub>, B<sub>2</sub>, ... are independent, where each A<sub>i</sub> = B<sub>i</sub> or B<sub>i</sub><sup>C</sup>
Intuitively, we assume that knowing A means we know A's complement

∩<sub>i=2</sub><sup>n</sup> A<sub>i</sub>
∏P(Ai) = P(∩ Ai) = P(∩) + P(A1)
(1- P(Ai)) ∏ P(Ai) = P(A1^C ∩ ∏<sub>i=2</sub>Ai)
This shows A<sub>1</sub><sup>C</sup>, A<sub>2</sub>, ... , A<sub>n</sub> are independent
Therefore, B<sub>1</sub>, B<sub>2</sub>, ... , B<sub>n</sub> are independent

### Conditional Independence

Often times we have two events that we think of as independent but aren't. There might be a confounding variable C
If A,B,C are such that P(C) > 0 and P(A∩B|C) = P(A|C) P(B|C), then A,B are said to be conditionally independent given C

Consider two coins with bias p!=q
Pick a coin at random, and flip twice. H(i) = Event that flip i is heads
Are the two coinflips independent? No. If p is 1 and q is 0, then we know what the next coin flip is given our first coin flip.

P(H<sub>i</sub>) = p + q / 2
P(H<sub>1</sub> ∩ H<sub>2</sub>) = p<sup>2</sup> + q<sup>2</sup> / 2 != P(H<sub>1</sub>) P(H<sub>2</sub>) = (p+q)<sup>2</sup> / 2
C = \{pick coin p\} H<sub>1</sub>, H<sub>2</sub> conditionally independent given C

TODO:
Set calendar for homework self grades and resubmission time
Set midterm time on calendar
Choose discussion time
Countable vs uncountable
Go to discussion
