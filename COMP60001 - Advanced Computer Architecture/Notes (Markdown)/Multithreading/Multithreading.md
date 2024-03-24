
- - - 

***Multithreading***

- - - 

***Multithreading***

***Instruction Issue:***
- *sequential machine*
- *can execute a single instruction per cycle*
- → *reduced functional unit utilisation due to dependencies & corresponding stalls*

***Superscalar Issue:***
- *dynamically scheduled machine*
- *can execute multiple instructions per cycle*
- → *more performance but lower functional unit utilisation (many lost issue opportunities)*

***Predicated Issue:***
- → *improve utilisation of superscalar by adding predication*
- → *less lost issue opportunities - but some results thrown away*

***Chip Multiprocessor:***
- *multiple processor cores on a chip*
- → *execute two threads in parallel on separate cores*
- → *limited utilisation when only one of threads is running*

***Fine Grained Multithreading (FGMT):***
- ***different threads in different cycles*
- *multiplex multiple threads in parallel on a single core*
- → *threads fetched in round-robin fashion*
- → *fetch up to 4 instructions per cycle*
- → *filling pipeline stalls*
- → *intra-thread dependencies still limit performance*
- → *if at a given time there is only one thread to run → must wait for that thread to be fetched*

***Simultaneous Multithreading (SMT):***
- ***dynamic scheduling of operations from a pool of threads*
- *in a dynamically scheduled machine*
- *maintaining a PC for each thread*
- *→ fetch instructions from multiple threads per cycle & pack instructions together for simultaneous issue/dispatch/execute/commit in parallel with each other*
- → *instructions can be dynamically scheduled from a common pool of instructions between all threads*
- → *if at a given time there is only one thread to run - it can run as fast as possible*

- - - 

***Simultaneous Multithreading (SMT)***

***basic out-of-order pipeline:***
*(dynamically scheduled processor)*

***SMT pipeline:***
- *same as out of order pipeline + dynamic scheduling of instruction fetches of 4 PCs in parallel*
- → *instruction cache contains allocations from different threads*
- → *issue-side register alias tables (register maps) needed for each thread (containing software-visible registers for that thread)*
- → *common RUU/queue*
- → *allocates physical registers dynamically to each threads registers from a common pool*
- → *dynamically executes independent of specific-thread*
- → *allocates into data cache independent of specific-thread*
- *→ writes to registers & retires independent of specific-thread*

***protection issue:***
- *→ must ensure different threads operating in different protection domains (i.e. different users) can only access their particular memory*
- → *thread id used to determine which TLB entries are valid for that particular memory address*

***resources:***
- → *different threads competing for computational resources (as we wanted to increase utilisation)*
- → *but also competing for I-Cache & D-Cache*
- → *working set of program x number of threads*
- → *want threads to be dissimilar such that they have less contentions with each other*

- - - 

***SMT Performance***

***alpha:***
- → *factor of 2 in performance (with 4 simultaneous threads)*

***pentium 4:***
- → *hyperthreading (running two parallel threads on a single core) gives little performance improvement*
- → *simultaneous multithreading gives significant performance improvement*

- - - 

***SMT - Example***

***Intel Atom:***
→ *in-order*
→ *2-way SMT*
→ *2 instructions per cycle (from same or different threads)*

***important consideration:***
→ *what is cost of SMT?*
→ *what is the cost of different threads?*
→ *where are the contentions between threads likely to be?*
→ *should hardware units be shared or individual?*

- - - 

***SMT Issues***

- ***execution time dominated by slowest thread in system*
- → *risk of individual threads running slow & impacting execution time*

- ***threads contend for resources*
- → *may be symbiotic (e.g. one thread is memory intensive & another is arithmetic intensive)*
- → *may be destructive (e.g. thrashes the cache → both fighting for limited cache/ capacity → or other shared resources)*

- ***resources should be partitioned per thread or shared on-demand*
- → *performance issues*
- → *correctness issues*
- *→ side channel issues*

- ***threads may be scheduled unfairly*
- → *one thread may monopolise the whole CPU & block progress of other threads*
- → *e.g. slow thread that suffers many cache misses & fills RUU - blocking issue of other threads*

- ***side-channels*
- → *one thread may be able to observe another’s traffic & deduce what it’s doing*

- - - 

***SMT - Memory***

- ***threads exploit memory-system parallelism*
- → *can get a lot of memory accesses in flight*
- → *many levels of memory hierarchy can be in use at once*
- → ***latency hiding*** → *overlapping data accesses with compute (data accesses of some threads overlap with compute of other threads)*

- ***no. threads limited by no. registers***
- → *threads need lots of registers*
- → *number of logical registers x number of threads*
- → *dynamically scheduled processor pays heavy cost for register renaming (register-by-register)*

- ***static partitioning*
- → *no. threads limited by no. registers*
- → *rather than dynamically register renaming (as done in dynamically scheduled processor) - statically partition register file based on the number of registers each thread actually needs*
- → *allows more threads to run on machine (maximum occupancy)*
- *occupancy → maximum number of threads chosen to run concurrently on processor (chosen as a tuning parameter)*
- → *done by many GPUs*
- → *tradeoff: lots of lightweight threads (to maximise latency hiding) vs fewer heavyweight threads (to benefit from many registers)*

- - - 

***Static Partitioning***

→ *mapping threads into the register file*
→ *logical-to-physical register mapping*
- *if each thread needs few registers (lightweight) - lots of threads can coexist in the same physical register file*
- - → *more threads = higher occupancy & better latency hiding*
- *alternatively - fewer heavyweight threads*
- → *benefit from many registers each*

- - - 







