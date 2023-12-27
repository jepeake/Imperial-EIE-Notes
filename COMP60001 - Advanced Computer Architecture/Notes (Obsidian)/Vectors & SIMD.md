
- - - 

***Vectors & SIMD***

- - - 

***Vectors***

<br>

***Vector***

→ ***sequence of data elements of the same type that are processed by a single instruction***

<br>

- → *requires **vector instruction sets***
- → *can use **automatic vectorisation** (compiler selects & uses vector instructions)*
- → *can use **lane-wise prediction** (vectorising conditionals)*

<br>

- - - 

***SIMD***

<br>

***SIMD (Single Instruction Multiple Data)***
- → ***execution of a single instruction applied to multiple data elements simultaneously in parallel***
- → *each processor unit that performs the operation works on a different data element*
- → *form a vectorisation in which a single operation is applied to a vector of data*
- → *uses registers & instructions that hold/execute whole vectors of operands at once*

<br>

***Advantages:***
- → ***reduces Turing Tax** (overhead of general-purpose machine) by reducing fetch-execute overhead (many data elements executed on for one fetch)*
- → ***increases instruction-level parallelism (reducing instruction count & improving data throughput)***
- → *significantly **speeds up computational tasks that are parallelisable***

<br>

- - - 

***Vector Instruction Set Extensions***

<br>

***AVX512 Vector Instruction Set:***
- → ***extends the scalar processor with vector registers** (each 512bits wide)*
- → *extended registers used to store 8 doubles/16 floats/32 shorts/64 bytes (vector instructions for all operands)*
- → ***instructions executed in parallel** in 64/32/16/8 lanes*

<br>

- → *contains **predicate registers***
- → *each predicate register holds a predicate per operand (**per lane**)*
- → *k predicate registers hold up to 64bits (for maximum of 64 lanes)*

<br>

***Lane:***

*→ element-wide slice of a sequence of vectors*

<br>

- - - 

***Predication***

***Example Operation (Vector Addition):***

***Assembler:***
```
VADDPS zmm1 {k1}{z},zmm2,zmm3
```
- *zmm1 → destination register*
- *k1 → optional predicate register*
- *z → optional flag*
- *zmm2,zmm3 → source registers*

<br>

***C (Vector Intrinsics):*** 
```
res = _mm512_maskz_add_ps(k, a, b);
```

<br>

***predication:***
- *→ only performs operation on lanes with corresponding predicate bit set activate*

***two predication modes:***
- → *zero masking → inactive lanes produce zero*
- → *masking → inactive lanes do not overwrite their prior register contents*

<br>

- - - 

***Automatic Vectorisation***

<br>

***automatic vectorisation***
- → *compiler converts scalar operations into vector operations (compiler automatically vectorises the scalar operations)*

<br>

***compiler:***
- → *automatic vectorisation performed by compiler during the code compilation process*
- → *compiler analyses the code - identifies loops/sequences of operations that can be vectorised - & generates corresponding vector instructions*
- → *automatic vectorisation commonly done in loops → if the iterations of a loop are independent & involve operations on arrays/sequences of data → compiler can replace scalar operations with vector operations handling multiple data elements of the array (multiple iterations of loop) in parallel*

<br>

***conditions in which compiler can vectorise code:***
- ***known loop bounds** → compiler must know how many times a loop will execute (knows how many loop iterations need to run in parallel)*
- ***no dependencies** → each iteration of loop must be independent of others (iterations cannot run in parallel if dependencies between iterations)*
- ***alignment** → data should be aligned in memory for vector operations*

<br>

***problems:***
- → *loops have complex control structure (e.g. **may have unknown loop bounds**)*
- → ***data dependencies often exist** which are hard for compiler to understand*
- → ***issues with memory alignment***

<br>

***solutions: (if compiler cannot automatically vectorise code)***
- → ***restructure code** to simplify loops (so has known bounds, no dependencies)*
- → *using **compiler hints** (such as \#pragma ivdep) to inform compiler about independence of loop iterations (so compiler does not need to infer this itself)*
- → ***explicitly aligning data** structures in memory*
- → *using **SIMD Intrinsics** → function-like constructs that directly utilise SIMD instructions - giving programmer more control over vectorisation*
- *→ **OpenMP Pragmas** → directives that can be used to instruct the compiler to vectorise certain loops/code sequences*

<br>

- - - 

***Compiler Hints (#pragma ivdep)***

<br>

```c
void add (float *c, float *a, float *b)
{
 #pragma ivdep
	for(int i=0; i <= N; i++){
		c[i] = a[i] + b[i]
	}
}
```

- ***IVDEP → Ignore Vector Dependencies***
- → ***tells compiler to assume there are no carried loop dependencies (vectorisation is safe)***
- → ***compiler may still not vectorise***

<br>

- - - 

***OpenMP Pragmas***

<Br>

***Loopwise:***
```c
void add (float *c, float *a, float *b)
{
 #pragma omp simd
	for (int i=0; i <= N; i++)
		c[i]=a[i]+b[i];
```
- → *indicates that the loop can be parallelised & transformed into an SIMD Loop (loop can be executed concurrently using SIMD Instructions)*

<br>

***Functionwise:***
```cpp
#pragma omp declare simd
void add (float *c, float *a, float *b)
{
	*c=*a+*b;
}
```
- → *applied to a function enabling SIMD Instructions at the Function Level*

<br>

*OpenMP Pragmas:*
- → ***tell compiler to vectorise code***
- → ***compiler may still not vectorise***

<br>

- - - 

***SIMD Intrinsics***

<br>

```cpp
void add (float *c, float *a, float *b)
{
__m128* pSrc1 = (__m128*) a;
__m128* pSrc2 = (__m128*) b;
__m128* pDest = (__m128*) c;
for (int i=0; i <= N/4; i++)
*pDest++ = _mm_add_ps(*pSrc1++, *pSrc2++);
}
```
- → *vector instruction lengths are hardcoded in the data types & intrinsics*
- → ***tells compiler which specific vector instructions to generate***
- → ***compiler will vectorise***

<br>

- - - 

***SIMT (Single Instruction, Multiple Threads)***

<br>

***if a compiler still cannot vectorise code → SIMT***
- → *a single instruction stream (vector) is executed across multiple threads*
- → *each thread handles different data (a different lane)*
- → *uses prediction to allow different threads to follow different branches while remaining executing in parallel*

<br>

***vectorising the outer loop:***
```cpp
#pragma omp simd
for (int i=0; i<N; ++i) {
	if(){…} else {…}
	for (int j=….) {…}
	while(…) {…}
	f(…)
}
```
- → *in the body of the vectorised loop - each lane executes a different iterations of the loop*

<br>

- - - 

***Vector Execution***

<br>

→ ***consider n-wide vector operations with n-wide ALU (or maybe in smaller m-wide blocks)***

<br>

***Vector Pipelining:***
- *→ vector instructions are executed serially (element-by-element) using several pipelined functional units*

<br>

***Vector Chaining:***
- → *each word in vector pipelining is forwarded to the next instruction as soon as available*
- *→ functional units form a long pipelined chain*
- → *(without vector chaining each vector instruction would need to load & store data to memory from which the next instruction would retrieve this data → chaining passes the data between instructions rather than accessing memory)*

<br>

***uop Decomposition:***
- *consider a dynamically scheduled machine*
- *each n-wide vector instruction is split into m-wide uops at decode time*
- → *dynamically scheduled execution engine schedules their execution (possibly across multiple functional units)*
- → *uops committed together*

<br>

- - - 

***Vector Chaining:***

<br>

***given a machine with 8-wide FUs:***
- → *VMP for memory operations*
- → *VP0 for vector multiply operations*
- → *VP1 for vector addition operations*

<br>

***32-wide vector instructions:***
- → *each 32-wide vector instruction executed in **4 8-bit blocks** (block-by-block by the functional unit)*
- → *blocks can be chained together into one continuously active pipeline*
- → *here, the block (green squares) using the VMP is chained to the block (yellow triangles) using the VP0 which is chained to the block (blue triangles) using VP1*
- → *the block in the VMP forwards the result to the block in the VP0 as soon as available*
- → *forwarding implemented block by block*
- → *all FUs active in overlapped fashion*

<br>

- - - 

***uop Decomposition***

<br>

*example:*

***Jaguar Core***

- → *low power two-issue dynamically scheduled processor core*
- → *supports AVX-256 ISA (256bit vector instructions)*
- → *two 128-bit ALUs*
- → ***256-bit AVC instructions split into two 128-bit uops at decode time - scheduled independently until commit***

<br>

- - - 

***SIMD Architectures***

<br>

***reduces Turing Tax:***
- → *more work with fewer instructions (less overhead)*
- → *relies on compiler/programmer (to consider alignment, dependence, predication)*

<br>

***issues:***
- → *although simple loops are fine - more complicated loops cause issues*
- → *lane-by-lane predication allows conditionals to be vectorised but branch divergence may lead to poor utilisation (instructions issued but not used)*
- → *indirections (when instructions does not directly reference the actual data/address it needs) can be vectorised but remain hard to implement efficiently unless accessed happen to fall on a small number of distinct cache lines*

<br>

***vector ISA:***
- → *vector ISA provides a broad spectrum of microarchitectural implementation choices*
- → *raises level of abstraction at which code is presented to the machine - programmer/compiler has more choices about how to execute instructions*
- → *e.g. AVX*

<br>

- - - 

