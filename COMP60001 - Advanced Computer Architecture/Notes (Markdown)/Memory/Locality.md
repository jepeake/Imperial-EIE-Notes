
- - - 

***Locality***

- - - 

***Real Memory Reference Patterns***

→ ***not completely random memory accesses***

<br>

- ***localities** exist in program (**spatial & temporal**)*
- *opportunities to provide hierarchical storage - so can cache commonly used data*

- - - 

***Typical Memory Reference Patterns***

***memory reference patterns:***

- *strong patterns for **instruction fetching, stack accesses (for function calls), or data accesses (accessing array)***

<br>

***instruction fetching:***
- *→ loops → executing the same neighbouring instructions multiple times*

<br>

***data accesses:***
- *→ vector access (arrays) or scalar accesses (same value)*

- - - 

***Locality***

→ ***programs use a relatively small portion of the address space at any instant of time***

<br>

***Temporal Locality***
*→ if a location is referenced by a program it is likely to be referenced again in the near future (think of loops)*

<br>

***Spatial Locality***
*→ if a location is referenced it is likely that locations near it will be referenced again in the near future (think of accessing an array)*

<br>

- *most modern architectures are **heavily reliant on locality for speed***

- - - 

***Cache Design***

- ***→ exploit both types of locality***

<br>

***temporal locality:***
- → *exploit temporal locality by remembering the contents of recently accessed locations (storing them in cache)*

<br>

***spatial locality:***
- → *exploit spatial locality by fetching blocks of data around recently access locations (fetching multiple blocks at a time into cache)*

- - - 


