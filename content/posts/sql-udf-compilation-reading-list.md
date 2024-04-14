+++
title = "Reading list: UDF compilation"
date = "2024-01-21"
+++

Results on UDF compilation discussed in the "Server-side Logic Execution" [lecture](https://youtu.be/DRFuyLd29eA?si=2sKYDwJCRTc83eJm) of CMU 15-721 were interesting.

- The first result in this area was: K. Ramachandra, et al., _Froid: Optimization of Imperative Programs in a Relational Database_ (VLDB, 2017) https://arxiv.org/abs/1712.00498
    * A subset of UDF can be compiled to SQL.
        Then standard query optimization routines apply.
    * _Decorrelation of user defined function invocations in queries_ (2014, [link](https://ieeexplore.ieee.org/document/6816679)) is what the Froid paper is based on.
- C. Duta, et al., _Compiling PL/SQL Away_ (CIDR, 2020, https://arxiv.org/abs/1909.03291)
    
    * Instead of compiling UDFs to SQL, target "common table expressions".
    * Supports more UDF constructs (loops!).
    * Also, the code is set up as a middleware between the application and the database.
        UDFs are converted to SQL before they reach the database.
    * "More PL-ey".
- Umbra's optimizer is better at "de-correlating" than Froid. (source: lecture & links later in this post)
    * _Orthogonal optimization of subqueries and aggregation_ (SIGMOD, 2001, [link](https://dl.acm.org/doi/10.1145/375663.375748)) is what Froid uses to decorrelate subqueries.
    * _Unnesting Arbitrary Queries_ (BTW 2015, [link](https://btw-2015.informatik.uni-hamburg.de/res/proceedings/Hauptband/Wiss/Neumann-Unnesting_Arbitrary_Querie.pdf)) is what Umbra uses.
        The lecturer says "they can systematically decorrelate any subquery".
        What does "any subquery" here mean?

Looking up the lecturer, I found this paper on batching:
_Dear User-Defined Functions, Inlining isn't working out so great for us. Let's try batching to make our relationship work. Sincerely, SQL_ (CIDR 2024, [slides](https://samarch.xyz/slides/cidr2024.pdf))
- The gist of it seems to be that if the database system supports Neumann-style (the BTW 2015 paper above), then inlining is preferred.
    But there are cases where batching is better (e.g. certain query shapes against MS SQL).
