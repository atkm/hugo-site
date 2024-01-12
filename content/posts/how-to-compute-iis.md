+++
title = "How to compute IIS"
date = "2023-08-22"
mathjax = true
+++

[Gurobi says](https://support.gurobi.com/hc/en-us/articles/360041448572-How-does-Gurobi-compute-the-IIS-for-infeasible-models-) their algorithm is based on [the paper](https://pubsonline.informs.org/doi/10.1287/ijoc.2.1.61) *"Identifying Minimally Infeasible Subsystems of Inequalities"* from 1990.

This [thread from Gurobi](https://support.gurobi.com/hc/en-us/community/posts/4470121095057-How-Gurobi-compute-IIS-for-MIP-problem-) has some discussion on how Gurobi does it.

- [An article](http://www.sce.carleton.ca/faculty/chinneck/docs/GuieuChinneck.pdf) by Chinneck.
- [These slides](https://www.sce.carleton.ca/faculty/chinneck/docs/CPAIOR07InfeasibilityTutorial.pdf) from Chinneck.

## "Identifying Minimally Infeasible Subsystems of Inequalities"

By John Gleeson, Jennifer Ryan. 1990.

[Link](https://www.math.ucdavis.edu/~deloera/MISC/LA-BIBLIO/trunk/RyanJennifer2.pdf)

Notes:

- \\( y_i := 1 \text{ iff the i^th constraint is deleted.} \\)
- \\( S_i := \text{ the set of constraint indices in the j^th IIS. } \\)
- The MIP given in the beginning of page 2 says:
    * Minimize the number of constraints dropped, while dropping at least one constraint from each IIS.
    * Since *all* IIS are enumerated, a subsystem that this program produces (drop constraints with \\( y_i = 1 \\)) is feasible.
    * So this program produces "the minimum number of inequalities that must be dropped in order to achieve a consistent system".
- Obviously, enumerating all IIS is far from trivial.
    "The number of minimally infeasible subsystems is exponential in the number of constraints".
- The "pivoting method" that the paper proposes is based on [a paper](https://www.sciencedirect.com/science/article/abs/pii/0377221781901776) by Van Loon.
- The known vertex enumeration algorithm at the time was O(num\_constraints * num\_variables^2 * num\_extreme\_points).
    Looks like the situation hasn't changed since 1996.
    Wikipedia lists the [Avis–Fukuda](https://en.wikipedia.org/wiki/Vertex_enumeration_problem) algorithm with the same complexity.

## SCIP

It's possible to compute IIS with SCIP. Never tried it.

- [thread 2](http://listserv.zib.de/pipermail/scip/2015-December/002613.html)
    * MinIISC is a stand-alone application separate from SCIP.
- [thread 1](http://listserv.zib.de/pipermail/scip/2012-October/001095.html)

[MinIISC](https://scipopt.org/doc/html/MINIISC_MAIN.php) computes a minimal (in cardinality) IIS.
The doc cites [*"Finding the minimum weight IIS cover of an infeasible system of linear inequalities"*](https://link.springer.com/article/10.1007/BF02284626).

- The program described in the "Solution Approach" section is the same as the program in the page 2 of "Identifying Minimally Infeasible Subsystems of Inequalities".
- The main problem that the paper deals with is the enumeration of IIS.
- The "Implementation" section cites *"Branch-and-Cut for the Maximum Feasible Subsystem Problem"*.

## Summary

There are two papers that IIS computation is based on.

1. *"Identifying Minimally Infeasible Subsystems of Inequalities"* by John Gleeson and Jennifer Ryan. 1990.
2. *"Finding the Minimum Weight IIS Cover of an Infeasible System of Linear Inequalities"* by Mark Parker and Jennifer Ryan. 1996.

## TODO

Understand the two papers (i) the vertex-IIS correspondence theorem in the 1990 paper, and (ii) the algorithm in the 1996 paper.
