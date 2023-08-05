+++
title = "Notes: Data Diffs"
date = "2023-08-05"
+++

I came across [Data diffs: Algorithms for explaining what changed in a dataset](https://blog.marcua.net/2022/02/20/data-diffs-algorithms-for-explaining-what-changed-in-a-dataset.html).

- The two papers discussed are:
    1. [Scorpion](http://sirrice.github.io/files/papers/scorpion-vldb13.pdf) by [Eugene Wu](http://www.cs.columbia.edu/~ewu/) and [Sam Madden](http://db.csail.mit.edu/madden/).
        Finds common properties of outlier points.
    2. [DIFF](http://www.bailis.org/papers/diff-vldb2019.pdf).
        SQL implementation of Scorpion and similar.
        The computated can be distributed.
- An author of the DIFF paper, Peter Bailis, founded sisudata.com.

The [comments](https://news.ycombinator.com/item?id=36888667) on HN list some related work:

- [Dolt](https://www.dolthub.com/) is a company whose product is a version-controlled SQL database.
- A [Spark implementation](https://github.com/G-Research/spark-extension/blob/master/DIFF.md) of DIFF by G-Research.
- [TerminusDB](https://terminusdb.com/). Another version-controlled db company.

Searching for "data diffs" on HN finds related work:

- [1](https://news.ycombinator.com/item?id=36889656): [How to check two SQL tables are the same](https://github.com/remysucre/blog/blob/main/posts/sql-eq.md).
    At a glance, no interesting things on the comment thread.
- [2](https://news.ycombinator.com/item?id=34002631): [data-diff](https://github.com/datafold/data-diff/).
    Compare datasets across different databases (like Postgres & Snowflake).
