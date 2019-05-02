---
title: Points of Order
pdf: points_of_order.pdf
slideNumber: true
controls: true
---


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


# C++20: The Language & Core

# C++20: The Library
