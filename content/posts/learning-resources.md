+++
title = "Learning Resources"
date = "2018-04-04"
+++
Freely available online resources that I've found useful for learning Statistics, Machine Learning, Distributed Computing, Database Systems, and other CS and SWE topics.

## Statistics and Machine Learning
### ML
Expositions on machine learning that are freely accessible (like the ones on Coursera) tend to sweep theoretical foundation and mathematical rigor under the carpet; these are resources that don't skimp on the hard stuff.

- Stanford CS229 Machine Learning
  * One-line summary: the course teaches you how to set up a cost function based on a model and data, and figure out how to optimize it.
  * Topics: matrix calculus review, logistic regression, gradient descent and its variants, discriminative and generative algorithms, linear programming and SVM, theory of learning (e.g. VC dimension, uniform convergence), K-means clustering and the EM algorithm, PCA, reinforcement learning, ... among many others.
  * [Stanford Engineering Everywhere](https://see.stanford.edu/course/cs229) publishes course materials (including video lectures) from 2008.
  [Resources](http://cs229.stanford.edu/) from the most recent iteration of the course (except video lectures) is also accesible.
  Newer iterations of the course spend more time on deep learning towards the end of the course.
  * Fun homework problems!

- `mathematicalmonk`'s machine learning [playlist](https://www.youtube.com/playlist?list=PLD0F06AA0D2E8FFBA)
  * Authored by Jeff Miller
  * The lectures provide good balance of intuition and mathematical detail.
  * His exposition on MCMC is the best I could find online.

- There is an online textbook with a gitbook (blue) interface that seems to be written by someone with a pure math background. I can't find it nor remember its title.

### Stat
- [Cosma Shalizi](http://www.stat.cmu.edu/~cshalizi/) (CMU)
  * His lecture notes are fantastic.
  They provide an in-depth discussion of assumptions behind statistical models, as well as statistical fallacies and pitfalls.
  * [36-401 Modern Regression](http://www.stat.cmu.edu/~cshalizi/mreg/15/)
      + A deep dive into the linear model. --- What is a statistical model? How to check assumptions? What confidence sets are and what they aren't. Is R-squared distraction or nuisance? "Does anyone know when the correlation coefficient is useful, as opposed to, used?" Multicollinearity is hard. --- I really enjoyed reading the notes.
  * [36-402 Undergraduate Advanced Data Analysis](http://www.stat.cmu.edu/~cshalizi/uADA/17/)
      + Builds on 36-401 to cover more modern techniques.
  * [36-350 Statistical Computing](http://www.stat.cmu.edu/~cshalizi/statcomp/14/)
      + Introduction to R, functional-style programming, and numerics.
      + Topics: R data types, split-apply-combine, testing, random number generation, distributions, optimization, bootstrap, MCMC, regexp.
      + Sections on optimization, simulation, and MCMC are good reading for numerical computing in general, not specific to R.

- Wasserman, "All of Statistics"
  * Theoretical statistics at the advanced-undergraduate level. Comparable to Casella & Berger, "Statistical Inference"?
  * This is a good text for self-study, as it doesn't pack an overwhelming amount of material.
  Every chapter, including end-of-chapter exercises, can be done in a day.
  * Later chapters feel like copy-pastes from a lecture note.

## Distributed Things
- [Big Data Analysis with Scala and Spark](https://www.coursera.org/learn/scala-spark-big-data) (Coursera)
  * Part of the Functional Programming in Scala specialization

- [U of W CSEP552 Distributed Systems](https://courses.cs.washington.edu/courses/csep552/18wi/)
  * Lecture videos.
  * also see: CMU 15-440 and MIT 6.824. Topics and assignments are similar.

- [data Artisans Flink Training](http://training.data-artisans.com/)
  * Provides exercises that cover most APIs (Streaming, Dataset, Connectors, Fault Tolerance, Table, SQL, CEP). Java or Scala. Exercises come with solutions.
  * Should read this article as a pre-requisite: [The world beyond batch: Streaming 101/102](https://www.oreilly.com/people/09f01-tyler-akidau).

- [Intro to Hadoop and MapReduce by Cloudera](https://www.udacity.com/course/intro-to-hadoop-and-mapreduce--ud617) (Udacity)
  * a small number of exercises to get started with HDFS and MapReduce
  * Can be done in a few days
  * VM provided

- [Real-Time Analytics with Apache Storm](https://www.udacity.com/course/real-time-analytics-with-apache-storm--ud381) (Udacity)
  * Can be done in a few days
  * Vagrant file and Flask+D3 app provided

## Database Systems
- [CMU 15-445 Intro to Database Systems](http://15445.courses.cs.cmu.edu/)
  * Great lecturer
  * Discusses details of database systems unlike Stanford's CS145.
  * This course is a pre-req to [CMU 15-721 Advanced Database Systems](http://15721.courses.cs.cmu.edu/), which covers in-memory systems. The first lecture of the advanced course (History of Database Systems) is a fun watch.

- [MongoDB University](https://university.mongodb.com/courses/catalog)
  * Each course provides lectures and hands-on exercises with tools in the MongoDB ecosystem.
  * M001: MongoDB Basics
     + hands-on with MongoDB Compass
  * M101J: MongoDB for Java Developers
  * M103: Basic Cluster Administration
  * M201: MongoDB Performance

## CS and SWE (topics not covered above)
- MIT 6.005 Software Construction (OCW)
  * Software engineering best practices. What constitutes a good specification and test? Immutability. Interfaces. Equality. Specification and tests for abstract data types and concurrent programs.

- [Practical Concurrent and Parallel Programming](http://www.itu.dk/people/sestoft/itu/PCPP/E2016/)
  * Covers most of "Java Concurrency in Practice" by Goetz with a Java 8 twist, and a bit of "The Art of Multiprocessor Programming".
  * Each week comes with a simple coding assignment, which takes about an hour.

- Functional Programming in Scala Specialization (Coursera)
  * Assignments are fun (i.e. moderately challenging) unlike most Coursera courses.
  * Course 1 & 2 comprise an introduction to functional programming and the Scala type system.
  Most of lecture time is spent on discussing various functional programming constructs, but the instructor (Odersky) dips toes into theoretical details sometimes, which I liked.
  * Course 3 covers data parallelism, parallel scan (aka prefix sum), and combiners.
  The discussion on combiners using conc-tree as an example is interesting. Programming assignments on Week 2 (reductions and prefix sums) and Week 4 (Barnes-Hut simulation) are great!

- [Sophomoric Parallelism and Concurrency](https://homes.cs.washington.edu/~djg/teachingMaterials/spac/) (Dan Grossman, U of W)
  * A short introductory course on parallel data structures and concurrency using Java's fork-join framework.
  * Homework problems are food for thought.

- Algorithms Part 1 & 2 by Princeton (Coursera)
  * A standard 2nd-year course on algorithms. The instructor sprinkles historical overview, "war stories", and applications of algorithms in each of his lectures, which are entertaining.
  * Good set of assignments to work through. They don't merely ask you to write a data structure introduced in lecture; each assignment requires some problem solving, such as figuring out what data structures should be used to achieve given performance requirements.

- Parallel, Concurrent, and Distributed Programming in Java Specialization by Rice (Coursera)
  * A massively watered-down version of [COMP322](https://wiki.rice.edu/confluence/display/PARPROG/COMP322).
  * The course focuses on concurrent programming, and it does a good job of giving an overview of various concurrency models and constructs.
  Topics covered are: fork-join, race conditions, barriers and phasers, chucking in parallel computing, structured and unstructured locks, liveness, critical section, object-based isolation, monitor pattern, actor model, linearizability, pipeline parallelism,  pub-sub, message passing model, client-server.
  * Programming assignments are "fill-in-the-blank" style, and not edifying --- except the assignments on parallel Boruvka MST and Sieve of Eratosthenes, which are fun.

- Cloud Computing Applications Part 1 & 2 by UIUC (Coursera)
  * Overview of the "big data" ecosystem. Gives brief descriptions of major technologies (Ceph, HBase, Kafka, etc.) and vendors (e.g. Hortonworks, Cloudera).

- [CMU 15-213 Introduction to Computer Systems](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/) (Fall 2015)
  * Comparable to MIT 6.004 Computation Structures, which is available on OCW. This CMU course takes a "programmer's perspective", and skips circuits.
  * [Video lectures](https://scs.hosted.panopto.com/Panopto/Pages/Sessions/List.aspx#folderID=%22b96d90ae-9871-4fae-91e2-b1627b43e25e%22)

- [ops-class.org](https://www.ops-class.org/)
  * A course on operating systems. All material (including video lectures) online.
  * The same author created [internet-class](internet-class.org).
