
- - - 

***Cache Miss Rate Reduction in Hardware***

- - - 

***Average Memory Access Time***

***AMAT = Hit Time + Miss Rate x Miss Penalty***

*Three ways to improve AMAT:*
- ***Reduce Miss Rate***
- *Reduce Miss Penalty*
- *Reduce Hit Time*

- → *reducing the miss rate can be done in **hardware** or software*

- - - -

***Types of Misses - The 3 C’s***

***Compulsory**:*
- *the first access to a block is not in the cache → so the block must be brought into the cache*
- → *misses even in an infinite cache*

***Capacity**:*
- *if the cache cannot contain all the blocks needed during the execution of the program (working set) → cache misses will occur due to blocks being discarded & later retrieved*
- → *misses in Fully Associative Cache of size X (would miss in any type of cache of size X)*

***Conflict**:*
- *if block placement strategy is set associative or direct mapped, conflict misses (in addition to compulsory & capacity misses) will occur → a block can be discarded & later retrieved if too many blocks map to its set*
- → *misses in n-way associative cache of size X (due to some compromise made in associativity of cache)*

*Coherence:*
- *missed caused by cache coherence → data may have been invalidated by another processor or I/0 device*

- - - 

***3Cs Absolute Miss Rate***

***benchmark study:***

***compulsory misses:***
- → *compulsory misses are vanishingly few*

***increasing associativity:***
- → *biggest improvement from 1-way → 2-way*
- *some improvement from 2-way → 4-way → 8-way associativity*

- - - 

***3Cs Relative Miss Rate***

*% misses*

***increasing cache size:***
- → *for small cache → most misses capacity misses*
- *as cache size increases → increasing associativity helps moreso*
- → *compulsory misses become a more significant fraction of miss rate*

***relative miss rate:***
 - *relative miss rate → depends on program*
- → *benchmarks typically have low compulsory miss rates*

- - - 

***Benchmarks***

***SPEC CPU Benchmarks:***
- → *a suite of commonly used CPU benchmarks*
- → *concerns CPU-intensive applications*
- → *much of the published research depends on the SPEC CPU benchmarks*

- → *each **benchmark report** includes elaborate details of software & hardware configuration*
- → *including details of compiler optimisation flags*
- → *for **base:** compiler flags for all benchmark programs*
- → *for **peak:** per-benchmark tuning of compiler flags*

- → *different systems achieve different relative performance on different programs in the benchmark suite*
- → *performance is averaged across the suite to produce the overall speed result*
- → *the geometric mean is used*

<br>

***Integer Benchmarks:***
- → *measure performance of executing integer arithmetic*
- → *integer arithmetic used for logical operations/addressing/counting*
- → *tend to make more intensive use of pointers & hard-to-predict branches*
- → *hard to parallelise*

- → *integer operations common in general-purpose computing - such OS/Text Processing/Databases etc.*

<br>

***Floating Point Benchmarks:***
- → *measure performance of executing floating-point arithmetic*
- → *floating-point arithmetic used for precise calculations*
- → *may benefit from more automatic parallelisation*

- → *floating -point operations common for applications that require large range of numbers/precise measurements - such as DSP/Graphics/Scientific Computing*
- → *floating-point operations in general more complex than integer operations & may take longer to execute - require handling numbers of varying magnitude & precision*

<br>

***Speed:***
- → ***execution time for one run of the program (possibly using multiple cores)***

<br>

***Rate:***
- → ***maximum throughput of completed jobs/second***

<br>

***Criteria for Good Benchmark Design:***
- ***Relevance:** benchmarks should represent the workloads & applications that will be commonly used*
- ***Verifiability:** results from the benchmark should be reproducible & verifiable by third parties*
- ***Fairness:** the benchmark suite should be fair & not favour any particular architecture or system*
- ***Transparency:** the methodology & operations of the benchmark tests should be open & transparent*
- ***Scalability:** a good benchmark should be scalable to different system sizes & configurations - should provide meaningful data across a range of system capabilities*
- ***Diversity:** the suite should include a variety of tests that cover different aspects of system performance - such as CPU, memory, I/O, & graphics*
- ***Metric Simplicity & Relevance:** the performance metrics used should be simple to understand & yet meaningful - should accurately reflect the performance characteristics that are most relevant to users*
- ***Regular Updates:** the benchmark suite should be regularly updated to reflect changes in usage patterns & user expectations*

- - - 

***Reducing Miss Rate***

- → ***Change Block Size***

- → ***Change Associativity***

- → ***Change Compiler***

- - - 

***Increasing Block Size***

→ ***exploits spatial locality to reduce miss rate***

→ ***increases miss penalty***

<br>

***spatial locality:***
- *initially the **miss rate improves with increasing block size → due to spatial locality** (more data copied into cache at once)*

<br>

***for small caches:***
- → *with very large cache blocks, when the cache rate is small → **miss rate deteriorates as large block sizes take up a large amount of the total cache capacity** (1KB cache with 256byte blocks → 4 blocks → 4 distinct memory regions in cache)*
- → *if cache lines are too large → may not be well utilised (large amount of data may be fetched into the cache & then never used)*
- → *if speculatively-loaded data is loaded into the large cache blocks → a large amount of space becomes wasted*

<br>

***miss penalty:***
- → ***miss penalty will increase with larger blocks**, as larger blocks take longer to load*
- *(for a 256k cache with 256byte blocks → appears larger blocks give smaller miss rate, but will also give much higher miss penalty)*

- - - 

***Increasing Associativity***

***increasing associativity:***
- *increasing associativity → increases depth of cache selector logic (more blocks to select from)*
- *increasing associativity → decreases miss rate → but cache hit time is increased slightly (due to greater selector logic depth)*

- *for larger cache sizes → increasing associativity does not improve AMAT*

<br>

***way prediction:***
- *want the **cycle/hit time of a direct-mapped cache (no selector logic) & miss rate of an associative cache***
- *solution → **way prediction (predict which way to select for each cache block - without need for selector structure)***

- - - 

***Victim Cache***

**→ *combines fast hit time of direct mapped & miss rate of associative cache (avoiding conflict misses)***

→ *adding a buffer to place data discarded from cache*

<br>

***structure:***
- *direct mapped cache above*
- *fully-associative cache below (buffer/victim cache)*
- → *access both in parallel*
- → *fully associative can be very small → small energy cost & fast cycle time*
- *→ direct mapped cache → fast cycle time*

<br>

***operation:***
- ***on access** → check both direct-mapped & victim cache in parallel*
- ***on hit of direct-mapped cache** → use data*
- ***on miss of direct-mapped cache** → check for hit victim cache & allocate into direct-mapped cache (displacing data) (otherwise → allocate from lower levels of memory hierarchy)*
- ***on replacement** → allocate this displaced data into the victim cache*

- - - 

***Skewed-Associative Cache***

→ ***reduce conflict misses by using different indices in each cache way***

→ *introducing a **hash function***

<br>

***hash function:***
- → *a mathematical function that converts any digital data into an output string with a fixed number of characters*
- → *e.g. XORing some index bits with tag bits & reordering index bits*

<br>

***conventional set-associative:***
- *→ in conventional 2-way set associative cache → to blocks map to the same set → can store 2 of A,B,& C but not all 3*

<br>

***skewed set-associative:***
- *→ in skewed 2-way set associative cache → depending on the hash function → all 3 could map to the same set in the left way & all 3 could map to different sets in the right way
- → *A,B,C conflicting in the left way*
- → *more likely that the A,B,C will map to different sets → can store all 3 at once in cache*

<br>

***loops & arrays:***

***skewed-associative:***
- → *suppose traversing three arrays A/B/C*
- → *suppose* $f_0(A[i]) = f_0(B[i]) = f_0(C[i])$ & $f_1(A[i]) = f_1(B[i]) = f_1(C[i])$
- *→ A/B/C conflict in both ways of the skewed associative cache (only 2 can be stored at once)*
- → *unlikely that this is the case for each element in the array we are traversing*
- → ***remaps with pseudo-random cache functions on every index of the arrays A/B/C***

***set-associative:***
- → *suppose traversing three arrays A/B/C*
- → *if same mapping function (lower order k bits) used for both ways (2-way set-associative)*
- → *can easily be unlucky due to power of 2 alignment & $f(A[i]) = f(B[i]) = f(C[i])$*
- → *three arrays map to same location in both ways*
- *→ the conflict would remain for every element in the arrays A/B/C (arrays addresses in sequence & no pesudorandomness)*
- → ***set associative worse performance than skewed-associative for random addresses & much worse performance for array accesses***

<br>

***skewed-associative vs set-associative:***
- *with skewed-associativity → **can get same miss rate with reduced associativity***
- → ***more predictable average performance** (pesudorandomness spreads ‘unlucky’ outcomes uniformly → hash changing on every access)*
- → ***harder to write a program that is free of associativity conflicts** (cannot predict hash functions outputs) → easy for set-associative*

<br>

***costs**:*
- *needs one address decoder per way (**added hardware**)*
- ***hash function latency** (slight computational cost)*
- ***difficulty of implementing LRU** replacement policy*
- *index hash uses translated bits*

- - - 

***Prefetching***

***larger cache blocks:***
- → *exploit spatial locality to improve miss rate*
- → *but - larger blocks take a long time to load → higher miss penalty*

***prefetching:***
- ***→ a better way of exploiting spatial locality (fetching data ahead of time before they are needed for execution)***
- → *adds a stream buffer → hardware prefetching mechanism*

<br>

***operation:***
- ***on access** → direct-mapped cache & stream buffer accessed in parallel*
- ***on a direct-mapped cache hit** → use that data*
- ***on a direct-mapped cache miss** → 
- → *check the stream buffer, if data available, forward to CPU*
- → *fetch data for that miss into the direct-mapped cache (either from the stream buffer or next level of memory hierarchy)*
- → *initiate fetch for the next cache line from memory hierarchy into stream buffer*

<br>

***spatial locality:***
- → ***stream buffer exploits spatial locality by prefetching next cache lines before they are needed** during execution (predicting that the next cache lines are going to be needed by execution)*
- → ***prediction mechanism** could be changed to enhance accuracy & predict when a sequence of misses occurs (similar to branch prediction mechanism)*
- → ***much faster to access the cache lines from stream buffer** - rather than accessing main memory*

<br>

***in modern CPUs:***
- → *more likely in modern CPUs that **data will be prefetched & allocated into the direct-mapped cache** (rather than having a stream buffer)*
- → *having speculatively executed data in the direct-mapped cache*
- → *with bigger cache - **having speculatively executed data directly in the cache less of a concern** (& stream buffer not needed)*
- → ***many modern CPUs have hardware prefetching***

<br>

***memory accesses:***
- → *adds extra speculative memory accesses → **extra memory bandwidth needed***

<br>

***multiple stream buffers:***
- → *can be scaled up using **multiple stream buffers** → **each maintaining a separate prefetch stream***
- → *for each new miss → a new stream buffer allocated*

<br>

***multi-way stream buffer:***
- → *accesses multiple stream buffers simultaneously*
- → *one stream is good for I-Cache misses*
- -. *multiple stream often important for D-Cache misses (e.g. traversing multiple arrays)*
→ *can also prefetch k values at a time - rather than only the next value (stream of values)*

- - - 

***Decoupled Access Execute***

- → *most programs have no dependence between the instructions that generate memory addresses (integer) & instructions that use memory results (floating-point)*
- → ***separate the instructions that generate addresses from the instructions that use memory results***
- → ***let the address-generation side of the machine run ahead***

<br>

- ***run instructions that generate memory addresses on the access processor***
- → *when data arrives from memory system → execute arithmetic operations on execute processor*

<br>

- → ***access processor runs ahead of execute processor (runahead)***
- → *can exploit parallelism in access processor to stream values to execute processor as fast as possible*

<br>

- → ***assumes no dependencies between FP results & addresses being generated***
- *dependency occurs → loss of decoupling event*

- - - 

***Simple Decoupled Access Execute Machine***

- ***two pipelines: integer & floating point***
- ***integer pipeline:** control operations, incrementing pointers, address calculation for load/store*
- ***floating point pipeline:** multi-cycle data computations*

<br>

***operation:***
- *instructions issued from integer pipeline*
- *example: **load instruction***
- *let the load address go to the memory & have microOp telling the floating point pipeline what operation needs to be done and in how many cycles the load data will be received directly from the integer pipeline*
- *addresses are generated in the integer pipeline - and data is forwarded directly to the floating point pipeline*
- *microOps stored in microOp queue (FIFO) - storing FP operations*
- *load data is stored in the load data queue*
- *floating point pipeline fetches the microOp & load data, performs the computation, & updates register*

<br>

- *for a **store instruction** - store address can be generated early by the integer pipeline (if it is not blocked) & put in the store address queue*
- *the floating point pipeline then later puts the data to be stored in the store data queue after executing and finding data value*
- *decoupling → addresses produced in the integer pipeline, data received & produced in the floating point pipeline*

<br>

- ***assuming in-order execution***

<br>

***dependencies:***
- *want to check if we have a RAW dependency (there is an earlier store instruction that wants to store data at any address that is then loaded from later - it might not have stored intime for the load) - by checking the load address against the queued store addresses*
- *if RAW dependency - stall pipeline*
- *this is a check for correctness rather than performance*

- - - 

***Summary***

***Reducing Miss Rate Through Hardware:***

***Increase Cache Capacity***
- → *reduces capacity & conflict misses*
- → *but bigger cache will be slower / will have to be pipelined*

***Increase Block Size***
- → *exploits spatial locality to reduce miss rate*
- → *increases miss penalty*

***Increase Associativity:***
- → *reduces conflict misses*
- → *but loses fast hit-time of direct-mapped cache*

***Victim Cache:***
- → *combines fast hit time of direct mapped & miss rate of associative cache (avoiding conflict misses)*
- → *reduces miss rate due to associativity conflicts*

***Skewed Associative Cache:***
- → *reduce conflict misses by using different indices in each cache way*
- → *introducing a hash function*
- → *can achieve same miss rate with reduced associativity*

***Hardware Prefetching:***
- → *a better way of exploiting spatial locality (fetching data ahead of time before they are needed for execution)*
- → *reduces miss rate by spatial locality & does not increase miss penalty (as larger blocks)*

***Decoupled Architecture:***
- → *separate the instructions that generate addresses from the instructions that use memory results*
- → *let the address-generation side of the machine run ahead*
- → *reduces miss delays by issuing load instructions early enough*

- - - 


