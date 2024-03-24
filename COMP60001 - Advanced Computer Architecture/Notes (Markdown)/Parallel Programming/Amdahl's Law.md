
- - - 

***Amdahl’s Law***

- - -

***Amdahl’s Law***

**→  *the overall performance improvement gained by optimising a single part of a system is limited by the fraction of time that improved part is actually used***

- - - 

***Parallel Machine***

- *→ **the speedup of the latency of the execution** of a program as a function of the number of processors executing it is **limited by the serial part of the program***

- *example: if a program needs 20hours to execute using a single thread - but a one-hour portion cannot be parallelised*
- *only the remaining 19hours (p = 0.95) of the program can be parallelised*
- *so - regardless of how many threads/cores are devoted to a parallelised execution of the program - the minimum execution time is always more than 1 hour*
- *so the theoretical speedup is less that 20x single-thread performance*

- - - 

***Theoretical Speedup***

- *$S_{latency}$ = theoretical speedup of execution of the whole task*
- *$S_{latency}(s) = \frac{1}{(1-p)+p/s}$*

- *s = speedup of the part of the task that benefits from improved system resources*
- *p = proportion of execution time that the part benefitting from improved resources originally occupied*

- - -

