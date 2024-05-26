+++
title = "Automated Debugging of Mixed-Integer Models (Draft)"
date = "2024-04-14"
mathjax = true
+++

## Motivating Example

Suppose that you have an [MLIP](https://en.wikipedia.org/wiki/Linear_programming#Integer_unknowns):

$$\begin{align}
P_0: \text{maximize} \quad & c_1x_1 + c_2x_2 \\\\
\text{subject to} \quad & a_1x_1 \leq L_1  \tag{1L, R_1 limit} \\\\
\quad & a_2x_2 \leq L_2  \tag{2L, R_2 limit} \\\\
\quad & b_{1,1}x_1 + b_{2,1}x_2 \geq D_1  \tag{1D} \\\\
\quad & b_{1,2}x_1 + b_{2,2}x_2 \geq D_2  \tag{2D} \\\\
\quad & x_1 \geq 0, \quad x_2 \geq 0  \\\\
\quad & x_1, x_2 \in \mathbb{N}
\end{align}$$

Think of inequalities \\( (1L) \\) and \\((2L)\\) as supply limits; actions \\(x_1\\) and \\(x_2\\) consume resources, and \\(L_1\\) and \\(L_2\\) represent limits of the resources they consume.
The inequalities \\((1D)\\) and \\((2D)\\), on the other hand, represent demand for a resource.
Assume that the set of demanded resources and the set of consumed resources are disjoint.
Actions \\(x_1\\) and \\(x_2\\) are required to supply at least \\(D_1\\) and \\(D_2\\) units of the resources.
Think of \\(x_1\\) and \\(x_2\\) as translators of supplied resources to the demanded resource---this model chooses the optimal consumption of resources to provide the demanded resource.
Because both actions provide both demanded resources, an optimal solution may be a mix of the two actions (one action may be better than the other for providing one resource, and vice versa for the other resource).

Further suppose that this model is solved at a regular cadence, say, for planning of something, and *all coefficients* are read from some data sources, which can vary over time.
For example, the organization in charge of \\(c_1\\), which represents the cost of action \\(x_1\\), may change their estimate.

Now, you solve the model one day, and find that the consumption of \\(R_1\\), the resource that \\(x_1\\) consumes, is too high.
Suppose that the use of \\(x_1\\) was encouraged because, \\(c_2\\), the cost of \\(x_2\\), was too high due to misconfiguration.
One way to root cause the issue would be to stare at the model, and to notice that the value of \\(c_2\\) is greater than usual---hopefully you keep the coefficients of the objective terms in a database table.

*Another way*, which is what this article is about, is to re-solve the model with a forcing constraint.
Suppose that, in the undesirable solution, you have \\(a_1x_1 = R_1^\ast\\).
Then, add the constraint \\(a_1x_1 <= 0.5 R_1^\ast\\) and re-solve the model.
The added constraint *reduces the consumption of resource \\(R_1\\)* relative to the baseline solution.
The objective value of the new solution is greater than the original solution, which has to be the case, because you added a constraint.
The interesting part is that the driver of the increase (i.e. the objective term(s) with the greatest diffs) *indicates why the consumption of \\(R_1\\) is higher than your expectation* in the original solution.
In this case, you would find that the increase in \\(c_2 x_2\\) is greater than the decrease in \\(c_1 x_1\\).
This leads you to look at the \\(c_i's\\) and the \\(x_i's\\).
From there, you conclude that \\(x_1\\) was chosen because the objective coefficient of the alternative was high.

### This can be automated(!)

Here is the punchline: __everything described above can be automated__, with a fair degree of generality.
The "core" logic---everything from anomaly detection to suggesting the culprit consants---can be done programmatically by adding a small amount of problem context to the MIP model.
Through "model annotations" is how we will add the problem context to the model; more about this in the details section.
With more added problem context, you can also automate the one remaining step, i.e. the deduction from the list of potential culprits to the conclusion that \\(c_2\\) is the root cause.

The "core" logic is more context-agnostic, and it's easy to write an implementation that supports *a class of models*.
For example, the core logic defined for the supply-demand problem in the motivating example easily generalizes to supply-demand problems with the same foundational structure.
That is, it applies to a model with different numbers of variables and constraints, or even a different number of classes of constraints (e.g. additional limits imposed on \\(x_1\\) and \\(x_2\\)), as long as it follows the same fundamental structure that decision variables are translators of consumed resources to demanded resources.

Thus, the core logic is the more interesting part as a technique, by a wide margin. The rest of the logic is a lot more context-dependent, and automating it requires a collection of case-by-case hardcoding.

## Violation of hard and soft constraints in the re-solve

Before going into the details of this resolve-based technique, it's important to discuss the two other classes of the outcome of a re-solve, infeasibility and soft constraint violation.

### Case: infeasibility

Consider once again the model \\(P_0\\) from the previous section.
Suppose that, as before, the consumption of \\(R_1\\) is higher than what is desirable; but, unlike before, it is caused by a lack of \\(R_2\\).

Thinking about what the model does, one can reason that the re-solve with a forcing constraint is infeasible.
The model converts \\(R_1\\) and \\(R_2\\) to the demanded resources.
In the current scenario, there is a lack of \\(R_2\\).
The forcing constraint added to the re-solve limits the consumption of \\(R_1\\).
Then, the re-solve is infeasible, because there is not enough resources to convert to the demanded resources.

When the re-solve is infeasible, we use its [IIS](https://www.gurobi.com/documentation/current/refman/py_model_computeiis.html) to root-cause the issue.
The IIS in this case consists of \\(2L\\), the forcing constraint, and \\(1D\\) (or \\(2D\\)).
This IIS encodes exactly why the model is infeasible.
You then conclude that \\(R_1\\) usage is high (w.r.t. your expectation) because \\(R_2\\) alone is not enough to meet the demand (encoded by \\(1D\\)).
Generally speaking, root-causing is often easier when the model is infeasible; more about this later.

### Case: soft constraint violation

For the sake of illustration, make one of the demand inequalities in the example, say \\((1D)\\), soft by introducing a slack variable \\(s_D\\):
$$
b_{1,1}x_1 + b_{2,1}x_2 \geq D_1 + s_D  \tag{1D'}
$$
with a large objective coefficient \\(\gamma\\).

Now, consider the same scenario as in the infeasibility case, where \\(R_1\\) consumption is high because of a lack of \\(R_2\\).
Because \\(1D\\) is now soft, the re-solve is feasible, so you study the changes in objective terms.
In this case, the increase in the penalty term \\(\gamma \cdot s_D\\) is the greatest.
So you conclude that the soft constraint corresponding to the penalty term is violated, and therefore that there is insufficient \\(R_2\\).

## Outline of the method to debug a model by re-solving

This programmatic technique can reliably root-cause various kinds of issues with a MIP model.
Specifically, here is a rough outline of the technique:

1. [Trigger phase] There is a *class* of actions that you want to see either more or less of.
    For example, consider a modified \\(P_0\\) with more than one variable that consume \\(R_1\\). Then, you can form a class of actions that consumes \\(R_1\\).
2. [Re-solve phase] To identify the factor(s) blocking the desired solution, add constraints that *force the desired solution*, and re-solve the model.
    The LHS of the forcing constraints must be a linear combination of variables belonging to the class of interest. 
    That constraints are constructed from a certain class of variables is not a strict requirement, but it makes programmatic construction of these constraints possible, and the metadata introduced are re-used in the analysis phase.
    More on this later.
3. [Analysis phase] If the re-solve is infeasible, analyze the IIS.
    Else, study the changes in objective values.
    Specifically, group objective terms by their categories, and rank categories by their aggregate changes.
    This will teases out why the solution that you want to see is not optimal.

All of these phases can be done programmatically (with a little implementation effort), which means that you can attach a post-processing step after a solve that detects and diagnoses issues with the model.

This article describes the re-solve phase first, as it is the core idea of this technique, and, narrative-wise, it is a natural way to introduce model annotations.

## Implementation of the Re-solve Phase

Data setup: let's say we keep a database table `mip_solution` that holds the values of objective terms in the baseline and re-solve solutions.
The columns for re-solve are empty until the re-solve phase is completed.

```
mip_solution table schema:
- variable name (string/enum)
- objective coefficient (numeric)
- objective category (string/enum) := the category that the objective term belongs to.
    Every objective term must belong to at least one category.
- var_value_baseline (numeric) := the value of the variable in the baseline solution
- obj_value_baseline (numeric) := the value of the objective term (c*x) in the baseline solution
- var_value_resolve (numeric) := the value of the variable in the re-solve
- obj_value_resolve (numeric) := objective value in the re-solve
```


### Annotations on variables

The first way to think of *annotations on variables* is as a map from model variables to their attributes.
If your model is an [MPModelProto](https://github.com/google/or-tools/blob/82750ac12f1ee5354e1c7869894d9af3508778f2/ortools/linear_solver/linear_solver.h#L529) this map can be added to it as a [proto extension](https://protobuf.dev/programming-guides/extension_declarations/).


```
// Annotations comprise a map<int64, map<string, string>> field in the model
// proto. The int64 key of the map is the index of the variable in MPModelProto.
// Below, the variable name is inlined for the purpose of illustration.
// You can definitely get fancy with the type of the inner map, as string keys
// and values don't encourage clean code.
{
    x0: {
        // You can have numeric annotations.
        "R1": "10.0",
        "R2": "0.0",  // This means that x0 doesn't consume R_2, so it's fine to
                      // omit this annotations in an actual implementation.
        // Annotations can also mark categorical attributes of a variable.
        "location": "somewhere",
        "color": "red",
    },
    x1: {
        // ... annotations for x1.
    }
}
```

### Creating the forcing constraint

Suppose that, like in the motivating example, you want to reduce the consumption of \\(R_1\\).
The LHS of the forcing constraint consists of the variables that consume \\(R_1\\), and the RHS of it is the amount of consumption of \\(R_1\\) in the baseline solution.
The following query collects the variables that can consume \\(R_1\\) by looking for the "R1" annotation.
The query also returns the values of the annotations, which become the coefficients of the linear combination.

```
SELECT
  // In the application layer, form the linear combination SUM(anno.value * variable_name)
  variable_name,
  anno.value
FROM mip_solution
  INNER JOIN annotations AS anno  // A join on variable index
WHERE
  anno.key = "R1"
  // You could pass in other filters based on annotations, like annotation("color" )="red".
;
```

(The linear combination you get is the same as the LHS of \\((1L)\\), because the constraint is also about consumption of \\(R_1\\).)

Now, to compute the RHS of the forcing constraint, we first need to know how much \\(R_1\\) was computed in the baseline solution.
That is obtained by plugging in the baseline solution into the linear combination that we just formed:

```
SELECT
  // The amount of consumption of R_1 in the baseline solution.
  SUM(var_value_baseline * anno.value)
FROM mip_solution
  INNER JOIN annotations AS anno  // A join on variable index
WHERE
  anno.key = "R1"
  // This query must have the same filters that you used in the query for
  // forming the linear combination.
;
```


### REDUCE and INCREASE directives for the Re-solve phase (WIP)

Determination of the reduction factor is . 
The reduction factor can depend on the 

### INCREASE is tricky (WIP)

### Pointwise vs. uniform (WIP)

## Implementation of the Analysis Phase

The following SQL pseudocode makes the feasible case of the analysis phase concrete.


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
FROM mip_solution
GROUP BY objective_category
// The category with the greatest aggregate increase is what you should direct
// your attention to.
ORDER BY per_category_obj_value_diff DESC
LIMIT 1;
```

To give you a sense of what this query returns, the list of objective categories may look like:

- One class of costs associated with the decision variables.
- Another class of costs associated with the decision variables.
- Penalty terms for one class of soft constaints.
- Penalty terms for another class of soft constaints.
- ...


After that, identify the most influential terms within the category.
```
SELECT
  *
FROM mip_solution
WHERE objective_category =
  $(the category that you identified in the previous query)
ORDER BY obj_value_resolve - obj_value_baseline DESC
// If you have subcategories within this category, a GROUP BY sum of objective
// diffs would be very useful.
;
```


### Interpretation of the analysis query result

In the domain that I worked in, returning the most influential objective category and the most influential terms within the category was enough for experienced *owners of the model* to immediately see what the issue was.
The IIS returned in the infeasibility case was similarly telling.

In most cases, it was possible to return actionable messages that are comprehensible even by the *consumers of the solutions*, who did not know the mathematical details of the model.
Creating a user-friendly message out of the query result or the IIS obviously requires knowledge of the model domain.

## Implementation of the Trigger Phase (WIP)

The trigger phase determines whether there is a problem with the solution.
Since  is entirely domain-specific, 

In the domain that I worked in, if the  \\( \frac{R_i}{\sum_k R_k} \\).
Re-use the idea from `INCREASE`.

## Putting it all together: an end-to-end explanation system

You can think of the three phases to comprise an interface `MIP_EXPLAIN` with the following inputs and outputs:

`MIP_EXPLAIN` *input*:

- A `REDUCE` or `INCREASE` directive.
    * `REDUCE` takes a variable annotation and a reduction factor as arguments.
    * `INCREASE` takes two variable annotations and an increase factor.
        One annotation is for forming the linear combination, and the other is for the unit of measurement.

`MIP_EXPLAIN` *output*:

- An IIS, or
- the objective category that is responsible for the objective increase. If available, the subcategory responsible for the increase of the objective category. (Recurse if there are more levels of nested category.)

Furthermore, one can wrap this interface inside a more domain-specific system that turns the debugging process truly end-to-end:

- A domain-specific logic that detects anomaly in the baseline solution, and constructs requests to `MIP_EXPLAIN`.
The section on the Trigger Phase describes one such logic that utilizes an idea used in the definition of `INCREASE`.
- A domain-specific logic that parses the `MIP_EXPLAIN` output into a human-digestible text.
    The section for the analysis phase describes how to do this.
    Or maybe you can even trigger some process to fix the root cause... Who knows.

With this, you get a post-processing step in a MIP solver pipeline that *detects an anomaly, and returns the reason for the anomaly in a human-digestible text*.
In a real system, you would have the analysis phase export evidences that lead to its conclusion (e.g. the full objective breakdown, not just the dominant term), so that the conclusion can be manually validated if necessary.
Obviously, it's possible to use the end-to-end explanation system manually as well.

### Implementation framework

SQL + application logic (in whatever language) is the right way to implement `MIP_EXPLAIN`, as the core logic of all phases boils down to collecting variables with certain annotations, and doing aggregation over them.
If latency isn't a concern, you can skip having to maintain a database table for `MIP_EXPLAIN` (which we called `mip_solution`) by reading the model and its baseline solution with a language that supports dataframe (python pandas, julia, ...) and writing sql against it.

Writing the debugging system in the same language as the rest of the model solver pipeline is important, as you will very likely re-use components from model generation and the baseline solve.

Oh, and the MIP solver needs to support IIS ([ref]({{<ref "how-to-compute-iis">}})).
Gurobi supports it.
SCIP doesn't seem to.

The domain-specific logic that wraps `MIP_EXPLAIN` is tedius to implement, as it needs to deal with various cases one-by-one.
SQL + application logic is also the right way to implement the wrapper, too.

### Benefits, and when this is worth the implementation effort (WIP)

It comes with all the organizational benefits that you would expect from automating a previously manual process:

- A rapid 

Implementing this technique is not that much work as a software project, but this investment is worth it only if the model is solved regularly, and the fundamental model structure remains the same.

At the organization where I developed this, the model was generated programmatically, so model annotations were generated with the model.
The only time when I had to make changes to the automated debugging process was when the team introduced a new class of decision variables.


## Are annotaitons really necessary? (WIP)

In theory, the information that the annotations embody is a subset of the information in the model.
For example, at one point in this exposition, I pointed out that the annotations about \\(R_1\\) embody the same information as \\((1L)\\), a constaint on the consumption of \\(R_1\\).
The point of annotations is to organize the model data that are most relevant for analysis in a format that is suitable for writing queries.

There are good reasons to create a structure to store attributes of the model as annotations, effectively duplicating information.
Manipulation of the model for performance reasons, such as rescaling of constants.

## Conclusion (WIP)

The analysis phase is the trickiest part. Corner cases, model-dependent.
It's easy to reliably produce analyses that are useful to the model owners, that leads to basically an immediate identification of the root cause.
It's significantly more difficult to .

### Is this method transferable to other models?

This section is the main reason why I wanted to write up this post.

I don't have experience working with other MIP models, so I'm curious how transferrable this method is.
I developed this method for 

Another thought: if the model was continuous, then there must surely be scenarios where you can avoid a re-solve by taking advantage of duality theory.

# Backlog

In addition to hinting at  this technique is that it eliminates possibilities.
For example, a data configuration that leads to a high usage of \\(R_1\\) is a low supply of \\(R_2\\) (i.e. small value of \\(L_2\\)).
If this was the case, the re-solve would have been infeasible, and the [IIS](https://www.gurobi.com/documentation/current/refman/py_model_computeiis.html) would have contained three inequalities \\((2L)\\), \\((\text{1D or 2D})\\), and the added constraint.
In words, this IIS says that there is not enough resources to meet the demand.
In the example, the fact that adding the constraint did not make the model infeasible precludes the possibility of insufficient supply of \\(R_2\\).

