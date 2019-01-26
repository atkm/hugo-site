+++
title = "Learning Resources - Computer Science (Systems)"
date = "2018-12-06"
+++
Freely available online resources that I've found useful for learning computer science, especially systems.
Most recently updated on 2019-01-25.

## Systems
Beyond CMU 15-213.

- [CMU 15-440 (Fall version)](https://www.synergylabs.org/courses/15-440/syllabus.html) (Ongoing)
    * The textbook (Van Steen & Tanenbaum) is readable.
        The slides are informative as supplements to corresponding textbook chapters and referenced articles.
    * Lab assignments are done in Go. Tons of debugging concurrency issues.
        0. Go warm-up. A simple key-value database server.
        1. Implement a subset of TCP starting with UDP.
            Build a work distribution application on top of the protocol.
        2. (WIP) Raft 
        3. ?

- [CMU 15-445 Intro to Database Systems](http://15445.courses.cs.cmu.edu/)
    * Great lecturer
    * Video lectures from Fall 2017 as well as Fall 2018 are available.
    * Lab assignment starter code is not available outside CMU.
    * Discusses details of database systems unlike Stanford's CS145.
    * This course is a pre-req to [CMU 15-721 Advanced Database Systems](http://15721.courses.cs.cmu.edu/), which covers in-memory systems. The first lecture of the advanced course (History of Database Systems) is a fun watch.
    * Alternatives are courses that use SimpleDB assignments (e.g. MIT 6.830, HMC CS133, U of W CSE444, Berkeley CS186).

- [CMU 15-418 Parallel Computer Architecture](http://15418.courses.cs.cmu.edu/spring2016/)
    * Topics: parallel programing models and their implementations in software as well as hardware, cache coherence, interconnect, granularity of synchronization,
        heterogeneous architecture, domain-specific parallel programming systems, memory/cache contention and throughput, costs of moving data,
        (possible) convergence of scientific/high-performance computing and data-intensive/distributed computing,
    * Recurring themes: locality, locality, locality, bandwidth limits, pipelining,
        hardware technologies that software programmers benefit from being aware of
    * Video lectures available

- Other systems courses
    * CMU courses tend not to make starter code public, while Stanford and MIT are much more likely to have theirs available to the public.
    * OS: Stanford CS140, Harvard CS161 (OS/161), MIT 6.828 (JOS).
        CMU 15-410 doesn't make all projects available for non-CMU people.
        Text -- Anderson & Dahlin, Operating Systems: Principles and Practice
    * Networks: CMU 15-441 (next: Spring 2019), Stanford CS144
    * Compilers: CMU 15-411 (next: Fall 2019?), Stanford CS143, MIT 6.035, Harvard CS153
    * Distributed systems: MIT 6.824

## Intro (Done)
Courses that are usually covered in the 1st and 2nd years of a CS curriculum (ref: [CMU](https://www.csd.cs.cmu.edu/sample-undergraduate-course-sequence)). 

- [CMU 15-213 Introduction to Computer Systems](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/) (Fall 2015)
    * Discusses on higher levels of abstraction than MIT 6.004 Computation Structures.
    This course doesn't cover circuits or make you write assembly code, for example.
    * Took 5 weeks to complete (lab assignments, video lectures, textbook chapter reading).
        Highlights:
        * malloclab -- segregated-list implementation of malloc. Analyze trace files to optimize performance.
        * shlab -- processes, signal handling. Getting signal concurrency right.
        * attacklab -- stack discipline. Buffer overflow attack and ROP attack! 
        * bomblab  -- understand a program behavior from an assembly code.
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
    * Each assignment (reading + coding) takes one day to complete, so 11 days in total.

- Functional Programming in Scala Specialization (Coursera)
    * Assignments are fun (i.e. moderately challenging) unlike most Coursera courses.
    * Course 1 & 2 comprise an introduction to functional programming and the Scala type system.
        Most of lecture time is spent on discussing various functional programming constructs, but the instructor (Odersky) dips toes into theoretical details sometimes, which I liked.
    * Course 3 covers data parallelism, parallel scan (aka prefix sum), and combiners.
        The discussion on combiners using conc-tree as an example is interesting.
        Programming assignments on Week 2 (reductions and prefix sums) and Week 4 (Barnes-Hut simulation) are great.

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
