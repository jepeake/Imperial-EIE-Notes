
- - - 

***Synchronisation & Memory Models***

- - - 

***Parallel Programming Overview***

→ *power considerations & multicore***

→ *shared-memory parallel programming*
	→ *dynamic load-balancing*
	→ *distributed-memory parallel programming (why harder to program but more robust performance)*

→ *cache coherency***
	*→ design space of snooping protocols based on broadcasting invalidations & requests

→ ***atomic operations & locks***

→ ***sequential consistency***

- - - 

***Synchronisation***

***synchronisation:***
→ ***coordination of concurrent processes to ensure correct execution***
- → *in multi-process systems - different processes may run in parallel & access/modify shared resources*
- → *without synchronisation - leads to race conditions where outcome depends on order of execution*

***race condition:***
- → *situation in which behaviour of system depends on order of execution of multiple processes executing in parallel*
- → *multiple processes are accessing shared resources & which process is executed first may vary from one execution of the program to another*
- → *correctness of the program can vary unpredictably*

***locks/mutexes:***
- → *used to ensure that only one thread can access a resource at a time*
- → *when a thread acquires a lock - no other thread can access the locked section until the lock is released

***semaphore:***
- → *a generalised lock*
- → *maintains a count & a thread can decrease or increase the count*
- → *used to control access to a resource pool or to synchronise activities*

***barrier synchronisation:***
→ *used to make sure the multiple threads reach a certain point in the execution before any of them proceed*

***atomic operations:***
- → *operation that is performed as a single step that cannot be interrupted or observed in an incomplete state*
- → *used to manipulate shared data without a lock - avoiding the overhead & potential deadlocks in lock-based synchronisation*
- → *can be used to fetch & update memory*

***to avoid synchronisation bottlenecks:***
-  *fast non-contended path → when there is no competition for resources - synchronisation should still perform efficiently*
- *efficiency in high contention → when many processes are competing for the same resource - synchronisation should still perform efficiently*
- *fairness → all processes should have equal opportunity to access shared resources - preventing scenarios where some threads are starved of resources while others monopolise them*

- - - 

***Atomic Operations***

***Test-And-Set:***
- → *atomic operation that tests a value (typically a synchronisation variable like a lock) & sets it if a certain condition is met (typically if the lock is free)*
- → *if the value is 0 (indicating the lock is available) - it sets the value to 1 (indicating the lock is now taken) & returns the original value*
- → *used to implement mutexes - where a thread can acquire a lock before entering a critical section*

***Fetch-And-Increment:***
- → *atomic operation that reads the current value of a memory location & then increments that value atomically - meaning no other operation can interrupt or be interleaved during this process*
- → *the original value before incrementing is returned*

***Atomic Exchange:***
- → *atomically swaps a value between a register & a memory location*
- → *use to implement synchronisation mechanisms*
- → *i.e. a thread wanting to acquire a lock would use atomic exchange to swap a 1 into the memory location of the lock variable & if the previous value was 0 - the lock was acquired successfully*

→ ***all operations consists of a load & store in a single operation***

- - - 

***Atomics in GPUs***

- → *GPUs typically have no cache coherency protocol for the L1 Caches*
- → *atomic operations on global memory have to be handled on **L2 Cache Controllers***

- - - 

***Atomics in a Cache Coherent Multicore***

*need to perform uninterrupted load & store*
→ ***use two instructions*

***Load Linked + Store Conditional:***
- ***load linked** → reads a value from memory & links it to a store conditional operation - returning the initial value of the memory location*
- ***store conditional** → attempts to write a value to the memory location that was previously read by a load-linked instruction - returning 1 if succeeds & 0 if otherwise*
- → *store succeeds only if no other writes to the memory location occurred (if no invalidation received for that address) between the load-linked & store-conditional instructions **(check using the linked address)** 
- → *ensures atomic operation*

***example:***
![[Pasted image 20231204000251.png|500]]

***fetch & increment using load-linked & store-conditional:***
![[Pasted image 20231204000528.png|500]]
- → *loads the value - increments it - & attempts to store the new value conditionally*
- → *if another process has written to the location - the increment succeeds*

→ ***cannot put memory access instructions between load-linked & store-conditional*

- - - 

***User Level Synchronisation Operations***
*using Atomic Exchange*

***Spin Locks:***
- → ***synchronisation mechanism where a processor continuously checks (spins) to see if a lock is available*
- → *the lock is attempted using an atomic exchange that tries to set the lock to a non-zero value (indicating the lock is taken)*
- → *if the exchange is successful (the previous value was 0) the lock is acquired*
- → *if not - the processor continues to spin & checks the lock variable repeatedly*
![[Pasted image 20231204001049.png|500]]

***Spin Locks in Multicore:***
- → *in a **multicore processor with cache coherences** - core spins on a local cached copy of the lock variable to minimise bus traffic & avoid overloading the memory with continuous read requests*
- ***problem** → atomic exchange operation includes a write (which invalidates other cached copies of the lock variable in a cache-coherent system - leading to significant bus traffic)*
- ***solution (test & test & set) →*** *rather than continuously attempting the exchange operation - processor first reads the lock variable (test) & then if the lock appears to be free attempts the atomic exchange (test and set)*
![[Pasted image 20231204001402.png|500]]

***if a spin lock is released with many cores spinning on it:***
- → *all cores spinning on the lock need to be informed & leads to significant bus traffic*
- → *the core that wins the lock is typically the first one to successfully execute when the lock is released - which is non-deterministic*

- - - 

***Ticket Locks***

***ticket lock:***
→ *used to ensure fairness among processes trying to acquire a lock*

***example:***
![[Pasted image 20231204215321.png|300]]
- → *ticket lock initialised setting both next_ticket & now_serving to 0 (no tickets issued yet & service at beginning)*
- ***ticketLock_acquire** → called by a thread that wants to acquire the lock - the fetch_and_inc function atomically increments the next ticket variable & returns its old value - this ensures each thread gets a unique ticket number*
- → *each thread then enters a spin loop - repeatedly checking whether its ticket number matches the now_serving variable*
- → *loop continues until thread’s turn to be served*
- - ***ticketLock_release** → when a thread is finished with the shared resource - it calls this function to release the lock - now_serving variable is incremented & allows the next thread in line to exit the spin loop in ticketLock_acquire to gain access to the shared resource*

- → ***ticket locks explicitly hand off access to the next in line*
- → ***ensures fairness as strictly follows the order in which threads requested access*

- - - 

***Lock Behaviour with High Core Counts***

***Scalable Lock:***
→ *maintains a constant number of cache misses per lock acquisition - regardless of the number of cores*
→ ***non-scalable locks** can lead to performance degradation as the number of cores increases due to increased contention & cache coherence traffic*

***Waiters***
- → *scalable locks have a queue of processes waiting to acquire the lock*
- → ***each waiter spins on its own queue entry** - providing more efficiency that all waiters spinning on the same lock variable by reducing cache coherence traffic*

- - - 

***Memory Consistency Models***

***Memory Consistency Models:***
- → ***define the order in which memory operations (reads & writes) appear to be executed in the processor*
- → *provides a contract between the hardware & software allowing developers to understand how memory operations will behave*

***Sequential Consistency:***
- → *a strict model that **requires the memory operations to appear as if they were executed in some sequential order*
- → *the operations of each individual processor appear in this sequence in the order specified by its program*

***SC for Synchronisation:***
- → *optimisation that may **delay memory accesses until all invalidations are done*
- → *less strict that full sequential consistency as can reorder some memory operations - but ensures that synchronisation between processors is maintained*

***Weak Consistency:***
- → *model that can **permit reads & writes to be executed out of order under certain conditions** (unlike strict ordering of sequential consistency)*
- → *increased flexibility of memory operations can improve speed*
- → *several model variants characterised by attitude towards RAR, WAR, RAW, WAR*
- → *model dictates how operations can be reordered & if they must be synchronised*

***Fence Instructions:***
- → *fence (memory barrier) instructions are special instructions that can **enforce sequential consistency at certain points*
- → *act as synchronisation points that ensure all memory operations issued before the fence are completed before any issues after the fence*
- → *expensive in terms of performance as they limit the optimisations the processor can perform*

***Synchronised Programs:***
- → *most programs do not have issues with weak consistency models as long as they are **explicitly synchronised*
- → *a program is synchronised if all accesses to shared data are ordered by explicit synchronisation operations (using locks, semaphores…)*
![[Pasted image 20231204222253.png|400]]

***Nondeterministic Programs:***
- → *programs that are willing to be **nondeterministic (unpredictable behaviour) are not synchronised*
- → *nondeterministic programs will have data races*
- → *data races occur when two threads access the same variable concurrently & at least one of the accesses is a write*

- - - 

***Summary***

- ***shared memory parallel programs must synchronise*

- *synchronisation primitives can be executed together*
	- → *at the memory (in GPUs)*
	- → ***or in the CPU (leads to issues with cache coherency traffic when spinning & when a contented lock is released)

- *while older ISAs offer test&set, compare-and-swap, & atomic exchange synchronisation primitives → these are hard to implement

- ***load-liked, store-conditional** provides a solution that is easy to implement on a cache coherent CPU*
	- → *operation only succeeds if no invalidation occurs in-between*

- ***test-and-test-and-set** reduces contention for cache line ownership when spinning*

- ***ticket locks** provide fairness*

- ***scalable locks** limit coherency traffic on lock release*

- ***weak memory consistency models** mean processes cannot reliably observe ordering of remote events unless explicit synchronisation takes place*

- - - 




