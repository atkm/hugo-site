+++
title = "Learning Resources - Computer Science (Systems)"
date = "2018-12-06"
+++

Freely available online resources that I've found useful for learning computer science, especially systems.
Last updated on 2019-11-23.

## Systems
Beyond CMU 15-213.

- [CMU 15-440 (Fall version)](https://www.synergylabs.org/courses/15-440/syllabus.html)
    * The textbook (Van Steen & Tanenbaum) is readable.
        The slides are informative as supplements to corresponding textbook chapters and referenced articles.
    * Lab assignments are done in Go. Tons of debugging concurrency issues.
        0. Go warm-up. A simple key-value server.
        1. Implement a subset of TCP starting with UDP.
            Build a work distribution application on top of the protocol.
        2. Raft 
        3. Sharded key-value server
    * Progress: done (labs, assignments, lectures).
    * Other distributed systems courses: CMU 15-440 spring version, MIT 6.824.

- [CMU 15-445 Intro to Database Systems](http://15445.courses.cs.cmu.edu/)
    * Great lecturer
    * Video lectures from Fall 2017/2018 are available.
    * Lab assignment starter code is not available outside CMU.
    * Discusses details of database systems unlike Stanford's CS145.
    * This course is a pre-req to [CMU 15-721 Advanced Database Systems](http://15721.courses.cs.cmu.edu/), which covers in-memory systems. The first lecture of the advanced course (History of Database Systems) is a fun watch.
    * Progress: in progress (1.5 labs left)
    * Alternatives are courses that use SimpleDB assignments (e.g. MIT 6.830, HMC CS133, U of W CSE444, Berkeley CS186).

- [CMU 15-418 Parallel Computer Architecture](http://15418.courses.cs.cmu.edu/spring2016/)
    * Topics: parallel programing models and their implementations in software as well as hardware, cache coherence, interconnect, granularity of synchronization,
        heterogeneous architecture, domain-specific parallel programming systems, memory/cache contention and throughput, costs of moving data,
        (possible) convergence of scientific/high-performance computing and data-intensive/distributed computing,
    * Recurring themes: locality, locality, locality, bandwidth limits, pipelining,
        hardware technologies that software programmers benefit from being aware of
    * Video lectures available
    * Progress: watched lectures.

- [UCSD CSE131 Compiler Construction](https://ucsd-cse131-f19.github.io/)
    * Focus on the compiler backend.
    * Unlike Stanford's [course](https://web.stanford.edu/class/cs143/), it skips lexical analysis and parsing.
        I was interested in learning code generation and optimization, so this course worked well for me.
    * Progress: done.

- [MIT 6.172 Performance Engineering of Software Systems](https://ocw.mit.edu/courses/6-172-performance-engineering-of-software-systems-fall-2018/)
    * Topics are different from CMU's 15-418.
        Labs teach the mechanics (~boring details) of doing performance work.
    * Working with cilk and llvm sounds interesting.
    * Progress: watched lectures, done some labs.

- Other systems courses
    * CMU courses tend not to make starter code public, while Stanford and MIT are much more likely to have theirs available to the public.
    * OS: Stanford CS140, Harvard CS161 (OS/161), MIT 6.828 (JOS).
        CMU 15-410 doesn't make all projects available for non-CMU people.
        Text -- Anderson & Dahlin, Operating Systems: Principles and Practice
    * Networks: CMU 15-441, Stanford CS144.
    * Compilers: CMU 15-411 (next: Fall 2019?), Stanford CS143, MIT 6.035, Harvard CS153.

## Computer Architecture

- [ETH Design of Digital Circuits](https://safari.ethz.ch/digitaltechnik/spring2019)
    * The first third of the course covers some basics of circuits.
        The rest covers computer architecture topics: pipelining, out-of-order execution, branch prediction,
            VLIW, systolic arrays, SIMD.
        Lecture notes for MIT's 6.004 are good supplementary reading, as they go deeper into more basic topics.
    * Assignments are done with Verilog on an FPGA.
        They are rudimentary compared with what is discussed in lectures---writing a single-cycle MIPS is the final assignment.
        A follow-up [course](http://www.archive.ece.cmu.edu/~ece447/s15/doku.php?id=labs) has assignments on topics from the second half of the course (pipelining, branch prediction, cache coherence).
    * The assignments felt a bit too easy.
        Most time was spent on learning the tools (verilog and IDE---I used Quartus).
    * Progress: completed all labs, watched lectures.
    * Computer architecture [courses](https://people.inf.ethz.ch/omutlu/teaching.html) from the same instructor.
        His lectures on memory systems are interesting.

- [MIT 6.004 Lecture Notes](https://computationstructures.org/lectures/info/info.html)
    * The lectures start with representations of information at the analog level, and end with software topics like virtual memory, operating systems, and parallel computing.
    * The course ties together areas in EECS.
        The main topics of the course are digital circuits and computer organization, but it also touches on neighboring topics like:
            - information entropy, error detection/correction
            - a tiny bit of analog circuits through looking at the RAM
            - circuit timing (throughout the circuits section of the lectures)
            - circuit design tradeoffs (area, power consumption, throughput, latency)
            - computability (FSM, turing machine)
            - compilers, IR optimization
            - OS scheduler
            - cache coherence
            - ... and more.
    * It gives you a sense of how information theory, electronics, computer organization concepts, CS theory, and software concepts are part of one big thing.
        Highly recommended!


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

- Algorithms Part 1 & 2 by Princeton (Coursera)
    * A first course on algorithms, which does not cover analyses of algorithms in depth.
    * The instructor sprinkles historical overview, "war stories", and applications of algorithms in each of his lectures, which are entertaining.
    * Good set of assignments to work through. They don't merely ask you to write a data structure introduced in lecture; each assignment requires some problem solving, such as figuring out what data structures should be used to achieve given performance requirements.
    * The assignments are the same as Princeton's [COS226](http://www.cs.princeton.edu/courses/archive/fall18/cos226/syllabus.php).
    * Each assignment (reading + coding) takes one day to complete, so 11 days in total.

- MIT 6.005 Software Construction (OCW)
    * Software engineering best practices. What constitutes a good specification and test?
    Immutability. Interfaces. Equality of types. Specification and tests for abstract data types and concurrent programs.

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
