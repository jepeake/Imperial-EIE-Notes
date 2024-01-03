
- - - 

***GPUs***

- - - 

***GPUs***

***CPUs →** latency-orientated systems that optimise making a single core run a single thread faster (using larger caches, branch prediction, speculation - to avoid stalls on data dependencies)*

***GPUs →** throughput-orientated systems that use massive parallelism to hide latency*

***workload consisting of thousands of threads:***
- → *never speculate (always another thread ready to execute)*
- → *no speculative branch execution (perhaps no branch prediction)*
- → *can use multithreading (FGMT or SMT) due to high parallelism to hide cache access latency (& perhaps memory latency)*

***control overhead:***
- → *launching 10,000s threads at once*
- → *taking branches in different directions*
- → *accessing random memory blocks*

- - - 

***Comparison with CPUs***

![[Pasted image 20231122224636.png|500]]

***GPUs:***
- → *simpler cores*
- → *much greater area devoted to **computation rather than control logic***
- → ***many functional units** (implementing the **SIMD** model)*
- → ***much less cache per core** (as thousands of thread with **super-fast context switching** that can **use fine-grained multithreading to hide cache access latency)***
- *→ **dropped sophisticated branch prediction** mechanisms*

- - - 

***Example (NVIDIA G80)***

![[Pasted image 20231122224914.png|600]]

***cores:***
- *16 processor cores (streaming multiprocessors)*
- → *each with computation resources, shared fast memory, & cache*
- → *packaged in pairs (TPCs)*

***memory:***
- → *high-speed interconnection network connecting processors to an array of memory interfaces
- → *offchip DRAM*
- → *L2 Cache*
- → *Raster Operation Processor (ROP)*

- - - 

***Texture/Processor Cluster (TPC)***

![[Pasted image 20231123103434.png|400]]

***SM:***
- → *streaming multiprocessor*
- → *two SMs share some resources in a TPC*

***SP:***
- *→ streaming processor
- → *8 per SM*
- → *execute 32-wide vector instructions*

***SMC:***
- → *streaming multiprocessor controller*

***MT Issue:***
- → *multithreaded instruction fetch & issue unit*
- → *fetching & issuing instructions from multiple threads according to multiple PCs*
- → *selects thread that is ready to go & schedules it according to some priority*

***C Cache:***
- → *constant read-only cache*

***I Cache:***
- → *instruction cache

***Geometry Controller:***
- → *directs all primitive & vertex attribute & topology flow in the TPC*

***SFU:***
- → *special function unit*
- → *computes transcendental functions (sin, cos, log, 1/x)*

***Shared Memory:***
- → *scratchpad memory (user-managed cache)*
- → *can explicitly transfer data into a local SRAM - compute with that data - & transfer out of that memory*
- → *shared by all threads present on SM*
- → *threads can cooperate to load data into shared memory & compute on that data*

***Texture Cache:***
- → *performs interpolation (estimation of intermediated values between discrete data points in graphics rendering)*
- → *indexed by a location in a texture*
- → *locality in 2D*
- → *accessed by points in space (i.e. floating point number)*

- - - 

***GPU Microarchitecture***

***thread: smallest unit of execution on a GPU (like a lane on a SIMD CPU)***
***warp: specific collection of threads that are executed simultaneously by GPU (like threads in CPU)***

***warps & threads:***
- → ***many fetch-execute devices** (16 SMs)
- → *each SM uses **fine-grain multithreading FGMT** to run 32 warps per SM*
- → ***high number of warps to tolerate high memory access latency** (can hide memory access latency of some warps by performing computation with other warps simultaneously)*
- → ***wraps run on SMs***
- → ***threads run on SPs***

- → ***MT Issue selects which warp to issue from in each cycle (FGMT)** (which warp has received data from memory & is ready)*
- → *each warp’s instructions are 32-wide SIMD instructions (**each warp contains 32 threads**)*
- → *instruction executed in four steps (using 8 SPs)*
- → ***each thread in a warp executes the same instruction in lock-step on different data***
- → ***SIMT (single instruction multiple thread) → each thread executes in a SIMD fashion + has its own instruction address & register state** (as divergence can occur if threads in a warp need to execute different instructions due to threads having to cover all branches of a conditional - leading to inefficiencies)*
- → *workload for each thread should be designed to **minimise divergence & maximise throughput** (not having to execute different branch paths)*
- → ***uses lanewise predication** (to avoid divergence)*

***memory:***
- → *each SM has local explicitly-programmed **shared scratchpad memory** (user-managed cache)*
- → ***different wraps on the same SM can share data** in shared memory*
- → *SMs have local L1 Data Cache (but no cache-coherency protocol)*
- → *machine is programmed by **launching 1000s of threads at once** (grouped into warps) & flushing L1 Data Cache (data is not exchanged between SMs during execution of an individual kernel & therefore no coherency protocol needed)*
- → ***multiple DRAM channels on-chip** - each including an L2 Cache (each data value can only be in one L2 location - no cache coherency issue)*

- - - 

***CUDA***

***CUDA (Compute Unified Device Architecture):***
- → ***using GPUs for general-purpose computation***
- → *needed to manage thousands of threads*

***CUDA Execution Model:***
![[Pasted image 20231123112157.png|300]]
- *CUDA → C Extension
- → *serial CPU code & parallel GPU code (kernels)*

***GPU Kernel:***
- → *each kernel is a C function (which can be run by 1000s of threads)*
- → ***each thread executes an instance of a kernel function on a single element of data**
- → *a group of threads form a **thread block** (larger structure containing several warps)*
- → *blocks are unit of allocation into SMs*
- → *thread blocks are organised into a **grid***
- → *threads within the **same thread block can synchronise execution & share access to local scratchpad memory***

***hierarchy of parallelism:***
- → *hierarchy of parallelism to handle thousands of threads*
- → *threads - warps - thread blocks - grids*
- → *thread blocks are **dynamically allocated to SMs & run to completion***
- → *threads within a block run on the same SM & can share data & synchronise*
- → *different blocks in a grid cannot interact with each other*

- - - 

***CUDA - Example*** 

***DAXPY:***
→ *parallel loop that performs scalar a * vector x + vector y*
![[Pasted image 20231123121218.png|500]]
***in CUDA:***
- → *split into CUDA Kernel + Host Code which launches threads*
- → ***Kernel Code → specifies code to be run by a single thread (each thread runs an instance of the kernel)***
- → *kernel specifics one iteration of the parallel loop*
- → *possible as overheads of managing threads is so low*
- → *thread first finds point in iteration space by reading registers set by thread launch mechanism*
- → *threads launched with kernel name, kernel parameters, & size of grid/size of blocks*
- → ***thread uses size of grid/size of blocks to locate to determine place in iteration space (i)***

***when compiled:***
→ *intermediate assembly code*
![[Pasted image 20231123121912.png|500]]
- → *i calculated from thread/block IDs*
- → *conditional stores in predicate register*
- → *if all bits of predicate register are true - branch to finish (if any bit set - execute body using predicate bits to select which threads are active)*
- → *x[i] & y[i] loaded*
- → *compute calculation & store*

- → ***32-wide SIMD vector instruction executes 32 instances of this kernel (for each thread) in parallel (one per lane)***

- - - 

***Running DAXPY on a GPU***

***scheduling:***
- → *work scheduled in **thread blocks***
- → ***each block contains some number of warps (groups of 32 threads)***
- → *trying to **fill machine with enough warps such that FGMT can fully occupy the machine & have enough memory accesses in-flight to hide the memory access latency***

![[Pasted image 20231123122516.png|500]]

- → *may break up blocks into smaller blocks to reduce load-balancing issue (work not distributed evenly among SMs)*

![[Pasted image 20231123123042.png|500]]

***inside each SM:***
- → *warps are dynamically scheduled*
- → *each warp is a 32-wide SIMD instruction*
- → ***each warp executes 32 CUDA threads (32 kernel instances) in SIMD lock-step** (executing same instruction of 32 data elements in parallel - one thread per lane)*
- → *each SM is shared by many warps (& many threads - possibly from the same or different thread blocks)*

***registers:***
- → *finite no. of registers are partitioned between warps*
- → ***partitioning of registers may limit number of warps that can be co-scheduled onto an SM***
- → ***large number of concurrent warps with small no. registers vs less warps with higher no. registers each***

***shared memory:***
- → *scratchpad memory (user-managed cache) **shared by all warps (& therefore all blocks)***
- → *shared memory **partitioned between blocks** (if multiple blocks - how much shared memory does each block need)
- → *partitioning can be configured*
- → ***imposes limit on number of blocks that can be allocated to an SM***

→ ***no. blocks that can be allocated into SM can be limited by size of register file / size of shared scratchpad memory***

- - - 

***SIMT (Single Instruction Multiple Thread)***

***SIMD (Single Instruction Multiple Data) →*** *single instruction operating on multiple data elements simultaneously (same operation applied across different data elements in parallel)*

***Multi-Threading →*** *executing multiple threads in parallel - where each thread can be executing different instructions & operating on different data*

***SIMT (Single Instruction Multiple Thread)***
→ ***multiple threads execute the same instruction at the same time on different data***
→ *like SIMD - but **each thread in SIMT can follow its own control path** (while all threads start executing the same instruction - they can diverge based on their specific data & control conditions)*
→ *allows more flexibility than traditional SIMD*

***program correctness:***
→ *programmers can ignore SIMT executions & program will still be correct - but can achieve performance improvements if threads in a warp don’t diverge*

***SM:***
- → *SM has a **SIMT multithreaded instruction unit** that creates, manages, schedules, & executes threads in groups of warps*
- → *each SM **manages a pool of warps***
- → *individual **threads** composing a SIMT warp **start together at the same program address (free to branch & execute independently)**
- → *at instruction issue time - select ready-to-run warp & issue next instruction to that warp’s active threads*

![[Pasted image 20231123123537.png|250]]

- → *each instruction operates on a **32-wide vector***
- → *each **thread** (element-wide slice of a vectors) **contains the state of a CUDA thread***
- → *when **successive instructions** from the same warp are executed - **progresses the execution of all threads in the warp***
- → *warps interleaved with other warps through FGMT (in a round-robin fashion) (e.g. to hide access latency of warp accessing memory)*
- → *the **FGMT** arrangement means that while one warp is executing - other warps are scheduled/executed → demonstrating how the **GPU maximises throughput by keeping as many threads active at any given time***
- → *SIMT shares SM Instruction Fetch & Issue unit across 32threads - but **requires full warp of active threads for fully performance efficiency***

- - - 

***Branch Divergence***

***threads can diverge in a warp:***
- → *a warp serially executes each path - disabling some of the threads (less efficient)*
- → *when all paths complete - threads reconverge (threads must be synchronised back to a common execution path)*

***divergence only occurs in a warp:***
- → *different wraps execute independently*

***control-flow coherence:***
- → *when all the threads in a warp go the same way - good utilisation (spatial branch locality)*
- → *to optimise program - **minimise divergence within threads in a warp***

***predicate bits can be used to handle divergent paths without branching:***
![[Pasted image 20231202214149.png|500]]

- - - 

***SIMD vs SIMT***

***SIMD:***
- → ***single instruction executed on multiple data***
- → *a single control unit dispatches the same instruction to multiple processing elements & they perform same operation on different pieces of data*

- → *exploits **spatial locality through adjacent loop iterations** across adjacent data*

- → *threads cannot follow their own control path - **divergence not possible***

- → ***SMT means a small number of threads run on the same core** to hide memory latency*

- → *load accesses **accesses adjacent locations in memory***

- → ***designed to process data in a tightly packed, orderly fashion - efficient for operations on large arrays where the data is stored contiguously in memory***


***SIMT:***
- → ***single instruction executed on multiple data using multiple threads***
- → *processor manages & executes many threads in parallel (each thread considered independent) - grouped into warps*
- → ***one thread per lane***

- → *exploits **spatial locality through adjacent threads** accessing adjacent data*

- → *each thread can follow own control path - **divergence possible***

- → ***SMT means multiple warps running on the same core** to hide memory latency*

- → *load can result in **different addresses accessed by each lane***

- → ***flexible & allows each thread within a warp to access different memory addresses - allowing for more data patterns***

- - - 

***Spatial Locality & Coalescing***

![[Pasted image 20231202221804.png|500]]

***SIMD:***
- → *vectorised the loop for SIMD execution (using directives or intrinsics)*
- → *directive (tells the compiler to generate vectorised code for the inner loop)
- → ***good spatial locality by accessed array elements in row-major order meaning data is accessed sequentially in memory***
- → *intrinsic (generates SIMD instructions)*
- → ***good spatial locality as processes chunks of array that are contiguous in memory*
- → ***good utilisation of cache***

***SIMT:***
- → *CUDA Kernel executes inner loop using multiple threads in parallel*
- → ***bad spatial locality as adjacent threads (with consecutive thread IDs) access different columns of the array (adjacent lanes) rather than contiguous elements (in the same column)***
- → ***poor utilisation of cache** (resulting in accesses to main memory which is inefficient)*

***Coalescing:***
- → ***ability of threads in a warp to access contiguous memory locations - allowing them to be combined into a single memory transaction***
- → *in SIMT - threads with adjacent IDs access data in different lanes (non-contiguous memory locations) & memory accesses cannot be coalesced*

- - - 

***Spatial Control Locality***

***SIMD:***
- → ***branch predictability***
- → *each individual branch is mostly taken or not taken (or well predicted by global history)*

***SIMT:***
- → ***branch coherence***
- → *adjacent threads in a warp all usually branch the same way (spatial locality for branches across threads)*
- → *if branches are coherent - good spatial locality*
- → *if branches not coherent - many wasted instruction opportunities*

- - - 

***Summary***

***CPUs vs GPUs***
- → *CPUs are latency-oriented - optimising single-thread performance*
- → *GPUs are throughput-oriented - with massive parallelism to hide latency*

***GPUs Architecture***
- → *simple cores with greater area for computation rather than control*
- → *many functional units implementing the SIMD model*
- → *less cache per core due to efficient multithreading hiding cache access latency*
- → *dropped sophisticated mechanisms like branch prediction*

***Thread Management***
- → *avoids speculation & speculative branch execution*
- → *uses FGMT or SMT to manage thousands of threads*
- → *implements SIMT to execute multiple threads with a single instruction*

***CUDA Execution Model***
- → *manages thousands of threads for general-purpose GPU computation*
- → *organises threads into hierarchy: threads - warps - thread blocks - grids*
- → *thread blocks are dynamically allocated to Streaming Multiprocessors & run to completion*

***Programming for GPUs***
- → *minimise thread divergence & maximise throughput for efficiency*
- → *leverage lanewise predication to avoid divergence*
- → *use shared memory effectively*

***Microarchitectural Details***
- → *SIMT allows a single instruction to operate across multiple threads*
- → *branch divergence occurs when threads in a warp take different execution paths - leading to inefficiency*
- → *spatial locality & coalescing are crucial for memory access efficiency - adjacent threads accessing contiguous memory is ideal*

***Optimisation Strategies***
- → *design workload to minimise divergence & maximise parallel execution*
- → *use shared memory & data coalescing techniques for efficient data handling & computation*

- - -
