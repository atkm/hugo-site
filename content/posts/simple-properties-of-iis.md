+++
title = "Simple properties of IIS"
date = "2023-08-21"
mathjax = true
+++

IIS stands for "Irreducible Infeasible Subsystem".
There is no Wikipedia entry for it.
I've been using the definition from a [Gurobi article](https://support.gurobi.com/hc/en-us/articles/15656630439441-How-do-I-use-compute-IIS-to-find-a-subset-of-constraints-that-are-causing-model-infeasibility-).

Let \\( I_0 \\) be a subset of constraints.
\\( I_0 \\) is an *IIS* iff:

1. \\( I_0 \\) is infeasible.
2.  Removing any single constraint ("or bound") from \\( I_0 \\) makes it feasible.

(I've never thought about what the "or bound" part is doing.)


#### Property 1: an IIS does not consist of disjoint infeasible systems

Example: the following constraints do not comprise an IIS.

1. \\( a \leq 0 \\)
2. \\( a \geq 1 \\)
3. \\( b \leq 0 \\)
4. \\( b \geq 1 \\)

because removing one constraint, say \\( a \leq 0 \\), leaves the infeasibility on \\( b \\).

Another non-IIS:

1. \\( a \geq 1 \\)
2. \\( b \geq 1 \\)
3. \\( a + b \leq 0 \\).

Removing \\( a \geq 1 \\) leaves the infeasibility on \\( b \\).



#### Property 2: removing a constraint in an IIS does not make the original system feasible.

\\( I_0 \\) is a collection of rows from \\( A x \leq B \\).

Example:

1. \\( a \geq 3 \\)
2. \\( b \geq 3 \\)
3. \\( a + b \leq 2 \\).
 
\\( \\{ a \geq 3,\ a + b \leq 2 \\} \\) is an IIS.
But removing \\( a \geq 3 \\) from the original system doesn't make it feasible.

#### Property 3: an IIS may not be "minimal".

By "minimal" here, I mean that a set of constraints has the smallest cardinality.
The [Gurobi article](https://support.gurobi.com/hc/en-us/articles/15656630439441-How-do-I-use-compute-IIS-to-find-a-subset-of-constraints-that-are-causing-model-infeasibility-) talks about this property a bit.
Some articles that I've come across use "minimal" to mean irreducible.
That is condonable, because "minimal" is the term that a [widely-cited paper](https://pubsonline.informs.org/doi/abs/10.1287/ijoc.2.1.61) uses.

Example:

1. \\( a \geq 1 \\)
2. \\( b \geq 1 \\)
3. \\( a + b \geq 2 \\).
3. \\( a + b \leq 1.5 \\)

The two subsystems \\( \\{ a + b \geq 2,\ a + b \leq 1.5 \\} \\) and \\( \\{ a + b \geq 2,\ a \geq 1,\ b \geq 1 \\} \\) are, in some sense, infeasible in the same way.
In this example, the former is the minimal IIS.
In practice, you often don't know which constraints in your model are linear combinations of other constraints, so I suppose minimality is a nice-to-have.
For my own work, the IIS that Gurobi returns has been useful.

This [stackexchange thread](https://or.stackexchange.com/questions/559/computational-complexity-to-compute-an-iis) cites a [paper](https://link.springer.com/article/10.1007/s10107-002-0363-5) that says finding the min-cardinality IIS is NP-hard, and is hard to approximate.

How solvers compute an IIS for another article.
