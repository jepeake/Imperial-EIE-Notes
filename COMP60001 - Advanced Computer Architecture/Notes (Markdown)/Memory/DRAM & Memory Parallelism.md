

- - - 

***DRAM & Memory Parallelism***

- - - 

***DRAM Array Design***

![[Pasted image 20231110192244.png|400]]

***square array of cells holding data***
- *DRAM requires larger capacity than SRAM & uses a single transistor design with **data stored as a charge on a capacitor***
- ***transistor operates as a switch activated by the wordline** to allow charge on capacitor to travel down bitline to be sensed*

***address split into row address & column address bits***
- *row address bits select row of cells to be activate (wordline)*
- *cells discharge*
- *cell state latched by column sense amplifiers
- *column address bits select data for output (bitline)*

![[Pasted image 20231110192910.png|400]]

- ***when charge travels down bitline → sensed by sense amplifiers (which determine if enough current has flowed to determine a 1 or a 0)***
- → *definitive signal (current flow) of 0s and 1s is latched for all of the bits in the entire row*
- → *column address bits then select which bit of the row to select*
- → *bit routed to data out*

***two phases:***
- *provide row address bits & decode binary values from activated word lines*
- *when charge flows → use column address bits to select column (can change column address bits to access different elements within row)*

- *when charge is drained out of a capacitor → signal/stored data lost*
- → ***once signal sensed & latched → data must be written back to selected row** (reactivating transistor to recharge capacitor where a 1 is stored)*

![[Pasted image 20231110193506.png|500]]

- - - 

***DRAM Packaging***

***discrete packaging:***

![[Pasted image 20240103135852.png|300]]

- → ***DIMM (Dual Inline Memory Module)** contains multiple DRAM chips with clock/control/address signals connected in parallel (sometimes with buffers to drive signals to all chips)

***M1: (in-package integration)***

![[Pasted image 20230915160908.png|400]]

- *processor & DRAM much closer → **higher bandwidth, shorter wiring, custom interfaces***
- *cannot change the memory - **fixed memory***

- - - 

***DRAM Operation***

***Three Steps in Read/Write Access to a Given Bank:***

***Row Access (RAS):***
- → *decode row address & enable addressed row*
- → *bitlines share charge with storage cell*
- → *small change in voltage detected by sense amplifiers which latch whole row of bits*
- → *sense amplifiers drive bitlines full rail to recharge storage cells*

***Column Access (CAS):***
- → *decode column address to select small number of sense amplifier latches (4/8/16/32 depending on DRAM package)*
- → *on read - send latched bits out to chip pins*
- → *on write - change sense amplifier latches which then charge storage cells to required value*
- → *can perform multiple column accesses on same row without another row access (burst mode)*

***Precharge:***
- → *charges bit lines to known value - required before next row access*

- → ***each step has a latency of around 15-20ns in modern DRAMs***
- → *various DRAM standards (DDR/RDRAM) have different ways of encoding the signals for transmission to the DRAM - but all share the same core architecture*
- *→ DRAM latency not decreasing with new generations → optimisations focussing on higher capacity*

- - - 

***DRAM Refresh***

![[Pasted image 20231110193643.png|400]]

***Refresh:***
- *after a while the **charge leaks → & data cannot be reliably read***
- → ***must refresh every cell periodically (e.g. every 64ms)***
- → *usually managed by a microcontroller on the DRAM chip (do not want to use CPU directly - as that is occupied with other execution)*
- → *refresh a few word lines at a time in a cyclic fashion*

- → *DRAM unavailable during refresh - every few microseconds & transaction may be delayed*
- → *refresh may be triggered more frequently if device is hot*
- → ***refresh is a significant energy cost***

- - - 

***DRAM Timing Characteristics***

- *once a row has been selected → **the whole row is latched & copied into a row buffer***
- → ***different elements of the row (i.e. from different columns) can be accessed with lower latency** - as readily available in the buffer*

- $t_{RAC}$ - *row access time - time taken to access a particular row in DRAM*
- $t_{RC}$ - *row cycle time - minimum time before which a new row can be accessed*
- $t_{CAC}$ - *column access time - time taken to access a particular column within already selected row*
- $t_{PC}$ - *precharge time - minimum time between access to columns in the DRAM* 

![[Pasted image 20231110194130.png|500]]

- → ***row access cycle time longer than row access time***
- → *data must be written back after it is read  (as when charge is drained out of a capacitor → signal/stored data lost)*

- - - 

***DRAM Chip 

***architecture of DRAM Chip:***

![[Pasted image 20231110194232.png|500]]

- *→ **several DRAM arrays (banks)** → each with a sense amplifier array*
- → *data latch for each DRAM array*
- → *decoder for each DRAM array to access different rows of different arrays in parallel*
- → *once data latched - can use column addresses (column decoder for each array) to get data out
- → *DRAM microcontroller controlling DRAM array → including automatic refreshing*

- - - 

***Error Correction***

***failures:***
- *failures over time → proportional to the number of bits*
- *→ as DRAM cells shrink physically in size → become more vulnerable*
- → *interference from neighbouring cells*
- → *radiation (such as high energy cosmic rays/ single-event upsets/bit-flips)*

***solution:***
- ***add redundancy through parity/ECC bits (+ additional bits to indicate if even number or odd number of 1s → can be checked when data read)***
- *could detect error + throw exception → want a more sophisticated code*

- ***common configuration: random error correction***
- → *SEC-DED (single error correct - double error) → Hamming Code*
- → *e.g. 64 data bits + 8 parity bits*
- → *costs substantial space overhead (additional DRAM needed) & required hardware to check if correct (+ need to log corrections & handle failures)*

- *want to handle failures of whole chips - not just single bits*
- → ***RAID (Redundant Array of Independent Disks)***
- → *storing the same data in different places on multiple hard disks or solid-state drives (SSDs) to protect data in the case of a drive failure*

- - - 

***RowHammer***

- → ***changing state of DRAM memory cells by repeatedly accessing (hammering) adjacent rows of memory***
- → ***when a specific memory row is accessed repeatedly & rapidly - can cause bit flips in adjacent rows of memory cells due to the electromagnetic interference

![[Pasted image 20231110202003.png|250]]

***attack:***
- *consider a program that **repeatedly writes to the yellow rows** (hammering the yellow rows → row hammer attack)*
- ***bits in purple row may be interfered with** (bit flip in purple row)*

- → *when a row is accessed repeatedly - it can **inadvertently cause charge leakages in adjacent rows due to electromagnetic interference - leading to bit flips***
- → *want to hammer yellow rows to change purple row **before a refresh occurs** (closer to refresh - row is more vulnerable - as signals began to decay)*

***testing:***
- → *this is how DRAMs are commonly tested*
- → *common DRAMs can be flipped despite passing manufacturing tests*

***vulnerability:***
- → ***can be exploited as a security vulnerability to gain unauthorised access to data***
- *may enable a program to change data that belongs to another process or OS*
- *& gain access to private data*

***mitigations:***
- ***ECC** (provide some protection)*
- ***adaptive refresh** (target row refresh, TRR) → count accesses & refresh potential victim rows early (if yellow rows are being accessed frequently → refresh purple row early)*
- *both ECC + TRR may be overcome*

- - - 

***Further Topics***

- *DRAM Energy Optimisations*
- *Stacking*
- *Processing in Memory, Near Memory Processing*
- *Bulk Copy & Zeroing*
- *Non Volatile Memory*

- - - 

***Main Memory Organisations***

![[Pasted image 20240103145759.png|500]]

***simple:***
- *CPU, Cache, Bus, & Memory - **same width (32 or 64 bits)***

***wide**:*
- ***transfer large cache lines at once***
- *memory array consists of **several DRAM arrays/banks → but all indexed with the same address***
- ***parallel data transfer & simultaneous activation of multiple DRAM arrays***
- *provides **greater BW** but does not give flexibility*

***interleaved:***
- ***parallel data transfer & parallel addressing***
- → *can index **multiple banks with different addresses***
- → *activating multiple DRAM arrays each accessing different data locations*
- → ***more flexibility***
- → *need to **spread accesses across banks** (e.g. using low order address bits - such that array accesses successive different banks)*
- *problem: can accidently write a program that only accesses one of the banks (→ can introduce a more complex mapping/hash function)*

- - - 

***Summary***

***DRAM:***
- ***DRAM arrays have separate row address latency***
- *DRAM reads are **destructive → needs to be written back***
- *DRAM needs **periodic refresh***

***Memory Parallelism:***
- *DRAM packages typically have multiple DRAM arrays → **can be row accessed in parallel***
- ***Wider Memory** → **increases transfer BW** to transfer up to a whole row in a single transaction*
- ***Interleaved Memory** → **allowing multiple addresses** to be accessed in parallel*

- ***Bank Conflicts** → occur when two different rows in the same DRAM array are required at the same time*

- *Need **ECCs** which cost space and time*

- - - 
