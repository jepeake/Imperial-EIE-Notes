
- - - 

***Block Placement & Write Policies***

- - - 

***4 Questions for Memory Hierarchy*

- *Where can a block be placed in the cache? (**Block Placement**)*
- *How is a block found if it is in the cache? (**Block Identification**)*
- *Which block should be replaced on a miss? (**Block Replacement**)*
- *What happens on a write? (**Write Strategy**)*

- - - 

***Block Placement*

***where can a block be placed in the cache?***

![[Pasted image 20231117173010.png|400]]

***increasing associativity:***
→ *more comparators & larger energy cost*
→ *better hit rate (but diminishing returns)*
→ *reduced storage layout sensitivity (more predictable → less likely that two addresses which map to the same cache block are overwriting each other as in direct-mapped)*

- - - 

***Block Identification***

***how is a block found if it is in the cache?***

![[Pasted image 20231117173347.png|500]]

***tag***
- *tag associated with data - used to identify block*
- *tag is the high bits of the block address, that does not include the index or block offset*

***increasing associativity***
- *as associativity is increased - if total cache size stays the same - increases the number of blocks per set → fewer index bits needed→ smaller tags

***reduce overhead of tag checking***
- *to reduce overhead of tag checking (identifying blocks) → use larger cache blocks → tag memory overhead per cached item reduces*

- - - 

***Block Replacement***

***which block should be replaced on a miss?***

*with direct-mapped → only one choice*

*with set/fully associative → choice:*
→ *ideally: least-soon to be re-used*
→ *LRU (least recently used)*
→ *random*

- *want to remove item from cache that is least likely to be used in the future*
- *LRU: assumes future will be similar to past, hard to implement, need to track which ways most recently accessed (commonly used, expensive to implement)*
- *random eviction can also be used → easy hardware but will lose some temporal locality*

- → *LRU only beats random for small caches*

- *for example: a program that sweeps over an array, but does not quite fit all of array into cache, would be bad for LRU, as LRU data is data you want to access soon once you hit end of array*

- *also FIFO (aka. Round Robin) → used in highly associative caches*
- *Not Most Recently Used (NMRU) → FIFO with exception for the most recently used block or blocks*

- *to decrease memory access time → first want to increase the hit rate (for fast accesses), then when we have a miss, we want to use the most effective replacement policy → second-order effect*

- - - 

***Write Strategy***

***what happens on write?***

- *when performing a write/load instruction - need to check cache to see if there is a cached copy of that data*
- *if cached copy is present, must be updated - should we also update the next level of memory hierarchy, main memory?*

***on cache hit***
- → ***write-through (WT) or write-back (WT) strategies***

***write-through*** 
- → *information written to both the block in the cache & the block in lower-level memory*
- → *always combined with write buffers so CPU does not have to wait for lower-level memory*

***write-back*** 
- → *information written only to block in cache*
- → *modified cache block is only written to main memory at some time later, when the cache needs to evict a block of data from the cache*
- → *write-back has a clean/dirty bit associated with each block, used to track if the data in the cache block has been modified since it was loaded from main memory (dirty) or whether it matches the corresponding data in main memory (clean)*
- → *if you displace a cache block that is dirty, must be written to main memory*

***absorbs repeated writes***
- *WB absorbs repeated writes & gives big energy advantage - less often needs to access lower levels of memory hierarchy*

***performance issue***
- *WB may lead to performance issue as WBs may block writing to higher levels of memory hierarchy - unless a WB buffer used*

***writing into an array***
- *for WB, if we writing into an array, can write all values into cache without having to propagate each time to the main memory (absorbing the repeated writes)*
- *WT - main memory data updates more often*
- *but, with WT, other processors may have cached copies of their own, which will be used, even though main memory has been updated, need to have coherent caches*

***example***
- *at the point of a store instruction (write), data is loaded into the cache (and if it’s a WB policy, the data is not also loading into main memory) so the data in cache no longer matches the (old) data in main memory - so the data in that cache block has a dirty bit associated with it*
- *instead of immediately writing the modified data back to the main memory, the cache holds onto the data until it’s absolutely necessary to update the main memory*
- *you can keep writing to, and reading from that cache block, to access the data of the associated tag, without having to go to main memory*
- *at the point you need to replace that block, i.e. due to a cache miss on that index (trying to read a value from that index but that tag is not in the cache, so goes to main memory to fetch that block and put it into the cache) → you need to update the main memory with the value for that block that is being evicted, otherwise the data will be lost*

- *this delay can lead to increased cache performance in cache as multiple writes to the same location can be aggregated into a single write into main memory
- *this optimization reduces memory bus traffic and minimizes the performance impact of frequent write operations*

***WT vs WB:***

*WT Advantages:*

- ***reduced complexity** → every write operation immediately updates main memory - reducing complexity*
- ***consistency →** guarantees data consistency between cache and main memory (but not consistency between caches)*
- ***simplicity →*** *can lead to easier cache coherence management*

*WT Disadvantages:*

- ***higher memory bus traffic →*** *due to immediate writes to main memory, can lead to contention and slower performance*
- ***higher write latency →*** *even though writes to cache and main memory are simultaneous, there is a slight latency overhead as the CPU must wait for the write to be acknowledged by both the cache and the main memory before proceeding with the next operation*
- ***inefficiency →*** *for write-heavy workloads, constant updates to main memory are inefficient*

*WB Advantages:*
- ***reduced memory bus traffic:*** *instead of generating memory traffic every time we write to memory, consolidate all writes in cache, until we are done with updating values, and then write to memory*
- ***lower write latency →*** *lower latency for write operations as data is initially written to the cache and is only later written to main memory*
- ***aggregates writes →*** *multiple writes to the same memory location can be aggregated into a single write to main memory, optimising bus usage*
- ***efficiency*** → *can lead to more efficient system & better overall performance, due to reduced memory bus contention and faster write acknowledgement*

*WB Disadvantages:*
- ***increased complexity →*** *makes cache design more complex, for each load/store instruction may have 0, 1 or 2 accesses to memory (0→hit,data is in the cache already, 1→miss,fetch data from memory, and put into cache, 2→miss & dirty bit, evict data from cache to main memory, fetch data from memory and put into cache) & managing when to write back to main memory adds complexity to cache management*
- ***cache coherency issues →*** *required more complex cache coherence protocols in multi-processor systems to ensure data consistency across caches*
- ***data integrity risks→*** *risk of data loss in the event of a system crash as most recent updates are only stored in the cache*

***on cache miss:***
- → ***write-allocate or no-write allocate***

- ***write-allocate**: fetch into cache*
- ***no-write allocate**: only write to main memory*

*common combinations:*
- *write th