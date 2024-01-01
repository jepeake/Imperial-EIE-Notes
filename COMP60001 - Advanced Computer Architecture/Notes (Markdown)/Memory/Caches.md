
- - - 

***Caches***

- - - 

***Intel Skylake Quad-Core Die***

***example:***

- *GPU*
- *System Agent*
- *4 CPU Cores connected with Ring Interconnect*
- *L3 Cache between cores (highest level)*
- *L2 Cache for each core*
- *L1 Data & L1 Instruction Cache embedded deeply in processor*

- - -

***Why are Caches Needed?***

- ***→ DRAM not providing fast access times for modern CPUs***
- *→ processor-memory performance gap increasing (clock rate vs access time of DRAM)*
- *→ caches needed*

- - - 

***Cache Algorithm***

- *for data cache → **on store/load → look at address & search cache tags to find match***
- *if match → cache **hit → return copy of data from cache***
- *if not a match → cache **miss → read block of data from main memory & return data to processor (updating cache)***

- - - 

***Types of Cache***
	
- *different style of cache designs (mappings)*

<br>

- ***direct mapped** → one-to-one mapping between data in memory and data in the cache, unique location in cache for each memory address*
- ***set-associative** → for a particular memory location, multiple places in cache can store the data (one location in each way)*
- ***fully associative** → one memory block can be mapped to any cache block*

- - - 

***Terminology***

***cache block (line)***:
- *the smallest portion of data that can be stored in the cache* 
- *(corresponds to a fixed-size portion of memory that is loaded from the main memory into the cache when needed)* 
- *e.g. 32byte block*
- → ***determines the granularity of caching (larger blocks reduce overhead of fetching data from main memory due to spatial locality but also increase likelihood of replacing useful data & increasing miss rates)***

<br>

***cache set:*** 
- *a group of cache blocks within a cache (in set-associative)*
- *in fully associative cache → one set contains all of the blocks*
- *in set associative cache → each set contains are few blocks which are indexed by the same tag*

<br>

***ways:***
- *ways represent the number of cache blocks within a single cache set*
- *(2-way set associative cache → each cache set has 2 blocks → cache is split into 2 ways)*

<br>

***cache level:*** *a specific cache within a multi-level cache hierarchy (L1, L2, L3)*

- - - 

***Direct Mapped Cache***

- *32x32 = 1024 (1KB) - 32, 32 Byte Blocks in Cache*

- *in direct-mapped cache, take the address that the processor is trying to access, and take the first 5 bits to indicate which byte we want to select, the second 5 bits to index which row/block we want to select from, and the remaining bits to determine the tag (+ 1 valid bit)*

- ***tag** - used to determine if the data in that block is the data the we want - metadata that tells us if we have a cache hit or not*
- *once data is written into the cache, valid bit changed to 1, to indicate that data is valid*
- *when CPU is turned on, no data in cache blocks, and valid bits are 0*

- *when data allocated into cache, data put into 32 byte block, and bits 31:9 of address put into tag section - to be checked upon next access*

<br>

*yields some additional hardware:*

- *cache tag must be compared to the address bits 31:9 of the requested address, and hit given valid bit*
- *first 5 bits passed to determine byte selection*

- - - 

***Associativity Conflicts in Direct-Mapped***

- *5 bit cache index - only 32 blocks mapped to whole main memory address space*
- *overlap - repeats every 32 addresses*
- *location 0 and 32 map to the same cache location. 1 and 33 map to the same cache location*

<br>

***example:***
- *need 512B - 1KB cache should be enough to hold all array values*
- *arrays A & B would be allocated in adjacent memory by compiler*
- *program repeatedly re-reads both A & B*

- *memory has 256 words of A. followed by 256 words of B*
- *with direct-mapped cache, 256 is a multiple of 32, so, location A+0 has identical address bits (5 bits) with B+0, and so on*
- *so map to the same cache block & repeatedly overwrite each other*
- *only allocating to a subset of cache, and leaving large part of cache unused*
- *in fact, due to overwriting, cache not used at all*

- - - 

  ***Two-Way Set Associative Cache***

- *similar to direct-mapped cache, but in parallel, in two-halves*
- *use same index bits to access into two direct-mapped cache structures*
- *two tags to compare with - data may be in either halve*
- *check & use multiplexor to take which cache block contains a cache hit*

- *each cache structure is called a **way***

- - - 

***Disadvantage of Set Associative Cache***

- *extra hardware on the critical path - slowing down (given only have 5-8 gate delays)*
- *may only discover you have a hit - after data becomes available*
- ***need more comparators - so more hardware/transistors (higher energy cost)***

- - -

***Intel Pentium 4 Level-1 Cache***

*example:*

- *4 ways: 4 parallel cache structures*

- - - 

***Fully Associative Cache***

- *access all of the cache blocks at the same time & do a tag comparison for every block in the cache*
- → ***a lot of extra hardware (complex & expensive to implement) & higher energy cost***
- → ***increased latency due to time taken searching through all entries for a match***

- - - 

