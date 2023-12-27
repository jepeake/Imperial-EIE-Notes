
- - - 

***Performance Modelling***

- - - 

***Arithmetic Intensity***

<br>

→ ***ratio of computational operations to memory operations (FLOPs/Bytes Transferred)***

$Artihmetic \ Intensity = \frac{Number \ of \ FLOPs}{Number \ of \ Bytes \ Transferred}$

<br>

***high arithmetic intensity → performs many operations per memory access***
- ***high arithmetic intensity** programs → typically **compute-bound** (limited by CPU speed)*
- ***low arithmetic intensity** programs → typically **memory-bound** (spend more time waiting for data to be transferred from memory)*

<br>

- *peak GLOP/s → theoretical maximum FP operations per second (compute performance)*
- *peak GB/s → theoretical maximum transfer to/from memory per second (memory performance)*
- *Ops/Byte → to achieve peak compute-performance must to x operations for each byte fetched from main memory*
- *Ops/Word → to achieve peak compute-performance must to x operations for each word fetched from main memory*

<br>

- - - 

***Performance***

<br>

*Considering* $Work = Flops + Transfers$

*Assuming Flops & Memory Operations evenly distributed throughout code* - $Execution \ Time = max \ \{\frac{Flops}{Peak Floprate}, \frac{Transfers}{Bandwidth}\}$

*As* $Performance = Flops / Execution \ Time$

$Performance = \frac{Flops}{max\{\frac{Flops}{Peak Floprate}, \frac{Transfers}{Bandwidth}\}} = min \ \{Peak \ Floprate, \frac{Flops}{Transfers} \times Bandwidth\}$

*where second term = arithmetic intensity* $\times$ *memory bandwidth*

$Optimal \ Arithmetic \ Intensity = \frac{Theoretical \ Peak \ Floprate}{Theoretical \ Memory \ Bandwidth}$

*so*

$Performance = min \ \{Peak \  Floprate, Peak \ Floprate\} = PeakFloprate$

*if the arithmetical intensity is larger than the optimal intensity & **compute-bound***

& *if the arithmetic intensity is smaller than the optimal intensity & **memory-bound***

$Performance = \frac{Flops}{Transfers} \times Bandwidth \leq PeakFloprate$

<br>

- - - 

***Roofline Model***

<br>

***arithmetic intensity vs achieved compute performance (log-log)***

→ *graph that predicts the maximum possible performance of a program (given machines peak performance & memory bandwidth)*

<br>

- *low arithmetic intensity → memory-bound & memory bandwidth (peak GB/s) limits performance*
- *high arithmetic intensity → compute-bound & peak performance (peak GFLOP/s) limits performance*
- → *typically memory-bound*
- → *peak performance (peak GLOP/s) → **roof** on the maximum performance (limit for high-arithmetic intensity)*

<br>

***if memory bandwidth limits performance:***
- → *optimise data movement*
- → *optimise data accesses to reach memory bandwidth*
- → *reduce amount of data program needs to move (increasing arithmetic intensity)*
- → ***performance reaches a higher factor of peak performance***

<br>

***roofline model used to contrast different machines***
***ridge point:***
- → *offers insight into the machine’s overall performance potential*
- → *should application be limited by memory bandwidth or peak performance (by calculating program’s arithmetic intensity & comparing to graph)*
- → *(longer ridge point → machine capable of reaching peak performance for a wider range of programs)*

<br>

- - - 

