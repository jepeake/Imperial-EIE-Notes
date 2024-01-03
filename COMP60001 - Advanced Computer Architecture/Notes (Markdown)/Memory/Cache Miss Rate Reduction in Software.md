
- - - 

***Cache Miss Rate Reduction in Software***

- - - 

***AMAT:***

***Average Memory Access Time***

*AMAT = Hit Time + Miss Rate x Miss Penalty*

*Three ways to improve AMAT:*
- ***Reduce Miss Rate***
- *Reduce Miss Penalty*
- *Reduce Hit Time*

→ *reducing the miss rate can be done in hardware or **software***

- - - 

***Software Prefetching***

- ***→ reduce miss rate by anticipating data needs of a program & prefetching that data before execution time***
- → *uses **explicit prefetch instructions in software** → inserted by programmer or compiler*

<br>

***prefetch instructions:***
- *→ initiate a load of a cache line into the cache (without stalling waiting for the data to arrive)*

<br>

***effectiveness of software prefetching depends on the timing of the prefetch instruction:***
- *if the prefetch is too close to the instruction using the prefetched data → the cache line will not have has time to arrive from main memory or the next cache level & instruction will stall (reducing effectiveness of prefetch)*
- *if the prefetch is too far ahead of the instruction using the prefetched data, the prefetched cache line may have already been evicted again before the data is used (causing another fetch of the cache line → stall) → cache line has to be fetched twice from memory*

<br>

***care is needed to ensure prefetch accesses don’t have unwanted side effects:***
- → *prefetch instructions may target addresses that would cause a page fault/exception*
- → *prefetches of addresses that would result in a page fault/exception are silently squashed*
- *in example → R10KCBARRIER macro used to control behaviour of prefetched instructions & prevent accessing of memory mapped IO registers*

<br>

***executing prefetch instructions has a cost:***
- → *instruction has to be decoded & uses some execution resources*
- → *a prefetch instruction that always prefetches cache lines that are already in the cache will consume execution resources without providing any benefit*
- → *important to verify that prefetch instructions really prefetch data that is not already in the cache*

<br>

***cache miss rate of a prefetch instruction needed to be useful depends on purpose:***
- → *prefetch instruction that fetches data from main memory **only needs small improvement in the miss rate** to be useful → due to the high main memory access latency*
- → *prefetch instruction that fetches cache lines from a cache further away from the processor may need higher miss rate improvement*

<br>

***usefulness of software prefetching:***
- → *almost never pays off on sophisticated processors →  as hardware prefetching is good (may be useful on simpler processors)*
- *commonly, software prefetching fetches slightly more data than is actually used (e.g. at the end of the loop, may continue prefetching k instructions past the loops end)*

- - - 

***Reordering I-Cache Structure***

- ***→ to reduce I-cache misses (assuming a direct-mapped I-cache) → want to reduce conflict misses***
- → *if two functions F/G map to the same location in direct-mapped cache & F/G called in a loop → F/G calls will cause conflict miss in I-cache*
- → *select I-cache layout using a **callgraph, branch structure, & profile data***

<br>

***reordering functions in instruction cache:***
- *construct the call graph → showing which loops call which functions*
- ***use call graph to choose I-cache layout (in such a way that functions called in the same loop do not overlap & avoids associativity conflicts)***
- *e.g. D, B, E all called in loop 2 → do not overlap in I-cache*
- *A. B. C overlap → okay as they are not called in the same loop*

- - - 

***Reducing Data-Cache Misses***

***Storage Layout Transformations:***
- ***storage layout transformations** → change way in which variables are declared & address at which data is stored (improves spatial locality & cannot change temporal locality → cannot change order of execution)*
- ***Merging Arrays** → improves spatial locality by merging two arrays into a single array of compound elements*
- ***Permuting a Multidimensional Array** → improve spatial locality by matching array layout to traversal order*

<br>

***Iteration Space Transformations:***
- ***iteration space transformations** → change the ordering in which loops are executed (improves temporal locality → transforms a particular loop & can change spatial locality)*
- ***Loop Interchange** → change nesting of loops to access data in order stored in memory*
- ***Loop Fusion** → combine two independent loops that have the same looping & some variables overlapping*

- - - 

***Array Merging***

→  ***improves spatial locality by merging two arrays into a single array of compound elements***

<br>

***given two arrays of values (val & key):***
- *initially stored as two sequential arrays (val array & key array) → sequentially in memory*
- *array merging → change storage layout & store two values (val & key) together in a struct → then storing structs in a single array (array of key-value pairs)*
- → *giving key-value pairs adjacent in memory*

- *array merging: **struct of arrays (sequential in memory) vs array of structs (adjacent in memory)***
- → *transpose operation (2xn → nx2)*

<br>

***examples:***
- → *if we’re searching for a value (search for key first & then access value) → good to have arrays stored sequentially, can search through all keys first without looking at values*
- → *if we have a hash table → want to fetch key-value pairs → good to have arrays stored adjacently in a struct (rather than having to traverse through two sequential arrays)*

<br>

***improves spatial locality:***
- *key-value pairs are commonly needed together (when a key is accessed - typically its value will be accessed soon)*
- *by storing these pairs in adjacent memory locations → improves spatial locality as values commonly accessed together stored in adjacent memory locations*
- *when key accessed & brought into cache - value likely to be brought into cache also (as whole cache lines accessed at once to exploit spatial locality)*
- *reduces miss rate - as when value later accessed → will have already been brought into the cache with the key*

- - - 

***Permutating a Multidimensional Array***

→ ***improve spatial locality by matching array layout to traversal order (type of loop interchange)***

<br>

- *MM1 accesses data in column-major order*
- *MM2 accesses data in row-major order*

  <br>

- *in C & C++→ matrices stored in row-major order*
- → ***accessing data should be done in row-major order to match array layout & traversal order***
- → ***significantly improves spatial locality (contiguous row elements can be accessed together) & therefore reduces cache mises***

- - - 

***Loop Interchange***

→ ***change nesting of loops to access data in order stored in memory***

<br>

***row major memory layout → row values are stored contiguously:***
- *before → the innermost loop is completing 5000 times before the outmost loop iterates - meaning that the nested loop strides through 5000 elements corresponding to the first element in each row → which are not stored contiguously in row-major order*
- *after → the innermost loop iterates through all elements in a row (changing column index for a fixed row) before the outermost loop iterates to the next row → matching row major order & improving spatial locality*

<br>

→ ***interchanging the nested loops matches the row major order & accesses row elements sequentially***

<br>

***minimising stride access:***
- → ***stride**: memory distance between elements accessed in succession*
- → ***large strides lead to poor cache utilisation** (more cache misses & loading new lines to cache)*
- → *matching memory layout → reduces strides & reduces cache misses*

- - -

***Loop Fusion***

 → ***combine two independent loops that have the same looping & some variables overlapping***

<br>

***combine the two independent (nested) loops with the same loop conditions:***
- *performing S2 immediately after S1*
- *after fusion → much less loop overhead & don’t need to reload array a & c from memory (accessed in both operations)*
- *→ transpose of the iteration space*

<br>

- ***improved spatial locality → both loops accessing the same data → 2 misses per access to a & c vs 1 miss per access***

<br>

***concatenating loops:***
- *→ compiler may avoid even storing to array a (can just store into a variable & use straight away)*
- → *safe transformation as long array a not needed after S2*
- → ***array contraction: values transferred in scalar instead of via array**
- → *reduces misses further by no accesses to array a*

<br>

***stencil loops:***
- → *loops in which dependencies do not align nicely*
- *e.g. 1D convolution filters*
- → *dependencies do not align nicely due to new values depending on existing neighbouring values*
- - *computation of one point cannot be done independently of computation of neighbouring points*
- → *cannot directly transpose iteration space*
- → *not directly fusable*
- → ***make fusable by shifting***
- → ***aligns dependencies***

<br>

- *can we do array contraction in this shifted case? → more difficult*

- *V cannot be contracted to a scalar → as value needed 2 iterations earlier*
- → *can contract V to a smaller array that respects that dependency (→ 4 element array)*

- - - 

***Summary***

***Miss Rate Reduction in Software:***

***Software Prefetching:***
- ***→ reduce miss rate by anticipating data needs of a program & prefetching that data before execution time***
- → *uses **explicit prefetch instructions in software** → inserted by programmer or compiler*
- *(if they work better than predictive prefetch hardware → possible in simple CPUs)*

***Reordering I-Cache Structure:***
- → *select I-cache layout using a **callgraph, branch structure, & profile data***
- ***→ use call graph to choose I-cache layout (in such a way that functions called in the same loop do not overlap & avoids associativity conflicts)***

***Storage Layout Transformations:***
- ***storage layout transformations** → change way in which variables are declared & address at which data is stored (improves spatial locality & cannot change temporal locality → cannot change order of execution)*
- ***Merging Arrays** → improves spatial locality by merging two arrays into a single array of compound elements*
- ***Permuting a Multidimensional Array** → improve spatial locality by matching array layout to traversal order*

***Iteration Space Transformations:***
- ***iteration space transformations** → change the ordering in which loops are executed (improves temporal locality → transforms a particular loop & can change spatial locality)*
- ***Loop Interchange** → change nesting of loops to access data in order stored in memory*
- ***Loop Fusion** → combine two independent loops that have the same looping & some variables overlapping*

- - -

