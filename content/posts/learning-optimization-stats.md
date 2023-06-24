+++
title = "Learning Optimization, Stats, and ML"
date = "2023-06-23"
+++

## Optimization

I'm interested in learning mathematical optimization (LP, ILP, convex opt), and stochastic problems of a similar flavor (stochastic control?).
Probably for historical reasons, convex optimization is taught in EE, and integer programming is in OR.

[Stanford EE263 Linear Dynamical Systems](https://see.stanford.edu/Course/EE263)
- An applied linear algebra course. Interesting content, good problems.
- "Linear methods can solve lots of things" is a statement that I often hear but without examples.
    This course makes that statement concrete.
    You learn lots of ways to reduce problems to solving Ax=b.
- Problems that apply linear algebraic techniques to control were fun to work out.
    Programming problems were good, too.
- I learned parts of linear algebra that I didn't pick up during my pure math studies.
- Discussions of numerical aspects of linear algebra during lectures were useful.
- Video lectures by Stephen Boyd are on youtube. The lectures are good.
- This course would be good for reviewing linear algebra in the future.
- Progress: done.

[Stanford EE364A Convex Optimization](https://see.stanford.edu/Course/EE364A)
- As with EE263, interesting content, good problems, good lecturer.
- The appeal of the subject is that a mathematically interesting (enough) object is useful for engineering problems.
- Duality theory for convex optimization is slick.
    Would love to find an application of it one day.
- Having the applied linear algebra background knowledge from EE263 was essential.
    Knowing some stats theory would have been useful, too, as some problems came from stats.
- Progress: mostly done. Skipped some problems. Would be good to go through it again.

[Stanford CS261A A Second Course in Algorithms](https://timroughgarden.org/w16/w16.html)
- Came across this course when I was looking for references on LP duality.
    The course goes through some algorithmic problems that 
- The lecture notes are well-written.
- Would like to get through the exercises one day.
- Progress: finished lectures.

[Discrete Optimization (Coursera)](https://www.coursera.org/learn/discrete-optimization)
- The course covers: LP/MIP, Constraint Programming, and other heuristics.
- The programming assignments are non-trivial unlike most coursera courses.
- I hadn't heard of constraint programming. If the lecturer didn't talk about his experience using it, I would have been dubious of its usefulness.
- Progress: one assignment left (out of six).

[MIT 15.053 Optimization Methods In Management Science](https://ocw.mit.edu/courses/15-053-optimization-methods-in-management-science-spring-2013/)
- The math used is elementary.
    Stanford's CS261A covers similar materials in a more interesting (harder) way.
- Useful bits are:
    * Detailed examples of simplex methods using tableau.
    * MIP modelling techniques.
- I can't find a good course for ILP/MIP. The "Optimization over integers" text may be the best bet.

[MIT 6.003 Signals and Systems](https://ocw.mit.edu/courses/6-003-signals-and-systems-fall-2011/)
- The first course in signal processing.
    It seems like techniques developed in EE have had wide applicability, so I thought it'd be good to be familiar with its terminology and tools used.
- Interestingly, this course had a lot of overlap with the complex analysis courses I took.
    I developed a caricaturistic description of complex analysis that it is about proving convergence of objects used in signal processing.
- Progress: watched most lectures. I don't feel particularly motivated to go through the problems.

I can't find a course that talks about stochastic methods, like hidden markov models and MDP.
[Stanford's EE365](https://web.stanford.edu/class/ee365/index.html) has the topics, but the lecture notes are thin, and they don't follow textbooks.

## Stats / ML

I should learn inference and regression.
[Stanford's STATS202](https://web.stanford.edu/class/stats202/intro.html) looks good for intro to ML.

[MIT 6.041 Probabilistic Systems Analysis and Applied Probability](https://ocw.mit.edu/courses/6-041-probabilistic-systems-analysis-and-applied-probability-fall-2010/)
- One thing that I've come to understand after taking several EE courses is that I should learn math from EE if I want to learn applicable math.
    This course's emphasis is on actually computing probability (like, getting a concrete real number at the end), which is different from probability courses I took from math departments.
    I feel a lot more comfortable modelling probabilistic systems after going through this course and seeing the pitfalls.
    Learning some techniques to compute probabilities that aren't trivial otherwise was another good thing about this course.
- Progress: 3/4 done.

## Algorithms

Stanford's CS261A technically fits here.

[Stanford CS265 Randomized Algorithms](https://www.youtube.com/watch?v=9DnWi8IWr24&list=PLkvhuSoxwjI_JL7GYcJHK7-EK55t0KYGO&ab_channel=MaryWootters)
- The topics are really interesting, but I don't know how useful they may be.
- Tim Roughgarden's offering [here](https://timroughgarden.org/f19/f19.html).

Stanford's [CS168 The Modern Algorithmic Toolbox](https://web.stanford.edu/class/cs168/) looks useful.
[CS364A Algorithmic Game Theory](https://timroughgarden.org/f13/f13.html) looks interesting.
