
- - - 

***Locality***

- - - 

***Real Memory Reference Patterns***

→ ***not completely random memory accesses***

![[Pasted image 20230915164812.png|400]]

- ***localities** exist in program (**spatial & temporal**)*
- *opportunities to provide hierarchical storage - so can cache commonly used data*

- - - 

***Typical Memory Reference Patterns***

***memory reference patterns:***
![[Pasted image 20230915164917.png|400]]

- *strong patterns for **instruction fetching, stack accesses (for function calls), or data accesses (accessing array)***

***instruction fetching:***
- *→ loops → executing the same neighbouring instructions multiple times*

***data accesses:***
- *→ vector access (arrays) or scalar accesses (same value)*

- - - 

***Locality***

→ ***programs use a relatively small portion of the address space at any instant of time***

***Temporal Locality***
*→ if a location is referenced by a program it is likely to be referenced again in the near future (think of loops)*

***Spatial Locality***
*→ if a location is referenced it is likely that locations near it will be referenced again in the near future (think of accessing an array)*

![[Pasted image 20230915165237.png|400]]

- *most modern architectures are **heavily reliant on locality for speed***

- - - 

***Cache Design***

- ***→ exploit both types of locality***

***temporal locality:***
- → *exploit temporal locality by remembering the contents of recently accessed locations (storing them in cache)*

***spatial locality:***
- → *exploit spatial locality by fetching blocks of data around recently access locations (fetching multiple blocks at a time into cache)*

- - - 


