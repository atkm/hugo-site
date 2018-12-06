+++
title = "Learning Resources - Computer Science (Systems)"
date = "2018-12-06"
+++
Freely available online resources that I've found useful for learning computer science, especially systems.

## Intro
Courses that are usually covered in the 1st and 2nd years of a CS curriculum (ref: [CMU](https://www.csd.cs.cmu.edu/sample-undergraduate-course-sequence)). 

- [CMU 15-213 Introduction to Computer Systems](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/) (Fall 2015)
  * Discusses on higher levels of abstraction than MIT 6.004 Computation Structures.
    This course doesn't cover circuits or writing assembly code, for example.
  * Starter code for the assignemnts are available on the [website](http://csapp.cs.cmu.edu/3e/labs.html) for the companion textbook.
  * [Video lectures](https://scs.hosted.panopto.com/Panopto/Pages/Sessions/List.aspx#folderID=%22b96d90ae-9871-4fae-91e2-b1627b43e25e%22)
  * Alternatives are Stanford CS 107 & 110 (which teach from the same textbook) and Berkeley 61C.

- MIT 6.005 Software Construction (OCW)
  * Software engineering best practices. What constitutes a good specification and test?
  Immutability. Interfaces. Equality of types. Specification and tests for abstract data types and concurrent programs.

- Algorithms Part 1 & 2 by Princeton (Coursera)
  * A first course on algorithms, which does not cover analyses of algorithms in depth.
  * The instructor sprinkles historical overview, "war stories", and applications of algorithms in each of his lectures, which are entertaining.
  * Good set of assignments to work through. They don't merely ask you to write a data structure introduced in lecture; each assignment requires some problem solving, such as figuring out what data structures should be used to achieve given performance requirements.
  * The assignments are the same as Princeton's [COS226](http://www.cs.princeton.edu/courses/archive/fall18/cos226/syllabus.php).
  * Each assignment (reading + coding) takes one day to complete.

- Functional Programming in Scala Specialization (Coursera)
  * Assignments are fun (i.e. moderately challenging) unlike most Coursera courses.
  * Course 1 & 2 comprise an introduction to functional programming and the Scala type system.
  Most of lecture time is spent on discussing various functional programming constructs, but the instructor (Odersky) dips toes into theoretical details sometimes, which I liked.
  * Course 3 covers data parallelism, parallel scan (aka prefix sum), and combiners.
  The discussion on combiners using conc-tree as an example is interesting. Programming assignments on Week 2 (reductions and prefix sums) and Week 4 (Barnes-Hut simulation) are great.

## Systems (Ongoing)
Beyond CMU 15-213.

- Stanford CS 140 (Operating systems)
  * The text (Anderson & Dahlin, Operating Systems: Principles and Practice) is readable.
  * Alternatives are Berkeley CS 162 (also uses Pintos), Harvard CS 161 (uses OS/161), ops-class.org (uses OS/161).
    CMU 15-410 doesn't make all projects available for non-CMU people.

- [CMU 15-445 Intro to Database Systems](http://15445.courses.cs.cmu.edu/)
  * Great lecturer
  * Video lectures from Fall 2017 as well as Fall 2018 are available.
  * Discusses details of database systems unlike Stanford's CS145.
  * This course is a pre-req to [CMU 15-721 Advanced Database Systems](http://15721.courses.cs.cmu.edu/), which covers in-memory systems. The first lecture of the advanced course (History of Database Systems) is a fun watch.

- Other systems courses
  * Distributed systems: CMU 15-440 (materials are different for Fall and Spring semesters), MIT 6.824
  * Compilers: CMU 15-411, Stanford CS 143 (is the one on Stanford Lagunita comparable?)
  * CMU 15-418 Parallel Computer Architecture. Video lectures available.

## Others

- [Practical Concurrent and Parallel Programming](http://www.itu.dk/people/sestoft/itu/PCPP/E2016/)
  * Covers most of "Java Concurrency in Practice" by Goetz with a Java 8 twist, and a bit of "The Art of Multiprocessor Programming".
  * Each week comes with a simple coding assignment, which takes about an hour.
    11 assignments in total.

- [Sophomoric Parallelism and Concurrency](https://homes.cs.washington.edu/~djg/teachingMaterials/spac/) (Dan Grossman, U of W)
  * A short introductory course on parallel data structures and concurrency using Java's fork-join framework.
  * Homework problems are food for thought.

- Computer Networks (David Wetherall, U of W)
  * An overview of networking from the physical layer all the way up to the application layer.
  * [Video lectures](http://media.pearsoncmg.com/ph/streaming/esm/tanenbaum5e_videonotes/tanenbaum_videoNotes.html).
   There used to be a course on Coursera based on the same videos.
  * The lecturer is an author of "Computer Networks" (5th ed).
