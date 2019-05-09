---
title: Points of Order
# pdf: points_of_order.pdf
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
\newcommand{\N}{\mathbb{N}}
\newcommand{\Q}{\mathbb{Q}}
\newcommand{\R}{\mathbb{R}}
\newcommand{\C}{\mathbb{C}}


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

- **Herb Sutter**, **Jens Maurer** and **Walter E. Brown** for writing the
  original P0515, and oh-so-much of their time and patience
- **Tony van Eerd** and **Lisa Lippincott**, for their input around what
  "strong" means, and **Arthur O'Dwyer** for further detailed discussion
- **Agustín Bergé**, **Tim Song** and **Richard Smith** for further deep
  discussions about implementation and specification
- And **David Stone** for his extensive work in getting the library and
  `operator==` right.

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

<!-- _asdf -->

## The Punchline

Through this process, the _equivalence_ $\sim$ on $S$ **induces** an _equality_
on the quotient set $S/\sim$.

This is true for any equivalence relation.

If we are not being careful with our words, we just say that $\sim$ is an
equality over the set of its equivalence classes.


## Bonus:

Note the cheap testing - checking for _equality_ of equivalence classes is the
same as checking for _equivalence_ of any two representatives.

This is far faster than checking for set equality, which can be $O(n^2)$.


# Equality and Equiv.: C++

## Context: Types

What is a type?

From Elements of Programming:

- A **value type** is a correspondence between a _species_ and a `datum`.
- **datum**: a finite sequence of $1$s and $0$s.
- **Species**: describes a set of common properties of _essentially equivalent_
  entities.

Oh well, that got us far :).


## The Point

Equality and Type are inextricably linked. Effectively, EoP says:

- a type is how we map _datums_ to entities;
- what an entity *is* is determined by the Equivalence, which separates
  different entities.

This equivalence over _entities_ induces an _equality_ on the type.


## Let me Repeat

The equivalence over entities ~~induces~~ _defines_ ~~an~~ the equality on the
type.


## Example: `int32_t`

- Species: subset of $\Z$.
- Equivalence: $=_{\Z}$ <!-- _asdf -->
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
- Datum: an arbitrarily long sequence of bytes, with length, location and
  capacity information, possibly disjoint.
- Induced equality: `std::string operator==`

Note that we ignore `capacity()`. This is *obvious* from the _species_ that
`std::string` is related to.


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

1. `x == y` :: baseline
2. `strcmp(x, y)` :: ignores everything after a `\0`
3. `stricmp(x, y)` :: also ignores case
4. `x.size() == y.size()` :: coarser than 5 and 1, independent of 2 and 3
5. `x == y && x.capacity() == y.capacity()` :: finest


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


## Put another way

`std::string` is a quotient space modulo `.capacity()`.

$f(x, y)$ is **not a function** on this quotient space, it is a _multifunction_.


# Equality Take-Aways

##

Equality is the finest equivalence on the type

(perhaps not on the representation)

## 

There is no such thing as weak equality.

<div class='footnotes'>
Ask Tony van Eerd if you don't believe your own brain.
</div>

## 

Equality (`operator==`) *encodes* the species of the type

## 

$x = y \imp f(x) = f(y)$ substitutability:

- works for math
- confusing notion in programming.

It's better to think in terms of the domain, where the _species_ is from.


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

`t != u;` rewrites to !(t == u)

Of course, additional order switching may happen afterwards.


## Extra types in the library

There are none for equality.

`std::strong_equality` and `std::weak_equality` are a figment of the
imagination. They are meaningless.

There is no such thing as weak equality.


# Order

We've figured out equality.

Order should be easy.


# Order Relations

## Definition

An <dfn>Order</dfn> is an _antireflexive_, _asymmetric_, _transitive_ relation.

Let's break this down.

Let $\Omega$ and $\Delta$ be relations.


## Reflexivity and Antireflexivity

$\Omega$ is <dfn>reflexive</dfn> iff an element is always related to itself<br>
  $\forall x: x\Omega x$
```{.render_dot}
digraph G {
  x -> x [style=dotted];
}
```

$\Omega$ is <dfn>antireflexive</dfn> iff an element is never related to itself<br>
  $\forall x: \neg x\Omega x$.<br>
  EoP calls this <dfn>strict</dfn>.


```{.render_dot}
digraph G {
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
    subgraph sg {
    rank=same;
     
    x -> y [style=solid];
    y -> x [style=dotted];
  }
}
```

$\Omega$ is <dfn>asymmetric</dfn> iff it never goes both ways<br>
  $\forall x, y: x\Omega y \Rightarrow \neg y\Omega x$

```{.render_dot}
digraph G {
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
    subgraph sg {
    rank=same;
     
    x -> y [style=solid];
    y -> z [style=solid];
    x -> z [style=dotted];
  }
}
```

Transitivity allows extension of local to global reasoning through induction.

<aside class="notes" data-markdown>
Transitivity is what allows the extension to global reasoning. It's what allows
stepping more than one step, which effectively means any number of steps.
</aside>


## Partial Order

Antireflexive, Assymetric, Transitive

```{.render_dot}
digraph G {
    subgraph sg {

    a -> b -> d;
    a -> c -> d;
  }
}
```

This is the order of directed acyclic graphs.<br>
$b$ and $c$ are not ordered with respect to each other.

<aside class="notes" data-markdown>
Note: for transitive relations, we don't draw the transitive closure arrows.
</aside>


## Linear (or Total) Order

Linear orders obey the trichotomy law:

An order is <dfn>linear</dfn> iff there exists an equivalence $\sim$, such that
exactly one of the statements is true for any given pair $(x, y)$:

- $x < y$
- $x > y$
- $x \sim y$


## Example:

```{.render_dot}
digraph G {
    subgraph sg {

    subgraph cluster_1 { a; } 
    subgraph cluster_2 { b; c; }
    subgraph cluster_3 { d }

    a -> b -> d;
    a -> c -> d;

  }
}
```
Graph of $\sim$ and $\prec$.

$^\prec/_\sim$<!-- _a--> is a linear order on $S/_\sim$.<!-- _a-->


## Strong Order

An order is <dfn>strong</dfn> if its equivalence ($\sim$) is the equality ($=$)
on the set.

An order is <dfn>weak</dfn> if this need not be true<br>
(every strong order is a weak order).


# Practice Proof 1

## Problem Statement

Let $\lt$ be a **weak** ordering over the set $S$. Prove that 
$$
\Delta := \neg(x \lt y) \wedge \neg(y \lt x)
$$
is an equivalence relation.

## The proof: idea

$\lt$ is a weak ordering, which means that there is some equivalence relation
$\sim$ on $S$ such that exactly either $x \lt y$ or $y \lt x$ or $x \sim y$.

We will prove that $\Delta$ is $\sim$.


## The proof (2)

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
    a -> c [style=solid];
    b;
}
```

* $a \Delta b$ and $b \Delta c$, since neither pair are $\prec$-related.
* If $\Delta$ were transitive, it would follow $a \Delta c$.
* However, $a \prec c$, which means $\neg (a \Delta c)$.
* We have found a counterexample.$\Box$


# Natural Order

## Definition

Every value type is defined through its equality, which could be called a natural equivalence on the type.

Some types also have a <dfn>natural order</dfn>.

The natural order on a type induces the same equivalence classes as its
equality, and is therefore always a strong order.


## Examples
- Points in a DAG have a natural partial order
- $\N$ have a natural order that follows from their inductive definition
- $\Z$'s follows from $\N$'s for magnitude and a rule for the sign.
- $\R$ have an order they inherit from $\Q$
- $\C$ ... do not have a natural order. They are inherently two-dimensional.


# Takeaways:

## 

Every linear order has an associated equivalence relation $\sim$.

This equivalence is $x \sim y := \neg(x < y) \wedge \neg(x > y)$.

## 

This equivalence induces equivalence classes.

By extension, we can say that a linear order induces equivalence classes.

##

If the equivalence is the equality, the order is strong.

##

Partial orders do not induce equivalence classes!

##

Some types have a natural strong order.


# Properties of Order Relations

## Finer Order

We say a linear order $<$ is <dfn>finer</dfn> than linear order $\prec$ iff:
$\forall x, y \in S: x < y \imp x \preceq y$.

Effectively, if $<$ distinguishes between the elements one way, then $\prec$
must either not distinguish between them, or must say the same thing.


## Reversal

If $<$ is a linear order, then so is $>$, and they induce the same equivalence
classes.

If there is a weak order, there are always at least two orders!


# C++: `std::strong_ordering`

## Definition

```cpp
struct strong_ordering {
    static const strong_ordering::less;
    static const strong_ordering::equal;
    static const strong_ordering::greater;
    static const strong_ordering::equivalent; // <- because is-a weak order
    /* comparison operators, conversions, etc */
};
```

## Intended Usage

Let's call a function that returns one of the above types an <dfn>order</dfn>,
since it's not a predicate.

Say `order(x, y)` is a linear order.

If `order(x, y) == 0` iff `x == y`, then `order` should return
`std::strong_ordering`.

In other words, `std::strong_ordering` is a semantic guarantee that
`order(x, y)` induces the same equivalence classes as `==`.


## Examples

`strcmp(char const*, char const*)` should be returning `std::strong_ordering`,
or a type derived from it. It is a strong order wirth respect to
`strcmp(x, y) == 0`, which is its equality.

The lexicographical order on `std::complex<float, float>` (if we disallow NaN
and infinities) is a strong order, but not a natural order.

There are infinitely many strong orders on the complex numbers: can you think
of them?


# C++: `std::weak_ordering`

##  Definition

```cpp
struct weak_ordering {
    static const weak_ordering::less;
    static const weak_ordering::equivalent;
    static const weak_ordering::greater;
    /* notice - no ::equal */
    /* comparison operators, conversions, etc */
};
```

## Intended Usage

Every order that induces _different_ equivalence classes than the equality on the
type is a weak order.

Or: if `order(x, y) == 0` need not be iff `x == y`, it's a weak order.

This is true even if the `order(x, y)` is _finer_ than the natural order on the
type.


## Examples

`stricmp(char const*, char const*)` is a weak order on null-terminated strings
with `strcmp`-induced equality.

`strcmp(char const*, char const*)` is a **weak** order on null-terminated
case-insensitive strings with `stricmp`-induced equality.


# `std::partial_ordering`

## Definition

```cpp
struct partial_ordering {
    static const partial_ordering::less;
    static const partial_ordering::equivalent;
    static const partial_ordering::greater;
    static const partial_ordering::unordered;
    /* comparison operators, conversions, etc */
};
```

## On Partial Orders

- They don't induce equivalence classes - no choice between strong and weak.

That said, partial orders should always be able to treat `==` as an element
disambiguator, because of where they are used (graphs, scheduling, etc).

Making a weak order that is finer (consistent with) a partial order is called
<dfn>linearization</dfn>, or sometimes scheduling.


## Examples

- "ancestor" and "predecessor" in a DAG
- teams in a league that did not compete
- instructions in a basic block in SSA


# C++: `<=>` semantics

## Three-way comparison operator

An operator that lets us _consicely_ encode the _natural order_ on the type.


## Obvious corrolary

`<=>` should always return `std::strong_ordering`, if `<=>` exists.

This follows directly from representing natural orders.


## If you don't follow this advice?

`(x < y) + (x == y) + (x > y)` need not be `1`. You will break poor,
unsuspecting code that looks correct.

Madness.


## Further: `==` is an optimization of `<=>`

Repeat: `<=>` should always return `std::strong_ordering`, if `<=>` exists.

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


# C++: `<=>` behaviour

## Autogenerated `<=>`

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

## Autogeneration properties

If you say `auto`, you get `std::common_comparison_category_t<...>` etc. It's
insane anyway - `<=>` should not return `std::weak_ordering`.

Just specity `std::strong_order` and you will get compile-time errors for insane
code.


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


# C++: `<=>` Usage

## When else should I actually call `<=>` by name?

We saw an example of using `<=>` directly for implementing `<=>`. Where else?
Binary search of all kinds:

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

## Don't actually call `<=>`

Just call the regular comparison operators.

- They will do the right thing
- Less codegen, because only one function.


# Beyond `<=>`

That's really all that's to know about `<=>`.

The rest of the complexity is elsewhere.


# Why `std::weak_ordering`

## When do we use it

Almost all named comparators should return `std::weak_ordering`.

Most named comparators are of the form:

```cpp
auto make_compare(auto projection) {
  return [projection](auto const& x, auto const& y) -> std::weak_ordering {
    return projection(x) <=> projection(y);
  }
}
```

(`<=>` on the codomain of `projection` is probably a strong order)


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

Don't make `operator<=>` return `std::partial_order`. Ever. I will find you, and
I will ... not accept your pull request.


# Natural Orderings

Let me repeat again, so that it sinks in for real:

`<=>` expresses natural strong orderings.

If it ain't natural and strong, it doesn't deserve the name.


# Any-order for your type

Say your type `T` does not have a natural strong ordering, but it *allows* for
one.  What do you do?

You provide a `strong_order(T const& x, T const& y)` (in your namespace)
customization point!

(`std::strong_order` will dispatch to it, or `<=>` if it returns
`std::strong_ordering`)


# Not A Natural

## Let's take `complex<T>`

Complex numbers don't have a natural ordering, but we still want to put them
into maps. A lexicographic ordering is fast, and it would make sense for it to
be shipped with complex. So how do we do that?

Also, let's make complex Regular, because it will be a better example that way.


## Should we define `<=>`?

## Should we define `<=>`?

NO! `<=>` is for natural orderings, and complex numbers don't have one.


## Example (cont.):

Defining for complex:

```cpp
struct my_complex {
  float real;
  float imag;

  bool operator==(complex other) {
    return std::strong_order(real, other.real) == 0 &&
           std::strong_order(imag, other.imag);
  }

  friend std::strong_ordering strong_order(my_complex const& x,
                                           my_complex const& y) {
    if (auto r = std::strong_order(x.real, y.real); r != 0) { return r; }
    return       std::strong_order(x.imag, y.imag);
  }
  friend std::weak_ordering weak_order(my_complex const& x,
                                           my_complex const& y) {
    if (auto r = std::weak_order(x.real, y.real); r != 0) { return r; }
    return       std::weak_order(x.imag, y.imag);
  }
  friend std::partial_ordering partial_order(my_complex const& x,
                                             my_complex const& y) {
    if (auto r = x.real <=> y.real; r != 0) { return r; }
    return       x.imag <=> y.imag;
  }
};
```


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

# Fin

Thanks for listening!

- Do define order
- Be aware of what your _species_ is.

Have fun and optimize!



