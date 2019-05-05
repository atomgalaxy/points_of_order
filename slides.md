---
title: Points of Order
pdf: points_of_order.pdf
slideNumber: true
controls: true
---
\newcommand{\@@}{\cdot}
\newcommand{\true}{\text{true}}
\newcommand{\false}{\text{false}}

\newcommand{\iff}{\Leftrightarrow}
\newcommand{\imp}{\Rightarrow}

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


# Floating Point (iec559 types)

## At long last...

`float`, `double` and `long double` are offering the iec559 ordering primitives.

Lawrence Crowl deserves the shout-out for this one; while my own (joint) paper
brought it home, he started it many years ago.


## Strong Order

`std::strong_order(float, float)` will get you an actual strong order on floats.
`NaN`s compare equal, but `NaN`s with different payload compare different, as do
the infinities.

## Weak Order

`std::weak_order(float, float)` will get you what you (probably) want - an
ordering on floats that treats `NaN`s the same, and infinities the way you'd
expect.

## Partial Order?

Partial order is what you get with `<=>` on ice559 types. It's there because
it's currently there, and we couldn't fix it. This keeps `float` as one of the
non-Regular types.


# A Type with no Natural Ordering

## Let's take `complex<T>`

Complex numbers don't have a natural ordering, but we still want to put them
into maps. A lexicographic ordering is fast, and it would make sense for it to
be shipped with complex. So how do we do that?

Also, let's make complex Regular, because it will be a better example that way.


## Should we define `<=>`?

NO! `<=>` is for natural orderings, and complex numbers don't have one.

## So... what?

We just define `==` (to fix Regular) and provide the customization points!

```cpp
template <typename T>
struct complex {
  T real;
  T imag;

  bool operator==(complex other) {
    return std::strong_order(real, other.real) == 0 &&
           std::strong_order(imag, other.imag);
  }

  // != is obtained by rewriting
  friend std::strong_ordering strong_order(complex x, complex y) {
    if (auto r = std::strong_order(x.real, y.real); !std::is_eq(r)) { return r; }
    return std::strong_order(x.imag, y.imag);
  }
  friend std::weak_ordering weak_order(complex x, complex y) {
    if (auto r = std::weak_order(x.real, y.real); !std::is_eq(r)) { return r; }
    return std::weak_order(x.imag, y.imag);
  }
};
```


<!-- end floating point -->

<!--
TODO:
- partial orders - which algorithms are they used in
- key functions are exactly what defines weak orders

- suggestion: cmp_proxy protocol?
- `==` is the finest equivalence that makes sense for a type, and *possibly* <=>
  is then the finest ordering that makes sense for a type (since it is strong).
  - types that just have a partial ordering *should* implement the
    `partial_order` customization point, and not <=>
- usefulness: < is a global relation that's usually O(input), not O(set) - that
  is exactly what makes it such an amazing optimization tool.

philosophy:


needs internet:
- define operator <=> for a class that can't forward it (how does the standard
  synthesize it?)
- the rewriting rules: have a few slides describing those
- fallback* functions - at least mention them. Maybe when we're defining the
  class that is defining its own?

maybe:
- lattices?
  - lattices have intervals

next pass:
- order the slides so they make sense

finalizing:
- doublecheck is_eq name in complex
- figure out how to do picture-on-the-side of the slide
-->
