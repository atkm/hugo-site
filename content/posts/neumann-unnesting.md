+++
title = "Notes for \"Unnesting Arbitrary Queries\" (BTW, 2015)"
date = "2024-02-04"
draft = true
mathjax = true
+++

Paper: T. Neumann and A. Kemper, _Unnesting Arbitrary Queries_, in _BTW_, 2015

A _correlated_ query is a subquery that refers to a term in the outer table.
This paper shows that correlated queries can *always* be decorrelated.
The impression that I got from this paper and related reading is that *all* query optimizers before this paper relied on hard-coded patterns to unnest subqueries.
It may not be fair to describe their patterns as "hard-coded", as these patterns may support some level of generality.
The point, however, is that one pattern may apply to some types of nesting but not to others.
The technique described in this paper, on the other hand, can decorrelate an arbitrary correlated query.

## Paper notes

### Section 3.2. General Unnesting

* "
_... as we will see, we can always reach this state.
An even nicer goal would be to reach a state where the resulting regular join can be substituted by existing attributes, eliminating the join altogether.
We will discuss that in Section 4._
"

A summary of the crux of this paper.
"[T]his state" refers to a state where a dependent join can be re-written as a non-dependent join.

* "
_Using these transformations, each dependent join is either eliminated at some point by substitution, or ends up in front of a base relation, in which case it can be transformed into a non-dependent join._
"

A base relation has no free variable, so it cannot be dependent on another relation.
So, by using the preceding rules to push a dependent join down the tree, the join *must* eventually be over two base relations.

### Section 7. Conclusion

* "
_... our analysis of commercial and open source database systems revealed that their optimizations only cover special patters that (arguably) are the most important use cases for query nesting._
"

## Uses

This unnesting technique is implemented in Hyper and Umbra.

DuckDB also [implements](https://duckdb.org/2023/05/26/correlated-subqueries-in-sql.html) this technique (posted 05/26/2023).

## Thoughts

This is very cool!
Reading something like this makes me want to learn query optimizer implementation.
Also, this paper provided an incentive for me to finally learn the definitions of relational algebra.

It would be a good exercise in relational algebra to write a formal proof that this method results in de-correlating nested queries.
It would also be good to take a look at DuckDB's implementation to get a sense of what query optimization implementations look like.
