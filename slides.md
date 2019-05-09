---
title: Points of Order
pdf: points_of_order.pdf
slideNumber: true
controls: true
theme: black

---

\newcommand{\@@}{\cdot}
\newcommand{\true}{\text{true}}
\newcommand{\false}{\text{false}}

\newcommand{\iff}{\Leftrightarrow}
\newcommand{\imp}{\Rightarrow}
\newcommand{\Z}{\mathbb{Z}}


# Points of Order {bg=#fff}

Gašper Ažman

May 9th, 2019

<img src='assets/cpp_now_logo.svg' style='height: 3ex'>


# Intro

## Agenda

We'll look at

- How order works in mathematics
- How C++ models that
- How C++ supports _you_ in getting it right.


<aside class="notes" data-markdown>
Hello, thank you for coming!

This today will be a joint math and C++ talk.
</aside>


## Thank You, Coauthors!

- **Barry Revzin**, for writing so many papers about this I've lost count
- **Jeff Snyder**, for writing P0863 and then co-authoring P0891R2 with me
- **Lawrence Crowl**, for merging the venerable P0100 with P0891R2


## Thank You, Collaborators!

- **Herb Sutter**, **Jens Maurer** and **Walter E. Brown** for writing the original P0515,
  and oh-so-much of their time and patience
- **Tony van Eerd** and **Lisa Lippincott**, for their input around what
  "strong" means, and **Arthur O'Dwyer** for further detailed discussion
- **Agustín Bergé**, **Tim Song** and **Richard Smith** for further deep
  discussions about implementation and specification

## 💖💖💖

And by far the most, my wife, **Zoetica Ebb**, for keeping me alive and sane
through all this.

🌌🦄🌈


# Equality and Equivalence

## Equality

In set theory, $x = y$ iff they are the same element.

Let $S = \{a, b, c, d\}$.

```{.render_dot}
digraph {
    label=S;
    a -> a;
    b -> b;
    c -> c;
    d -> d;
}
```
Graph of $=_S$

Let that sink in. This is crucial for understanding order.


## Equivalence

Sometimes, that is too **fine**, and we need a **coarser** relation that is
_like_ equality, but is not equality.


```{.render_dot}
digraph {
    label=S;
    {a, b} -> {a, b};
    {c, d} -> {c, d};
}
```
Graph of $\sim$

Let's call this relation on $S$ "$\sim$". $a \sim b$ and $c \sim d$.


## Equivalence Classes

Equivalence relations like $\sim$ induce equivalence classes over the elemements:

```{.render_dot}
digraph {
    subgraph cluster_X {
        {a, b} -> {a, b};

        label="E";
        graph [style=dotted];
    }

    subgraph cluster_Y {
        {c, d} -> {c, d};

        label="F";
        graph [style=dotted];
    }
    label=P;
}
```

$P = \{E, F\}$, where $E = \{a, b\}$ and $F = \{c, d\}$.


## Quotient Set

```{.render_dot}
digraph {
    subgraph cluster_X {
        {a, b} -> {a, b};

        label="E";
        graph [style=dotted];
    }

    subgraph cluster_Y {
        {c, d} -> {c, d};

        label="F";
        graph [style=dotted];
    }
    label=P;
}
```

The set $P = \{E, F\}$ is also called a **quotient set**, and denoted $S/\sim$.


## Equality on the Quotient Set

We can test equality on $P$ cheaply through $\sim$!
```{.render_dot}
digraph {
    subgraph cluster_X {
        {a, b} -> {a, b};

        label="E";
        graph [style=dotted];
    }

    subgraph cluster_Y {
        {c, d} -> {c, d};

        label="F";
        graph [style=dotted];
    }
    label=P;
}
```

$(X, Y\in P: X =_P Y) \iff (\exists x \in X, y \in Y: x \sim y)$<br>
It's true for any element.


## The Punchline

Through this process, the _equivalence_ $\sim$ on $S$ **induces** an _equality_
on the quotient set $S/\sim$.

This is true for any equivalence relation.

If we are not being careful with our words, we just say that $\sim$ is an
equality over the set of its equivalence classes.


## Bonus:

Note the cheap testing - checking for _equality_ of equivalence classes is the same as checking for _equivalence_ of any two representatives.

This is far faster than checking for set equality, which can be $O(n^2)$.


# Equality and Equiv.: C++

## Context: Types

What is a type?

From Elements of Programming:

- A **value type** is a correspondence between a _species_ and a `datum`.
- **datum**: a finite sequence of $1$s and $0$s.
- **Species**: describes a set of common properties of _essentially equivalent_ entities.

Oh well, that got us far :).


## The Point

Equality and Type are inextricably linked. Effectively, EoP says:

- a type is how we map _datums_ to entities;
- what an entity *is* is determined by the Equivalence, which separates
  different entities.

This equivalence over _entities_ induces an _equality_ on the type.


## Let me Repeat

The equivalence over entities ~~induces~~ _defines_ ~~an~~ the equality on the type.


## Example: `int32_t`

- Species: subset of $\Z$.
- Equivalence: $=_{\Z}$
- Datum: 32 consecutive bits
- Induced equality: bitwise comparison


## Example: null-terminated case-insensitive string (ascii)

- Species: strings of (abstract) ascii characters, ending in `0`
- Equivalence: case-insensitive comparison
- Datum: an arbitrarily long sequence of bytes
- Induced equality: `stricmp(x, y) == 0`

Yes, it's an **equality**.


## Example: `std::string`

- Species: strings of abstract characters.
- Equivalence: character-by-character comparison.
- Datum: an arbitrarily long sequence of bytes, with length, location and capacity
  information, possibly disjoint.
- Induced equality: `std::string operator==`

Note that we ignore `capacity()`. This is *obvious* from the _species_ that `std::string` is related to.


# Equivalence: _finer_ and _coarser_

## Examples:

Some equivalences over `std::string`:

- `x == y`
- `strcmp(x, y)`
- `stricmp(x, y)`
- `x.size() == y.size()`
- `x == y && x.capacity() == y.capacity()`


## Definitions

Let $\Omega$ and $\Delta$ be equivalences over the same set.

$\Delta$ is <dfn>finer</dfn> than $\Omega$ iff<br>
whenever $\Delta$ deems elements equivalent, so does $\Omega$.

$\forall x, y: x \Delta y \Rightarrow x \Omega y$

We say that $\Omega$ is coarser than $\Delta$.


## Exercise:

1. `x == y`
2. `strcmp(x, y)`
3. `stricmp(x, y)`
4. `x.size() == y.size()`
5. `x == y && x.capacity() == y.capacity()`

Which ones are finer than others?


## Answers:

1. `std::string::operator==` - finest
2. `strcmp(x, y)` - ignores everything after a `\0`
3. `stricmp(x, y)` - also ignores case
4. `x.size() == y.size()` - coarser than 5 and 1, independent of 2 and 3
5. `x == y && x.capacity() == y.capacity()` - finest


## So what about 5?

It's not an equality:

- for one, it's not `==`, and that's what equality is *defined* as
- it's not in tune with the _species_ of `std::string`


It's not even an equivalence over `std::string`!

It *is* an equivalence (and even an equality) on some other type that shares
`std::string`'s representation, but not its _species_.


## Proof by counterexample

The trick: substitute equal element in equation.

Symmetry:
```cpp
bool eq(x, y) { return x == y && x.capacity() == y.capacity(); }
std::string a{}, b{};
a.reserve(1);
eq(a, a) => eq(a, a)  // substitute the last a for b, since a == b
eq(a, a) => eq(a, b)  // implication does not hold (true => false)
```
`eq` is not an equivalence relation over `std::string`.


## Equality Take-Aways

- Equality is the finest equivalence on the type
    - (perhaps not on the representation)
- There is no such thing as weak equality
- Equality (`op==`) *encodes* the species of the type
- $x == y \imp f(x) == f(y)$ substitutability works for math, but is a
  confusing notion in programming. It's better to think in terms of the domain,
  where the _species_ is from.


# C++20: Equality Rules

## Defaulted operator==:

```cpp
struct X : Base {
  int a;
  Y b;
  Z c;

  // bool operator==(X const&) constexpr const = default
  bool operator==(X const& y) const {
    Base const& base = *this;
    Base const& y_base = y;
    return base == y_base && a == y.a && b == y.b && c == y.c;
  }

};
```

## Order Switching: `t == u` $\to$ `u == t`

Let `T` and `U` be types, and `t` and `u` values of those types.

```t == u```

The language will try:

* `t.operator==(u)`, `operator==(t, u)`
* `u.operator==(t)`, `operator==(u, t)`


These form an overload set. Same goes for `!=`


## `t != u` $\to$ `!(t == u)`

If `operator!=` is not defined for `T`, then rewriting happens:

```cpp
t != u; // rewrites to !(t == u)
```

Of course, additional order switching may happen afterwards.


# Order




# Spoilers

## Like, what is this about?

Ordering and spaceships.


## Why spoilers?

Because they're better than overviews.


## But what is this really about?

* When and how do I define `<=>`
* When and how do I use `<=>`
* Have you finally solved `float`?
* can I put random stuff into `std::map` yet?
* What's with the rewriting rules? (There are rewriting rules?)


## You said these are spoilers

Well... more like a semi-accurate preview so we can get the first pass out of
the way, and find better questions.

Here are the answers:


## When do I define `<=>`

When your type has a natural strong$^{\dagger}$ ordering.

<div class='footnotes'>
$^\dagger$`questions.push("what are std::partial_ordering and std::weak_ordering for then?");`

If you don't believe me, ask Titus Winters.

`questions.push("what is a natural ordering?");`
</div>

## How do I define `<=>`

```
std::strong_ordering operator<=>(T const& other) = default;
```

`<=>` replaces `<, <=, >=, >` and sometimes `==` and `!=`.


## When do I use `<=>`

you don't, directly$^\dagger$. Use normal comparisons.

<div class='footnote'>
$^\dagger$ unless chaining or you know better.

`questions.push("chaining?");`
</div>

<!-- this is super cool because you need less codegen -->
## Have you solved `float`?

Yeah.

## Can I put random stuff into maps yet?

`questions.pop("std::strong_order and std::weak_order???");`

We have new customization points for types that just want to enable uses that
need *an* order, and don't care what the order *means*.


## What's with the rewriting rules?

There are rewriting rules. Basically:


- `x < y` gets rewritten into `(x <=> y) < 0` (and similar for other
  operators)
- `x != y` gets rewritten into `!(x == y)`, and `==` will be autogenned from
  member `==`s.


## Fin

That's all the spoilers. Here are the better questions we've found:

- What is a natural ordering?
- How do we correctly chain calls to `<=>` when implementing it?
- What are the new `std::strong_order`, `std::weak_order` and
  `std::partial_order` customization points for?
- What exactly are the rewriting rules?


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


# Properties of Relations

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


## Reflexivity

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


## Symmetry

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

## Transitivity

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


## Equivalence

$\Omega$ is an <dfn>equivalence</dfn> relation iff it is _reflexive_,
_symmetric_ and _transitive_.

Equality is an equivalence relation.


## Order

$\Omega$ is an <dfn>order</dfn> iff it is _antireflexive_, _asymmetric_ and
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

<!-- TODO quizzes. -->

# Order Relations

An Order is an antireflexive, asymmetric, transitive relation.

- We omit the transitive closure edges when drawing them.

TODO make examples of order drawings.


## Strong Order

Linear order, obeys trichotomy law.

- Induced equivalence classes are same as equality on the type, which means they
  are singletons.
- Given a set `S`, the strong order on `S` is the finest order on `S`.


## Weak Order

Linear order, obeys trichotomy law.
- Most often induced with key functions or projections.
- Induced equivalence classes can be interesting.


## Partial Order

Not a linear order. This is the order of DAGs.


# Properties of Order Releations

## Finer Order

We say a linear order `<` is <dfn>finer</dfn> than linear order `\prec` iff:
$\forall x, y \in S: x < y \imp x \preceq y$.

Effectively, if `<` distinguishes between the elements one way, then `\prec`
must either not distinguish between them, or must say the same thing.


## Reversal

If `<` is a linear order, then so is `>`, and they induce the same equivalence
classes.


# Practice Proof 1

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


# Practice Proof 2

## Problem statement

Let $\prec$ be a **partial** ordering over the set $S$. Prove that 
$$
\neg(x \prec y) \wedge \neg(y \prec x)
$$
need not be an equivalence relation. Let's call it $\Delta$ again.


## The rub

$\neg(x \prec y) \wedge \neg(y \prec x)$

- reflexivity: It's obviously reflexive, since $x \prec x$ is always
  `false`.$\Box$
- symmetry: the expression is symmetric. $\Box$
- transitivity: ... we might find something.


## Transitivity

Consider this graph of the relation $\prec$.

```{.render_dot}
digraph G {
    bgcolor=transparent;
    color=white;

    a -> c [style=solid];
    b;
}
```

* $a \Delta b$ and $b \Delta c$, since neither pair are $\prec$-related.
* If $\Delta$ were transitive, it would follow $a \Delta c$.
* However, $a \prec c$, which means $\neg (a \Delta c)$.
* We have found a counterexample.$\Box$


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



## `std::strong_ordering operator<=>(T const&)`

```cpp
struct X : Base {
  int a;
  Y b;
  Z c;

  // std::strong_ordering operator<=>(X const&) const = default;
  std::strong_ordering operator<=>(X const& y) const {
    Base const& base = *this;
    Base const& y_base = y;
    if (std::strong_ordering r = base <=> y_base; r != 0) { return r; }
    if (std::strong_ordering r =    a <=> y.a;    r != 0) { return r; }
    if (std::strong_ordering r =    b <=> y.b;    r != 0) { return r; }
    return                          c <=> y.c;
  }
};
```




## Relational Operators $\to$ `<=>`

If they are defined, nothing changes. If not:

`t < u` rewrites to `(t <=> u) < 0`

More generally, for $@ \in \{<, <=, >=, >\}$ 

`t @ u` rewrites to `(t <=> u) @ 0`


## Order Switching: `<=>`

```cpp
     t  <  u            // rewrites to
    (t <=> u) < 0       // or
0 < (u <=> t)
```

Same for the other operators.

The rules for `<=>` are completely symmetric, regardless of whether it's
declared as a member or not.



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
- Herb Sutter, Jens Maurer, Walter E. Brown, Barry Revzin, Jeff Snyder, David
  Stone for working on getting ordering correct in the language.


# Floating Point (iec559 types)

## At long last...

`float`, `double` and `long double` are offering the iec559 ordering primitives.

Lawrence Crowl deserves the shout-out for this one; while my own (joint) paper
brought it home, he started it many years ago with P0100.


## Strong Order

`std::strong_order(float, float)` will get you an actual strong order on floats.
`NaN`s compare equal, but `NaN`s with different payload compare different, as do
the infinities.


## Weak Order

`std::weak_order(float, float)` will get you what you (probably) want - an
ordering on floats that treats `NaN`s the same, and infinities the way you'd
expect.


## Partial Order?

Partial order is what you get with `<=>` on `ice559` types. It's there because
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


# Adapting `<=>` for use with Map - just use the regular operators.

```cpp
struct strong_ordering_less {
  template <typename T>
  bool operator(T const& x, T const& y) {
    return std::is_lt(std::strong_order(x, y));
  }
};

struct weak_ordering_less {
  template <typename T>
  bool operator(T const& x, T const& y) {
    return std::is_lt(std::weak_order(x, y));
  }
};


std::map<T, strong_ordering_less>
```


# Partial Orders and Lattices




# Bibliography:
- order switching: https://htmlpreview.github.io/?https://github.com/BRevzin/cpp_proposals/blob/master/118x_spaceship/d1630r0.html

-

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

- what about orders that are "stronger than strong"
- an ordering induces equivalence classes
- every type that has a strong ordering has at least two - the opposite way as
  well.
- strong_ordering is a signal that == induces the same equivalence classes
- an ordering finer than strong_ordering is still a weak_ordering!
- interplay with std::hash?

- How would I use an ordering derived from std::strong_order in map?
 

maybe:
- lattices?
  - lattices have intervals
- ordering for pointers.

next pass:
- order the slides so they make sense

finalizing:
- doublecheck is_eq name in complex
- figure out how to do picture-on-the-side of the slide


-->

<!--
Topics:

- Goals for today
    - What is equality
    - What is equivalence

- Equality and Equivalence:
    - Math:
        - Equality: an equivalence relation that tells elements of a set apart.
        - Equivalence: a reflexive, symmetric and transitive relation.
    - Language:
        - Defaulting
        - Rewriting Rules









-->


