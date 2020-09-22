---
title: Software Model Checking
subtitle: Notes on Model Checking
author: Ranjit Jhala
header-includes: |
  \include{commands}
  \usepackage{stmaryrd}
---


Preliminaries
-------------

A simple program is defined via a transition relation as $P = (X, L, l_0, t)$ where
$X$ is the set of variables, $L$ set of control locations with $l_0 \in L$
initial locations and $T$ the set of transitions. Given $(l, \rho, l') \in T$,
$l, l'$ are the source and end locations of the trasition and $\rho$ is a
constraint over free variables from $X \cup X'$, where variables from $X$
represent values from $l$ and variables from $X'$ represent values from $l'$.

The labels and transitions define a control flow graph (CFG).
For a given program, its CFG captures its control flow.
A program can be converted to its simple program representation as:

 - Assignment `x := e` becomes $x' = e \wedge \underset{y \in X / \{x\}}{\bigwedge}y =
   y'$

A state $s$ is an of valuation variables (X), represented as $s = v.X$.
Given $(s, s') \in v.X \times v.X'$, say $(s, s') \vDash \rho$ if the
valuations satisfy the constraint.

A *finite computation* is defined as $\langle l_0, s_0\rangle, \langle l_1, s_1
\rangle, \dots \langle l_k, s_k \rangle \in (L \times v.X)^{*}$ where
$l_0 \in L_0$ (initial location)
and for each $i \in \{0, \dots, k-1\}$  there exists a
transition $(l_i, \rho, l_{i+1}) \in T$ such that $(s_i, s_{i+1}) \vDash \rho$

Similarly an *infinite computation* is defined $\langle l_0, s_0\rangle, \langle l_1, s_1
\rangle, \dots  \in (L \times v.X)^{\omega}$


### Properties

Intuitively, safety properties state "bad things don't happen".
Liveness properties state "good things eventually happen".
Mathematically, a property $\Pi \subset (L \times v.X)^{\omega}$,
is basically defined as a set of traces and a traces $\sigma$
is said to satify $\Pi$ iff $\sigma \in \Pi$.

A safety property is one such that for every infinite computation
$\sigma \in \Pi$, and for every prefix $\sigma'$, there exists
a $\beta \in (L \times v.X)^{\omega}$ such $\sigma' \beta \in \Pi$.
Contrapositively, a trace $\sigma'$ is unsafe if there exists no trace $\beta$
that makes it safe (i.e. $\sigma . \beta \in \Pi$).
In other words, $\Pi$ is a safety property iff and only if it's prefix
closed.

The **verification problem** takes as input a program P and property $\Pi$
and return true if all traces of P are in $\Pi$ and false otherwise.
It can also be stated in terms of reachability: given a program P and
a location $\epsilon$, the verification problem return true iff
no exeuctions reach $\epsilon$.

### Stateful Search

Let $\finitePgm$ be the class of programs where each variable
ranges over a finite domain. The verification problem has
an algorith with the following steps:

1. Takes as input the property and the error state to avoid.
2. Maintain queues reach and worklist for reached states and
   states to work on.
2. Add all initial states to the queue of states to explore.
3. Dequeue a state $(l, s)$, and if it's not in reach, add it.
4. For each $(l, \rho, l') \in T$, add all states $s' \in \text{Post}(s, \rho)$
   to worklist.
5. Check if $\epsilon \in$ reach.

State space explosion is the main problem that must be handled.
There are two main ares of research:

1. Collapse states into equivalent classes and perform the search
   on one candidate from each class. Requires showing that
   any bug in the original system can also be intercepted in the
   reduced system. Involves techiniques like Partial order reduction,
   symmetric reduction, etc.

2. Compositional approaches involve proving safety on smaller sub
   programs to compose a proof for the main program. Basically
   corresponds to the Hoare logic composition rule, which says

```
            {A1} P1 {G1}     {A2} P2 {G2}     G1 -> A2
            ------------------------------------------
                        {A1} P1 ; P2 {G2}
```


#### Sytematic State Space Exploration

All non determinism is factored into two sources: input from environment
and scheduling choices. In contrast to exhaustive exploration, systematic
exploration proceeds by iterating over the space of all possible schedules
and simply executing the process under each schedule. The advantage is
that the need for a formal executable semantics is eliminated.
Mostly used in conjunction with a test harness. While a regular test
simply executes the program with some fixed schedule, a systematic
exploration based tool would execute the program with the same input
but with all possible schedules. Challeneges include a mechanism
that's needed to keep track of states, generation of schedules,
discovering executions where states are the same regardless of schedule.

#### Stateless Search

The key idea behind "stateless search" is to
not store the set of visited states while performing the search.


### Symbolic Concrete Model Checking

Instead of the enumerative approach that relies on
enumerating concrete states, symbolic model checking
uses an *abstract data type* symreg to represent states.
An interpretation function $\llbracket \_ \rrbracket : \text{symreg} \to
\powerset{v.X}$. The advantage is amplified by advances in constraint solvers.



#### Example: Propositional Logic

Consider finite domains. To simplify, we assume n boolean variables.
A boolean vector of length n can represent the state space.
A symbolic state is represented via a function that
returns true iff the representative vector is in the
symbolic state. This basically boils down to the Mathching Logic's representation
of states, where for given concrete state $c$ and symbolic pattern
$\varphi$, $c$ is in the symbolic state iff $\rho(c) \in \rho(\varphi)$,
where $\rho$ is the interpretation into the finite domain.

Note that the focus here is on the representation of states. An enumerative
representation will require $2^n$ states, but a symbolic representation
requires just $\top$ to represent the set of all states.

#### Example: First Order Logic

In this case we're dealing with programs with infinite domains.
A symbolic region's representation involves a formula $varhpi$ with free variables
that correspond to program variables. The interpretation is given $\llbracket
\varphi \rrbracket = \{s \mid  s \vDash \varphi \}$.

### Invariants and Synthesis

The set of reachable states $R$ is defined as the smallest set closed under the
Post operation. In other words $\Post(R) \subseteq R$. Safety w.r.t to a label
$\epsilon$ is guaranteed as long as $\epsilon$ does not appear in $R$.
Instead of calculating $R$, one could calculate $R'$ such that (1) set of
initial states is contained in $R'$ and (2) $R'$ is closed under the Post
operation. If $\epsilon$ doesn't appear in $R'$, then the program is still
safe, even though the $R'$ may be an over approximation of $R$.



























