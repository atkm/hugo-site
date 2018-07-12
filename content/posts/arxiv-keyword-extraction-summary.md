+++
title = "Keyword Extraction from arXiv - Summary"
date = "2018-07-11"
+++

In [the previous articles](/posts/arxiv-keyword-extraction-part1) on arXiv keyword extraction, I focused on details of setting up an infrastructure to serve the algorithm.
Since then, I wrote a more complete keyword extraction algorithm, and deployed it to Google Cloud Platform.
This article a more high-level overview of the end product.

## Motivation
I built this app to help myself keep learning mathematics.
When I was in graduate school, I learned the terminology of my area of study through attending seminars --- whenever the speaker used a term that I wasn't familiar with, I would write it down and look it up later.
A keyword is a good starting point to learn about a research area, as googling for it usually leads to lecture notes, Stack Exchange posts, or a Wikipedia article.

{{< figure src="/images/keyword-extraction-screenshot-at.png" title="screenshot" 
    width="100%">}}

## Data
I used [metha](https://github.com/miku/metha) to obtain all metadata of math articles.
The fields of an article metadata are:

- article ID
- title
- authors
- abstract
- categories
- submitted.

In PostgreSQL, metadata are modeled by many-to-many relationships between "articles", "authors", and "categories" tables.
The tables are managed with SQLAlchemy.

## Method
The keyword extraction algorithm acts on the set of all articles in a category selected by the user.
The algorithm consists of three steps:

1. Represent each metadata with the concatenation of its title and abstract. Create tf-idf vectors of 1-grams from the representation.
2. Run a k-means clustering of the metadata using their tf-idf vector representations. Clusters represent sub-categories.
3. Within each cluster, rank 2- and 3-grams with tf-idf. Return the top 5 keywords from each cluster.

I inspected resulting keywords from the math.DS category, which was my field of study, to judge the quality of the algorithm.

Ideas that went into this algorithm:

- I chose to run k-means on tf-idf vectors instead of bag-of-words vectors to increase the weight of significant terms when generating clusters.
- If the tf-idf ranking is run without clustering, general terms that describe the category dominate the ranking. For math.DS, for example, such terms would be "dynamical systems", "differential equations", "mechanics", "flows", and so on.
    Such terms are too general to serve as a pointer to a research area.
    The tf-idf ranking results in more interesting terms when run on clusters, because it looks for significant terms in sub-categories.
    Furthermore, increasing the number of clusters results in identifying a finer division of articles into more specialized disciplines.
    I call this level of specialization as "specificity".
    The algorithm exposes the control of specificity, and the user may choose its value.
    The algorithm returns keywords from more specialized disciplines when specificity is higher.
- Words that appear frequently in academic papers (e.g. "we prove", "in this paper") are removed.
- Characters used for LaTeX formatting (e.g. backslash, dollar, umlauts) are removed.

## Deployment
The algorithm is deployed to a GCP Compute Engine instance as a Flask app.
PostgreSQL and Redis run in the same instance.

## Ideas for Further Work
- Convert plural nouns to their singular form to prevent both the singular and plural forms from appearing in the ranking.
- Parallelize the tf-idf ranking computation by cluster.
- Support other disciplines on arXiv, such as physics, computer science, and statistics.
- Implement scheduled updates of articles on the Compute Engine instance.
- Implement a "new keywords" feature by comparing keywords extracted from newer articles with those from older articles.
- This app is a derivative of a far more ambitious idea, which is to generate summaries of prominent research interests.
    A summary may contain a description of the research interest, pointers to influential arXiv articles, and pointers to relevant expository articles.
    The keyword extraction algorithm presented in this article is a tiny step in that direction.
    I don't see the idea coming to fruition anytime soon, but it'd be great to see a product like that someday!
