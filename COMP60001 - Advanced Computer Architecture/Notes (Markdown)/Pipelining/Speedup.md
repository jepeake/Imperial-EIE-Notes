
- - - 

***How fast?***

- - - 

- *simple 5-stage pipeline can run at 5-9GHz*
- ***limited by critical path through slowest pipeline stage logic** (pipeline stage with longest path of logic elements) - determines clock rate*

<br>

- ***tradeoff:*** *do more per cycle? or increase clock rate?*
- *do more per cycle: deeper logic structure between each pipeline latches*
- *increase clock rate: more pipeline stages with less in each stage*

<br>

- *at 3GHz, clock period = 330ps, 10 gate delays*

<br>

- *not all gates take the same length of time*
- ***fan-out***: *how many other gates are driven by the output of a gate*
- *determines time it takes for signal to propagate through a gate*
- *example unit of combinational logic depth = FO4 (fan out 4)*
- *4 gates driven by the gate*

<br>

- *pipeline latches account for 3-5 FO4 delays*
- *leaving 5-8 FO4 delays for actual work*

- - - 


