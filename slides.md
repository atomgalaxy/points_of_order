---
title: Points of Order
pdf: points_of_order.pdf
slideNumber: true
controls: true
---
\newcommand{\@@}{\cdot}
\newcommand{\true}{\text{true}}
\newcommand{\false}{\text{false}}


# Points of Order {bg=#fff}

Gašper Ažman

May 9th, 2019

<img src='assets/cpp_now_logo.svg' style='height: 3ex'>


# Spoilers

## Like, what is this about?

Ordering and spaceships.


## Why spoilers?

Because they're better than overviews.


## But what is this really about?


* When and how do I define `<=>`
* When and how do I use `<=>`
* Why should I even care?
* Have you finally solved `float`?
* can I put random stuff into `std::map` yet?


## You said these are spoilers

Here are the answers:


## When do I define `<=>`

When your type has a natural strong$^{\dagger}$ ordering.

<div class='footnotes'>
$^\dagger$`questions.push("what are std::partial_ordering and std::weak_ordering for then?");`<br/>

If you don't believe me, ask Titus Winters.
</div>

## How do I define `<=>`

```
std::strong_ordering operator<=>(T const& other) = default;
```

`<=>` replaces `<, <=, >=, >` and sometimes `==` and `!=`.


## When do I use `<=>`

you don't, directly$^\dagger$. Use normal comparisons.


<small>$^\dagger$unless chaining or you know better.</small>

<!-- this is super cool because you need less codegen -->
## 

3. Why should I even care?
4. Have you finally solved float ordering?
5. can I put random stuff into maps yet?

# Overview

1. The Math <small>(order theory)</small>
2. The Application <small>(Element of Programming)</small>
3. C++ 20: The Language & Core <small>(`<=>`, `==` and `std::strong_ordering`)</small>
4. C++ 20: The Library <small>(`std::strong_order`, `variant`, `vector` & `optional`)</small>


# The Big Questions





# Math Plan

1. Relations
2. Equivalence theory
3. Order theory

<!--
We'll do an introduction to order theory first. Order is a relation, and just
what it means is very related to what equality and equivalence mean. So, to
start off easy, we'll go through the parts that deal with equivalence theory
first, and then expand to order theory.
-->

# Relations

## Definition
A relation $\Omega$ on a set $S$ is a subset of the set of pairs of elements
from $S$: 
$$
\Omega \subseteq S \times S
$$

For $x, y \in S$, we say that they are <dfn>$\Omega$-related</dfn> iff $(x, y)
\in \Omega$.

We write this as $x \Omega y$ in infix notation.


## Example: $\leq$ As a matrix

$S = \{ a, b, c, d \}$

$\leq_S$ | $a$ | $b$ | $c$ | $d$
---------+-----+-----+-----+-----
     $a$ |$\@@$|     |     | 
     $b$ |$\@@$|$\@@$|     |    
     $c$ |$\@@$|$\@@$|$\@@$|    
     $d$ |$\@@$|$\@@$|$\@@$|$\@@$

Every dot is a pair in $\leq_S \, \subseteq S \times S$.


## Example: $\leq$ as a graph

$S = \{ a, b, c, d \}$

```{.render_dot}
digraph G {
  bgcolor=transparent;
  a -> a;
  a -> b;
  a -> c;
  a -> d;
  b -> b;
  b -> c;
  b -> d;
  c -> c;
  c -> d;
  d -> d;
}
```

Every arrow is a pair in $\leq_S \, \subseteq S \times S$.


## Relations as predicates

For a relation $\Omega$ over $S$, there is a binary predicate $S \times S \to Bool$:

$$
\Omega(x, y) = ((x, y) \in \Omega) ? \true : \false;
$$

Example:

$$
\leq_S(x, y)
$$

# Overview

Order theory deals with specific kinds of binary _relations_ over sets of
elements.

Relations have many properties. These 7 matter for today:

- reflexivity & antireflexivity
- symmetry, asymmetry (or strictness)
- transitivity
- equivalence
- order (partial, weak, strong, total)

Let $\Omega$ and $\Delta$ be relations.

<aside class="notes" data-markdown>
- "Sets" is how we get around the platonic equality problem - we just have
  elements, and elements are always only equal to themselves, because sets come
  with equality built in.
</aside>


# Reflexivity

$\Omega$ is <dfn>reflexive</dfn> iff an element is always related to itself<br>
  $\forall x: x\Omega x$
```{.render_dot}
digraph G {
  bgcolor=transparent;
  x -> x [style=dotted];
}
```

$\Omega$ is <dfn>antireflexive</dfn> iff an element is never related to itself<br>
  $\forall x: \neg x\Omega x$.<br>
  EoP calls this <dfn>strict</dfn>.


```{.render_dot}
digraph G {
  bgcolor=transparent;
  x -> x [style=dashed, color=red];
}
```


<aside class="notes" data-markdown>
Reflexivity is about the relationships of elements to themselves. It allows
reasoning about elements uniformly.
</aside>


# Symmetry

$\Omega$ is <dfn>symmetric</dfn> iff it always goes both ways<br>
  $\forall x, y: x\Omega y \Rightarrow y\Omega x$
```{.render_dot}
digraph G {
  bgcolor=transparent;
    subgraph sg {
    rank=same;
     
    x -> y [style=solid];
    y -> x [style=dotted];
  }
}
```

$\Omega$ is <dfn>antisymmetric</dfn> iff it never goes both ways<br>
  $\forall x, y: x\Omega y \Rightarrow \neg y\Omega x$

```{.render_dot}
digraph G {
  bgcolor=transparent;
  subgraph sg {
  rank=same;
  
  x -> y [style=solid];
  y -> x [style=dashed, color=red];
  }
}
```

<aside class="notes" data-markdown>
Symmetry is about the relationships of elements between each other. It tells us
about element neighbourhoods and allows single-stepping.
</aside>

# Transitivity

$\Omega$ is <dfn>transitive</dfn> iff it allows skipping intermediates:<br>
  $\forall x, y, z: x\Omega y \land y\Omega z \Rightarrow x\Omega z$
```{.render_dot}
digraph G {
  bgcolor=transparent;
    subgraph sg {
    rank=same;
     
    x -> y [style=solid];
    y -> z [style=solid];
    x -> z [style=dotted];
  }
}
```


<aside class="notes" data-markdown>
Transitivity is what allows the extension to global reasoning. It's what allows
stepping more than one step, which effectively means any number of steps.
</aside>


# Equivalence

$\Omega$ is an <dfn>equivalence</dfn> relation iff it is _reflexive_,
_symmetric_ and _transitive_.

Equality is an equivalence relation.


# Order

$\Omega$ is an <dfn>order</dfn> iff it is _antireflexive_, _asymmetric_ and
_transitive_.


# Quiz #1

Reflexive, Antireflexive, Symmetric, Asymmetric, Transitive, Equivalence, Order

Which is this:

```{.render_dot}
digraph G {
  x -> x;
  y -> y;
  z;
  x -> y;
  y -> z;
  z -> x;
}
```

# Equivalence class

An equivalence relation splits a set into equivalence classes. The equivalence
relation _induces_ an equality on the set of equivalence classes, but it is not
an equality over the orignial set.


# Finer-than

Let $\Omega$ and $\Delta$ be equivalences over the same set.

$\Delta$ is <dfn>finer</dfn> than $\Omega$ iff when $\Delta$ deems elements equivalent, so does $\Omega$.

$\forall x, y: x \Delta y \Rightarrow x \Omega y$

You can think of $\Delta$ having a "better resolution".

Trivially, equality on a set is the _finest_ of all equivalence relations.


# What is equality

What is life? What is meaning?

<aside class="notes" data-markdown>
Math: You have a set. The set has elements. An element is only equal to itself.

That might not be very useful - equality on the real numbers is a very complicated affair, for instance - crucially, though, there is only one $\pi$.
</aside>


# The Application
- why
- algorithms
- data structures


# C++20: The Language & Core

# C++20: The Library
