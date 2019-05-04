---
title: Points of Order
pdf: points_of_order.pdf
slideNumber: true
controls: true
---

\newcommand{\iff}{\Leftrightarrow}
\newcommand{\imp}{\Rightarrow}

# Points of Order {bg=#fff}

Gašper Ažman

May 9th, 2019

<img src='assets/cpp_now_logo.svg' style='height: 3ex'>


# Overview

1. The Math <small>(order theory)</small>
2. The Application <small>(Element of Programming)</small>
3. C++ 20: The Language & Core <small>(`<=>`, `==` and `std::strong_ordering`)</small>
4. C++ 20: The Library <small>(`std::strong_order`, `variant`, `vector` & `optional`)</small>


# The Math: Order Theory

## Overview

Order theory deals with specific kinds of binary _relations_ over sets of
elements.

Relations have many properties. These 8 matter:

- reflexive & antireflexive
- symmetric, asymmetric (or strict) and antisymmetric
- transitive
- equivalence
- order (partial, weak, strong, total)

Let $\Box$ be a relation.

<aside class="notes" data-markdown>
- "Sets" is how we get around the platonic equality problem - we just have
  elements, and elements are always only equal to themselves, because sets come
  with equality built in.
</aside>


## Reflexivity

$\Box$ is <dfn>reflexive</dfn> iff an element is always related to itself<br>
  $\forall x: x\Box x$
```{.render_dot}
digraph G {
  bgcolor=transparent;
  x -> x [style=dotted];
}
```

$\Box$ is <dfn>antireflexive</dfn> iff an element is never related to itself<br>
  $\forall x: \neg x\Box x$.<br>
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


## Symmetry

$\Box$ is <dfn>symmetric</dfn> iff it always goes both ways<br>
  $\forall x, y: x\Boxy \Rightarrow y\Box x$
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

$\Box$ is <dfn>antisymmetric</dfn> iff it never goes both ways<br>
  $\forall x, y: x\Boxy \Rightarrow \neg y\Box x$

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

## Transitivity

$\Box$ is <dfn>transitive</dfn> iff it allows skipping intermediates:<br>
  $\forall x, y, z: x\Box y \land y\Box z \Rightarrow x\Box z$
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


## Equivalence

$\Box$ is an <dfn>equivalence</dfn> relation iff it is _reflexive_,
_symmetric_ and _transitive_.


## Order

$\Box$ is an <dfn>order</dfn> iff it is _antireflexive_, _asymmetric_ and
_transitive_.


## Quiz #1

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


# The Application
- why
- algorithms
- data structures


# When should I actually call `<=>` by name?

Binary search of all kinds. End.

```cpp
iterator map<T>::find(T const& needle) {
  node* current = _head;
  while (current != nullptr) {
    std::weak_ordering const result = current->value <=> needle;
    switch (result) {
      case ::std::weak_ordering::equivalent:
        return {current};
      case ::std::weak_ordering::less:
        current = current->left;  break;
      case ::std::weak_ordering::greater;
        current = current->right; break;
  }
  return end();
}
```


# `==` is an optimization of `<=>`

The first thing you do after calling `lower_bound` is compare the return value.

Don't break this code:
```cpp
auto match = std::lower_bound(begin, end, needle);
//           ^^^^^^^^^^^^^^^^ uses <, which rewrites to <=>
if (match != end() || *match != needle) {
//                           ^^ rewrites to ==
  /* not found */
} else {
  /* found */
}
```


# Reference

- How do I write my own `<=>` correctly?

# Beyond `<=>`

# The purpose of `std::weak_ordering`

## When?

A _total ordering_ is a _weak ordering_ when its equivalence is different from
equality _on the type_.

Let this sink in.

`compare(x, y) == 0` is different from `x == y`.

<div class='footnotes'>
`questions.push("But what about case_insensitive_string?!!");`
</div>


## The Great Reveal: it's for 3-way comparators

`std::weak_ordering case_insensitive_compare(std::string const& x, std::string const& y)`.

When your comparator is not compatible with the `==` on the type, use a
`std::weak_ordering`.

It's a bit niche.


# So what about `case_insensitive_string`?

## Oh, everyone's favorite bugbear.

With case insensitive string, you have to decide:


## You are telling the truth

You _really_ don't care about case.

`==` is an **equality** on the _set of equivalence classes_ over
`case_sensitive_string`.

($\Rightarrow$ `<=>` is _strong_!)


## You are lying

You do care about case (`==` distinguishes case).

STOP! Rename your type, and provide a strong `<=>`.

Repeat after me:<br>
`x == y` is an _optimization_ of `(x <=> y) == 0`.

Anything else is insanity.


# The purpose of `std::partial_ordering`

It's to express partial orders, d0h!

Don't make `operator<=>` return `std::partial_order`.


# Natural Orderings

`<=>` expresses natural strong orderings.


# Any-order for your type

Say your type `T` does not have a natural strong ordering, but it *allows* for
one.  What do you do?

You provide a `strong_order(T const& x, T const& y)` (in your namespace)
customization point!

(`std::strong_order` will dispatch to it, or `<=>` if it returns
`std::strong_ordering`)



# Shoutouts

- Editors of cppreference.org. You guys make everyone's lives so much easier!

- Arthur O'Dwyer: for giving a 90-minute talk on _An allocator is a handle to a
  heap_. Much indirect inspiration was had.


# Practice 1

## Problem Statement

Let $\lt$ be a **weak** ordering over the set $S$. Prove that 
$$
\neg(x \lt y) \wedge \neg(y \lt x)
$$
is an equivalence relation.

$\lt$ is a weak ordering, which means that there is some equivalence relation
$\sim$ on $S$ such that exactly either $x \lt y$ or $y \lt x$ or $x \sim y$.

We will prove that $\Delta$ is $\sim$.


## The proof

We are trying to prove that if $x \Delta y \iff x \sim y$.

Trichotomy law: exactly one of $x \lt y$, $y \lt x$, $y \sim x$ is true.<br>
Grouping the inequality terms gives us:
$$\neg(x \lt y) \wedge \neg(y \lt x) \iff x \sim y$$

$x \Delta y$<br>
$\iff \neg(x \lt y) \wedge \neg(y \lt x)$ (rewrite to definition)<br>
$\iff x \sim y$ (lemma above)

$\Box$


# Practice 2

## Problem statement

Let $\subset$ be a **partial** ordering over the set $S$. Prove that 
$$
\neg(x \subset y) \wedge \neg(y \subset x)
$$
need not be an equivalence relation. Let's call it $\Delta$ again.


## The rub

$\neg(x \subset y) \wedge \neg(y \subset x)$

- reflexivity: It's obviously reflexive, since $x \subset x$ is always
  `false`.$\Box$
- symmetry: the expression is symmetric. $\Box$
- transitivity: ... we might find something.

## Transitivity

Consider this graph of the relation $\subset$.

```{.render_dot}
digraph G {
  bgcolor=transparent;
  a -> c [style=solid];
  b;
}
```

* $a \Delta b$ and $b \Delta c$, since neither pair are $\subset$-related.
* If $\Delta$ were transitive, it would follow $a \Delta c$.
* However, $a \subset c$, which means $\neg (a \Delta c)$.
* We have found a counterexample.$\Box$


# C++20: The Language & Core



# C++20: The Library

<!--
TODO:
- figure out how to do picture-on-the-side of the slide
- floating point
- define strong_order and weak_order for complex
- define operator <=> for a class that can't forward it (how does the standard
  synthesize it?)
- lattices?
  - lattices have intervals
- partial orders - which algorithms are they used in
- key functions are exactly what defines weak orders
- order the slides so they make sense
- fallback* functions - at least mention them
- the rewriting rules: have a few slides describing those
- suggestion: cmp_proxy protocol?
- `==` is the finest equivalence that makes sense for a type, and *possibly* <=>
  is then the finest ordering that makes sense for a type (since it is strong).
  - types that just have a partial ordering *should* implement the
    `partial_order` customization point.
- usefulness: < is a global relation that's usually O(input), not O(set) - that
  is exactly what makes it such an amazing optimization tool.
-->
