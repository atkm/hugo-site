+++
title = "Notes for \"Efficiently Compiling Efficient Query Plans for Modern Hardware\" (VLDB, 2011)"
date = "2024-01-22"
draft = true
+++

Paper: T. Neumann, _Efficiently Compiling Efficient Query Plans for Modern Hardware_, in _VLDB_, 2011

This was the primary reading for the lecture on query compilation in CMU's 15-721.
The lecture motivates this paper with the limitation of Hique.
Namely, the improvement in execution time from code specialization is cancelled out by the compilation time, which sits on the critical path.

Redshift does an interesting thing in this space.
They generate C++ code, but [cache the compiled code](https://aws.amazon.com/blogs/big-data/fast-and-predictable-performance-with-serverless-compilation-using-amazon-redshift/) to amortize the compilation latency.

## Paper notes

### Page 539

* "
_As the tuples must be produced one at a time, the table scan operator has to remember where in the compressed stream the current tuple is and jump to the corresponding decompression code when asked for the next tuple._
"

They bring up this as an example of "poor code locality and complex book-keeping", which is mentioned two sentences prior.
The previous example of the `next` function being a virtual call is already an example of poor code locality, so I didn't really understand why they re-emphasized it.
I would think that the benefit of code specialization in this example is the same as the previous example.

For the "complex book-keeping" part, I don't see how remembering the current offset in the compressed stream is complex.


* "
_Pipelining means that an operator can pass data to its parent operator without copying or otherwise materializing the data._
"

I was initially confused by this statement, because operators still have to return something in the tuple-by-tuple processing model.
The point that they are making is that what the operators return can fit in registers or L1.
Another thing to note is that the space that a child operator returns to may not be reused by its parent operator if thier return types are different.
That said, would it not be possible to create an untyped scratch space (that fits into, say, L3) that is shared by all operators in a pipeline?
Either way, nothing about the technique described in this paper precludes the size of operator output.
In fact, the paper later talks about vectorizing operator execution.

### Page 540

* "_Processing is data centric and not operator centric.
Data is processed such that we can keep it in CPU registers as long as possible.
Operator boundaries are blurred to achieve this goal._"

Nothing to add here. I just thought this was worth highlighting.

* "_As long as query processing was dominated by disk I/O, the iterator model worked fine._"

My impression was that disk performance hasn't changed much in some time (like a decade).
I need to go find references.

* "_... as tuples are passed via function calls to arbitrary functions -- which always results in evicting the register contents._"

Right! It's not like they get cleared, but the callee is free to do whatever they wish with the registers.
The caller or callee must save registers on the stack.
In page 543, the paper talks about how saved registers should remain in cache, but "it becomes noticeable" when repeated.

### Page 541

Nothing was highlighted here.

### Page 542

The `produce-consume` API is set up so the produce calls go from the root operator to the leaves, then the children call consume on parents.
Consume is where data processing happens.
In this sense, this is a push-based processing model.

* "_... compiling a complex query could take multiple seconds._"

The lecture slides suggest that the compile time alone is similar to the execution time of un-specialized, un-optimized code.
This should have the data: K. Krikellas, et al., _Generating Code for Holistic Query Evaluation_ (ICDE, 2010) should have the data.

The paper says that, in contrast, LLVM compilation takes single-digit ms.

* "_C++ does not offer total control over the generated code, which can lead to suboptimal performance._"

Is this something that compilers can become better at?
The paper talks more about this later.
The answer is that there are contexts that the compiler cannot be aware of.

* "_... the LLVM assembler is strongly typed, which caught many bugs that were hidden in our original textual C++ code generation._"

I find it interesting that writing an assembly (sure, it's an LLVM assembly but still) helped catching bugs.

* "_one can easily mix LLVM and C++, as C++ methods can be called directly from LLVM and vice versa.
... The complex part of the query processing (e.g. complex data structure management or spilling to disk) is written in C++...
The different operators are connected together by LLVM code ..._"
This highlights how much of execution time is spent in function call overhead

* "_The C++ "cogwheels" are pre-compiled;
only the LLVM "chain" for combining them is dynamically generated.
Thereby we achieve very low query compilation times._"

This means that the bulk of compilation time in a C++ codegen scheme comes from the complex logic.

### Page 543

* "_... the tuple access itself and the further tuple processing (filtering, materialization in hash table) is implemented in LLVM assembler code.
C++ code is called from time to time (like when allocating more memory),
but interaction of the C++ parts is controlled by LLVM._"

More description on the responsibilities of C++ vs. LLVM.

* "_For optimal performance it is important that the hot path, i.e. the code that is executed for 99% of the tuples, is pure LLVM.
... While staying in LLVM, we can keep the tuples in CPU registers all the time,
which is about as fast as we can expect to be._"

Reiteration of what has been said before.

* "_... contrary to the simple examples seen so far in the paper it is not possible or even desirable to compile a complex query into a single function._"

Two straightforward reasons.
One is that complex logic is better handled by C++.
The other is that inlining results in large code.

* "_... we have to keep track of all attributes and remember if they are currently available in registers.
Materializing attributes in memory is a deliberate decision, similar to spooling tuples to disk.
Of course not from a performance point of view [...], but from a code point of view, materialization is a very complex step that should be avoided if possible._"

LLVM provides an unlimited number of virtual registers, and generates instructions to spill to memory when there are not enough physical registers.
The quote says that, just like how disk-based systems manage disk access via a buffer pool manager, they write their own memory spilling code instead of relying on what LLVM could generate.
The number of registers is machine-dependent, so I assume this approach is not portable. (What architectures don't have the same number of general-purpose registers as x86-64?)
Also, this seems like a case where the abstraction of unlimited registers gets in the way of the programmer.

But then, in section 4.3 they say: "_As mentioned before, we keep all tuple attributes in (virtual) CPU registers._"
So I don't know in what sense spilling to memory is "a deliberate decision".

* "_In general, we try to load attributes as late as possible.
... However, when these values are needed on the critical path [...], it makes sense to compute these values a bit earlier than strictly necessary._"

You can almost do this in C++ with `__builtin_prefetch`.
The difference is that `__builtin_prefetch` reads data into cache, whereas this approach in the paper reads data into registers.

* "_... the query compiler must produce code that allows for good branch prediction._"

There is no difference between LLVM and C++ in terms of branch prediction.

### Page 544

* "_The code generator is relatively compact.
In our implementation, the code generation for all algebraic operators required for SQL-92 consists of about 11,000 lines of code, which is not a lot._"

* "_Traditional block-wise processing has the great disadvantage of creating additional memory access.
However, processing more than one tuple at one is indeed a very good idea, as longas we can keep the whole block in registers.
In particular, when using SIMD registers this is often the case._"

They are fine with batched-processing as long as everything fits in registers.

* "_... it can help delay branching, as predicates can be evaluated and combined without executing branches immediately._"

I didn't get this.
The papers references here are: V. Raman et al. "_Constant-time query processing_" (_ICDE_ 2008) and K. A. Ross "_Conjunctive selection conditions in main memory_" (_PODS_ 2002).

### Page 545

* "_As we are mainly interested in an evaluation of raw query processing speed here, we ran a setup without concurrency, i.e. we loaded 12 warehouses, and then executed the TPC-C transactions single-threaded and without client wait times.
Similarly, the OLAP queries are executed on the 12 warehouses single-threaded and without concurrent updates._"

Experimental setup. No concurrent processing.

* _Table 2: OLAP Performance of Different Engines_"

The LLVM-generated code does clearly better than C++-generated ones.
An exception is Q5, which is join-heavy, where the difference is less pronounced (LLVM still wins).
This is because joins create pipeline breakers where data must be materialized.

VectorWise on Q5 does as well as HyPer+LLVM---why?

* "_... in particular, the aggregation query Q1, the differences are large.
Q1 highlights this very well, as in principle the query is very simple, just one scan and an aggregation._"

* "_... it is interesting to see how the generated machine code performs regarding branching and cache effects.
[...] In the experiment, we compared the LLVM version of HyPer with MonetDB
MonetDB performs operations in compact, tight loops, and can therefore be expected to have a low number of branch mispredictions_."

Why did they not compare the C++ version of HyPer with its LLVM version?
Branch misprediction rates should look similar, since their code organization are similar.
But they really should have compared the cache miss rates between the two versions!
Their big claim in this paper is that the improvements in execution time come from keeping hot data in registers.
If that's the case, they should have shown experimentally that the LLVM version should have fewer L1 cache misses than the C++ version.

## Thoughts

For most of the course, we are happy if tuples are in memory at access time.
This paper, on the other hand, tries to keep hot data in *registers*.
Clear benefits of this processing model are observed in the execution time benchmark results (section 6.1, Table 2).

That said, it's unclear from the paper why C++ is not able to generate executables that is as efficient as LLVM-generated ones, as the optimizations discussed in section 4.3 can be implemented in C++.

So, it's not obvious whether this execution time improvement is worth the complexity of generating LLVM.
A motivation that this paper cites for improving processing efficiency is improvements in disk performance---but have disks improved that much?
Perhaps reading their Umbra paper will shed insights into this point.

The compile time improvement, on the other hand, is a clear win.
But caching compiled code (like Redshift) is an alternative to reducing compilation time without having to generate LLVM IR.

A side note on compilation time: in the [Umbra talk](https://youtu.be/pS2_AJNIxzU?si=D2Q0bBpSKVwKohqu) that the author gave in 2022 (the paper is from 2011), he says that they generate *bytecode* because compilation time of generated LLVM assembly is superlinear.
The bytecode generated directly is not as optimized as what LLVM emits.
So, while going from C++ to LLVM was a strict win (ignoring development overhead), one must consider a tradeoff between compile time and execution time when choosing between LLVM and bytecode.
