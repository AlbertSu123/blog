---
title: 'Intro to Financial Engineering'
date: '2023-01-19'
tags: ['user-interface', 'class', 'berkeley']
draft: false
summary: 'Notes from CS 160. Interest Rate analysis, optimal portfolio selection, risk measures, arbitrage, hedging strategies, stock derivative pricing(forwards, futures, options)'
---

The goal of finance is to avoid arbitrage.

Textbook: Investment Science, second edition

cashflow a = (a<sub>1</sub>, ..., a)<sub>n</sub>)
a: cash received at period i. If a<sub>i</sub> < 0, this signifies cash paid.

Assumption

1. More cash is better
2. time between periods i and i+1 for all i are fixed
3. All a<sub>i</sub> are certain

## Comparing Cashflows

Assumption: There exists an ideal bank where anyone can deposit(borrow) x at period i (for all i=0, 1) and set (pay) x (i+1) at period i+1
Why doesn't this assumption work? Because you could borrow 10 million dollars, and whenever you need to repay interest, you just borrow more money to pay the interest

What cashflows x=(x<sub>0</sub>, x<sub>1</sub>) can be generated from a=(a<sub>0</sub>, a<sub>1</sub>) through an ideal bank with interest rate r per period?

Answer: let b<sub>i</sub> be the balance at the bank
B<sub>0</sub> = a<sub>0</sub> - x

B<sub>1</sub> = B<sub>0</sub>(1+r) + a<sub>1</sub> - x<sub>1</sub>

need B<sub>1</sub> = 0

B<sub>1</sub> = (a<sub>0</sub> - x<sub>0</sub>)(1+r) + a<sub>1</sub> - x<sub>1</sub> = 0

x<sub>0</sub> + (1 + r) x<sub>0</sub> = a<sub>0</sub> + (1 + r)a<sub>1</sub>

For n > 2, a = (a<sub>0</sub>, a<sub>1</sub>, ..., a<sub>n</sub>), x=(x<sub>0</sub>, x<sub>1</sub>, ..., x<sub>n</sub>)

Keep depositing a, withdraw x at each timestep
B<sub>0</sub> = a<sub>0</sub> - x<sub>0</sub>

B<sub>1</sub> = B<sub>0</sub>(1+r) + a<sub>1</sub> - x<sub>1</sub>

B<sub>2</sub> = B<sub>1</sub>(1+r) + a<sub>2</sub> - x<sub>2</sub>

B<sub>i</sub> = B<sub>i-1</sub>(1+r) + a<sub>i</sub> - x<sub>i</sub> = 0

Conclusion

1. a can be converted to x iff PV(a) = PV(x)
2. Given a = (a<sub>0</sub>, a<sub>1</sub>, ..., a<sub>n</sub>) and b = (b<sub>0</sub>, b<sub>1</sub>, ..., b<sub>n</sub>)
   a is (better, equal, worse) than b iff (PV(a)>PV(b), PV(a)=PV(b), PV(a)< PV(b))
