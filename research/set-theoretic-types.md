# Semantic Subtyping and Set-Theoretic Types

## Core model

Semantic subtyping interprets each type as a set of values and defines `A <: B` exactly when the set denoted by `A` is included in the set denoted by `B`. Under that interpretation, bottom denotes the empty set, top denotes the universe of values, and two types are equivalent when they denote the same set. [Frisch, Castagna, and Benzaken, 2008](https://doi.org/10.1145/1391289.1391293) ([author-hosted paper](https://www.irif.fr/~gc/papers/semantic_subtyping.pdf))

The set-theoretic constructors are union (`A | B`), intersection (`A & B`), and complement (`not A`); difference can be expressed as intersection with a complement. These operators apply to the denotations, not merely to surface syntax. The paper studies them together with ordinary type constructors such as base, product, and function types. [Frisch et al., 2008](https://doi.org/10.1145/1391289.1391293)

An arrow type describes functions that, when given an argument in its domain type, return a value in its result type. The familiar rule for plain arrows is contravariance in the domain and covariance in the result; with unions, intersections, and negation, however, subtyping must account for the denoted sets of functions rather than stop at that one structural rule. [Frisch et al., 2008](https://doi.org/10.1145/1391289.1391293)

## Subtyping and decision constraints

The set interpretation gives a useful reduction: `A <: B` holds exactly when `A & not B` denotes the empty set. Thus an exact subtype checker needs a decision procedure for emptiness (or an equivalent inclusion procedure) for its chosen type grammar and value model. [Frisch et al., 2008](https://doi.org/10.1145/1391289.1391293)

The decision result is tied to the paper's defined type constructors and their semantics; it does not establish decidability or a particular cost bound for every language that adds Boolean types. In particular, adding constructors or changing how functions, products, or recursive types are interpreted requires re-establishing how emptiness and inclusion are decided for that extended system. [Frisch et al., 2008](https://doi.org/10.1145/1391289.1391293)

This makes the reasoning boundary semantic, not just syntactic: overlap, emptiness, and inclusion between combined types must agree with the language's value interpretation. A collection of local variance or constructor rules alone is not a substitute for specifying that interpretation and its decision procedure. [Frisch et al., 2008](https://doi.org/10.1145/1391289.1391293)

## Implications for Aura

Aura's README lists Elixir-like set-theoretic types as a goal while describing Aura as exploratory, so these are questions to resolve rather than recommendations to adopt a particular design. [Aura README at the research base commit](https://github.com/mwilsoncoding/aura/blob/193e223dd314ea8725642eb083e231e78924f3f2/README.md)

- Specify the universe of values and the denotation of each type constructor before defining subtype rules; inclusion is relative to those choices. [Frisch et al., 2008](https://doi.org/10.1145/1391289.1391293)
- Decide which Boolean constructors are part of the language and whether their meaning is full set union, intersection, and complement. This determines which overlaps, emptiness facts, and subtype relations need to be handled. [Frisch et al., 2008](https://doi.org/10.1145/1391289.1391293)
- State the intended guarantee for subtype queries, including the supported constructor set and any recursion, and evaluate its cost against that exact definition; results for one formal type language do not transfer automatically to another. [Frisch et al., 2008](https://doi.org/10.1145/1391289.1391293)

## Primary sources

- Alain Frisch, Giuseppe Castagna, and Véronique Benzaken. “Semantic subtyping.” *Journal of the ACM* 55(4), 2008. [DOI](https://doi.org/10.1145/1391289.1391293) · [author-hosted paper](https://www.irif.fr/~gc/papers/semantic_subtyping.pdf)
- Alain Frisch, Giuseppe Castagna, and Véronique Benzaken. “Semantic subtyping.” *LICS 2003*. [DOI](https://doi.org/10.1109/lics.2002.1029823)
- Giuseppe Castagna and Alain Frisch. “A gentle introduction to semantic subtyping.” *PPDP 2005*. [DOI](https://doi.org/10.1145/1069774.1069793)