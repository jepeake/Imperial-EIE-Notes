
- - - 

***Cache Performance***

- - - 

***CPU Cache Interaction***

- *add data cache & instruction cache to 5-stage pipeline

![[Pasted image 20230915202202.png|400]]

***cache hit***
- *if cache hit, use cache memory as previously done in 5-stage pipeline*

***cache miss***
- *on cache miss → refill cache data from lower levels of the memory hierarchy
- → *stall CPU as need to wait for data to return from memory - as longer latency to access memory hierarchy & still doing in-order execution so cannot keep executing instructions while we wait*
- *→ still execute other instructions already in the pipeline (drain the pipeline)*

- - - 

***Improving Cache Performance***

- *cache performance measured by AMAT (Average Memory Access Time)*

***AMAT = Hit Time + Miss Rate x Miss Penalty***

*to improve performance:*
- *reduce hit time*
- *reduce miss rate*
- *reduce miss penalty*

- *L1 Cache is typically 32KB - as we want to maintain the short hit time/latency*
- *balance of cache size (for good hit rate) vs hit time*

- - - 

***Causes of Cache Misses: The 3 C’s***

***Compulsory →*** *first reference to a line (aka. cold start misses)*
→ *misses that would occur even with an infinite cache*

***Capacity →*** *cache too small to hold all data needed by program (working set)*
→ *misses that would occur even with the perfect replacement policy*

***Conflict →*** *misses that occur due to collisions due to line-placement strategy*
→ *misses that would not occur with full associativity*

- *capacity → common → cache added as an intermediate layer to hide some of the memory latency - but cache cannot hold all of the data that a program needs - will need to go to main memory at some point and miss cache*

- *conflict → multiple memory blocks map to the same cache tag - when the set with that tag is full - need to evict a way when we have a cache miss for data with that tag (limited associativity, would not occur with full associativity as no two memory blocks would have the same tag)*

******

***Effects of Parameters on Performance***

***Larger Cache Size***
- → ***hit time increases** (physical distance to data in cache increases)*
- → *reduces capacity & conflict misses (conflict misses reduce as you have more blocks - so more unique tags - and less memory locations map to the same tag*)
- → ***miss rate decreases***
- → ***miss penalty stays the same***

***Higher Associativity***
- → ***hit time increases** (more comparators to check for hits)*
- → *reduces conflict misses (more places to store data with the same tag → more blocks in a set)*
- → ***miss rate decreases***
- → ***miss penalty stays the same***

***Larger Block Size***
- ***hit time increases** (slightly longer to read larger blocks of data)
- → *reduces compulsory misses (more data loaded into cache at once → filling up the cache faster)*
- → *increases conflict misses & miss penalty (increases conflict misses due to fewer sets in cache & fewer tags  - more opportunities for conflicts - higher miss penalty as more data to be reloaded on miss)*
- → ***miss rate may increase***
- → ***miss penalty increases***

- - - 

***Miss Rate***

- *size of cache vs miss rate*

![[Pasted image 20230915205527.png|600]]

- *very small compulsory misses*
- *when cache size is small → mostly capacity misses (when cache large enough to fit entire locality of program - less capacity misses)* 
- *biggest reduction in misses direct-mapped cache → two way set associative cache*

- - - 

***Block Size & Spatial Locality***

→ ***can increase block size to exploit spatial locality***
- *e.g. a 4 Word Block (copies 4 Words from Memory at at time)*

![[Pasted image 20231117172829.png|600]]

***less tag overhead***
- *less tag overhead → bigger offsets - more bits used to index which word inside the block - less bits for the tag (block address)*

***burst transfers***
- *burst transfers → transferring multiple words at once*

***disadvantages***
- *disadvantages: higher miss penalty - more conflict misses (fewer number of blocks)*

- - - 

***Miss Rate vs Block Size

- *for different cache capacities*

![[Pasted image 20230915212728.png|500]]

- *at first - bigger block size → lower miss rate (due to spatial locality) → beyond 64 bytes - get higher miss rate for smaller caches (block/block size too big relative to cache size)*
- *blocks more flat for bigger caches*
- *64bytes block/block size commonly used (**spatial locality vs conflict misses/miss penalty balance**)*

- - - 
