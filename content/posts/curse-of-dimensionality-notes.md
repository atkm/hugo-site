+++
title = "Notes: Curse of Dimensionality"
date = "2018-10-20"
mathjax = true
+++

"Curse of Dimensionality", in the sense of Bellman as well as in machine learning.

## Geometry
In \\( \mathbb{R}^n \\) where \\( n \\) is large,

- The volume of the unit ball goes to 0 as the dimension goes to infinity.
- Most of the volume of a high-dimensional ball is concentrated around its shell.
- Pick two vectors on the surface of the unit ball independently.
The two are orthogonal with high probability.
- Johnson-Lindenstrauss lemma: a set of points can be embedded (almost isometrically w.r.t. \\( L_2 \\)) into a space of a much lower dimension.
The \\( L_1 \\) analogue doesn't hold.
Look up fast JL transform.
- Distances between points are concentrated: \\( \frac{ \max\_{x, y} d(x,y) - \min\_{x, y} d(x,y) } { \min\_{x, y} d(x,y) } \to 0 \\) where \\( d \\) is the \\( L_p \\) norm where \\( p \geq 1 \\).


## ML
Suppose the data has \\( p \\) features,

- Suppose that the features are binary categorical.
The cardinality of the feature space grows exponentially with \\( p \\).
Suppose the size of the training set is fixed; then, the model is highly likely to overfit when \\( p \\) is large.
- Suppose that the features are continuous and in \\( [0,1] \\).
Then, most points are located outside of the sphere inscribed in the cube.


## Reference

- [Princeton COS 521](https://www.cs.princeton.edu/courses/archive/fall13/cos521/lecnotes/lec11.pdf)
- [Why is Euclidean distance not a good metric in high dimensions?](https://stats.stackexchange.com/questions/99171)
- [The Curse of Dimensionality in classification](http://www.visiondummy.com/2014/04/curse-dimensionality-affect-classification/)
- Domingos, "A Few Useful Things to Know about Machine Learning" ([link](https://homes.cs.washington.edu/~pedrod/papers/cacm12.pdf))
