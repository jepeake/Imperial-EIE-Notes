
- - - 

***Shared Memory vs Distributed Memory***

- - - 

***Parallel Programming Overview***

→ ***power considerations & multicore***

→ ***shared-memory parallel programming*
	→ *dynamic load-balancing*
	→ *distributed-memory parallel programming (why harder to program but more robust performance)*

→ *cache coherency*
	*→ design space of snooping protocols based on broadcasting invalidations & requests*

→ *atomic operations & locks***

→ *sequential consistency*

- - - 

***Microprocessor Trend***

- → *single thread performance non-longer increasing with Moore’s Law*
- → *as clock frequency hit a maximum point (power & cooling requirements)*
- → *power hit a maximum point*
- → ***to exploit transistor count & increase performance (without increasing clock frequency) → increase number of cores*

- - - 

***Power Constraint***

***Dynamic Power:***
- → ***power consumed when signals change** (when a transistor changes start from 0 to 1 or vice versa)*
- → *proportional to the square of the supply voltage*

***Static Power:***
- → ***power consumed when gates powered-up** (by leakage current from transistors in a static state)*

***Dennard Scaling:***
- → ***dynamic power gets smaller if we make the transistors smaller** & so can increase clock rate with smaller transistors*
- → ***end of Dennard Scaling →*** *as transistors become smaller  - **static power starts to dominate** - particularly at high voltages (needed for high clock rates) & so cannot increase clock rates to get performance

***Power vs Clock Rate:***
- → ***power increases with clock rate***
- → *due to high static leakage due to high voltage (static power)*
- → *& high dynamic switching (dynamic power)*

***Clock vs Parallelism:***
- → *much more efficient to use lots of parallel units at a low clock rate & low voltage (less energy per operation)*

***Solutions:***
→ ***compute fast & then turn system off*

→ ***compute just fast enough to meet deadline*

→ ***clock/power gating***
- → *turn functional units/cores off when they are not being used*

→ ***dynamic voltage/clock regulation*
- → *reduce clock rate dynamically*
- → *reduce supply voltage dynamically*
- → *such as when CPU is not a bottleneck (or battery is low)* 

→ ***run on lots of cores - each at a low clock rate*

→ ***turbo mode*
- → *boost clock rate only when one core is active*

- - - 

***Programming a Parallel Computer***

*two types of possible memory:*
- → ***Shared Memory*
- → ***Distributed Memory*

***Shared Memory:***
- → *allows **multiple processors to access the same memory space***
- → *makes parallel programming easier as **all processors can directly read & write to the same memory locations without the need for complex message-passing protocols** (needed in distributed memory systems)*
- → *need to ensure **cache coherence***
- → ***locks, semaphores, & barriers*** *used to avoid race conditions when processors accessing the same shared memory*

***Distributed Memory:***
- → ***each processor has its own private memory***
- → *processors communicate through **message passing***
- → *introduces **communication overhead***
- → *more complex programming needed - as requires explicit management of synchronisation & communication*

***Shared vs Distributed***
- ***ease-of-use →*** *shared memory is typically easier to program due to its straightforward memory model (but can become complex for a large number of processors due to cache coherence issues)*
- ***scalability →*** *distributed memory scales better but requires careful management of communication & synchronisation*
- ***suitability*** → *shared memory is suitable for small to medium-scale parallelism (i.e. multicore processors) while distributed memory is preferred for large-scale parallelism (i.e. computer clusters)*
- ***performance*** → *shared memory is faster as can communicate with just a load store (no message passing overhead - as in distributed)*

- - - 

***Example - Parallel Programming***

***code:***
- → *first loop operates on rows in parallel*
- → *second loop operates on columns in parallel*
- → *with **distributed memory - would have to program message passing between to transpose the array***
- → *with **shared memory - both loops can access the same memory & no message passing needed***

- - - 

***Shared-Memory Parallel (OpenMP)***

***OpenMP (Open Multi-Processing):***
- → *application programming interface (API) for **shared-memory parallel programming***
- → *typically found in **multicore processors***

***Features:***
- ***Directive-Based Model →*** *OpenMP used compiler directives (pragmas) that tell the compiler how to parallelise the code*
- → *allows for incremental parallelisation (can begin with a serial program & parallelise step by step)*
- ***Synchronisation Mechanisms*** → *contains mechanisms like barriers, critical sections, locks, & atomic operations to synchronise execution of threads (avoiding race conditions)*
- ***Data Environment*** → *provides mechanisms for specifying how variables should be treated in parallel regions (shared, private, reduction) - helping to manage the data scope & lifecycle across threads*
- ***Runtime Library Routines*** → *offers a set of library routines for various purposes like setting number of threads, querying number of threads…*
- ***Environment Variables*** → *control aspects like number of threads to use etc.*

***example:***

***#pragma omp parallel for*** → *directive tells compiler that **next loop should be executed in parallel***

- ***default(shared) private(i)** → tells the compilers all variables except i are shared by all threads*
- ***schedule(static,chunk)** → tells the compiler that iterations of the parallel loop should be distributed in equal sized blocks to each thread (chunk parameter specifies the size of each chunk of iterations) & these chunks of iterations are statically scheduled*
- ***reduction(+:result)** → performs a reduction on the variables that appear in the argument list (combining values of private copies of result from all threads into a single value)*

***Static Scheduling:***
- → ***effective when workload is evenly distributed** across the iterations of the loop*
- → ***minimises runtime overhead** associated with scheduling*
- → *simple way to **balance the load** when programmer has prior knowledge that each iteration will approximately take the same amount of time*

***Reduction:***
- → ***simplifies the code** by managing the private copies & combining results (otherwise would require manual synchronisation & thread-safe programming)*
- → *leads to better performance by **minimising synchronisation overhead** & allowing concurrent computation*
- → ***avoids race conditions** that can occur when multiple threads try to update a shard variable simultaneously*

- - -

***Implementing Shared-Memory Parallel Loop (OpenMP)***

***Serial Loop vs Shared-Memory Parallel Loop (OpenMP):***

***Serial Loop:***
- → *iterating over arrays A and B and computing there element wise sum into array C*
- → ***runs on a single thread in a sequential manner***

***Shared-Memory Parallel Loop:***
- → *executed by every core*
- → ***uses self-scheduling to distribute the iterations of the loop across multiple threads in a shared-memory environment***

***Barrier Synchronisation:***
- → ***synchronisation mechanism which forces all threads to wait until every thread has reached point in the code before any can proceed***
- → *ensures that all threads complete their current tasks before any of them move onto subsequent tasks (avoiding race conditions in which threads modify data which other threads are still using)*
- → *synchronises threads at certain points in program execution*

***Optimisations:***
- → ***working in chunks:** instead of fetching one iteration at a time - a thread could fetch a chunk of iterations to work on - reducing overhead of frequent synchronisation for fetching new iterations*
- → ***avoiding unnecessary barriers:** placing barriers only where necessary can minimise the waiting time for threads*
- → ***exploiting cache affinity**: by scheduling iterations that access nearby data elements to the same thread - can take advantage of cache*

- - - 

***FetchAndAdd***

*FetchAndAdd:*
→ ***atomic operation to get the next un-executed loop iteration***
→ *fetches value from a memory location - increments the value - stores the result back to the same memory location*

***atomic operations:***
- → *executed entirely as a single step without possibility of interruption*

***lock-based implementation:***
- → *automatically increments the value at address pointed to by i & returns the old value*
- *→ lock/unlock mechanism used to ensure no two threads can modify i simultaneously (leading to race conditions)*
- → ***using locks can be expensive as they cause significant overhead & can lead to contention (other threads waiting for the lock to be released)*

***hardware-supported atomic operations:***
- → *many modern processors have built-in support for atomic operations that are more efficient that lock-based approaches*

***combining operations:***
- → *when scaling to large system even atomic operations can lead to contention as many threads may attempt to increment the same memory location simultaneously*
- → *can **combine FetchAndAdd operations in the network layer***
- → *when two FetchAndAdd messages meet - combined into one FetchAndAdd that returns two values (single operation)*
- → *requires support from network hardware connecting processors*

- - - -

***Distributed-Memory Parallel (MPI)***

*(alternative to shared memory parallel & OpenMP)*

***Message-Passing Interface (MPI):***
→ ***a standard API for parallel programming using message passing***
→ *defines the syntax & semantics of a core of library routines for writing portable message-passing programs on parallel computers*
→ *views a parallel program as a collection of processes that have their own local memory
→ processes communicate & synchronised by sending & receiving messages*

***Features:***
- ***point-to-point communication** → sending & receiving messages between pairs of processors (with functions like MPI_Send & MPI_Recv)
- ***collective communication** → operations involving groups of processes, such as broadcasting a message to all processes or gathering messages from all processes*
- ***synchronisation** → provides mechanisms for synchronising processes*

***operations:***

- → *uses **Single Program Multiple Data (SPMD)***

- - - 

***Single Program Multiple Data (SPMD)***

***SPMD:***
- → *parallel programming model*
- → *multiple processors executing the same program - operating on different sets of data*
- → *each processor must figure out which part of the task needs to perform*

***advantages:***
- ***highly scalable** → each processor operates independently - making SPMD suitable for multicore processors or large distributed memory supercomputers*
- ***flexible** → allows each processor to execute tasks independently*
- ***simplified programming** → as each processor executes the same program*

- - - 

***Stencils***

***Stencil***
→ *each element is updated using a weighted sum of neighbour values*

***example:***
- → *nested loop that performs a stencil operation*
- → *for each element in array B - computes the new value as the average of its four neighbours in array A*

***parallelisation:***
- → *to parallelise stencil operation - **divide array among multiple processes & each process updates a portion of the array***
- → ***each process needs data from neighbouring partitions** to compute values at the edge of the partition (dependence)*
- → *MPI requires explicit communication between processes - whereas OpenMP can use shared memory*

***using OpenMP (Shared Memory):***

- → *uses **directive** to **parallelise the execution of the nested loop***
- → ***private** ensures each thread has its own instance of the loop*
- → ***collapse** allows for the collapsing of two nested loops into a single parallel loop to increase granularity of work done by each thread*
- → *nested loop calculates new values for the array elements based on the neighbouring values - stencil operation*
- → *after the stencil operation another parallel loop used to copy the new values from B to A (needed to allow next iteration of stencil)*
- → *two directives needed as their is a **barrier synchronisation** between the two nested loops to ensure all updates to B completed before any values copied back to A*

- - - 

***Initialisation***

- → *MPI using **SPMD (Single Program Multiple Data)** parallel programming model*
- → *multiple processors executing same program on different data (figuring out which part of task they need to perform)*

***Initialising Partitions of Array of Data:***

- ***myrank →*** *a unique identifies of a processor in MPI*
- ***comm →*** *MPI communicator - a group of processes that can communicate with each other using MPI*
- ***MPI_COMM_SIZE*** → *MPI call determines how many processors in a group*
- ***MPI_COMM_RANK*** → *determines the rank of the calling process within the communicator (assigning unique identifier myrank to the calling process)*

- → ***computes size of the local block** of data the each processor will handle (partition of the array)*
- → *computes which other processors are **neighbouring***
- → ***allocates local arrays** that each processor will work on (block + one more element on either side that processor depends on)*

- - - 

***Jacobi2D***

→ ***example of stencil computations with MPI***

→ *sweeps over A computing the moving average of neighbouring four elements***
→ *computes new array B from A then copied back into B***
→ *tries to overlap communication with computation***
- → *do while loop iterates until a convergence on condition is met*
- → ***calculates the new values for the boundary elements** of B by taking the average of neighbouring elements in A*
- → *uses **non-blocking send & receive operations** to **exchange boundary data** between neighbouring processes*
- → ***computes new values for the interior elements** of B (not including the boundaries) - each element of B is updated to be the average of its four neighbours in A*
- → *new values in B care copied back to A - preparing A for next iteration*

- - - 

***OpenMP vs MPI (Shared vs Distributed Memory)***

***OpenMP:***
→ ***shared memory*

***easy to implement:***
- → *only need to add pragmas to code & code runs in parallel*

***limited scalability:***
- → *best suited to systems with a limited number of processors sharing memory (i.e. multicore processors)*

***hides communication:***
- → *programmer does not need to think about communication*
- → *programmer does not need to allocate storage for data to be received from neighbouring processors*

***unintended sharing:***
- → *can lead to difficult bugs (i.e. accidently reading data in shared memory that is being updated by another thread)*

***MPI:***
→ ***distributed memory*

***added complexity:***
- → *programmer must make data partitioning explicit*
- → *programmer must define communication*

***highly scalable:***
- → *highly scalable & can be used effectively on large distributed systems (including supercomputers)*

***may require more copying of data:***
- → *need to copy data between memory of different processors

***makes communication & synchronisation explicit:***
- → *allows programmer to schedule computation that avoids unnecessary overlaps (potentially increasing communication efficiency)*

- - - 

***Summary***

***why multicore?***
- → *limits of instruction level parallelism*
- → *limits of SIMD parallelism*
- → *parallelism at low clock rates is energy efficient*

***how to program a parallel machine?***
- → *explicitly managed threads*
- → *parallel loops*
- → *message passing (distributed memory)*

***communication & synchronisation between cores***
- → *needed in MPI*

- - - 
