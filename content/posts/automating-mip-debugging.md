+++
title = "Automated Debugging of Mixed-Integer Models (Draft)"
date = "2024-04-14"
mathjax = true
+++

## Motivating Example

Suppose that you have an [MLIP](https://en.wikipedia.org/wiki/Linear_programming#Integer_unknowns):

$$\begin{align}
\text{maximize} \quad & c_1x_1 + c_2x_2 \\\\
\text{subject to} \quad & a_1x_1 \leq L_1  \tag{1L} \\\\
\quad & a_2x_2 \leq L_2  \tag{2L} \\\\
\quad & b_{1,1}x_1 + b_{2,1}x_2 \geq D_1  \tag{1D} \\\\
\quad & b_{1,2}x_1 + b_{2,2}x_2 \geq D_2  \tag{2D} \\\\
\quad & x_1 \geq 0, \quad x_2 \geq 0  \\\\
\quad & x_1, x_2 \in \mathbb{N}
\end{align}$$

Think of inequalities \\( (1L) \\) and \\((2L)\\) as supply limits; actions \\(x_1\\) and \\(x_2\\) consume resources, and \\(L_1\\) and \\(L_2\\) represent limits of the resources they consume.
The inequalities \\((1D)\\) and \\((2D)\\), on the other hand, represent demand for a resource.
Actions \\(x_1\\) and \\(x_2\\) are required to supply at least \\(D_1\\) and \\(D_2\\) units of the resources.
Think of actions as translators of supplied resources to the demanded resource, so this model chooses the optimal consumption of resources to provide the demanded resource.
Because both actions provide both demanded resources, an optimal solution may be a mix of the two actions (one action may be better than the other for providing one resource, and vice versa for the other resource).

Further suppose that this model is solved at a regular cadence, say, for planning of something, and *all coefficients* are read from some data sources, which can vary over time.
For example, the organization in charge of \\(c_1\\), which represents the cost of action \\(x_1\\), may change their estimate.

Now, you solve the model one day, and find that the consumption of \\(R_1\\), the resource that \\(x_1\\) consumes, is too high.
Suppose that the use of \\(x_1\\) was encouraged because, \\(c_2\\), the cost of \\(x_2\\), was misconfigured in the data source.
One way to root cause the issue would be to stare at the model, and to notice that the value of \\(c_2\\) is greater than usual---hopefully you keep the coefficients of the objective terms in a database.

*Another way* to root cause is through re-solving the model with a forcing constraint.
Suppose that, in the undesirable solution, you have \\(a_1x_1 = \hat{R_1}\\).
Then, add the constraint \\(a_1x_1 <= 0.5 \hat{R_1}\\) and re-solve the model.
The added constraint *reduces the consumption of resource \\(R_1\\)* relative to the baseline solution.
Comparing solutions of the baseline model and the modified model reveals the root cause.
The objective value of the new solution is greater than the original solution, which has to be the case, because you added a constraint.
The interesting part is that *the source* of the increase indicates why the consumption of \\(R_1\\) is higher than your expectation in the original solution.
In this case, you would find that the increase in \\(c_2 x_2\\) is greater than the decrease in \\(c_1 x_1\\).
This leads you to look at the \\(c_i's\\) and the \\(x_i's\\), which is a step in the right direction.

Another benefit of this technique is that it eliminates possibilities.
For example, a data configuration that leads to a high usage of \\(R_1\\) is a low supply of \\(R_2\\) (i.e. small value of \\(L_2\\)).
If this was the case, the re-solve would have been infeasible, and the [IIS](https://www.gurobi.com/documentation/current/refman/py_model_computeiis.html) would have contained three inequalities \\((2L)\\), \\((\text{1D or 2D})\\), and the added constraint.
In words, this IIS says that there is not enough resources to meet the demand.
In the example, the fact that adding the constraint did not make the model infeasible precludes the possibility of insufficient supply of \\(R_2\\).

## Supporting soft constraints

This method continues to work well in presence of soft constraints.

Make one of the demand inequalities in the example, say \\((1D)\\), soft by introducing a slack variable \\(s_D\\):
$$
b_{1,1}x_1 + b_{2,1}x_2 \geq D_1 + s_D  \tag{1D'}
$$
with a large objective coefficient \\(\gamma\\).

Suppose that, as before, the consumption of \\(R_1\\) is higher than what is desirable, but, unlike before, it is caused by a lack of \\(R_2\\).
Unlike when \\((1D)\\) is hard, adding the constraint \\(a_1x_1 <= 0.5 \hat{R_1}\\) no longer makes the model infeasible.
It is, however, still possible to attribute the excessive consumption of \\(R_1\\) to the lack of \\(R_2\\).

As before, you study the changes in objective terms.
In this case, the increase in the penalty term \\(\gamma \cdot s_D\\) is the greatest.
So you conclude that, the soft constraint corresponding to the penalty term is violated, and therefore that there is insufficient \\(R_2\\).

## Outline of the method to debug a model by re-solving

This technique can reliably root-cause various kinds of issues with this class of MIP models.
Specifically, the technique is:

1. [Trigger phase] There is a *class* of actions that you want to see either more or less of.
    For example, the class may be identified with consumption of a specific resource.
2. [Re-solve phase] To identify the factor(s) blocking the desired solution, add constraints that *force the desired solution*, and re-solve the model.
    The LHS of the constraints should be a linear combination of variables belonging to the class of interest. 
    (That constraints are constructed from a certain class of variables is not a strict requirement, but it makes programmatic construction of these constraints possible. More on this later.)
    Re-solving the model with forcing constraints teases out why the solution that you want to see is not optimal.
3. [Analysis phase] If the re-solve is infeasible, return the IIS.
    Else, study the changes in objective values.
    Specifically, group objective terms by their categories, and rank categories by their aggregate changes.

The list of objective categories may look like:

- One class of costs associated with the decision variables.
- Another class of costs associated with the decision variables.
- Penalty terms for one class of soft constaints.
- Penalty terms for another class of soft constaints.
- ...

As we will see later, all of these phases can be done programmatically (with little implementation effort), which means that you can attach a post-processing step after a solve that detects and diagnoses issues with the model.

## Implementation: Analysis phase

The following SQL pseudocode makes the feasible case of the analysis phase concrete.

Data setup:
```
Table schema:
- objective term name (string/enum)
- objective coefficient (numeric)
- objective category (string/enum) := the category that the objective term belongs to.
    Every objective term must belong to at least one category.
- var_value_baseline (numeric) := the value of the variable in the baseline solution
- obj_value_baseline (numeric) := the value of the objective term (c*x) in the baseline solution
- var_value_resolve (numeric) := the value of the variable in the re-solve
- obj_value_resolve (numeric) := objective value in the re-solve
```

There is one row for every objective term.

Example: suppose that c*X is an objective term. The value of X in the baseline
solution is 3.0. Then,
- objective term name = 'X'
- objective coefficient = c
- var_value_baseline = 3.0
- obj_value_baseline = 3.0 * c.

The first step in the analysis phase is a simple aggregate query that identifies
the most influential objective category.

```
SELECT
  objective_category,
  SUM(obj_value_resolve - obj_value_baseline) AS per_category_obj_value_diff,
GROUP BY objective_category
// The category with the greatest aggregate increase is what you should direct
// your attention to.
ORDER BY per_category_obj_value_diff DESC
LIMIT 1;
```
After that, identify the most influential terms within the category.
```
SELECT
  *
WHERE objective_category =
  $(the category that you identified in the previous query)
ORDER BY obj_value_resolve - obj_value_baseline DESC
// If you have subcategories within this category, a GROUP BY sum of objective
// diffs would be very useful.
;
```

In the domain that I worked in, returning the most influential objective category and the most influential terms within the category was enough for experienced *owners of the model* to immediately see what the issue was.
In special cases, it was even possible to return actionable messages to *consumers of the solutions*, who did not know the mathematical details of the model.

## Implementation of the Trigger and Re-solve phases (WIP)

This section assumes that the MIP model is implemented as an [MPModelProto](https://github.com/google/or-tools/blob/82750ac12f1ee5354e1c7869894d9af3508778f2/ortools/linear_solver/linear_solver.h#L529) from Google's or-tools, though the implementation is so simple that there really should be no obstacle porting the idea to other solver libraries.

### Annotations on variables

### Trigger conditions based on annotations

### Implementations of REDUCE and INCREASE directives for the Re-solve phase

### INCREASE is tricky.

## Conclusion (WIP)

The analysis phase is the trickiest part. Corner cases, model-dependent.
It's easy to reliably produce analyses that are useful to the model owners, that leads to basically an immediate identification of the root cause.
It's significantly more difficult to .
