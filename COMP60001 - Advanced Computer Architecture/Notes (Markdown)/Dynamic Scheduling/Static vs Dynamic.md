
- - - -

***Pipelines & Scheduling***

- - - -

***Static Scheduling:***
- *instructions are scheduled at compile time*
- *compiler tries to optimise the order of instructions to minimise hazards*
- *scheduling is determined ahead of execution*
- *cannot adapt to runtime events*

***Dynamic Scheduling:***
- *instructions are scheduled at runtime*
- *processor determines order of execution*
- *processor inspects the state of the pipeline & availability of operands to decide which instructions to execute next*
- *can adapt to runtime events*

- - - 

***Pipelines***

- *→ in dynamic scheduling, deeper pipelines are often used*
- *each stage is made shorter, allowing for higher clock frequencies*
- *→ increases penalty of branch misprediction*

- *dynamic scheduling can deal better with hazards in deeper pipelines as it can reorder instructions on-the-fly to keep the pipelines filled*

- *dynamically scheduling pipelines often use out-of-order execution*
- *→ instructions executed as soon as their operands are read, rather than following the program order*
- *→ can improve throughput*
- *reservation stations, ROBs can deepen the processing stages*

- *deeper pipelines can do more speculative work → which pays off if the speculations are correct*

- - - 





